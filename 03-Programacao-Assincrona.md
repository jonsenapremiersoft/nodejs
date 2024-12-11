# Programação Assíncrona em Node.js

## Introdução

A programação assíncrona é um pilar fundamental no desenvolvimento com Node.js, permitindo que operações demoradas (como leitura de arquivos, requisições HTTP ou consultas a banco de dados) sejam executadas sem bloquear o fluxo principal da aplicação. Vamos explorar em detalhes os conceitos e implementações modernas.

## Modelo de Event Loop do Node.js

O Node.js utiliza um modelo de event loop single-threaded que permite operações não-bloqueantes. Este modelo funciona da seguinte maneira:

![Event Loop](https://miro.medium.com/v2/resize:fit:1120/1*pfxxWSTQfE8qutc1AO7AWw.png)

```javascript
console.log('Início');

setTimeout(() => {
    console.log('Execução assíncrona');
}, 1000);

console.log('Fim');

// Saída:
// Início
// Fim
// Execução assíncrona
```

## Callbacks

Callbacks são funções passadas como argumentos para outras funções, executadas após a conclusão de uma operação assíncrona. Embora ainda sejam encontradas em códigos legados, não são mais consideradas a melhor prática devido ao "callback hell".

```javascript
const fs = require('fs');

fs.readFile('arquivo.txt', 'utf8', (erro, dados) => {
    if (erro) {
        console.error('Erro na leitura:', erro);
        return;
    }
    
    fs.writeFile('novo-arquivo.txt', dados, (erro) => {
        if (erro) {
            console.error('Erro na escrita:', erro);
            return;
        }
        console.log('Arquivo processado com sucesso');
    });
});
```

### Problemas com Callbacks

* Difícil manutenção do código
* Tratamento de erros complexo
* Dificuldade em lidar com operações sequenciais
* Código profundamente aninhado ("callback hell")

## Promises

Promises são objetos que representam a eventual conclusão (ou falha) de uma operação assíncrona. São a base para um código mais limpo e manutenível.

### Anatomia de uma Promise

```javascript
const minhaPromise = new Promise((resolve, reject) => {
    // Operação assíncrona
    const sucesso = true;
    
    if (sucesso) {
        resolve('Operação concluída');
    } else {
        reject(new Error('Falha na operação'));
    }
});
```

### Métodos principais

```javascript
// Encadeamento de Promises
minhaPromise
    .then(resultado => {
        console.log(resultado);
        return 'Próximo valor';
    })
    .then(novoResultado => {
        console.log(novoResultado);
    })
    .catch(erro => {
        console.error(erro);
    })
    .finally(() => {
        console.log('Executa sempre');
    });

// Execução paralela
Promise.all([promise1, promise2, promise3])
    .then(resultados => {
        console.log('Todos concluídos:', resultados);
    });

// Primeira Promise a resolver
Promise.race([promise1, promise2, promise3])
    .then(resultado => {
        console.log('Primeiro a concluir:', resultado);
    });
```

## Async/Await

Async/Await é uma sintaxe moderna que permite escrever código assíncrono de forma síncrona, tornando-o mais legível e manutenível. É construída sobre Promises.

### Sintaxe Básica

```javascript
async function processarDados() {
    try {
        const dados = await buscarDados();
        const dadosProcessados = await processarDadosEmBatch(dados);
        const resultado = await salvarNoBanco(dadosProcessados);
        return resultado;
    } catch (erro) {
        console.error('Erro no processamento:', erro);
        throw erro;
    }
}
```

### Padrões Modernos com Async/Await

```javascript
// Execução paralela com async/await
async function buscarMultiplosDados() {
    try {
        const [usuarios, produtos, pedidos] = await Promise.all([
            buscarUsuarios(),
            buscarProdutos(),
            buscarPedidos()
        ]);
        
        return {
            usuarios,
            produtos,
            pedidos
        };
    } catch (erro) {
        console.error('Erro na busca:', erro);
        throw erro;
    }
}

// Iteração assíncrona
async function processarLista(items) {
    const resultados = [];
    
    for await (const item of items) {
        const resultado = await processarItem(item);
        resultados.push(resultado);
    }
    
    return resultados;
}
```

## Melhores Práticas

1. Prefira async/await sobre Promises puras e callbacks
2. Sempre utilize try/catch para tratamento de erros
3. Para operações independentes, use Promise.all() para execução paralela
4. Evite await dentro de loops para operações independentes
5. Utilize funções utilitárias para timeout e retry:

```javascript
// Função de timeout
const timeout = (ms) => new Promise(resolve => setTimeout(resolve, ms));

// Função de retry
async function withRetry(fn, maxRetries = 3, delay = 1000) {
    for (let i = 0; i < maxRetries; i++) {
        try {
            return await fn();
        } catch (erro) {
            if (i === maxRetries - 1) throw erro;
            await timeout(delay);
        }
    }
}

// Exemplo de uso
async function buscarDadosComRetry() {
    return withRetry(async () => {
        const response = await fetch('https://api.exemplo.com/dados');
        return response.json();
    });
}
```

## Utilitários Modernos

### Async Iterators

```javascript
// Gerador assíncrono
async function* geradorDeDados() {
    for (let i = 0; i < 5; i++) {
        await timeout(1000);
        yield `Dado ${i}`;
    }
}

// Consumindo o gerador
async function consumirDados() {
    for await (const dado of geradorDeDados()) {
        console.log(dado);
    }
}
```

### AbortController

```javascript
// Cancelamento de operações assíncronas
async function buscarComTimeout() {
    const controller = new AbortController();
    const timeoutId = setTimeout(() => controller.abort(), 5000);
    
    try {
        const response = await fetch('https://api.exemplo.com/dados', {
            signal: controller.signal
        });
        clearTimeout(timeoutId);
        return response.json();
    } catch (erro) {
        if (erro.name === 'AbortError') {
            throw new Error('Operação cancelada por timeout');
        }
        throw erro;
    }
}
```

## Conclusão

A programação assíncrona em Node.js evoluiu significativamente, com async/await emergindo como o padrão mais moderno e legível. Combinando estes conceitos com padrões de projeto adequados, podemos criar aplicações robustas e eficientes.

Para aprofundamento, recomenda-se explorar:
* Streams em Node.js para processamento de grandes volumes de dados
* Event Emitters para comunicação baseada em eventos
* Workers Threads para processamento paralelo real
* Bibliotecas como `bluebird` para recursos avançados de Promises
