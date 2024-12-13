# API de Biblioteca Digital

## 1. Preparando o Ambiente

```bash
# Cria uma pasta para seu projeto
mkdir biblioteca-api
cd biblioteca-api

# Inicia um projeto Node.js
npm init -y
```

Instale as dependências:

```bash
# Instala as principais ferramentas
npm install express pg dotenv typescript

# Instala os tipos necessários para o TypeScript
npm install @types/express @types/pg @types/node --save-dev

# Instala uma ferramenta para desenvolvimento que reinicia o servidor automaticamente
npm install ts-node-dev --save-dev
```

### Explicando cada dependência:
- `express`: Framework para criar nossa API
- `pg`: Biblioteca para conectar com PostgreSQL
- `dotenv`: Para gerenciar variáveis de ambiente
- `typescript`: Para usar TypeScript
- Os pacotes `@types/*`: Adiciona suporte a tipos para nossas bibliotecas
- `ts-node-dev`: Reinicia nosso servidor quando salvamos alterações

## 2. Configurando o TypeScript

Vamos configurar o TypeScript. Crie um arquivo `tsconfig.json` na raiz do projeto:

```json
{
  "compilerOptions": {
    "target": "es2017",
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

## 3. Criando o Banco de Dados

Abra o DBeaver e execute estas queries. Vou explicar cada uma:

```sql
-- Cria o banco de dados
CREATE DATABASE biblioteca;

-- Cria a tabela de livros
CREATE TABLE livros (
    id SERIAL PRIMARY KEY,  -- SERIAL cria um número único automaticamente
    titulo VARCHAR(100) NOT NULL,  -- NOT NULL significa que é obrigatório
    autor VARCHAR(100) NOT NULL,
    ano_publicacao INTEGER,
    disponivel BOOLEAN DEFAULT true  -- começa como true por padrão
);
```

## 4. Estruturando o Projeto

Vamos organizar nossos arquivos assim:
```
biblioteca-api/
├── src/
│   ├── config/        (configurações do projeto)
│   ├── controllers/   (lógica da aplicação)
│   ├── routes/        (rotas da API)
│   └── server.ts      (arquivo principal)
├── .env              (variáveis de ambiente)
└── package.json
```

Crie estas pastas:
```bash
mkdir src
mkdir src/config
mkdir src/controllers
mkdir src/routes
```

## 5. Configurando a Conexão com o Banco

Crie um arquivo `.env` na raiz do projeto:

```env
DB_HOST=localhost
DB_USER=seu_usuario
DB_PASS=sua_senha
DB_NAME=biblioteca
DB_PORT=5432
```

Agora crie `src/config/database.ts`:

```typescript
import { Pool } from 'pg';
import dotenv from 'dotenv';

// Carrega as variáveis de ambiente do arquivo .env
dotenv.config();

// Cria a conexão com o banco
export const pool = new Pool({
    host: process.env.DB_HOST,
    user: process.env.DB_USER,
    password: process.env.DB_PASS,
    database: process.env.DB_NAME,
    port: parseInt(process.env.DB_PORT || '5432')
});
```

## 6. Criando o Controller

Crie `src/controllers/livroController.ts`:

```typescript
import { Request, Response } from 'express';
import { pool } from '../config/database';

// Aqui vamos criar funções para gerenciar os livros
export const livroController = {
    // Função para criar um novo livro
    async criar(req: Request, res: Response) {
        try {
            // Pega os dados enviados na requisição
            const { titulo, autor, ano_publicacao } = req.body;

            // Insere no banco de dados
            const result = await pool.query(
                'INSERT INTO livros (titulo, autor, ano_publicacao) VALUES ($1, $2, $3) RETURNING *',
                [titulo, autor, ano_publicacao]
            );

            // Retorna o livro criado
            res.status(201).json(result.rows[0]);
        } catch (error) {
            console.error('Erro ao criar livro:', error);
            res.status(500).json({ erro: 'Erro ao criar livro' });
        }
    },

    // Função para listar todos os livros
    async listar(req: Request, res: Response) {
        try {
            const result = await pool.query('SELECT * FROM livros');
            res.json(result.rows);
        } catch (error) {
            console.error('Erro ao listar livros:', error);
            res.status(500).json({ erro: 'Erro ao listar livros' });
        }
    }
};
```

## 7. Criando as Rotas

Crie `src/routes/livroRoutes.ts`:

```typescript
import { Router } from 'express';
import { livroController } from '../controllers/livroController';

const router = Router();

// Define as rotas e conecta com as funções do controller
router.post('/livros', livroController.criar);
router.get('/livros', livroController.listar);

export default router;
```

## 8. Criando o Servidor

Crie `src/server.ts`:

```typescript
import express from 'express';
import livroRoutes from './routes/livroRoutes';

const app = express();

// Permite receber JSON nas requisições
app.use(express.json());

// Adiciona nossas rotas
app.use('/api', livroRoutes);

// Inicia o servidor na porta 3000
const PORT = 3000;
app.listen(PORT, () => {
    console.log(`Servidor rodando em http://localhost:${PORT}`);
});
```

## 9. Configurando os Scripts

Adicione estes scripts no seu `package.json`:

```json
{
  "scripts": {
    "dev": "ts-node-dev src/server.ts",
    "build": "tsc",
    "start": "node dist/server.js"
  }
}
```

Para rodar o servidor em desenvolvimento:
```bash
npm run dev
```

## 10. Testando a API

Você pode usar o Insomnia ou Postman para testar. Aqui estão alguns exemplos:

### Criar um livro:
```
POST http://localhost:3000/api/livros
Content-Type: application/json

{
    "titulo": "O Senhor dos Anéis",
    "autor": "J.R.R. Tolkien",
    "ano_publicacao": 1954
}
```

### Listar livros:
```
GET http://localhost:3000/api/livros
```

## Exercícios para Praticar

1. **Buscar um livro específico**
   - Crie uma rota GET `/api/livros/:id`
   - Dica: Use `req.params.id` para pegar o ID

2. **Atualizar um livro**
   - Crie uma rota PUT `/api/livros/:id`
   - Permita atualizar título, autor ou ano

3. **Deletar um livro**
   - Crie uma rota DELETE `/api/livros/:id`

## Sugestões de Melhorias para Iniciantes

1. **Validações Básicas**
   - Verifique se título e autor não estão vazios
   - Verifique se o ano é um número válido
   ```typescript
   if (!titulo || !autor) {
       return res.status(400).json({ erro: 'Título e autor são obrigatórios' });
   }
   ```

2. **Buscas Simples**
   - Adicione uma rota para buscar livros por autor
   - Adicione uma rota para buscar livros por ano

3. **Mensagens de Erro Amigáveis**
   - Crie mensagens claras quando algo der errado
   - Adicione console.log para ajudar no debug

4. **Documentação Básica**
   - Comente seu código explicando o que cada parte faz
   - Crie um README.md simples explicando como rodar o projeto

5. **Organização**
   - Separe as queries SQL em um arquivo separado
   - Use constantes para mensagens de erro
