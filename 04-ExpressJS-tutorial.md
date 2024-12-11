# Tutorial de Express.js para Iniciantes

Express.js é um framework minimalista e flexível para Node.js, amplamente utilizado para construir APIs e aplicativos web. Neste tutorial, abordaremos os conceitos básicos para você começar a desenvolver com Express.js usando práticas modernas.

## Pré-requisitos
- Conhecimentos básicos de JavaScript e Node.js.
- Node.js instalado em sua máquina. [Baixar Node.js](https://nodejs.org/)

## Passo 1: Configurando o Projeto
1. Crie uma pasta para o seu projeto e navegue até ela:
   ```bash
   mkdir meu-projeto-express
   cd meu-projeto-express
   ```

2. Inicialize o projeto com npm:
   ```bash
   npm init -y
   ```

3. Instale o Express.js:
   ```bash
   npm install express
   ```

## Passo 2: Criando o Servidor Básico
Crie um arquivo chamado `index.js` e adicione o seguinte código:

```javascript
const express = require('express');
const app = express();
const PORT = 3000;

// Rota principal
app.get('/', (req, res) => {
  res.send('Bem-vindo ao Express.js!');
});

// Iniciando o servidor
app.listen(PORT, () => {
  console.log(`Servidor rodando em http://localhost:${PORT}`);
});
```

Execute o servidor:
```bash
node index.js
```
Abra o navegador e acesse `http://localhost:3000`. Você verá a mensagem: **Bem-vindo ao Express.js!**

## Passo 3: Adicionando Rotas
Rotas são um dos conceitos principais no Express.js. Vamos criar algumas rotas adicionais no `index.js`:

```javascript
// Rota para obter informações de usuários
app.get('/users', (req, res) => {
  res.json({ users: ['Alice', 'Bob', 'Charlie'] });
});

// Rota com parâmetro dinâmico
app.get('/users/:id', (req, res) => {
  const userId = req.params.id;
  res.send(`Usuário ID: ${userId}`);
});

// Rota POST para criar um usuário
app.post('/users', express.json(), (req, res) => {
  const newUser = req.body;
  res.status(201).json({ message: 'Usuário criado com sucesso!', user: newUser });
});
```

## Passo 4: Middleware
Middlewares são funções que manipulam requisições antes que elas cheguem às rotas. Vamos adicionar um middleware simples para logar informações das requisições:

```javascript
// Middleware para log de requisições
app.use((req, res, next) => {
  console.log(`${req.method} ${req.url}`);
  next();
});
```

Adicione esse trecho acima das definições de rota.

## Passo 5: Estruturando o Projeto
À medida que o projeto cresce, uma estrutura organizada é essencial. Aqui está um exemplo básico:

```
meu-projeto-express/
├── routes/
│   └── users.js
├── index.js
└── package.json
```

1. Mova as rotas relacionadas a `users` para um arquivo chamado `routes/users.js`:

```javascript
const express = require('express');
const router = express.Router();

// Rotas de usuários
router.get('/', (req, res) => {
  res.json({ users: ['Alice', 'Bob', 'Charlie'] });
});

router.get('/:id', (req, res) => {
  const userId = req.params.id;
  res.send(`Usuário ID: ${userId}`);
});

router.post('/', express.json(), (req, res) => {
  const newUser = req.body;
  res.status(201).json({ message: 'Usuário criado com sucesso!', user: newUser });
});

module.exports = router;
```

2. Importe e use as rotas em `index.js`:

```javascript
const userRoutes = require('./routes/users');
app.use('/users', userRoutes);
```

## Passo 6: Adicionando Variáveis de Ambiente
Para armazenar informações sensíveis como a porta do servidor, use variáveis de ambiente:

1. Instale o pacote `dotenv`:
   ```bash
   npm install dotenv
   ```

2. Crie um arquivo `.env`:
   ```env
   PORT=3000
   ```

3. Atualize `index.js`:

```javascript
require('dotenv').config();
const PORT = process.env.PORT || 3000;
```
