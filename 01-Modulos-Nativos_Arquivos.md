# Módulos Nativos Node.js - File System (fs) e Path

## Introdução

O Node.js possui diversos módulos nativos que nos auxiliam no desenvolvimento de aplicações. Hoje vamos explorar dois módulos essenciais: o `fs` (File System) e o `path`, que são fundamentais para manipulação de arquivos e diretórios.

## Módulo File System (fs)

O módulo `fs` permite interagir com o sistema de arquivos do computador. Vamos explorar suas principais funcionalidades:

### 1. Leitura de Arquivos (precisa ter um 'arquivo.txt' na raiz)

```javascript
const fs = require('fs');

// Leitura síncrona
const conteudo = fs.readFileSync('./arquivo.txt', 'utf-8');
console.log(conteudo);

// Leitura assíncrona
fs.readFile('./arquivo.txt', 'utf-8', (erro, conteudo) => {
    if (erro) {
        console.error('Erro ao ler arquivo:', erro);
        return;
    }
    console.log(conteudo);
});

// Usando Promises
const fsPromises = require('fs').promises;
async function lerArquivo() {
    try {
        const conteudo = await fsPromises.readFile('./arquivo.txt', 'utf-8');
        console.log(conteudo);
    } catch (erro) {
        console.error('Erro:', erro);
    }
}
```

### 2. Escrita em Arquivos

```javascript
// Escrita síncrona
fs.writeFileSync('./novo-arquivo.txt', 'Conteúdo do arquivo');

// Escrita assíncrona
fs.writeFile('./novo-arquivo.txt', 'Conteúdo do arquivo', (erro) => {
    if (erro) {
        console.error('Erro ao escrever:', erro);
        return;
    }
    console.log('Arquivo criado com sucesso!');
});

// Adicionar conteúdo ao final do arquivo
fs.appendFileSync('./log.txt', '\nNova linha de log');
```

### 3. Manipulação de Diretórios

```javascript
// Criar diretório
fs.mkdirSync('./nova-pasta');

// Ler conteúdo do diretório
const arquivos = fs.readdirSync('./');
console.log('Arquivos:', arquivos);

// Verificar se arquivo/diretório existe
const existe = fs.existsSync('./arquivo.txt');
console.log('Arquivo existe:', existe);
```

## Módulo Path

O módulo `path` ajuda a trabalhar com caminhos de arquivos e diretórios de forma consistente em diferentes sistemas operacionais.

### 1. Operações Básicas

```javascript
const path = require('path');

// Juntar caminhos
const caminhoCompleto = path.join('/pasta', 'subpasta', 'arquivo.txt');
console.log(caminhoCompleto); // /pasta/subpasta/arquivo.txt

// Resolver caminho absoluto
const caminhoAbsoluto = path.resolve('pasta', 'arquivo.txt');
console.log(caminhoAbsoluto);

// Extensão do arquivo
const extensao = path.extname('arquivo.txt');
console.log(extensao); // .txt
```

### 2. Componentes do Caminho

```javascript
const caminho = '/pasta/subpasta/arquivo.txt';

console.log(path.dirname(caminho));  // /pasta/subpasta
console.log(path.basename(caminho)); // arquivo.txt
console.log(path.parse(caminho));    // Objeto com informações detalhadas
```

## Exemplos Práticos

### 1. Logger de Aplicação (cria um arquivo de log)

```javascript
const fs = require('fs');
const path = require('path');

class Logger {
    constructor(diretorioLogs) {
        this.diretorioLogs = diretorioLogs;
        this.arquivoLog = path.join(diretorioLogs, 'app.log');
        
        // Garantir que o diretório existe
        if (!fs.existsSync(diretorioLogs)) {
            fs.mkdirSync(diretorioLogs, { recursive: true });
        }
    }

    log(mensagem) {
        const data = new Date().toISOString();
        const entrada = `[${data}] ${mensagem}\n`;
        
        fs.appendFileSync(this.arquivoLog, entrada);
    }
}

const logger = new Logger('./logs');
logger.log('Aplicação iniciada');
logger.log('Usuário fez login');
```

### 2. Organizador de Arquivos (precisar ter uma pasta 'downloads' na raiz do projeto com arquivos de extensões variadas para ele organizar)

```javascript
const fs = require('fs');
const path = require('path');

function organizarArquivos(diretorio) {
    // Ler arquivos do diretório
    const arquivos = fs.readdirSync(diretorio);
    
    arquivos.forEach(arquivo => {
        const caminhoArquivo = path.join(diretorio, arquivo);
        
        // Ignorar diretórios
        if (fs.statSync(caminhoArquivo).isDirectory()) return;
        
        const extensao = path.extname(arquivo).slice(1);
        const pastaDestino = path.join(diretorio, extensao);
        
        // Criar pasta da extensão se não existir
        if (!fs.existsSync(pastaDestino)) {
            fs.mkdirSync(pastaDestino);
        }
        
        // Mover arquivo
        const novoCaminho = path.join(pastaDestino, arquivo);
        fs.renameSync(caminhoArquivo, novoCaminho);
    });
}

organizarArquivos('./downloads');
```

## Melhores Práticas

1. Sempre utilize caminhos relativos com `path.join()` ou `path.resolve()` para garantir compatibilidade entre sistemas operacionais.

2. Prefira métodos assíncronos para não bloquear o event loop, especialmente em operações com arquivos grandes.

3. Trate erros adequadamente usando try/catch ou callbacks.

4. Use `fs.promises` para trabalhar com Promises e async/await, tornando o código mais limpo e manutenível.

5. Sempre verifique a existência de arquivos/diretórios antes de tentar acessá-los.

6. Utilize encoding UTF-8 ao ler arquivos de texto para evitar problemas com caracteres especiais.

7. Evite operações síncronas em produção, use-as apenas em scripts de configuração ou inicialização.

8. Implemente logs para operações críticas de arquivo para facilitar o debugging.

## Exercícios Práticos

1. Crie um sistema de backup que:
   - Monitore uma pasta específica
   - Copie arquivos novos/modificados para uma pasta de backup
   - Mantenha um arquivo de log das operações

2. Desenvolva um analisador de arquivos que:
   - Liste todos os arquivos de um diretório recursivamente
   - Calcule o tamanho total ocupado
   - Gere um relatório em JSON

3. Implemente um sistema de templates que:
   - Leia um arquivo HTML template
   - Substitua variáveis no formato {{variavel}}
   - Gere o arquivo final com as substituições
