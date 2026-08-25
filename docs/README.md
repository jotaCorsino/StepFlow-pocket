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

- `arquitetura-vigente.md` — visão técnica consolidada, incluindo Bloco 9;
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

**Fase 1 em andamento; Blocos 8 e 9 concluídos. Próximo: Bloco 10 — Exportação/impressão + ficha compacta técnica, ainda não iniciado.**

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

Permanecem para o Bloco 10:

- engines PDF/DOCX/impressão;
- template final da ficha A4;
- margens/tipografia/densidade;
- limites numéricos dos textos;
- preview;
- PDF específico da ficha;
- QR/barcode se aprovado.

Bloco 11 ainda fecha Backup/Restore técnico. Bloco 12 fecha parâmetros finais, regra editorial de categoria arquivada, estrutura oficial e plano da Fase 2.

Não há código funcional oficial.