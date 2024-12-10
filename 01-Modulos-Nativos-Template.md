# Implementando Template de E-mail com NodeJS

### Estrutura de Arquivos Sugerida
```
projeto/
  ├── templates/
  │   └── email.html
  ├── data/
  │   └── user.json
  └── index.js
```

### Exemplo de Template (email.html)
```html
<!DOCTYPE html>
<html>
<body>
    <h1>Olá {{nome}}!</h1>
    <p>Bem-vindo à {{empresa}}.</p>
    <p>Seu cargo será: {{cargo}}</p>
    <p>Data de início: {{dataInicio}}</p>
</body>
</html>
```

### Exemplo de Dados (user.json)
```json
{
    "nome": "João Silva",
    "empresa": "TechCorp",
    "cargo": "Desenvolvedor Senior",
    "dataInicio": "15/12/2024"
}
```

### Implementação Base (index.js)

```javascript
const fs = require('fs');
const path = require('path');

function processarTemplate(template, dados) {
    let resultado = template;
    
    // Encontrar todas as variáveis no formato {{variavel}}
    const variaveis = template.match(/\{\{([^}]+)\}\}/g) || [];
    
    // Substituir cada variável pelo valor correspondente
    variaveis.forEach(variavel => {
        const chave = variavel.replace(/\{\{|\}\}/g, '');
        resultado = resultado.replace(variavel, dados[chave] || '');
    });
    
    return resultado;
}

async function gerarHTML(templatePath, dadosPath, outputPath) {
    try {
        // Ler o template e os dados
        const template = await fs.promises.readFile(templatePath, 'utf-8');
        const dados = JSON.parse(await fs.promises.readFile(dadosPath, 'utf-8'));
        
        // Processar o template
        const htmlFinal = processarTemplate(template, dados);
        
        // Salvar o resultado
        await fs.promises.writeFile(outputPath, htmlFinal);
        
        console.log('Template processado com sucesso!');
    } catch (erro) {
        console.error('Erro ao processar template:', erro);
    }
}
```

### Sugestões para Expandir o Exercício:

1. Adicionar suporte a loops:
```html
{{#each produtos}}
    <li>{{nome}} - R$ {{preco}}</li>
{{/each}}
```

2. Adicionar suporte a condicionais:
```html
{{#if premium}}
    <span class="badge">Premium</span>
{{/if}}
```

3. Permitir includes/partials:
```html
{{> header}}
<div class="content">
    <!-- conteúdo -->
</div>
{{> footer}}
```

4. Adicionar formatação de dados:
```html
<p>Data: {{formatDate data "DD/MM/YYYY"}}</p>
<p>Preço: {{formatMoney preco}}</p>
```

5. Criar uma CLI para uso do sistema:
```bash
node template-cli.js --template=email.html --data=user.json --output=resultado.html
```

### Dicas de Implementação:

1. Use expressões regulares para encontrar os padrões de substituição
2. Implemente validação de dados e templates
3. Adicione tratamento de erros robusto
4. Crie uma documentação clara de uso
5. Implemente cache de templates para melhor performance
6. Adicione suporte a diferentes tipos de arquivo (HTML, TXT, MD)
7. Crie testes unitários para garantir o funcionamento

### Bônus: Exemplo de Implementação com Cache

```javascript
class TemplateEngine {
    constructor() {
        this.cache = new Map();
    }

    async lerTemplate(caminho) {
        if (this.cache.has(caminho)) {
            return this.cache.get(caminho);
        }

        const template = await fs.promises.readFile(caminho, 'utf-8');
        this.cache.set(caminho, template);
        return template;
    }

    async processar(templatePath, dados, outputPath) {
        const template = await this.lerTemplate(templatePath);
        const resultado = this.processarTemplate(template, dados);
        await fs.promises.writeFile(outputPath, resultado);
    }
}
```
