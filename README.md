# Introdução ao Backend com Node.js

## **1. Fundamentos de Backend Web**
### O que é Backend?
- **Definição:** O backend é a camada "dos bastidores" de uma aplicação. Ele processa dados, executa lógica de negócios e comunica-se com o banco de dados.
- **Responsabilidades principais:**
  - Processar e responder às requisições do frontend.
  - Manipular e armazenar dados.
  - Garantir a segurança e integridade das informações.

### Arquitetura Cliente-Servidor
- **Cliente:** Navegador ou aplicativo que envia solicitações (HTTP) para o servidor.
- **Servidor:** Máquina que processa essas solicitações e envia respostas (normalmente em JSON ou HTML).

### Tecnologias comuns no backend
- **Linguagens:** Node.js (JavaScript), Python, Java, PHP, etc.
- **Bancos de dados:** MySQL, PostgreSQL, MongoDB.
- **Servidores web:** Apache, Nginx.

---

## **2. Fundamentos do Node.js**
### O que é Node.js?
- **Definição:** Um runtime de JavaScript baseado no mecanismo V8 do Google Chrome, que permite executar JavaScript no lado do servidor.
- **Características principais:**
  - **Event-driven (Baseado em eventos):** Ideal para aplicações que lidam com múltiplas conexões simultâneas.
  - **Non-blocking I/O (Entrada e saída não bloqueantes):** Operações assíncronas que melhoram o desempenho.
  - **Single-threaded:** Apesar de rodar em uma única thread, utiliza um modelo de eventos para gerenciar múltiplas requisições.

### Quando usar Node.js?
- Aplicações em tempo real (ex.: chats, games).
- APIs RESTful.
- Aplicações com alta concorrência e baixa latência.

---

## **3. Introdução ao Node.js e seu Ecossistema**
### História e Popularidade
- Lançado em 2009 por Ryan Dahl.
- Cresceu devido ao uso da linguagem JavaScript, que já era popular no frontend.

### Ecossistema
- **NPM (Node Package Manager):** Maior repositório de pacotes open-source do mundo.
- **Frameworks e bibliotecas populares:**
  - **Express.js:** Criação de servidores web simples e rápidos.
  - **Socket.io:** Comunicação em tempo real.
  - **Mongoose:** ODM para MongoDB.

---

## **4. Configuração do Ambiente de Desenvolvimento**
### Pré-requisitos
1. **Instalar o Node.js:**
   - Acesse [Node.js](https://nodejs.org/) e baixe a versão LTS recomendada.
2. **Verificar instalação:**
   ```bash
   node -v   # Verifica a versão do Node.js
   npm -v    # Verifica a versão do NPM
   ```

---

## **5. NPM e Gerenciamento de Pacotes**
### O que é o NPM?
- Gerenciador de pacotes do Node.js.
- Usado para instalar bibliotecas, ferramentas e gerenciar dependências.

### Comandos básicos do NPM
1. **Iniciar um projeto Node.js:**
   ```bash
   npm init -y
   ```
   - Cria o arquivo `package.json` com configurações padrão.

2. **Instalar pacotes:**
   - Pacote específico:
     ```bash
     npm install express
     ```
   - Pacote global:
     ```bash
     npm install -g nodemon
     ```

3. **Remover pacotes:**
   ```bash
   npm uninstall express
   ```

4. **Gerenciar scripts:**
   No `package.json`, adicione scripts como:
   ```json
   "scripts": {
     "start": "node index.js",
     "dev": "nodemon index.js"
   }
   ```
   Execute o script:
   ```bash
   npm run dev
   ```

### O que é o `package-lock.json`?
- Garante que todas as dependências instaladas tenham as mesmas versões em diferentes ambientes.

---

## **Prática**
1. **Configurar o ambiente:**
   - Instale o Node.js e configure um projeto com `npm init -y`.

2. **Criar um servidor básico com Node.js:**
   - Arquivo `index.js`:
     ```javascript
     const http = require('http');

     const server = http.createServer((req, res) => {
       res.statusCode = 200;
       res.setHeader('Content-Type', 'text/plain');
       res.end('Olá, mundo!');
     });

     server.listen(3000, () => {
       console.log('Servidor rodando em http://localhost:3000');
     });
     ```

3. **Instalar e usar o `nodemon` (https://nodemon.io/):**
   - Instale o `nodemon`:
     ```bash
     npm install -g nodemon
     ```
   - Inicie o servidor:
     ```bash
     nodemon index.js
     ```

---
