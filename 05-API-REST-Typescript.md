# API REST com Express e TypeScript

## Estrutura do Projeto

```
src/
  ├── server.ts
  ├── routes/
  │   └── userRoutes.ts
  ├── controllers/
  │   └── userController.ts
  ├── models/
  │   └── User.ts
  └── middleware/
      └── errorHandler.ts
```

## Configuração Inicial

### 1. Inicialização do Projeto

Primeiro, crie um novo projeto Node.js:

```bash
mkdir api-rest-express-typescript
cd api-rest-express-typescript
npm init -y
```

### 2. Configuração do package.json

package.json:

```json
{
  "name": "api-rest-express-typescript",
  "version": "1.0.0",
  "scripts": {
    "start": "ts-node src/server.ts",
    "dev": "nodemon src/server.ts",
    "build": "tsc"
  },
  "dependencies": {
    "express": "^4.18.2",
    "dotenv": "^16.0.3",
    "cors": "^2.8.5"
  },
  "devDependencies": {
    "@types/express": "^4.17.17",
    "@types/node": "^18.15.11",
    "@types/cors": "^2.8.13",
    "typescript": "^5.0.4",
    "ts-node": "^10.9.1",
    "nodemon": "^2.0.22"
  }
}
```

### 3. Configuração do TypeScript

Crie um arquivo tsconfig.json:

```json
{
  "compilerOptions": {
    "target": "es2020",
    "module": "commonjs",
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules"]
}
```

## Implementação

### 1. Modelo de Usuário (src/models/User.ts)

```typescript
interface User {
  id: number;
  name: string;
  email: string;
  age: number;
}

export default User;
```

### 2. Controlador de Usuário (src/controllers/userController.ts)

```typescript
import { Request, Response } from 'express';
import User from '../models/User';

// Simulando um banco de dados
let users: User[] = [];

export const userController = {
  // GET /users
  getAllUsers: (req: Request, res: Response) => {
    res.json(users);
  },

  // GET /users/:id
  getUserById: (req: Request, res: Response) => {
    const id = parseInt(req.params.id);
    const user = users.find(u => u.id === id);
    
    if (!user) {
      return res.status(404).json({ message: 'Usuário não encontrado' });
    }
    
    res.json(user);
  },

  // POST /users
  createUser: (req: Request, res: Response) => {
    const { name, email, age } = req.body;
    
    if (!name || !email || !age) {
      return res.status(400).json({ message: 'Dados incompletos' });
    }

    const newUser: User = {
      id: users.length + 1,
      name,
      email,
      age
    };

    users.push(newUser);
    res.status(201).json(newUser);
  },

  // PUT /users/:id
  updateUser: (req: Request, res: Response) => {
    const id = parseInt(req.params.id);
    const { name, email, age } = req.body;
    
    const userIndex = users.findIndex(u => u.id === id);
    
    if (userIndex === -1) {
      return res.status(404).json({ message: 'Usuário não encontrado' });
    }

    users[userIndex] = {
      ...users[userIndex],
      name: name || users[userIndex].name,
      email: email || users[userIndex].email,
      age: age || users[userIndex].age
    };

    res.json(users[userIndex]);
  },

  // DELETE /users/:id
  deleteUser: (req: Request, res: Response) => {
    const id = parseInt(req.params.id);
    const userIndex = users.findIndex(u => u.id === id);
    
    if (userIndex === -1) {
      return res.status(404).json({ message: 'Usuário não encontrado' });
    }

    users = users.filter(u => u.id !== id);
    res.status(204).send();
  }
};
```

### 3. Rotas (src/routes/userRoutes.ts)

```typescript
import { Router } from 'express';
import { userController } from '../controllers/userController';

const router = Router();

router.get('/users', userController.getAllUsers);
router.get('/users/:id', userController.getUserById);
router.post('/users', userController.createUser);
router.put('/users/:id', userController.updateUser);
router.delete('/users/:id', userController.deleteUser);

export default router;
```

### 4. Middleware de Erro (src/middleware/errorHandler.ts)

```typescript
import { Request, Response, NextFunction } from 'express';

export const errorHandler = (
  err: Error,
  req: Request,
  res: Response,
  next: NextFunction
) => {
  console.error(err.stack);
  res.status(500).json({
    message: 'Erro interno do servidor',
    error: process.env.NODE_ENV === 'development' ? err.message : undefined
  });
};
```

### 5. Servidor Principal (src/server.ts)

```typescript
import express from 'express';
import cors from 'cors';
import dotenv from 'dotenv';
import userRoutes from './routes/userRoutes';
import { errorHandler } from './middleware/errorHandler';

dotenv.config();

const app = express();
const PORT = process.env.PORT || 3000;

// Middlewares
app.use(cors());
app.use(express.json());

// Rotas
app.use('/api', userRoutes);

// Middleware de erro
app.use(errorHandler);

// Inicialização do servidor
app.listen(PORT, () => {
  console.log(`Servidor rodando na porta ${PORT}`);
});
```

## Explicação dos Endpoints

1. **GET /api/users**
   - Lista todos os usuários
   - Retorna: Array de usuários
   - Status: 200 OK

2. **GET /api/users/:id**
   - Busca um usuário específico pelo ID
   - Retorna: Usuário único
   - Status: 200 OK ou 404 Not Found

3. **POST /api/users**
   - Cria um novo usuário
   - Body: { name, email, age }
   - Retorna: Usuário criado
   - Status: 201 Created ou 400 Bad Request

4. **PUT /api/users/:id**
   - Atualiza um usuário existente
   - Body: { name?, email?, age? }
   - Retorna: Usuário atualizado
   - Status: 200 OK ou 404 Not Found

5. **DELETE /api/users/:id**
   - Remove um usuário
   - Retorna: Nada
   - Status: 204 No Content ou 404 Not Found

## Como Testar

1. **Instalação das Dependências**
   ```bash
   npm install
   ```

2. **Iniciando o Servidor**
   ```bash
   npm run dev
   ```

3. **Exemplos de Requisições**

   Criar um usuário:
   ```bash
   curl -X POST http://localhost:3000/api/users \
   -H "Content-Type: application/json" \
   -d '{"name":"João","email":"joao@email.com","age":30}'
   ```

   Listar usuários:
   ```bash
   curl http://localhost:3000/api/users
   ```

## Conceitos Importantes Implementados

1. **Tipagem Forte**
   - Uso de interfaces TypeScript para definir estruturas de dados
   - Tipagem de request e response do Express

2. **Arquitetura em Camadas**
   - Models: Definição de tipos/interfaces
   - Controllers: Lógica de negócios
   - Routes: Definição de endpoints
   - Middleware: Processamento de requisições

3. **Tratamento de Erros**
   - Middleware centralizado para erros
   - Respostas de erro padronizadas
   - Status HTTP apropriados

4. **Boas Práticas REST**
   - Uso correto dos métodos HTTP
   - URLs semânticas
   - Status codes apropriados
   - Respostas JSON consistentes
