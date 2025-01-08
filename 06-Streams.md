# Streams no Node.js: Entendendo o Fluxo de Dados

## O que são Streams?

Imagine que você está enchendo um copo com água da torneira. A água não cai toda de uma vez - ela flui continuamente. Streams no Node.js funcionam de maneira similar! Em vez de carregar um arquivo inteiro na memória (como pegar um balde cheio de água), as streams processam os dados aos poucos (como a água fluindo da torneira).

## Por que usar Streams?

Vamos pensar em um exemplo prático: imagine que você precisa ler um arquivo de vídeo de 1GB. Se você tentar carregar todo o arquivo na memória de uma vez, isso pode:
- Consumir muita memória RAM
- Fazer seu programa ficar lento
- Até mesmo travar se não houver memória suficiente

Com streams, você pode processar esse mesmo arquivo em pequenos pedaços (chunks) de, por exemplo, 64KB cada. É como assistir um vídeo no YouTube - você não precisa baixar o vídeo inteiro para começar a assistir!

## Tipos de Streams

### 1. Readable Streams (Streams de Leitura)
São como torneiras - você consegue "ler" (receber) dados delas.

Exemplo simples de leitura de um arquivo:

```javascript
const fs = require('fs');

// Criando uma stream de leitura
// É como abrir uma torneira que vai liberar o conteúdo do arquivo aos poucos
const readStream = fs.createReadStream('arquivo.txt', {
    // Definindo que queremos ler texto
    encoding: 'utf8',
    // Definindo o tamanho de cada "gole" (chunk) como 64KB
    highWaterMark: 64 * 1024 
});

// Quando receber um pedaço de dados...
readStream.on('data', (chunk) => {
    // chunk é cada pedaço do arquivo
    console.log('Recebi um pedaço do arquivo!');
    console.log('Tamanho deste pedaço:', chunk.length, 'caracteres');
    console.log('Conteúdo deste pedaço:', chunk);
});

// Quando terminar de ler todo o arquivo...
readStream.on('end', () => {
    console.log('Terminei de ler o arquivo inteiro!');
});

// Se acontecer algum erro...
readStream.on('error', (error) => {
    console.log('Ops, algo deu errado:', error.message);
});
```

### 2. Writable Streams (Streams de Escrita)
São como um papel onde você pode escrever - você envia dados para elas.

```javascript
const fs = require('fs');

// Criando uma stream de escrita
// É como pegar uma folha de papel para escrever
const writeStream = fs.createWriteStream('meu_texto.txt');

// Escrevendo dados (é como escrever linhas no papel)
writeStream.write('Olá! Esta é a primeira linha\n');
writeStream.write('Esta é a segunda linha\n');

// Finalizando a escrita (é como dizer "terminei de escrever")
writeStream.end('Esta é a última linha!\n');

// Quando terminar todo o processo de escrita...
writeStream.on('finish', () => {
    console.log('Terminei de escrever tudo!');
});

// Se acontecer algum erro durante a escrita...
writeStream.on('error', (error) => {
    console.log('Ops, não consegui escrever:', error.message);
});
```

### 3. Pipe: Conectando Streams
O pipe é como um cano que liga uma torneira (readable stream) a um ralo (writable stream). É uma forma fácil de conectar streams!

```javascript
const fs = require('fs');

// Imagine que queremos copiar um arquivo
const arquivoOriginal = fs.createReadStream('original.txt', 'utf8');
const arquivoCopia = fs.createWriteStream('copia.txt');

// Conectando a torneira ao ralo!
arquivoOriginal.pipe(arquivoCopia);

// Quando a cópia terminar...
arquivoCopia.on('finish', () => {
    console.log('Arquivo copiado com sucesso!');
});
```

## Exemplo Prático: Servidor de Arquivos

Vamos criar um servidor que envia um arquivo grande de forma eficiente:

```javascript
const http = require('http');
const fs = require('fs');

// Criando um servidor web
const servidor = http.createServer((requisicao, resposta) => {
    // Vamos supor que temos um arquivo grande de texto
    const arquivo = fs.createReadStream('arquivo_grande.txt');
    
    // Se der algum erro ao ler o arquivo...
    arquivo.on('error', (erro) => {
        resposta.statusCode = 500; // código de erro do servidor
        resposta.end('Desculpe, não consegui ler o arquivo :(');
    });

    // Configurando o tipo de conteúdo que vamos enviar
    resposta.writeHead(200, { 'Content-Type': 'text/plain' });
    
    // Conectando a stream de leitura do arquivo à stream de resposta
    // É como criar um cano direto do arquivo para o navegador!
    arquivo.pipe(resposta);
});

// Iniciando o servidor na porta 3000
servidor.listen(3000, () => {
    console.log('Servidor está rodando! Acesse http://localhost:3000');
});
```

## Exercícios Práticos para Fixação

1. **Exercício Básico**: 
   Crie um programa que leia um arquivo de texto e mostre seu conteúdo no console, usando streams.

2. **Exercício Intermediário**: 
   Crie um programa que copie um arquivo, mas transforme todo o texto em MAIÚSCULAS durante a cópia.

3. **Exercício Avançado**: 
   Crie um servidor web que permita fazer upload de arquivos grandes usando streams.

## Dicas Importantes

1. **Sempre trate erros**: Suas streams podem dar erro! Use `.on('error', ...)` para tratá-los.
2. **Lembre-se do encoding**: Para arquivos de texto, use `encoding: 'utf8'`.
3. **Feche suas streams**: Quando terminar, use `.end()` ou `.destroy()` para liberar recursos.
4. **Comece pequeno**: Faça testes com arquivos pequenos antes de trabalhar com arquivos grandes.

## Conclusão

Streams são como canos de água do Node.js - permitem que você trabalhe com dados grandes de forma eficiente, processando-os aos poucos. Isso é especialmente útil quando você precisa:
- Ler ou escrever arquivos grandes
- Fazer upload/download de arquivos
- Processar dados em tempo real
- Trabalhar com vídeos ou áudio
- Lidar com grandes volumes de dados
