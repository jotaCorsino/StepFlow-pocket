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
- `05-leitor-processo.md` — Reader, incluindo stepper compacto navegável, baixa densidade visual e contexto operacional de Atendimento;
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

- `arquitetura-vigente.md` — visão técnica consolidada, incluindo Bloco 9 e Bloco 10 / Etapas 1–5;
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
- `bloco-10-exportacao-impressao-ficha.md` — mapa técnico do Bloco 10; **Etapas 1–5 consolidadas, Etapa 6 próxima e ainda não aberta**;
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

**Fase 1 em andamento; Blocos 8 e 9 concluídos. Bloco 10 está em andamento com as Etapas 1 — Arquitetura de geração documental, 2 — PDF de Procedimentos, 3 — DOCX de Procedimentos, 4 — Impressão Windows de Procedimentos e 5 — Template físico de Procedimentos consolidadas. Etapa 6 — PDF + preview da Ficha compacta é a próxima, ainda não aberta.**

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

Direção visual transversal consolidada:

- clareza com baixa densidade textual permanente;
- cor, forma, símbolo, posição e ícones reconhecíveis podem substituir texto repetitivo quando claros;
- detalhes secundários podem aparecer sob demanda;
- cor nunca é o único meio para estado importante;
- Reader mantém `Visão geral` + uma página lógica por Etapa;
- stepper compacto de círculos/linhas é navegável e permanece separado do progresso operacional de checklist.

### Bloco 10 — Etapas 1–5

Fonte técnica detalhada: `04-planejamento/bloco-10-exportacao-impressao-ficha.md`.

Consolidado:

- geração documental no Host a partir de `DocumentModel` semântico e snapshot consistente;
- PDF com Typst embutido e runtime autocontido;
- DOCX OOXML Transitional via adaptador `docx-rs`, editável/refluível;
- impressão Windows local do PDF oficial por WebView2 `ShowPrintUI(System)`;
- Reader separado da geometria física e com stepper compacto navegável;
- Procedimento físico A4 retrato multipágina, margens-base 18 mm;
- sem capa exclusiva e sem sumário físico obrigatório v1;
- sem cabeçalho repetitivo, rodapé compacto;
- paginação automática sem truncamento/redução silenciosa;
- PDF com Noto Sans/Noto Sans Mono incorporadas;
- DOCX com Arial/Consolas referenciadas, sem embedding v1;
- escala-base 18 / 14 / 10,5 / 9 / 8 pt conforme papel semântico;
- limite rígido de uma A4 pertence à Ficha compacta, não ao Procedimento completo.

### Gate antes da Etapa 6

A Etapa 6 não pode ser aberta antes de:

```text
squash merge da Etapa 5
→ remoção da branch remota da Etapa 5
→ remoto somente com main
→ zero PRs abertos
```

### Extensão de produto consolidada

O StepFlow deve acomodar manutenção, TI, Service Desk, Help Desk, infraestrutura/servidores, redes, guias e outros procedimentos internos.

Permanecem consolidados:

- categorias configuráveis e múltiplas;
- separação `Procedimento × Atendimento/Execução × Equipamento`;
- Atendimentos como área operacional própria;
- Equipamento opcional/reutilizável;
- múltiplos Procedimentos por Atendimento;
- revisão exata utilizada preservada;
- ficha compacta com ou sem Equipamento;
- tipos mínimos `Servidor`, `Desktop`, `Notebook`;
- bateria contextual para Notebook;
- identidade central da empresa;
- Backup/Restauração em Configurações;
- safety backup antes de Restore normal;
- PDF/DOCX/impressão de Procedimentos pela revisão selecionada;
- baixa densidade visual como princípio transversal, com uso responsável de cor/forma/símbolo e detalhes secundários sob demanda;
- estados transversais comuns.

### Pendências ainda vigentes

- custo final Argon2id;
- senha mínima final;
- duração de sessão;
- entropia/tamanho final de token;
- Gerência × configuração da empresa;
- Gerência × Backup;
- regra editorial de nova revisão ainda referenciando categoria arquivada;
- Etapas 6–12 do Bloco 10;
- mecanismo técnico do Bloco 11;
- validações reais do ambiente corporativo.

### Regra de atualização deste painel

Todo avanço consolidado de **fase, bloco, tela ou etapa do bloco atual** deve atualizar este README **no mesmo checkpoint documental**. Um avanço não está documentalmente encerrado se este painel ficar atrasado.