# mae_prompt_generator


<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Gerador de JSON para M.A.E.</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 20px;
            background-color: #f5f5f5;
        }
        .container {
            max-width: 800px;
            margin: auto;
            background-color: #fff;
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 2px 4px rgba(0,0,0,0.1);
        }
        .campo-principal {
            margin-bottom: 20px;
        }
        label {
            display: block;
            margin-bottom: 5px;
            font-weight: bold;
            color: #333;
        }
        textarea, input, select {
            width: 100%;
            padding: 10px;
            border: 1px solid #ddd;
            border-radius: 4px;
            margin-bottom: 10px;
            font-family: inherit;
        }
        textarea {
            resize: vertical;
            min-height: 100px;
        }
        .button-group {
            margin: 20px 0;
            display: flex;
            gap: 10px;
            flex-wrap: wrap;
        }
        button {
            padding: 10px 15px;
            background-color: #346067;
            color: white;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            transition: background-color 0.3s;
        }
        button:hover {
            background-color: #2a4c52;
        }
        pre {
            background-color: #f8f8f8;
            padding: 15px;
            border-radius: 4px;
            overflow-x: auto;
            border: 1px solid #ddd;
        }
        .referencias-section {
            margin: 20px 0;
            border: 1px dashed #ccc;
            border-radius: 4px;
        }
        .collapse-trigger {
            width: 100%;
            padding: 10px;
            text-align: left;
            background: #f8f8f8;
            border: none;
            cursor: pointer;
            display: flex;
            align-items: center;
            gap: 10px;
        }
        .collapse-trigger:hover {
            background: #f0f0f0;
        }
        .collapse-trigger small {
            color: #666;
            margin-left: auto;
        }
        .collapse-content {
            padding: 15px;
            border-top: 1px dashed #ccc;
        }
        .referencias-lista {
            max-height: 200px;
            overflow-y: auto;
            margin-top: 10px;
        }
        .referencia-item {
            display: flex;
            justify-content: space-between;
            align-items: center;
            padding: 8px;
            border-bottom: 1px solid #eee;
        }
        .referencia-item button {
            padding: 2px 6px;
            background: #ff4444;
        }
        #outroLinkIA {
            margin-top: 10px;
        }
        .ia-selector {
            margin-bottom: 20px;
        }
    </style>
</head>
<body>

<div class="container">
    <h1>Gerador de JSON para M.A.E.</h1>
    
    <form id="maeForm">
        <div class="campo-principal">
            <label for="principios">Princípios:</label>
            <textarea id="principios" placeholder="Descreva detalhadamente os princípios..." required></textarea>
            
            <label for="titulo_resumido">Título Principal (máx. 5 palavras):</label>
            <input type="text" id="titulo_resumido" maxlength="50" 
                   placeholder="Resumo conciso do tema principal" required>
        </div>

        <div class="campo-principal">
            <label for="desenvolvidoPor">Desenvolvido por:</label>
            <textarea id="desenvolvidoPor" 
                      placeholder="Informações sobre autor/desenvolvedor..." required></textarea>
        </div>

        <div class="campo-principal">
            <label for="aplicacao">Aplicação em:</label>
            <textarea id="aplicacao" 
                      placeholder="Contexto detalhado de aplicação..." required></textarea>
        </div>

        <div class="referencias-section">
            <button type="button" class="collapse-trigger" onclick="toggleReferencias()">
                <span class="icon">▶</span> Sites de Referência (opcional)
                <small>Expandir para adicionar sites base para geração de URLs</small>
            </button>
            
            <div id="referenciasCollapse" class="collapse-content" style="display: none;">
                <div class="referencias-container">
                    <div class="referencias-input-group">
                        <input type="url" id="referenciaUrl" 
                               placeholder="https://site-referencia.com">
                        <button type="button" onclick="referencias.adicionar()">Adicionar</button>
                    </div>
                    <div id="listaReferencias" class="referencias-lista"></div>
                    <small>As URLs geradas serão baseadas nestes sites de referência</small>
                </div>
            </div>
        </div>

        <div class="ia-selector">
            <label for="iaEscolhida">Escolha sua IA:</label>
            <select id="iaEscolhida" onchange="verificarSelecaoIA()" required>
                <option value="">Selecione uma IA</option>
                <option value="https://chat.openai.com/">ChatGPT (OpenAI)</option>
                <option value="https://gemini.google.com/">Gemini (Google)</option>
                <option value="https://claude.ai/">Claude (Anthropic)</option>
                <option value="https://copilot.microsoft.com/">Copilot (Microsoft)</option>
                <option value="outro">Outra IA...</option>
            </select>
            
            <div id="outroLinkIA" style="display: none;">
                <label for="linkPersonalizado">Cole o link da IA:</label>
                <input type="url" id="linkPersonalizado" 
                       placeholder="https://sua-ia-preferida.com">
            </div>
        </div>

        <div class="button-group">
            <button type="button" onclick="gerarJSONePrompt()">1. Gerar JSON e Prompt</button>
            <button type="button" onclick="copiarPrompt()">2. Copiar Prompt</button>
            <button type="button" onclick="abrirIASelecionada()">3. Abrir IA Selecionada</button>
        </div>
    </form>

    <h6>JSON Gerado:</h6>
    <pre id="jsonOutput"></pre>
    
    <h6>Prompt Gerado:</h6>
    <pre id="promptOutput"></pre>

    <h6>Colar Texto da IA:</h6>
    <textarea id="gptText" rows="10"></textarea>
    <button type="button" onclick="processarTextoGPT()">4. Processar Texto</button>
</div>

<script>
// Configuração do sistema
const MAE_CONFIG = {
    versao: 'free',
    features: {
        referencias: {
            limiteDominio: false,
            limiteQuantidade: false,
            maxReferencias: Infinity,
            permiteDeletar: true
        }
    }
};

// Sistema de referências
const referencias = {
    lista: [],
    
    adicionar: function() {
        const input = document.getElementById('referenciaUrl');
        const url = input.value.trim();
        
        if (url && this.validarUrl(url)) {
            if (this.podeAdicionarReferencia(url)) {
                this.lista.push({
                    url: url,
                    timestamp: new Date().toISOString()
                });
                this.atualizarLista();
                input.value = '';
                this.salvarReferencias();
            }
        }
    },

    podeAdicionarReferencia: function(novaUrl) {
        const config = MAE_CONFIG.features.referencias;
        
        if (config.limiteQuantidade && this.lista.length >= config.maxReferencias) {
            alert('Limite de referências atingido. Faça upgrade para adicionar mais!');
            return false;
        }

        return true;
    },

    remover: function(index) {
        if (MAE_CONFIG.features.referencias.permiteDeletar) {
            this.lista.splice(index, 1);
            this.atualizarLista();
            this.salvarReferencias();
        } else {
            alert('Remoção de referências disponível apenas na versão PRO!');
        }
    },

    atualizarLista: function() {
        const container = document.getElementById('listaReferencias');
        container.innerHTML = '';
        
        this.lista.forEach((ref, index) => {
            const div = document.createElement('div');
            div.className = 'referencia-item';
            div.innerHTML = `
                <a href="${ref.url}" target="_blank">${ref.url}</a>
                <button onclick="referencias.remover(${index})">×</button>
            `;
            container.appendChild(div);
        });
    },

    validarUrl: function(url) {
        try {
            new URL(url);
            return true;
        } catch {
            alert('Por favor, insira uma URL válida');
            return false;
        }
    },

    salvarReferencias: function() {
        localStorage.setItem('mae_referencias', JSON.stringify(this.lista));
    },

    carregarReferencias: function() {
        const salvas = localStorage.getItem('mae_referencias');
        if (salvas) {
            this.lista = JSON.parse(salvas);
            this.atualizarLista();
        }
    },

    gerarUrlValida: function(categoria) {
        if (this.lista.length === 0) {
            return `https://exemplo.com/${categoria.toLowerCase()}`;
        }
        
        const refIndex = Math.floor(Math.random() * this.lista.length);
        return `${this.lista[refIndex].url}/${categoria.toLowerCase()}`;
    }
};

function escapeUnicode(str) {
    return str.replace(/[^\0-~]/g, function(ch) {
        return "\\u" + ("000" + ch.charCodeAt().toString(16)).slice(-4);
    });
}

function verificarSelecaoIA() {
    const select = document.getElementById('iaEscolhida');
    const outroLink = document.getElementById('outroLinkIA');
    
    if (select.value === 'outro') {
        outroLink.style.display = 'block';
    } else {
        outroLink.style.display = 'none';
    }
}

function toggleReferencias() {
    const content = document.getElementById('referenciasCollapse');
    const trigger = document.querySelector('.collapse-trigger .icon');
    
    if (content.style.display === 'none') {
        content.style.display = 'block';
        trigger.textContent = '▼';
    } else {
        content.style.display = 'none';
        trigger.textContent = '▶';
    }
}

function gerarJSONePrompt() {
    const principios = escapeUnicode(document.getElementById('principios').value);
    const desenvolvidoPor = escapeUnicode(document.getElementById('desenvolvidoPor').value);
    const aplicacao = escapeUnicode(document.getElementById('aplicacao').value);
    const titulo = escapeUnicode(document.getElementById('titulo_resumido').value);

    const jsonData = {
        "01-00-introducao": {
            "title": `1. ${titulo}`,
            "description": `Exploração de como os princípios de ${principios}, desenvolvidos por ${desenvolvidoPor}, podem ser aplicados em ${aplicacao}.`,
            "url": referencias.gerarUrlValida("introducao"),
            "category": `1. ${principios} em ${aplicacao}`
        }
    };

    const prompt = `
Tarefa: Criar uma estrutura JSON para o M.A.E. que explique como os princípios [${principios}], desenvolvidos por [${desenvolvidoPor}], podem ser aplicados em [${aplicacao}].
Requisitos:
- Formato JSON: Estruturar o conteúdo em formato JSON.
- Codificação Unicode: Utilizar codificação Unicode para caracteres especiais.
- Exemplos Práticos: Incluir exemplos práticos com formatação de texto com quebra de linha e parágrafos.
- URLs Relevantes: Incluir URLs relevantes e válidas para cada item.
- Estrutura de Categorias: Criar 7 categorias relevantes para o contexto de consultoria.
- Detalhamento das Categorias: Cada categoria deve conter uma descrição clara e objetiva.
`;

    document.getElementById('jsonOutput').textContent = JSON.stringify(jsonData, null, 2);
    document.getElementById('promptOutput').textContent = prompt;
}

function copiarPrompt() {
    const promptContent = document.getElementById('promptOutput').textContent;
    navigator.clipboard.writeText(promptContent).then(() => {
        alert("Prompt copiado para a área de transferência!");
    });
}

function abrirIASelecionada() {
    const select = document.getElementById('iaEscolhida');
    let url = select.value;
    
    if (url === 'outro') {
        url = document.getElementById('linkPersonalizado').value;
    }
    
    if (!url) {
        alert('Por favor, selecione uma IA ou forneça um link válido!');
        return;
    }
    
    abrirOuFocarJanela(url, 'IA_Window', 800, 600, 200, 100);
}

function abrirOuFocarJanela(url, title, width, height, left, top) {
    const newWindow = window.open(url, title, `width=${width},height=${height},left=${left},top=${top}`);
    if (newWindow.focus) {
        newWindow.focus();
    }
    document.getElementById('gptText').focus();
}

function processarTextoGPT() {
    const gptText = document.getElementById('gptText').value;
    try {
        const processedJSON = JSON.parse(gptText);
        const blob = new Blob([JSON.stringify(processedJSON, null, 2)], { type: "application/json" });
        const url = URL.createObjectURL(blob);
        const a = document.createElement("a");
        a.href = url;
        const nomeArquivo = prompt("Digite o nome do arquivo:", "resultado_gpt.json");
        if (nomeArquivo) {
            a.download = nomeArquivo;
            a.click();
            URL.revokeObjectURL(url);
        }
    } catch (error) {
        alert("Erro ao processar o texto. Verifique se o formato está correto.");
    }
}

// Inicialização
document.addEventListener('DOMContentLoaded', function() {
    referencias.carregarReferencias();
});
</script>

</body>
</html>