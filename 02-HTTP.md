# Fundamentos do NodeJS e Protocolo HTTP

## 1. Fundamentos do Protocolo HTTP

O HTTP (Hypertext Transfer Protocol) é o protocolo fundamental para comunicação na web. Ele funciona em um modelo cliente-servidor, onde:

### 1.1 Componentes Principais

**Cliente**: Geralmente um navegador web que envia requisições HTTP para um servidor.

**Servidor**: Programa que recebe requisições e retorna respostas HTTP.

### 1.2 Métodos HTTP Principais

- **GET**: Solicita dados do servidor
- **POST**: Envia dados para serem processados
- **PUT**: Atualiza dados existentes
- **DELETE**: Remove dados
- **PATCH**: Atualiza parcialmente um recurso

### 1.3 Códigos de Status HTTP

- **2xx**: Sucesso (200 OK, 201 Created)
- **3xx**: Redirecionamento
- **4xx**: Erro do cliente (404 Not Found, 400 Bad Request)
- **5xx**: Erro do servidor (500 Internal Server Error)

## 2. Módulo HTTP no NodeJS

O Node.js possui um módulo nativo chamado `http` que permite criar servidores web e fazer requisições HTTP.

### Exemplo Prático 1: Servidor Web Básico

```javascript
const http = require('http');

const server = http.createServer((req, res) => {
    res.writeHead(200, {'Content-Type': 'text/plain'});
    res.end('Olá, bem-vindo ao meu servidor!');
});

server.listen(3000, () => {
    console.log('Servidor rodando em http://localhost:3000');
});
```

### Exemplo Prático 2: API REST Simples para Gerenciar Tarefas

```javascript
const http = require('http');

let tarefas = [
    { id: 1, descricao: 'Estudar NodeJS', concluida: false },
    { id: 2, descricao: 'Fazer exercícios', concluida: true }
];

const server = http.createServer((req, res) => {
    // Configurando CORS para desenvolvimento
    res.setHeader('Access-Control-Allow-Origin', '*');
    res.setHeader('Content-Type', 'application/json');

    // GET /tarefas
    if (req.method === 'GET' && req.url === '/tarefas') {
        res.writeHead(200);
        res.end(JSON.stringify(tarefas));
        return;
    }

    // POST /tarefas
    if (req.method === 'POST' && req.url === '/tarefas') {
        let body = '';
        req.on('data', chunk => {
            body += chunk.toString();
        });

        req.on('end', () => {
            const novaTarefa = JSON.parse(body);
            novaTarefa.id = tarefas.length + 1;
            tarefas.push(novaTarefa);
            
            res.writeHead(201);
            res.end(JSON.stringify(novaTarefa));
        });
        return;
    }

    // Rota não encontrada
    res.writeHead(404);
    res.end(JSON.stringify({ erro: 'Rota não encontrada' }));
});

server.listen(3000, () => {
    console.log('Servidor rodando em http://localhost:3000');
});
```

## 3. Exercícios Práticos

### Exercício 1: Sistema de Lista de Compras
Crie um servidor HTTP que permita:
1. Listar todos os itens (GET)
2. Adicionar novo item (POST)
3. Marcar item como comprado (PUT)
4. Remover item (DELETE)

### Exercício 2: Chat Simples
Implemente um servidor de chat básico que:
1. Permita enviar mensagens (POST)
2. Listar mensagens recentes (GET)
3. Limpar histórico (DELETE)

## 4. Melhores Práticas

### 4.1 Segurança
- Sempre valide os dados de entrada
- Use HTTPS em produção
- Implemente rate limiting
- Nunca confie em dados do cliente

### 4.2 Performance
- Utilize streaming para arquivos grandes
- Implemente cache quando possível
- Comprima respostas (gzip/deflate)
- Monitore o uso de memória

### 4.3 Organização do Código
- Separe a lógica de rotas
- Use async/await para operações assíncronas
- Implemente tratamento de erros
- Mantenha logs adequados

### 4.4 Padronização de API
- Use verbos HTTP corretamente
- Retorne códigos de status apropriados
- Mantenha consistência no formato das respostas
- Documente os endpoints

## 5. Testando o Conhecimento

Para testar os exemplos e exercícios, você pode usar:

1. Ferramentas como Postman ou Insomnia
2. cURL no terminal:

```bash
# GET request
curl http://localhost:3000/tarefas

# POST request
curl -X POST http://localhost:3000/tarefas \
    -H "Content-Type: application/json" \
    -d '{"descricao": "Nova tarefa", "concluida": false}'
```

## 6. Recursos Adicionais

- Documentação oficial do Node.js (https://nodejs.org/pt/learn/getting-started/introduction-to-nodejs)
- MDN Web Docs sobre HTTP (https://developer.mozilla.org/pt-BR/docs/Web/HTTP)
- Ferramentas de desenvolvimento (Postman, Insomnia)
