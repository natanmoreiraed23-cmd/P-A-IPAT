<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Dashboard IPAT - Multi Seleção</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/PapaParse/5.4.1/papaparse.min.js"></script>
    <style>
        body { font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; background-color: #f3f4f6; }
        .card { background: white; border-radius: 8px; box-shadow: 0 2px 4px rgba(0,0,0,0.1); padding: 20px; }
        .negative { color: #ef4444; font-weight: bold; }
        .positive { color: #10b981; font-weight: bold; }
        .modal { position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.5); display: none; justify-content: center; align-items: center; z-index: 50; }
        .modal-content { background: white; border-radius: 8px; width: 95%; max-height: 95vh; overflow-y: auto; padding: 20px; }
        .table-container { overflow-x: auto; }
        table { width: 100%; border-collapse: collapse; font-size: 0.85rem; }
        th, td { padding: 8px; border-bottom: 1px solid #e5e7eb; text-align: left; }
        th { background-color: #f9fafb; font-weight: 600; white-space: nowrap; }
        tr:hover { background-color: #f0f9ff; cursor: pointer; }
        
        /* Estilo para o multiselect */
        .checkbox-list { max-height: 150px; overflow-y: auto; border: 1px solid #d1d5db; padding: 5px; border-radius: 4px; background: #fff; }
        .checkbox-item { display: flex; align-items: center; margin-bottom: 2px; }
        .checkbox-item input { margin-right: 8px; }
        .checkbox-item label { font-size: 0.8rem; color: #374151; cursor: pointer; white-space: nowrap; }
    </style>
</head>
<body class="p-4 md:p-6">

    <div class="max-w-full mx-auto space-y-6">
        
        <div class="card space-y-4">
            <div class="flex justify-between items-start">
                <div>
                    <h1 class="text-2xl font-bold text-gray-800">Dashboard IPAT</h1>
                    <p class="text-sm text-gray-600">Sistema de Análise de Produtividade Ajustada</p>
                </div>
                <div class="flex gap-2">
                    <button onclick="fazerBackup()" class="bg-gray-600 text-white px-3 py-1 rounded text-xs hover:bg-gray-700">Backup JSON</button>
                    <label class="bg-gray-600 text-white px-3 py-1 rounded text-xs hover:bg-gray-700 cursor-pointer">
                        Restaurar
                        <input type="file" id="fileRestore" accept=".json" class="hidden" onchange="restaurarBackup(this)">
                    </label>
                    <button onclick="limparDados()" class="bg-red-100 text-red-700 px-3 py-1 rounded text-xs hover:bg-red-200">Limpar</button>
                </div>
            </div>
            
            <div class="grid grid-cols-1 md:grid-cols-2 gap-4 border-b pb-4">
                <div>
                    <label class="block text-xs font-bold text-gray-700 mb-1">1. Detalhamento de Chamadas (.csv)</label>
                    <input type="file" id="fileChamadas" accept=".csv" class="block w-full text-sm text-gray-500 file:mr-4 file:py-1 file:px-2 file:rounded file:border-0 file:text-xs file:bg-blue-50 file:text-blue-700 hover:file:bg-blue-100">
                </div>
                <div>
                    <label class="block text-xs font-bold text-gray-700 mb-1">2. Eventos Atendentes (.csv)</label>
                    <input type="file" id="fileEventos" accept=".csv" class="block w-full text-sm text-gray-500 file:mr-4 file:py-1 file:px-2 file:rounded file:border-0 file:text-xs file:bg-blue-50 file:text-blue-700 hover:file:bg-blue-100">
                </div>
            </div>

            <div class="grid grid-cols-1 md:grid-cols-12 gap-4 items-start">
                <div class="md:col-span-2">
                    <label class="block text-xs font-bold text-gray-700">Data Inicial</label>
                    <input type="date" id="dataInicioFilter" class="border rounded p-1 w-full text-sm">
                </div>
                <div class="md:col-span-2">
                    <label class="block text-xs font-bold text-gray-700">Data Final</label>
                    <input type="date" id="dataFimFilter" class="border rounded p-1 w-full text-sm">
                </div>
                <div class="md:col-span-2">
                    <label class="block text-xs font-bold text-gray-700">Taxa Meta (%)</label>
                    <input type="number" id="taxaOcupacao" value="85" class="border rounded p-1 w-full text-sm">
                </div>
                 <div class="md:col-span-2 flex items-end h-full pt-4">
                    <button onclick="processarDados()" class="bg-blue-600 text-white px-6 py-2 rounded hover:bg-blue-700 w-full font-bold shadow">CALCULAR</button>
                </div>
                
                <div class="md:col-span-4">
                    <div class="flex justify-between items-end mb-1">
                        <label class="block text-xs font-bold text-gray-700">Filas (Selecione)</label>
                        <div class="space-x-1">
                            <button onclick="toggleFilas(true)" class="text-[10px] text-blue-600 underline">Todas</button>
                            <button onclick="toggleFilas(false)" class="text-[10px] text-blue-600 underline">Nenhuma</button>
                        </div>
                    </div>
                    <div id="filaContainer" class="checkbox-list">
                        <p class="text-gray-400 text-xs italic p-1">Carregue o arquivo de chamadas...</p>
                    </div>
                </div>
            </div>
        </div>

        <div id="kpiContainer" class="grid grid-cols-2 md:grid-cols-5 gap-3 hidden">
            <div class="card text-center p-3 border-l-4 border-blue-500">
                <h3 class="text-xs text-gray-500 font-bold uppercase">Recebidas</h3>
                <p class="text-2xl font-bold text-blue-600" id="kpiRecebidas">-</p>
                <p class="text-[10px] text-gray-400">Total Selecionado</p>
            </div>
            <div class="card text-center p-3 border-l-4 border-green-500">
                <h3 class="text-xs text-gray-500 font-bold uppercase">Atendidas</h3>
                <p class="text-2xl font-bold text-green-600" id="kpiAtendidas">-</p>
            </div>
            <div class="card text-center p-3 border-l-4 border-purple-500">
                <h3 class="text-xs text-gray-500 font-bold uppercase">Ocupação Média</h3>
                <p class="text-2xl font-bold text-purple-600" id="kpiOcupacao">-</p>
            </div>
            <div class="card text-center p-3 border-l-4 border-orange-500">
                <h3 class="text-xs text-gray-500 font-bold uppercase">Ocioso Médio</h3>
                <p class="text-2xl font-bold text-orange-600" id="kpiOcioso">-</p>
            </div>
            <div class="card text-center p-3 border-l-4 border-gray-800">
                <h3 class="text-xs text-gray-500 font-bold uppercase">IPAT Médio</h3>
                <p class="text-2xl font-bold" id="kpiIpat">-</p>
            </div>
        </div>

        <div id="mainTableCard" class="card hidden">
            <h2 class="text-lg font-bold mb-4 text-gray-800 border-b pb-2">Performance por Colaborador</h2>
            <div class="table-container">
                <table id="tabelaAgentes">
                    <thead>
                        <tr>
                            <th>Nome</th>
                            <th>Dias</th>
                            <th title="Total estimado de chamadas baseadas na ocupação">Estimativa</th>
                            <th title="Total de chamadas atendidas">Atendidas</th>
                            <th>Jornada Líq.</th>
                            <th>Tempo Falado</th>
                            <th>Pausas Prod.</th>
                            <th>Pausas Improd.</th>
                            <th>Tempo Ocioso</th>
                            <th>Ocupação</th>
                            <th>IPAT</th>
                        </tr>
                    </thead>
                    <tbody>
                        </tbody>
                </table>
            </div>
        </div>

    </div>

    <div id="modalDetalhe" class="modal">
        <div class="modal-content">
            <div class="flex justify-between items-center mb-4 border-b pb-2">
                <h2 class="text-xl font-bold text-gray-800" id="modalTitle">Detalhes</h2>
                <button onclick="closeModal()" class="text-gray-500 hover:text-red-600 text-3xl font-bold">&times;</button>
            </div>
            <div class="table-container">
                <table id="tabelaDetalheIntervalos">
                    <thead>
                        <tr>
                            <th>Intervalo</th>
                            <th>Status</th>
                            <th>Atendidas</th>
                            <th>Estimativa</th>
                            <th>Tempo Atend.</th>
                            <th>Tempo Ocioso</th>
                            <th>Pausa (Tipo)</th>
                            <th>Dur. Pausa</th>
                            <th>Acumulado Pausas</th>
                        </tr>
                    </thead>
                    <tbody id="modalBody"></tbody>
                </table>
            </div>
        </div>
    </div>

    <script>
        // ==========================================
        // ESTADO GLOBAL
        // ==========================================
        let rawDataChamadas = [];
        let rawDataEventos = [];
        
        // Configurações de Pausa
        const PAUSAS_PRODUTIVAS = ['pausa registro', 'registro', 'pausa cancelamento de plano', 'pausa mesa do acerto', 'pausa feedback', 'feedback'];
        const PAUSAS_IMPRODUTIVAS = ['pausa agua', 'agua', 'pausa banheiro', 'banheiro'];
        const PAUSAS_DEDUCAO_LIQUIDA = ['lanche', 'almoço', 'jantar'];

        // ==========================================
        // UTILITÁRIOS
        // ==========================================
        function timeToSeconds(timeStr) {
            if (!timeStr) return 0;
            let cleanStr = timeStr.split(' ')[0]; 
            const p = cleanStr.split(':');
            if (p.length < 2) return 0;
            let s = 0;
            if (p.length === 3) s = (+p[0]) * 3600 + (+p[1]) * 60 + (+p[2]);
            else s = (+p[0]) * 60 + (+p[1]);
            return s;
        }

        function secondsToTime(secs) {
            if (isNaN(secs) || secs < 0) secs = 0;
            const h = Math.floor(secs / 3600);
            const m = Math.floor((secs % 3600) / 60);
            const s = Math.floor(secs % 60);
            return `${String(h).padStart(2,'0')}:${String(m).padStart(2,'0')}:${String(s).padStart(2,'0')}`;
        }

        // Parse Seguro de Data (YYYY-MM-DD ou DD/MM/YYYY)
        function parseDate(dateStr) {
            if (!dateStr) return null;
            if (dateStr instanceof Date) return dateStr;
            
            // ISO
            if (dateStr.includes('-')) {
                // Se tiver T ou espaço, tenta construtor direto
                if (dateStr.includes('T') || dateStr.includes(' ')) return new Date(dateStr.replace(/-/g, '/'));
                // Se for só YYYY-MM-DD, cria com timezone local (append hora)
                return new Date(dateStr + ' 00:00:00');
            }
            // BR DD/MM/YYYY
            if (dateStr.includes('/')) {
                const parts = dateStr.split(' ');
                const dataParts = parts[0].split('/');
                let dateObj;
                if (dataParts[2].length === 4) {
                    dateObj = new Date(dataParts[2], dataParts[1]-1, dataParts[0]);
                } else {
                    dateObj = new Date(dateStr);
                }
                
                if (parts.length > 1) {
                    const timeParts = parts[1].split(':');
                    dateObj.setHours(timeParts[0], timeParts[1], timeParts[2] || 0);
                }
                return dateObj;
            }
            return new Date(dateStr);
        }

        // ==========================================
        // CARREGAMENTO ARQUIVOS
        // ==========================================
        document.getElementById('fileChamadas').addEventListener('change', (e) => loadCSV(e.target.files[0], 'chamadas'));
        document.getElementById('fileEventos').addEventListener('change', (e) => loadCSV(e.target.files[0], 'eventos'));

        function loadCSV(file, type) {
            if (!file) return;
            const reader = new FileReader();
            reader.onload = function(e) {
                const text = e.target.result;
                const lines = text.split('\n');
                if (lines.length < 6) {
                    alert('Arquivo inválido (menos de 6 linhas).');
                    return;
                }
                // Pular 5 linhas, cabeçalho na 6ª
                const csvContent = lines.slice(5).join('\n');
                
                Papa.parse(csvContent, {
                    header: true,
                    skipEmptyLines: true,
                    delimiter: ";", 
                    complete: function(results) {
                        if (type === 'chamadas') {
                            rawDataChamadas = results.data;
                            preencherFiltroFilas();
                        } else {
                            rawDataEventos = results.data;
                        }
                    }
                });
            };
            reader.readAsText(file, 'ISO-8859-1');
        }

        // ==========================================
        // NOVO FILTRO DE FILAS (MULTISELECT)
        // ==========================================
        function preencherFiltroFilas() {
            const container = document.getElementById('filaContainer');
            container.innerHTML = '';
            
            const filas = new Set();
            rawDataChamadas.forEach(r => {
                if (r.nomeFila) filas.add(r.nomeFila.replace(/"/g, '').trim());
            });
            
            // Ordenar alfabeticamente
            const sortedFilas = Array.from(filas).sort();

            if (sortedFilas.length === 0) {
                container.innerHTML = '<p class="text-xs text-red-500">Nenhuma fila encontrada.</p>';
                return;
            }

            sortedFilas.forEach(f => {
                const div = document.createElement('div');
                div.className = 'checkbox-item';
                
                const checkbox = document.createElement('input');
                checkbox.type = 'checkbox';
                checkbox.value = f;
                checkbox.checked = true; // Padrão todas marcadas
                checkbox.id = 'fila_' + f.replace(/\s+/g, '_');
                
                const label = document.createElement('label');
                label.htmlFor = checkbox.id;
                label.innerText = f;

                div.appendChild(checkbox);
                div.appendChild(label);
                container.appendChild(div);
            });
        }

        function toggleFilas(status) {
            const checkboxes = document.querySelectorAll('#filaContainer input[type="checkbox"]');
            checkboxes.forEach(cb => cb.checked = status);
        }

        // ==========================================
        // PROCESSAMENTO DE DADOS
        // ==========================================
        function processarDados() {
            if (rawDataChamadas.length === 0 || rawDataEventos.length === 0) {
                alert("Por favor, carregue os arquivos CSV de Chamadas e Eventos.");
                return;
            }

            // --- 1. Capturar Parâmetros ---
            const taxaOcupacao = parseFloat(document.getElementById('taxaOcupacao').value) / 100;
            
            // Captura de datas corrigida
            const dtInicioVal = document.getElementById('dataInicioFilter').value;
            const dtFimVal = document.getElementById('dataFimFilter').value;

            // Criar objetos Date garantindo o timezone local (YYYY, MM-1, DD)
            let dataInicioFiltro = null;
            if (dtInicioVal) {
                const [y, m, d] = dtInicioVal.split('-');
                dataInicioFiltro = new Date(y, m-1, d, 0, 0, 0); // 00:00:00
            }

            let dataFimFiltro = null;
            if (dtFimVal) {
                const [y, m, d] = dtFimVal.split('-');
                dataFimFiltro = new Date(y, m-1, d, 23, 59, 59, 999); // Final do dia
            }

            // Capturar Filas Selecionadas (Multiselect)
            const checkboxes = document.querySelectorAll('#filaContainer input[type="checkbox"]:checked');
            const filasSelecionadas = Array.from(checkboxes).map(cb => cb.value);

            if (filasSelecionadas.length === 0) {
                alert("Selecione pelo menos uma fila.");
                return;
            }

            // --- 2. Filtrar Chamadas ---
            let chamadasFiltradas = rawDataChamadas.filter(c => {
                const d = parseDate(c.data);
                // Filtro Data
                if (dataInicioFiltro && d < dataInicioFiltro) return false;
                if (dataFimFiltro && d > dataFimFiltro) return false;
                
                // Filtro Fila (Multi)
                const filaNome = c.nomeFila ? c.nomeFila.replace(/"/g, '').trim() : '';
                if (!filasSelecionadas.includes(filaNome)) return false;

                return true;
            });

            // --- 3. Criar Intervalos (10 min) ---
            let intervalos = {};

            const getIntervalKey = (dateObj) => {
                const m = Math.floor(dateObj.getMinutes() / 10) * 10;
                const h = dateObj.getHours();
                return `${dateObj.getFullYear()}/${String(dateObj.getMonth()+1).padStart(2,'0')}/${String(dateObj.getDate()).padStart(2,'0')} ${String(h).padStart(2,'0')}:${String(m).padStart(2,'0')}`;
            };

            // Preencher demanda
            chamadasFiltradas.forEach(c => {
                const horaStr = c.horaEntradaPos.split(' ')[0];
                const dataBase = parseDate(c.data);
                const [hh, mm, ss] = horaStr.split(':');
                dataBase.setHours(hh, mm, ss || 0);

                const key = getIntervalKey(dataBase);
                if (!intervalos[key]) intervalos[key] = { recebidas: 0, agentes: new Set(), chamadas: [] };
                
                intervalos[key].recebidas++;
                if (c.tipo === 'Atendida') {
                    const atendente = c.nomeAtendente ? c.nomeAtendente.replace(/"/g, '').trim() : "N/A";
                    intervalos[key].chamadas.push({ atendente: atendente, duracao: timeToSeconds(c.tempoAtendimento) });
                }
            });

            // --- 4. Processar Eventos (Jornada) ---
            let eventosFiltrados = rawDataEventos.filter(e => {
                const d = parseDate(e.data);
                if (dataInicioFiltro && d < dataInicioFiltro) return false;
                if (dataFimFiltro && d > dataFimFiltro) return false;
                return true;
            });

            let agentesPorDia = {};

            eventosFiltrados.forEach(e => {
                const nome = e.nomeAtendente.replace(/"/g, '').trim();
                const dataContabil = parseDate(e.data);
                const key = `${nome}|${dataContabil.toISOString().split('T')[0]}`;

                if (!agentesPorDia[key]) agentesPorDia[key] = { 
                    nome: nome, 
                    data: dataContabil, 
                    logins: [], 
                    pausas: []
                };

                const eventoTipo = e.evento.toLowerCase();
                if (eventoTipo === 'login') {
                    agentesPorDia[key].logins.push({ inicio: parseDate(e.dataInicio), fim: parseDate(e.dataFim) });
                } else if (eventoTipo === 'pausa') {
                    agentesPorDia[key].pausas.push({
                        tipo: e.nomePausa ? e.nomePausa.toLowerCase().replace(/"/g, '') : '',
                        inicio: parseDate(e.dataInicio),
                        fim: parseDate(e.dataFim)
                    });
                }
            });

            // --- 5. Mapear Supply (Agentes nos Intervalos) ---
            Object.values(agentesPorDia).forEach(agenteDia => {
                agenteDia.logins.forEach(sessao => {
                    let cursor = new Date(sessao.inicio);
                    cursor.setMinutes(Math.floor(cursor.getMinutes()/10)*10, 0, 0);
                    while (cursor < sessao.fim) {
                        const key = getIntervalKey(cursor);
                        if (!intervalos[key]) intervalos[key] = { recebidas: 0, agentes: new Set(), chamadas: [] };
                        intervalos[key].agentes.add(agenteDia.nome);
                        cursor.setMinutes(cursor.getMinutes() + 10);
                    }
                });
            });

            // --- 6. Calcular Capacidade (IPAT) ---
            Object.keys(intervalos).forEach(key => {
                const dados = intervalos[key];
                const qtdAgentes = dados.agentes.size;
                const demanda = dados.recebidas;
                
                // (Chamadas / Agentes) * OcupacaoMeta
                if (qtdAgentes > 0) {
                    dados.estimativaPorAgente = (demanda / qtdAgentes) * taxaOcupacao;
                } else {
                    dados.estimativaPorAgente = 0;
                }
            });

            // --- 7. Consolidar por Agente ---
            let agentesConsolidados = {};

            Object.values(agentesPorDia).forEach(registro => {
                const nome = registro.nome;
                if (!agentesConsolidados[nome]) {
                    agentesConsolidados[nome] = {
                        dias: new Set(),
                        jornadaBruta: 0,
                        jornadaLiquida: 0,
                        tempoFalado: 0,
                        pausasProd: 0,
                        pausasImprod: 0,
                        tempoDeducao: 0,
                        chamadasAtendidas: 0,
                        chamadasEstimadasTotal: 0,
                        detalheIntervalos: {}
                    };
                }

                agentesConsolidados[nome].dias.add(registro.data.getTime());

                // Jornada
                registro.logins.sort((a,b) => a.inicio - b.inicio);
                if (registro.logins.length > 0) {
                    const primeiroLogin = registro.logins[0].inicio;
                    let ultimoLogoff = registro.logins[registro.logins.length-1].fim;
                    agentesConsolidados[nome].jornadaBruta += ((ultimoLogoff - primeiroLogin) / 1000);
                }

                // Pausas
                registro.pausas.forEach(p => {
                    let dur = (p.fim - p.inicio) / 1000;
                    if (dur < 0) dur = 0;
                    let isProd = PAUSAS_PRODUTIVAS.some(term => p.tipo.includes(term));
                    let isImprod = PAUSAS_IMPRODUTIVAS.some(term => p.tipo.includes(term));
                    let isDeducao = PAUSAS_DEDUCAO_LIQUIDA.some(term => p.tipo.includes(term));

                    if (isProd) agentesConsolidados[nome].pausasProd += dur;
                    else if (isImprod) agentesConsolidados[nome].pausasImprod += dur;
                    if (isDeducao) agentesConsolidados[nome].tempoDeducao += dur;
                });
            });

            // Cruzar chamadas e estimativas
            Object.keys(intervalos).sort().forEach(keyIntervalo => {
                const dadosInt = intervalos[keyIntervalo];
                
                // Atribuir Estimativa para quem estava online
                dadosInt.agentes.forEach(agenteNome => {
                    if (agentesConsolidados[agenteNome]) {
                        if (!agentesConsolidados[agenteNome].detalheIntervalos[keyIntervalo]) {
                            agentesConsolidados[agenteNome].detalheIntervalos[keyIntervalo] = {
                                chamadasAtendidas: 0, tempoFalado: 0, estimativa: dadosInt.estimativaPorAgente,
                                status: 'Disponível', pausas: []
                            };
                        }
                        agentesConsolidados[agenteNome].chamadasEstimadasTotal += dadosInt.estimativaPorAgente;
                    }
                });

                // Atribuir Realizado
                dadosInt.chamadas.forEach(chamada => {
                    const nome = chamada.atendente;
                    if (agentesConsolidados[nome]) {
                        agentesConsolidados[nome].tempoFalado += chamada.duracao;
                        agentesConsolidados[nome].chamadasAtendidas += 1;
                        if (agentesConsolidados[nome].detalheIntervalos[keyIntervalo]) {
                            agentesConsolidados[nome].detalheIntervalos[keyIntervalo].chamadasAtendidas++;
                            agentesConsolidados[nome].detalheIntervalos[keyIntervalo].tempoFalado += chamada.duracao;
                            agentesConsolidados[nome].detalheIntervalos[keyIntervalo].status = 'Atendendo';
                        }
                    }
                });
            });

            // Adicionar detalhes de pausas nos intervalos
            Object.values(agentesPorDia).forEach(registro => {
                const nome = registro.nome;
                if (!agentesConsolidados[nome]) return;
                registro.pausas.forEach(p => {
                    let cursor = new Date(p.inicio);
                    cursor.setMinutes(Math.floor(cursor.getMinutes()/10)*10, 0, 0);
                    while (cursor < p.fim) {
                        const key = getIntervalKey(cursor);
                        if (agentesConsolidados[nome].detalheIntervalos[key]) {
                            agentesConsolidados[nome].detalheIntervalos[key].status = 'Pausa (' + p.tipo + ')';
                            agentesConsolidados[nome].detalheIntervalos[key].pausas.push({tipo: p.tipo});
                        }
                        cursor.setMinutes(cursor.getMinutes() + 10);
                    }
                });
            });

            // --- 8. Renderizar ---
            const listaAgentes = [];
            let kpiRecebidas = chamadasFiltradas.length;
            let kpiAtendidas = 0;
            let somaOcupacao = 0;
            let somaOcioso = 0;
            let somaIpat = 0;
            let countAgentes = 0;

            Object.keys(agentesConsolidados).forEach(nome => {
                const d = agentesConsolidados[nome];
                const qtdDias = d.dias.size;
                const divisor = qtdDias > 1 ? qtdDias : 1;

                d.jornadaLiquida = d.jornadaBruta - d.tempoDeducao;
                d.tempoOcioso = d.jornadaLiquida - d.pausasImprod - d.pausasProd - d.tempoFalado;
                if (d.tempoOcioso < 0) d.tempoOcioso = 0;

                d.ocupacao = (d.tempoFalado + d.pausasProd + d.pausasImprod) / (d.jornadaLiquida || 1);
                
                if (d.chamadasEstimadasTotal > 0) {
                    d.ipat = (d.chamadasAtendidas / d.chamadasEstimadasTotal) - 1;
                } else {
                    d.ipat = 0;
                }

                listaAgentes.push({
                    nome: nome,
                    dias: qtdDias,
                    jornadaLiqView: d.jornadaLiquida / divisor,
                    faladoView: d.tempoFalado / divisor,
                    prodView: d.pausasProd / divisor,
                    improdView: d.pausasImprod / divisor,
                    ociosoView: d.tempoOcioso / divisor,
                    ocupacao: d.ocupacao,
                    ipat: d.ipat,
                    // NOVAS COLUNAS
                    atendidasTotal: d.chamadasAtendidas, 
                    estimativaTotal: d.chamadasEstimadasTotal,
                    raw: d
                });

                kpiAtendidas += d.chamadasAtendidas;
                somaOcupacao += d.ocupacao;
                somaOcioso += (d.tempoOcioso / divisor);
                somaIpat += d.ipat;
                countAgentes++;
            });

            // KPIs
            document.getElementById('kpiContainer').classList.remove('hidden');
            document.getElementById('kpiContainer').classList.add('grid'); // Reset tailwind grid
            document.getElementById('kpiRecebidas').innerText = kpiRecebidas;
            document.getElementById('kpiAtendidas').innerText = kpiAtendidas;
            document.getElementById('kpiOcupacao').innerText = countAgentes ? (somaOcupacao/countAgentes * 100).toFixed(2) + '%' : '0%';
            document.getElementById('kpiOcioso').innerText = countAgentes ? secondsToTime(somaOcioso/countAgentes) : '00:00:00';
            
            const ipatMedio = countAgentes ? (somaIpat/countAgentes * 100) : 0;
            const elIpat = document.getElementById('kpiIpat');
            elIpat.innerText = ipatMedio.toFixed(2) + '%';
            elIpat.className = "text-2xl font-bold " + (ipatMedio >= 0 ? "positive" : "negative");

            // Tabela
            renderTabela(listaAgentes);
            document.getElementById('mainTableCard').classList.remove('hidden');
        }

        function renderTabela(lista) {
            const tbody = document.querySelector('#tabelaAgentes tbody');
            tbody.innerHTML = '';
            
            lista.forEach(agente => {
                const tr = document.createElement('tr');
                tr.onclick = () => abrirModal(agente);
                
                const ipatPct = (agente.ipat * 100).toFixed(2) + '%';
                const ipatClass = agente.ipat >= 0 ? 'positive' : 'negative';

                // NOVAS COLUNAS ADICIONADAS AQUI: Estimativa e Atendidas
                tr.innerHTML = `
                    <td class="font-medium text-gray-900">${agente.nome}</td>
                    <td>${agente.dias}</td>
                    <td class="text-blue-600 font-semibold">${agente.estimativaTotal.toFixed(2)}</td>
                    <td class="text-green-600 font-semibold">${agente.atendidasTotal}</td>
                    <td>${secondsToTime(agente.jornadaLiqView)}</td>
                    <td>${secondsToTime(agente.faladoView)}</td>
                    <td>${secondsToTime(agente.prodView)}</td>
                    <td>${secondsToTime(agente.improdView)}</td>
                    <td>${secondsToTime(agente.ociosoView)}</td>
                    <td>${(agente.ocupacao * 100).toFixed(2)}%</td>
                    <td class="${ipatClass}">${ipatPct}</td>
                `;
                tbody.appendChild(tr);
            });
        }

        // ==========================================
        // MODAL
        // ==========================================
        function abrirModal(agenteData) {
            const modal = document.getElementById('modalDetalhe');
            const tbody = document.getElementById('modalBody');
            document.getElementById('modalTitle').innerText = `Detalhes: ${agenteData.nome}`;
            tbody.innerHTML = '';

            const intervalos = agenteData.raw.detalheIntervalos;
            const keys = Object.keys(intervalos).sort();
            let contagemPausas = {};

            keys.forEach(key => {
                const intData = intervalos[key];
                let descPausas = [];
                let qtdPausaDisplay = "";

                if (intData.pausas && intData.pausas.length > 0) {
                   intData.pausas.forEach(p => {
                       let tipo = p.tipo.toLowerCase();
                       if (!contagemPausas[tipo]) contagemPausas[tipo] = 0;
                       contagemPausas[tipo]++;
                       descPausas.push(p.tipo);
                       qtdPausaDisplay += `${p.tipo}: ${contagemPausas[tipo]} <br>`;
                   });
                }
                
                let tempoOciosoInt = 600 - intData.tempoFalado;
                if (intData.status.includes('Pausa')) tempoOciosoInt = 0; 
                if (tempoOciosoInt < 0) tempoOciosoInt = 0;

                const tr = document.createElement('tr');
                tr.innerHTML = `
                    <td>${key.split(' ')[1]} <span class="text-xs text-gray-400">(${key.split(' ')[0]})</span></td>
                    <td>${intData.status}</td>
                    <td>${intData.chamadasAtendidas}</td>
                    <td>${intData.estimativa.toFixed(2)}</td>
                    <td>${secondsToTime(intData.tempoFalado)}</td>
                    <td>${secondsToTime(tempoOciosoInt)}</td>
                    <td>${descPausas.join(', ')}</td>
                    <td>${intData.status.includes('Pausa') ? 'Sim' : '-'}</td>
                    <td class="text-xs">${qtdPausaDisplay}</td>
                `;
                tbody.appendChild(tr);
            });

            modal.style.display = 'flex';
        }

        function closeModal() {
            document.getElementById('modalDetalhe').style.display = 'none';
        }
        window.onclick = function(event) {
            if (event.target == document.getElementById('modalDetalhe')) closeModal();
        }

        // ==========================================
        // BACKUP
        // ==========================================
        function fazerBackup() {
            const data = { chamadas: rawDataChamadas, eventos: rawDataEventos };
            const dataStr = "data:text/json;charset=utf-8," + encodeURIComponent(JSON.stringify(data));
            const a = document.createElement('a');
            a.href = dataStr;
            a.download = "backup_ipat.json";
            a.click();
        }
        function restaurarBackup(input) {
            const file = input.files[0];
            if (!file) return;
            const reader = new FileReader();
            reader.onload = function(e) {
                const data = JSON.parse(e.target.result);
                rawDataChamadas = data.chamadas || [];
                rawDataEventos = data.eventos || [];
                preencherFiltroFilas();
                alert('Restaurado com sucesso.');
            };
            reader.readAsText(file);
        }
        function limparDados() {
            if(confirm('Limpar tudo?')) {
                rawDataChamadas = []; rawDataEventos = [];
                document.getElementById('filaContainer').innerHTML = '';
                document.getElementById('kpiContainer').classList.add('hidden');
                document.getElementById('mainTableCard').classList.add('hidden');
            }
        }
    </script>
</body>
</html>
