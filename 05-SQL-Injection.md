# SQL Injection

Olá! Hoje vamos aprender sobre SQL Injection, uma das vulnerabilidades mais comuns em aplicações web, e como preveni-la em aplicações NodeJS com Express.

## O que é SQL Injection?

SQL Injection é uma técnica onde um atacante insere (injeta) código SQL malicioso através dos inputs da aplicação. Se bem-sucedido, o atacante pode:
- Ler dados sensíveis do banco
- Modificar dados 
- Apagar tabelas inteiras
- Em alguns casos, até executar comandos no servidor

## Exemplo Vulnerável

Veja este código Express que busca um usuário:

```javascript
app.get('/user', (req, res) => {
    const id = req.query.id;
    const query = `SELECT * FROM users WHERE id = ${id}`;
    
    db.query(query, (err, result) => {
        if (err) throw err;
        res.json(result);
    });
});
```

Este código é perigoso porque concatena diretamente o input do usuário na query. Um atacante poderia fazer:

```
GET /user?id=1 OR 1=1
```

A query se tornaria:
```sql
SELECT * FROM users WHERE id = 1 OR 1=1
```

Como `1=1` é sempre verdadeiro, isso retornaria TODOS os usuários do banco!

## Como Prevenir

### 1. Use Prepared Statements

A primeira e mais importante defesa é usar prepared statements. No Node, podemos usar o `?` como placeholder:

```javascript
app.get('/user', (req, res) => {
    const id = req.query.id;
    const query = 'SELECT * FROM users WHERE id = ?';
    
    db.query(query, [id], (err, result) => {
        if (err) throw err;
        res.json(result);
    });
});
```

### 2. Use ORMs

ORMs (Object-Relational Mappers) como Sequelize ou Prisma já incluem proteção contra SQL Injection:

```javascript
const user = await prisma.user.findUnique({
    where: {
        id: req.query.id
    }
});
```

### 3. Validação de Input

Sempre valide e sanitize os inputs:

```javascript
const express = require('express');
const { body, validationResult } = require('express-validator');

app.post('/user', 
    body('id').isInt(), // valida se é número inteiro
    (req, res) => {
        const errors = validationResult(req);
        if (!errors.isEmpty()) {
            return res.status(400).json({ errors: errors.array() });
        }
        // continua processamento...
    }
);
```

## Exercício Prático

Identifique os problemas de segurança neste código:

```javascript
app.post('/login', (req, res) => {
    const { username, password } = req.body;
    const query = `
        SELECT * FROM users 
        WHERE username = '${username}' 
        AND password = '${password}'
    `;
    db.query(query, (err, result) => {
        if (err) throw err;
        res.json(result);
    });
});
```

### Problemas:
1. Concatenação direta de input na query
2. Senha em texto plano
3. Sem validação de input
4. Sem prepared statements

### Correção:

```javascript
const bcrypt = require('bcrypt');

app.post('/login',
    body('username').trim().isLength({ min: 3 }),
    body('password').isLength({ min: 6 }),
    async (req, res) => {
        const errors = validationResult(req);
        if (!errors.isEmpty()) {
            return res.status(400).json({ errors: errors.array() });
        }

        const { username, password } = req.body;
        
        try {
            const query = 'SELECT * FROM users WHERE username = ?';
            const [user] = await db.query(query, [username]);
            
            if (!user || !await bcrypt.compare(password, user.password_hash)) {
                return res.status(401).json({ error: 'Invalid credentials' });
            }
            
            res.json({ message: 'Login successful' });
        } catch (err) {
            res.status(500).json({ error: 'Server error' });
        }
    }
);
```

## Dicas Finais

1. Nunca confie em input do usuário
2. Sempre use prepared statements ou ORMs
3. Valide todos os inputs antes de processar
4. Use HTTPS para proteger dados em trânsito
5. Mantenha suas dependências atualizadas
6. Implemente rate limiting para prevenir ataques de força bruta
