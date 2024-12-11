# Introdução às APIs REST

## 1. O que é uma API?

Uma API (Interface de Programação de Aplicações) é como um "cardápio" de funções que um sistema oferece para outros sistemas. Imagine um restaurante:
- O cliente (outro sistema) faz pedidos
- O garçom (API) recebe os pedidos e leva para a cozinha
- A cozinha (seu sistema) processa o pedido
- O garçom (API) entrega o resultado ao cliente

## 2. O que significa REST?

REST (Representational State Transfer) é um conjunto de regras para criar APIs. É como um "manual de boas práticas" que nos ajuda a criar APIs organizadas e padronizadas.

### As principais regras do REST são:

1. **Separação Cliente-Servidor**
   - O cliente não precisa saber como o servidor funciona por dentro
   - O servidor não precisa saber quem é o cliente

2. **Sem Estado (Stateless)**
   - Cada requisição deve conter todas as informações necessárias
   - O servidor não guarda informações entre requisições
   - É como se cada pedido fosse o primeiro pedido

3. **Padronização de Interface**
   - Usar URLs para identificar recursos
   - Usar métodos HTTP corretamente
   - Respostas em formatos padrão (geralmente JSON)

## 3. Métodos HTTP em Detalhes

### GET - Para buscar informações
```javascript
// Exemplo: Buscar todos os livros
app.get('/books', (req, res) => {
    res.json(books);
});

// Explicação:
// - '/books' é o caminho (endpoint) da API
// - req contém dados da requisição
// - res é usado para enviar a resposta
```

### POST - Para criar novos dados
```javascript
// Exemplo: Criar um novo livro
app.post('/books', (req, res) => {
    // req.body contém os dados enviados pelo cliente
    const newBook = {
        id: books.length + 1,
        title: req.body.title,
        author: req.body.author,
        createdAt: new Date()
    };
    
    books.push(newBook);
    // 201 significa "Criado com sucesso"
    res.status(201).json(newBook);
});
```

### PUT - Para atualizar dados (atualização completa)
```javascript
// Exemplo: Atualizar um livro existente
app.put('/books/:id', (req, res) => {
    // :id é um parâmetro de rota
    const book = books.find(b => b.id === parseInt(req.params.id));
    
    if (!book) {
        // 404 significa "Não encontrado"
        return res.status(404).json({ 
            message: 'Livro não encontrado' 
        });
    }
    
    // Atualiza todos os campos
    book.title = req.body.title;
    book.author = req.body.author;
    
    res.json(book);
});
```

### DELETE - Para remover dados
```javascript
// Exemplo: Remover um livro
app.delete('/books/:id', (req, res) => {
    const bookIndex = books.findIndex(b => b.id === parseInt(req.params.id));
    
    if (bookIndex === -1) {
        return res.status(404).json({ 
            message: 'Livro não encontrado' 
        });
    }
    
    books.splice(bookIndex, 1);
    // 204 significa "Sucesso, sem conteúdo para retornar"
    res.status(204).send();
});
```

## 4. Estrutura Completa de uma API

Vamos criar uma API mais organizada para gerenciar livros:

```javascript
const express = require('express');
const app = express();

// Middleware para processar JSON
app.use(express.json());

// Middleware para log de requisições
app.use((req, res, next) => {
    console.log(`${req.method} ${req.path}`);
    next();
});

// Nosso "banco de dados"
let books = [
    { 
        id: 1, 
        title: 'O Senhor dos Anéis', 
        author: 'J.R.R. Tolkien',
        year: 1954,
        available: true
    }
];

// Função auxiliar para validar livro
function validateBook(book) {
    const errors = [];
    
    if (!book.title) {
        errors.push('Título é obrigatório');
    }
    
    if (!book.author) {
        errors.push('Autor é obrigatório');
    }
    
    return errors;
}

// Rotas da API
app.get('/books', (req, res) => {
    // Suporta filtro por disponibilidade
    const available = req.query.available;
    
    if (available !== undefined) {
        const filtered = books.filter(b => b.available === (available === 'true'));
        return res.json(filtered);
    }
    
    res.json(books);
});

app.get('/books/:id', (req, res) => {
    const book = books.find(b => b.id === parseInt(req.params.id));
    
    if (!book) {
        return res.status(404).json({ 
            message: 'Livro não encontrado' 
        });
    }
    
    res.json(book);
});

app.post('/books', (req, res) => {
    const errors = validateBook(req.body);
    
    if (errors.length > 0) {
        return res.status(400).json({ errors });
    }
    
    const newBook = {
        id: books.length + 1,
        title: req.body.title,
        author: req.body.author,
        year: req.body.year,
        available: true,
        createdAt: new Date()
    };
    
    books.push(newBook);
    res.status(201).json(newBook);
});

app.put('/books/:id', (req, res) => {
    const book = books.find(b => b.id === parseInt(req.params.id));
    
    if (!book) {
        return res.status(404).json({ 
            message: 'Livro não encontrado' 
        });
    }
    
    const errors = validateBook(req.body);
    
    if (errors.length > 0) {
        return res.status(400).json({ errors });
    }
    
    book.title = req.body.title;
    book.author = req.body.author;
    book.year = req.body.year;
    book.updatedAt = new Date();
    
    res.json(book);
});

app.delete('/books/:id', (req, res) => {
    const bookIndex = books.findIndex(b => b.id === parseInt(req.params.id));
    
    if (bookIndex === -1) {
        return res.status(404).json({ 
            message: 'Livro não encontrado' 
        });
    }
    
    books.splice(bookIndex, 1);
    res.status(204).send();
});

// Iniciando o servidor
const PORT = 3000;
app.listen(PORT, () => {
    console.log(`Servidor rodando em http://localhost:${PORT}`);
});
```

## 5. Como Testar a API

### Usando Postman/Insomnia:

1. **GET /books**
   - Método: GET
   - URL: http://localhost:3000/books

2. **POST /books**
   - Método: POST
   - URL: http://localhost:3000/books
   - Body (JSON):
   ```json
   {
       "title": "1984",
       "author": "George Orwell",
       "year": 1949
   }
   ```

3. **PUT /books/1**
   - Método: PUT
   - URL: http://localhost:3000/books/1
   - Body (JSON):
   ```json
   {
       "title": "1984",
       "author": "George Orwell",
       "year": 1949
   }
   ```

4. **DELETE /books/1**
   - Método: DELETE
   - URL: http://localhost:3000/books/1

## 6. Boas Práticas

1. **Use os verbos HTTP corretamente**
   - GET para buscar
   - POST para criar
   - PUT para atualizar tudo
   - PATCH para atualizar parcialmente
   - DELETE para remover

2. **Use status HTTP apropriados**
   - 200: OK
   - 201: Criado
   - 204: Sem conteúdo
   - 400: Erro do cliente
   - 404: Não encontrado
   - 500: Erro do servidor

3. **Versione sua API**
   - Use prefixos como `/api/v1/books`
   - Ajuda na manutenção e evolução

4. **Valide os dados de entrada**
   - Sempre verifique os dados recebidos
   - Retorne erros claros

5. **Use nomes no plural para coleções**
   - `/books` em vez de `/book`
   - `/users` em vez de `/user`

## 7. Exercício Prático

Crie uma API para gerenciar uma lista de tarefas com os seguintes requisitos:

1. Cada tarefa deve ter:
   - ID
   - Título
   - Descrição
   - Status (pendente/concluída)
   - Data de criação
   - Data de conclusão

2. Implemente as seguintes rotas:
   - GET /tasks (listar todas)
   - GET /tasks/:id (buscar uma)
   - POST /tasks (criar)
   - PUT /tasks/:id (atualizar)
   - DELETE /tasks/:id (remover)

3. Adicione validações:
   - Título obrigatório
   - Descrição opcional
   - Status deve ser "pendente" ou "concluída"
