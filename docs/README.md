# Documentação do StepFlow Pocket

Esta pasta contém a documentação ativa do projeto. O `README.md` da raiz é o painel resumido de acompanhamento.

## Precedência

Em caso de divergência:

1. `../AGENTS.md`;
2. `05-progresso/registro-de-decisoes.md` — decisão consolidada mais recente;
3. documento específico vigente;
4. `01-produto/visao-geral.md`;
5. enunciado da tarefa dentro das decisões vigentes;
6. material histórico no Git.

Proposta não aprovada não pode ser implementada como decisão.

## Leitura eficiente para Codex

Sempre:

- `AGENTS.md`;
- enunciado da tarefa;
- este índice;
- documentos específicos indicados.

Conforme impacto:

- registro de decisões;
- plano da Fase 1;
- arquitetura vigente;
- contexto de ambientes;
- documento técnico relacionado.

## Índice vigente

### Governança — `00-governanca`

- `contexto-ambientes.md` — sessão normal do PO, sandbox Codex e ambiente corporativo;
- `metodo-padrao-trabalho-assistido.md` — processo PO + Assistente + Codex;
- `politica-capacidade-codex.md` — seleção de modelo/raciocínio.

Regras obrigatórias de execução ficam em `../AGENTS.md`.

### Produto — `01-produto`

- `visao-geral.md` — propósito, usuários, requisitos e limites;
- `categorizacao-atendimentos-equipamentos.md` — domínio `Procedimento × Atendimento × Equipamento`, categorias e regras operacionais consolidadas do Bloco 9.

### Telas — `02-telas`

- `README.md` — mapa do Bloco 8;
- `01-login.md` — Login;
- `02-shell-sidebar.md` — Shell;
- `03-dashboard.md` — Dashboard;
- `04-lista-pesquisa-processos.md` — Lista/Pesquisa de Processos;
- `05-leitor-processo.md` — Reader, incluindo contexto operacional de Atendimento;
- `06-editor-processo.md` — Editor de Processo + categorias;
- `07-historico-revisoes.md` — Histórico/Revisões;
- `08-lista-pesquisa-atendimentos.md` — Lista/Pesquisa de Atendimentos com Status/Período do Bloco 9;
- `09-atendimento-execucao-equipamento.md` — Atendimento/Execução + Equipamento + lifecycle/checklist;
- `10-usuarios-permissoes.md` — Usuários/Permissões;
- `11-meu-perfil.md` — Meu perfil;
- `12-configuracoes-categorias.md` — Configurações + Categorias;
- `13-backup-restauracao.md` — Backup/Restauração — UX;
- `14-exportacao-impressao-ficha.md` — Exportação/Impressão + ficha — UX;
- `15-estados-transversais.md` — Estados transversais.

### Arquitetura — `03-arquitetura`

- `arquitetura-vigente.md` — visão técnica consolidada, incluindo Bloco 9 e Bloco 10 / Etapas 1–3;
- `implantacao-pocket.md` — implantação/ciclo de vida central;
- `compatibilidade-windows-client.md` — Tauri/Windows/WebView2;
- `host-pocket.md` — Controller/Host;
- `launcher-distribuicao-client.md` — distribuição local do Client;
- `comunicacao-client-host.md` — HTTP/JSON/WebSocket;
- `autenticacao-sessao-autorizacao.md` — autenticação + matriz documental/operacional;
- `modelo-dados-schema-fase-1.md` — schema conceitual + lifecycle/checklist/snapshot;
- `concorrencia-fila-conflitos-eventos.md` — writer/fila/conflitos/eventos gerais.

### Planejamento — `04-planejamento`

- `roadmap.md` — fases;
- `plano-oficial-fase-1.md` — estado/gates/pendências;
- `bloco-9-atendimentos-execucao-checklist.md` — contrato operacional consolidado do Bloco 9;
- `bloco-10-exportacao-impressao-ficha.md` — mapa do Bloco 10; **Etapas 1–3 consolidadas, Etapa 4 próxima e ainda não aberta**;
- `tarefas-codex/README.md` — somente tarefas Codex ativas.

### Progresso — `05-progresso`

- `registro-de-decisoes.md` — decisões vigentes/pendências;
- `changelog-projeto.md` — marcos;
- `diario-de-progresso.md` — histórico, não fonte superior;
- `revisao-cruzada-fase-0.md` — evidência histórica.

### Templates — `templates`

- `template-analise-de-tela.md`;
- `template-preflight-capacidade-codex.md`;
- `template-tarefa-codex.md`.

## Estado atual

**Fase 1 em andamento; Blocos 8 e 9 concluídos. Bloco 10 está em andamento com as Etapas 1 — Arquitetura de geração documental, 2 — PDF de Procedimentos e 3 — DOCX de Procedimentos consolidadas. Etapa 4 — Impressão Windows de Procedimentos é a próxima, ainda não aberta.**

Consolidado no Bloco 9:

- lifecycle `Em andamento / Concluído / Cancelado`;
- criação no primeiro save;
- conclusão/cancelamento/reabertura;
- responsabilidade do Funcionário;
- checklist persistente em contexto de Atendimento;
- progresso derivado de checklist;
- revisão exata usada;
- snapshot histórico de Equipamento;
- matriz operacional;
- códigos `AT-000001` e `EQP-000001`;
- Gerência gere categorias por preset;
- lifecycle/capacidade da ficha.

### Bloco 10 — Etapa 1 consolidada

Contrato vigente:

- geração sob responsabilidade do Host;
- solicitação por identidade/revisão esperada;
- snapshot consistente antes da renderização;
- `DocumentModel` semântico entre domínio e renderers;
- leitura/transação SQLite encerrada antes da renderização;
- geração como leitura derivada, fora da fila de mutações;
- limite próprio de concorrência/backpressure;
- fluxo request → artefato sem job persistente inicial;
- transporte autenticado Host → Client;
- Host não grava em path arbitrário do Client;
- runtime autocontido, sem Office/LibreOffice/Adobe/Chrome externo/cloud obrigatório;
- ausência de persistência/histórico/backup de exportações por padrão.

### Bloco 10 — Etapa 2 consolidada

Contrato vigente:

- PDF de Procedimentos usa Typst embutido no Host Rust, com crates oficiais e adaptador interno StepFlow;
- sem `typst.exe`/CLI, browser ou conversor externo;
- template interno confiável/versionado;
- dados do domínio nunca são concatenados ao source Typst;
- renderer opera em mundo virtual controlado, sem pacotes/recursos remotos em runtime;
- PDF 1.7 e Tagged PDF são configurados explicitamente;
- Tagged PDF não implica conformidade formal PDF/UA/PDF-A;
- texto permanece selecionável/pesquisável/copiável;
- fontes são empacotadas/incorporadas sem depender das fontes do Windows;
- blocos semânticos conhecidos não são descartados silenciosamente;
- comandos/código permanecem texto e preservam whitespace;
- multipágina e quebra automática são obrigatórias;
- layout físico final continua reservado para a Etapa 5;
- conteúdo visual não depende implicitamente do relógio/ambiente da máquina central;
- falha do renderer não gera artefato parcial tratado como sucesso;
- recursos avançados e conformidades formais ficam fora da primeira versão.

### Bloco 10 — Etapa 3 consolidada

Contrato vigente:

- DOCX é gerado diretamente pelo Host Rust a partir do mesmo `DocumentModel`, sem conversão de PDF/Typst;
- saída é `.docx` OOXML/WordprocessingML real, com OOXML Transitional como baseline de compatibilidade;
- `docx-rs` é a biblioteca preferida sob adaptador interno StepFlow;
- sem Word/COM, LibreOffice, browser/headless, CLI conversor ou cloud;
- dados do domínio não viram XML/OOXML arbitrário;
- estilos/template são internos e versionados, sem `.docx`/`.dotx` fornecido pelo usuário em runtime;
- texto permanece selecionável, pesquisável, copiável e editável;
- blocos semânticos conhecidos não são descartados silenciosamente;
- passos/subpassos usam numeração/listas Word reais; checklist permanece documental;
- comandos/código preservam whitespace e permanecem texto;
- PNG/JPEG são baseline; SVG não é requisito direto do DOCX v1 e não pode ser omitido silenciosamente;
- DOCX é refluível e não promete paginação idêntica ao PDF;
- política tipográfica/embedding de fontes do DOCX permanece para Etapa 5/gate técnico e não é herdada automaticamente do PDF;
- macros/VBA, ActiveX, OLE, remote templates, conteúdo externo, anexos, assinatura, senha/DRM e importação de DOCX editado ficam fora da primeira versão;
- artefato incompleto/corrompido nunca é tratado como sucesso.

Etapas 4–12 permanecem pendentes. A Etapa 4 está somente marcada como próxima e ainda não foi aberta para análise.

Bloco 11 continua não iniciado. Bloco 12 fecha parâmetros finais, regra editorial de categoria arquivada, estrutura oficial e plano da Fase 2.

Não há código funcional oficial.