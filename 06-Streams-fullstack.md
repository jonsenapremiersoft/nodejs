# Sistema de Upload de Arquivos
## Objetivo:

Criar uma aplicação web moderna para upload de arquivos usando TypeScript, Tailwind CSS, React e NestJS, implementando streams para manipulação eficiente.

## Configuração do Projeto

### Backend (NestJS + TypeScript)

1. **Criar o Projeto**
```bash
nest new file-upload-server
cd file-upload-server

# Instalar dependências
npm install @nestjs/platform-express multer @types/multer
```

2. **Criar Interface para Response**
```typescript
// src/interfaces/upload.interface.ts
export interface UploadResponse {
  message: string;
  filename: string;
  size?: number;
  type?: string;
}
```

3. **Criar Controller com TypeScript**
```typescript
// src/upload/upload.controller.ts
import { 
  Controller, 
  Post, 
  UseInterceptors, 
  UploadedFile,
  BadRequestException 
} from '@nestjs/common';
import { FileInterceptor } from '@nestjs/platform-express';
import { createWriteStream } from 'fs';
import { join } from 'path';
import { UploadResponse } from '../interfaces/upload.interface';

@Controller('upload')
export class UploadController {
  @Post()
  @UseInterceptors(FileInterceptor('file'))
  async uploadFile(
    @UploadedFile() file: Express.Multer.File
  ): Promise<UploadResponse> {
    if (!file) {
      throw new BadRequestException('Nenhum arquivo enviado');
    }

    try {
      const uploadPath = join(__dirname, '..', '..', 'uploads');
      const writeStream = createWriteStream(
        join(uploadPath, file.originalname)
      );
      
      writeStream.write(file.buffer);
      writeStream.end();

      return {
        message: 'Arquivo enviado com sucesso',
        filename: file.originalname,
        size: file.size,
        type: file.mimetype
      };
    } catch (error) {
      throw new BadRequestException('Erro ao processar arquivo');
    }
  }
}
```

### Frontend (React + TypeScript + Tailwind)

1. **Criar Projeto React com TypeScript**
```bash
npx create-react-app file-upload-client --template typescript
cd file-upload-client

# Instalar Tailwind CSS
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
```

2. **Configurar Tailwind**
```javascript
// tailwind.config.js
module.exports = {
  content: [
    "./src/**/*.{js,jsx,ts,tsx}",
  ],
  theme: {
    extend: {},
  },
  plugins: [],
}
```

3. **Adicionar Tailwind ao CSS**
```css
/* src/index.css */
@tailwind base;
@tailwind components;
@tailwind utilities;
```

4. **Criar Tipos**
```typescript
// src/types/upload.types.ts
export interface FileUploadState {
  file: File | null;
  status: string;
  progress: number;
}

export interface UploadResponse {
  message: string;
  filename: string;
  size?: number;
  type?: string;
}
```

5. **Criar Componente de Upload**
```typescript
// src/components/FileUpload.tsx
import React, { useState, useCallback } from 'react';
import { FileUploadState, UploadResponse } from '../types/upload.types';

const FileUpload: React.FC = () => {
  const [uploadState, setUploadState] = useState<FileUploadState>({
    file: null,
    status: '',
    progress: 0
  });

  const handleFileChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    if (e.target.files && e.target.files[0]) {
      setUploadState({
        ...uploadState,
        file: e.target.files[0],
        status: 'Arquivo selecionado'
      });
    }
  };

  const handleUpload = useCallback(async () => {
    if (!uploadState.file) {
      setUploadState(prev => ({
        ...prev,
        status: 'Por favor, selecione um arquivo'
      }));
      return;
    }

    const formData = new FormData();
    formData.append('file', uploadState.file);

    const xhr = new XMLHttpRequest();

    xhr.upload.onprogress = (event) => {
      const progress = (event.loaded / event.total) * 100;
      setUploadState(prev => ({
        ...prev,
        progress,
        status: `Enviando: ${Math.round(progress)}%`
      }));
    };

    xhr.onload = () => {
      if (xhr.status === 200) {
        const response: UploadResponse = JSON.parse(xhr.response);
        setUploadState(prev => ({
          ...prev,
          status: response.message,
          progress: 100
        }));
      } else {
        setUploadState(prev => ({
          ...prev,
          status: 'Erro ao enviar arquivo',
          progress: 0
        }));
      }
    };

    xhr.onerror = () => {
      setUploadState(prev => ({
        ...prev,
        status: 'Erro na conexão',
        progress: 0
      }));
    };

    xhr.open('POST', 'http://localhost:3000/upload');
    xhr.send(formData);
  }, [uploadState.file]);

  return (
    <div className="max-w-xl mx-auto p-6">
      <div className="bg-white rounded-lg shadow-md p-6">
        <h2 className="text-2xl font-bold mb-6 text-gray-800">
          Upload de Arquivo
        </h2>
        
        <div className="mb-4">
          <label className="block text-gray-700 mb-2">
            Selecione um arquivo:
          </label>
          <input
            type="file"
            onChange={handleFileChange}
            className="block w-full text-gray-700 bg-gray-100 rounded-lg cursor-pointer"
          />
        </div>

        <button
          onClick={handleUpload}
          className="bg-blue-500 text-white px-4 py-2 rounded-lg hover:bg-blue-600 transition-colors w-full mb-4"
        >
          Enviar Arquivo
        </button>

        {uploadState.progress > 0 && (
          <div className="mb-4">
            <div className="w-full bg-gray-200 rounded-full h-2.5">
              <div
                className="bg-blue-500 h-2.5 rounded-full transition-all duration-300"
                style={{ width: `${uploadState.progress}%` }}
              ></div>
            </div>
          </div>
        )}

        {uploadState.status && (
          <p className={`text-center ${
            uploadState.status.includes('sucesso') 
              ? 'text-green-600' 
              : uploadState.status.includes('Erro') 
                ? 'text-red-600' 
                : 'text-blue-600'
          }`}>
            {uploadState.status}
          </p>
        )}
      </div>
    </div>
  );
};

export default FileUpload;
```

6. **Atualizar App.tsx**
```typescript
// src/App.tsx
import React from 'react';
import FileUpload from './components/FileUpload';

const App: React.FC = () => {
  return (
    <div className="min-h-screen bg-gray-100 py-12">
      <div className="container mx-auto">
        <h1 className="text-3xl font-bold text-center text-gray-800 mb-8">
          Sistema de Upload
        </h1>
        <FileUpload />
      </div>
    </div>
  );
};

export default App;
```

## Sugestões de Melhorias

1. **Validação de Arquivo**
```typescript
// src/utils/fileValidation.ts
export interface FileValidation {
  isValid: boolean;
  message: string;
}

export const validateFile = (file: File): FileValidation => {
  const MAX_SIZE = 5 * 1024 * 1024; // 5MB
  const ALLOWED_TYPES = ['image/jpeg', 'image/png', 'application/pdf'];

  if (!ALLOWED_TYPES.includes(file.type)) {
    return {
      isValid: false,
      message: 'Tipo de arquivo não permitido'
    };
  }

  if (file.size > MAX_SIZE) {
    return {
      isValid: false,
      message: 'Arquivo muito grande (máximo 5MB)'
    };
  }

  return {
    isValid: true,
    message: 'Arquivo válido'
  };
};
```

2. **Drag and Drop**
```typescript
// Adicionar ao FileUpload.tsx
const handleDrop = (e: React.DragEvent<HTMLDivElement>) => {
  e.preventDefault();
  if (e.dataTransfer.files && e.dataTransfer.files[0]) {
    setUploadState({
      file: e.dataTransfer.files[0],
      status: 'Arquivo selecionado',
      progress: 0
    });
  }
};
```

3. **Lista de Arquivos Enviados**
```typescript
interface UploadedFile {
  filename: string;
  size: number;
  uploadedAt: Date;
}

const [uploadedFiles, setUploadedFiles] = useState<UploadedFile[]>([]);
```

## Estrutura Final do Projeto

```
projeto/
├── backend/
│   ├── src/
│   │   ├── interfaces/
│   │   │   └── upload.interface.ts
│   │   ├── upload/
│   │   │   └── upload.controller.ts
│   │   ├── app.module.ts
│   │   └── main.ts
│   └── uploads/
└── frontend/
    ├── src/
    │   ├── components/
    │   │   └── FileUpload.tsx
    │   ├── types/
    │   │   └── upload.types.ts
    │   ├── utils/
    │   │   └── fileValidation.ts
    │   ├── App.tsx
    │   └── index.css
    ├── tailwind.config.js
    └── tsconfig.json
```
