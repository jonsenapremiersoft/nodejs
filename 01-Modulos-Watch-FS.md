# Monitoramento de Arquivos com fs.watch

O `fs.watch` é um recurso poderoso que permite monitorar mudanças em arquivos e diretórios em tempo real. É perfeito para criar sistemas de backup automático, hot-reload em desenvolvimento, ou qualquer automação baseada em alterações de arquivos.

### Sintaxe Básica

```javascript
const fs = require('fs');

fs.watch('./pasta-monitorada', (evento, arquivo) => {
    console.log(`Evento: ${evento}`);
    console.log(`Arquivo alterado: ${arquivo}`);
});
```

O evento pode ser:
- 'rename': Quando um arquivo é criado ou deletado
- 'change': Quando um arquivo é modificado

### Exemplo Prático: Sistema de Backup Automático

```javascript
const fs = require('fs');
const path = require('path');

class BackupSystem {
    constructor(pastaOrigem, pastaBackup) {
        this.pastaOrigem = pastaOrigem;
        this.pastaBackup = pastaBackup;
        this.logFile = path.join(pastaBackup, 'backup.log');

        // Criar pasta de backup se não existir
        if (!fs.existsSync(pastaBackup)) {
            fs.mkdirSync(pastaBackup, { recursive: true });
        }
    }

    iniciar() {
        console.log(`Monitorando a pasta: ${this.pastaOrigem}`);
        
        fs.watch(this.pastaOrigem, (evento, arquivo) => {
            // Ignorar arquivos temporários (comum em alguns editores)
            if (arquivo.startsWith('.')) return;
            
            this.realizarBackup(arquivo);
        });
    }

    realizarBackup(arquivo) {
        const origem = path.join(this.pastaOrigem, arquivo);
        const destino = path.join(this.pastaBackup, arquivo);
        
        // Verificar se o arquivo ainda existe
        // (necessário pois o evento pode ser disparado na deleção)
        if (fs.existsSync(origem)) {
            try {
                // Copiar arquivo
                fs.copyFileSync(origem, destino);
                
                // Registrar no log
                const data = new Date().toISOString();
                const logMessage = `[${data}] Backup realizado: ${arquivo}\n`;
                fs.appendFileSync(this.logFile, logMessage);
                
                console.log(`Backup realizado: ${arquivo}`);
            } catch (erro) {
                console.error(`Erro ao fazer backup de ${arquivo}:`, erro);
            }
        }
    }
}

// Uso do sistema de backup
const backup = new BackupSystem('./pasta-trabalho', './pasta-backup');
backup.iniciar();
```

### Considerações Importantes

1. **Granularidade dos Eventos**: O comportamento do `fs.watch` pode variar entre sistemas operacionais:
   - No Windows, você recebe eventos separados para cada alteração
   - No Linux/macOS, você pode receber múltiplas alterações em um único evento

2. **Eventos Duplicados**: Às vezes o `fs.watch` pode emitir múltiplos eventos para uma única alteração. Considere implementar um debounce:

```javascript
const debounce = new Set();

fs.watch('./pasta', (evento, arquivo) => {
    // Evitar eventos duplicados em um curto intervalo
    if (debounce.has(arquivo)) return;
    
    debounce.add(arquivo);
    setTimeout(() => debounce.delete(arquivo), 100);
    
    // Seu código aqui
});
```

3. **Monitoramento Recursivo**: O `fs.watch` não monitora subpastas automaticamente. Se você precisa disso, terá que implementar manualmente:

```javascript
function watchRecursive(pasta) {
    // Monitorar a pasta principal
    fs.watch(pasta, (evento, arquivo) => {
        console.log(`Alteração em: ${arquivo}`);
    });

    // Monitorar subpastas
    fs.readdirSync(pasta).forEach(item => {
        const caminho = path.join(pasta, item);
        if (fs.statSync(caminho).isDirectory()) {
            watchRecursive(caminho);
        }
    });
}

watchRecursive('./pasta-principal');
```

### Dicas para o Exercício de Backup

Para o exercício de backup sugerido anteriormente, considere implementar:

1. Verificação de integridade dos arquivos (comparar hash MD5/SHA)
2. Sistema de versionamento (manter várias versões do mesmo arquivo)
3. Compressão dos arquivos de backup
4. Filtros por tipo de arquivo
5. Interface de linha de comando para configuração
6. Relatórios periódicos de backup
7. Limpeza automática de backups antigos

Exemplo de como implementar versionamento:

```javascript
const crypto = require('crypto');

function gerarNomeVersionado(arquivo, pastaBackup) {
    const nome = path.parse(arquivo).name;
    const extensao = path.parse(arquivo).ext;
    const timestamp = new Date().toISOString().replace(/[:\.]/g, '-');
    
    return path.join(pastaBackup, `${nome}_${timestamp}${extensao}`);
}

function calcularHash(arquivo) {
    const conteudo = fs.readFileSync(arquivo);
    return crypto.createHash('md5').update(conteudo).digest('hex');
}
```
