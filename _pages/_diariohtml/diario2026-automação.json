// ===== SCRIPT DE AUTOMATIZAÇÃO =====
// Este script atualiza automaticamente o índice e o contador de entradas

(function() {
    'use strict';

    // ===== CONFIGURAÇÕES =====
    const CONFIG = {
        containerSelector: '#container',        // Seletor das entradas
        dateSelector: '.date',                  // Seletor da data dentro da entrada
        sectionPrefix: 'section-',              // Prefixo dos IDs das seções
        navSelector: '.sidebar-nav',           // Seletor da navegação
        navTitleSelector: '.nav-title',        // Seletor do título da navegação
        navCounterSelector: '.nav-counter',    // Seletor do contador
        navDividerSelector: '.nav-divider',    // Seletor do divisor
        todoId: 'todo',                        // ID da seção to-do
        maxPreviewLength: 30,                  // Tamanho máximo do preview
        debug: false                           // Modo debug (true para logs)
    };

    // ===== FUNÇÕES AUXILIARES =====

    /**
     * Obtém a data de uma entrada
     * @param {HTMLElement} entry - Elemento da entrada
     * @returns {string} Data no formato YYYY-MM-DD ou string vazia
     */
    function getEntryDate(entry) {
        const dateEl = entry.querySelector(CONFIG.dateSelector);
        if (!dateEl) return '';
        return dateEl.textContent.trim();
    }

    /**
     * Obtém o texto de preview de uma entrada
     * @param {HTMLElement} entry - Elemento da entrada
     * @param {number} maxLength - Tamanho máximo do preview
     * @returns {string} Texto do preview
     */
    function getEntryPreview(entry, maxLength = CONFIG.maxPreviewLength) {
        const pTags = entry.querySelectorAll('p');
        let text = '';
        
        // Pega o primeiro parágrafo que não seja a data
        for (let p of pTags) {
            if (p !== entry.querySelector(CONFIG.dateSelector) && p.textContent.trim()) {
                text = p.textContent.trim();
                break;
            }
        }
        
        // Se não encontrou parágrafo, pega qualquer texto
        if (!text) {
            text = entry.textContent.trim();
            // Remove a data se estiver no texto
            const dateText = getEntryDate(entry);
            if (dateText && text.startsWith(dateText)) {
                text = text.substring(dateText.length).trim();
            }
        }
        
        // Trunca se necessário
        if (text.length > maxLength) {
            text = text.substring(0, maxLength) + '...';
        }
        
        return text || 'Sem descrição';
    }

    /**
     * Extrai número da seção a partir do ID
     * @param {string} id - ID do elemento
     * @returns {number|null} Número da seção ou null
     */
    function extractSectionNumber(id) {
        const match = id.match(new RegExp(CONFIG.sectionPrefix + '(\\d+)'));
        return match ? parseInt(match[1]) : null;
    }

    /**
     * Gera um slug a partir de um texto
     * @param {string} text - Texto para gerar slug
     * @returns {string} Slug gerado
     */
    function generateSlug(text) {
        return text
            .toLowerCase()
            .replace(/[áàâãä]/g, 'a')
            .replace(/[éèêë]/g, 'e')
            .replace(/[íìîï]/g, 'i')
            .replace(/[óòôõö]/g, 'o')
            .replace(/[úùûü]/g, 'u')
            .replace(/[ç]/g, 'c')
            .replace(/[^a-z0-9\s-]/g, '')
            .replace(/\s+/g, '-')
            .replace(/-+/g, '-')
            .substring(0, 30);
    }

    /**
     * Verifica se uma string contém apenas números
     * @param {string} str - String a verificar
     * @returns {boolean} Verdadeiro se for numérica
     */
    function isNumeric(str) {
        return /^\d+$/.test(str);
    }

    // ===== FUNÇÕES PRINCIPAIS =====

    /**
     * Coleta todas as entradas do diário
     * @returns {Array} Array de objetos com dados das entradas
     */
    function collectEntries() {
        const entries = [];
        const containerElements = document.querySelectorAll(CONFIG.containerSelector);
        
        containerElements.forEach((entry, index) => {
            // Se não tiver ID, gera um
            if (!entry.id) {
                const date = getEntryDate(entry);
                if (date && date.match(/^\d{4}-\d{2}-\d{2}$/)) {
                    entry.id = CONFIG.sectionPrefix + date.replace(/-/g, '');
                } else {
                    entry.id = CONFIG.sectionPrefix + (index + 1);
                }
            }

            // Verifica se é a seção de to-do (pode ter data igual)
            if (entry.id === CONFIG.todoId) return;

            const date = getEntryDate(entry);
            if (!date) return;

            const preview = getEntryPreview(entry);
            
            entries.push({
                id: entry.id,
                date: date,
                preview: preview,
                element: entry
            });
        });

        // Ordena por data (mais recente primeiro)
        entries.sort((a, b) => {
            if (a.date < b.date) return 1;
            if (a.date > b.date) return -1;
            return 0;
        });

        return entries;
    }

    /**
     * Atualiza a navegação com as entradas coletadas
     * @param {Array} entries - Array de entradas
     * @param {HTMLElement} nav - Elemento da navegação
     */
    function updateNavigation(entries, nav) {
        if (!nav) {
            console.warn('Elemento de navegação não encontrado');
            return;
        }

        // Remove links antigos (mantém o título e divisor)
        const navTitle = nav.querySelector(CONFIG.navTitleSelector);
        const navDivider = nav.querySelector(CONFIG.navDividerSelector);
        
        // Remove todos os nós após o título
        const children = Array.from(nav.children);
        let removeStart = false;
        
        children.forEach(child => {
            if (child === navTitle) {
                removeStart = true;
                return;
            }
            if (removeStart && child !== navDivider) {
                child.remove();
            }
        });

        // Atualiza o contador no título
        if (navTitle) {
            let counterSpan = navTitle.querySelector(CONFIG.navCounterSelector);
            if (!counterSpan) {
                counterSpan = document.createElement('span');
                counterSpan.className = 'nav-counter';
                navTitle.appendChild(counterSpan);
            }
            const entryCount = entries.length;
            counterSpan.textContent = entryCount + ' entrada' + (entryCount !== 1 ? 's' : '');
        }

        // Insere os novos links
        const insertBeforeElement = navDivider || null;
        
        entries.forEach(entry => {
            const link = document.createElement('a');
            link.href = '#' + entry.id;

            const dateSpan = document.createElement('span');
            dateSpan.className = 'entry-date';
            dateSpan.textContent = entry.date;

            const previewSpan = document.createElement('span');
            previewSpan.className = 'entry-preview';
            previewSpan.textContent = entry.preview;

            link.appendChild(dateSpan);
            link.appendChild(previewSpan);

            // Insere antes do divisor ou no final
            if (insertBeforeElement) {
                nav.insertBefore(link, insertBeforeElement);
            } else {
                nav.appendChild(link);
            }
        });
    }

    /**
     * Atualiza a âncora do to-do list
     */
    function updateTodoAnchor() {
        let todoElement = document.getElementById(CONFIG.todoId);
        if (!todoElement) {
            todoElement = document.createElement('div');
            todoElement.id = CONFIG.todoId;
            todoElement.style.scrollMarginTop = '20px';
            document.body.appendChild(todoElement);
        }
    }

    /**
     * Log de debug
     * @param {string} message - Mensagem a ser logada
     * @param {*} data - Dados opcionais
     */
    function debugLog(message, data = null) {
        if (CONFIG.debug) {
            if (data) {
                console.log('[Diário HTML] ' + message, data);
            } else {
                console.log('[Diário HTML] ' + message);
            }
        }
    }

    // ===== FUNÇÃO PRINCIPAL =====

    /**
     * Função principal que executa a automatização
     */
    function main() {
        debugLog('Iniciando automatização do diário...');

        // Coleta as entradas
        const entries = collectEntries();
        debugLog('Entradas coletadas:', entries);

        // Atualiza a navegação
        const nav = document.querySelector(CONFIG.navSelector);
        if (nav) {
            updateNavigation(entries, nav);
            debugLog('Navegação atualizada');
        } else {
            console.warn('Elemento de navegação não encontrado');
        }

        // Atualiza âncora do to-do
        updateTodoAnchor();
        debugLog('Âncora do to-do atualizada');

        // Atualiza a última atualização
        const lastUpdateEl = document.getElementById('ultima-atualizacao');
        if (lastUpdateEl && entries.length > 0) {
            const lastDate = entries[0].date;
            const now = new Date();
            const dateObj = new Date(lastDate + 'T00:00:00');
            
            if (!isNaN(dateObj.getTime())) {
                const options = { 
                    year: 'numeric', 
                    month: 'long', 
                    day: 'numeric' 
                };
                const formattedDate = dateObj.toLocaleDateString('pt-BR', options);
                
                // Calcular dias desde a última atualização
                const diffTime = Math.abs(now - dateObj);
                const diffDays = Math.ceil(diffTime / (1000 * 60 * 60 * 24));
                const diffText = diffDays === 0 ? 'hoje' : `há ${diffDays} dias`;
                
                lastUpdateEl.innerHTML = `Última atualização: <strong>${formattedDate}</strong> (${diffText}) &bull; ${entries.length} registro${entries.length !== 1 ? 's' : ''}`;
            } else {
                lastUpdateEl.textContent = `Última atualização: ${lastDate}`;
            }
        }

        debugLog('Automatização concluída!');
    }

    // ===== EXECUÇÃO =====

    // Executa quando o DOM estiver pronto
    if (document.readyState === 'loading') {
        document.addEventListener('DOMContentLoaded', main);
    } else {
        main();
    }

    // ===== EXPOSIÇÃO PARA DEBUG =====
    // Permite executar novamente via console
    window.refreshDiario = main;

})();
        
