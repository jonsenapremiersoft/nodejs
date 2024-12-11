# Express.js

## 1. O que é Express.js?

https://expressjs.com/

O Express.js é um framework web para Node.js que simplifica o processo de construção de aplicações web e APIs. Ele foi criado para oferecer um conjunto mínimo de ferramentas para servidores web, mantendo a flexibilidade e não obscurecendo os recursos do Node.js.

**Principais características:**
- Minimalista e flexível
- Alto desempenho
- Suporte robusto para rotas
- Grande ecossistema de middleware
- Compatível com diversos template engines
- Fácil integração com bancos de dados

## 2. Por que usar Express.js?

O Express.js resolve vários problemas comuns no desenvolvimento web:

1. **Organização de código**: Fornece uma estrutura consistente para organizar rotas e middleware
2. **Produtividade**: Reduz o código boilerplate necessário para criar servidores web
3. **Performance**: Otimizado para alta performance e baixo uso de memória
4. **Manutenibilidade**: Facilita a manutenção do código através de sua arquitetura modular
5. **Escalabilidade**: Permite escalar aplicações de forma eficiente

## 3. Instalação e Configuração

### 3.1 Instalação

```bash
# Criar novo projeto
mkdir meu-projeto
cd meu-projeto

# Inicializar package.json
npm init -y

# Instalar Express
npm install express
```

### 3.2 Configuração Básica

```javascript
// app.js
const express = require('express');
const app = express();
const port = 3000;

// Configurações básicas
app.use(express.json()); // Para parsing de JSON
app.use(express.urlencoded({ extended: true })); // Para parsing de formulários

app.listen(port, () => {
  console.log(`Servidor rodando na porta ${port}`);
});
```

## 4. Middleware em Detalhes

O middleware é uma função que tem acesso ao objeto de requisição (req), ao objeto de resposta (res) e à próxima função middleware (next).

### 4.1 Tipos de Middleware

```javascript
// Middleware de Aplicação
app.use((req, res, next) => {
    console.log('Tempo:', Date.now());
    next();
});

// Middleware de Rota
app.get('/usuario/:id', (req, res, next) => {
    // Verifica se o usuário está autenticado
    next();
}, (req, res) => {
    // Manipula a requisição
});

// Middleware de Tratamento de Erros
app.use((err, req, res, next) => {
    console.error(err.stack);
    res.status(500).send('Algo deu errado!');
});
```

### 4.2 Middleware Comum

```javascript
// Middleware de logging
const logger = (req, res, next) => {
    console.log(`${req.method} ${req.url}`);
    next();
};

// Middleware de autenticação
const auth = (req, res, next) => {
    if (!req.headers.authorization) {
        return res.status(401).json({ message: 'Não autorizado' });
    }
    next();
};
```

## 5. Roteamento Detalhado

O sistema de roteamento do Express permite criar rotas de forma organizada e modular.

### 5.1 Rotas Básicas

```javascript
// Rota GET
app.get('/usuarios', (req, res) => {
    res.json([{ id: 1, nome: 'João' }]);
});

// Rota POST
app.post('/usuarios', (req, res) => {
    // Criação de usuário
    res.status(201).json({ message: 'Usuário criado' });
});

// Rota PUT
app.put('/usuarios/:id', (req, res) => {
    // Atualização completa
    const id = req.params.id;
    res.json({ message: `Usuário ${id} atualizado` });
});

// Rota DELETE
app.delete('/usuarios/:id', (req, res) => {
    // Remoção
    res.json({ message: 'Usuário removido' });
});
```

### 5.2 Rotas com Parâmetros

```javascript
// Parâmetros de URL
app.get('/usuarios/:id', (req, res) => {
    const id = req.params.id;
    res.json({ id: id });
});

// Query strings
app.get('/busca', (req, res) => {
    const termo = req.query.q;
    res.json({ termo: termo });
});
```

## 6. Manipulação de Requisições e Respostas

### 6.1 Objeto Request (req)

```javascript
app.get('/exemplo', (req, res) => {
    // Parâmetros da URL
    console.log(req.params);
    
    // Query strings
    console.log(req.query);
    
    // Headers
    console.log(req.headers);
    
    // Corpo da requisição
    console.log(req.body);
    
    // Cookies
    console.log(req.cookies);
});
```

### 6.2 Objeto Response (res)

```javascript
app.get('/respostas', (req, res) => {
    // Enviar JSON
    res.json({ mensagem: 'OK' });
    
    // Enviar texto
    res.send('Texto simples');
    
    // Definir status
    res.status(201).send();
    
    // Redirecionar
    res.redirect('/outra-rota');
    
    // Renderizar view
    res.render('index', { titulo: 'Página Inicial' });
});
```

## 7. Tratamento de Erros

```javascript
// Middleware de erro personalizado
app.use((err, req, res, next) => {
    // Log do erro
    console.error(err.stack);
    
    // Resposta de erro
    res.status(err.status || 500).json({
        message: err.message || 'Erro interno do servidor',
        error: process.env.NODE_ENV === 'development' ? err : {}
    });
});

// Tratamento de erros assíncronos
app.get('/async', async (req, res, next) => {
    try {
        // Operação assíncrona
        await algumaOperacao();
    } catch (error) {
        next(error); // Passa o erro para o middleware de erro
    }
});
```

## 8. Servindo Arquivos Estáticos

```javascript
// Servir arquivos da pasta 'public'
app.use(express.static('public'));

// Configurar múltiplos diretórios estáticos
app.use(express.static('public'));
app.use(express.static('uploads'));

// Prefixo virtual
app.use('/static', express.static('public'));
```

## 9. Segurança

```javascript
const helmet = require('helmet');
const cors = require('cors');

// Configurações de segurança básicas
app.use(helmet()); // Adiciona vários headers de segurança
app.use(cors()); // Habilita CORS

// Rate limiting
const rateLimit = require('express-rate-limit');
const limiter = rateLimit({
    windowMs: 15 * 60 * 1000, // 15 minutos
    max: 100 // limite de 100 requisições por IP
});
app.use(limiter);
```

## 10. Boas Práticas

1. **Estrutura de Diretórios**
```
projeto/
  ├── config/        # Configurações
  ├── controllers/   # Controladores
  ├── middleware/    # Middleware personalizado
  ├── models/        # Modelos
  ├── routes/        # Rotas
  ├── public/        # Arquivos estáticos
  ├── views/         # Templates
  ├── tests/         # Testes
  └── app.js         # Entrada da aplicação
```

2. **Separação de Responsabilidades**
```javascript
// routes/usuarios.js
const router = express.Router();
const UsuarioController = require('../controllers/UsuarioController');

router.get('/', UsuarioController.listar);
router.post('/', UsuarioController.criar);

module.exports = router;

// controllers/UsuarioController.js
class UsuarioController {
    static async listar(req, res) {
        // Lógica de listagem
    }
    
    static async criar(req, res) {
        // Lógica de criação
    }
}
```

3. **Variáveis de Ambiente**
```javascript
// config/config.js
require('dotenv').config();

module.exports = {
    port: process.env.PORT || 3000,
    dbUrl: process.env.DATABASE_URL,
    environment: process.env.NODE_ENV || 'development'
};
```
