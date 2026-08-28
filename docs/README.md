# Documentação do StepFlow Pocket

Esta pasta contém a documentação ativa do projeto. O `README.md` da raiz é o painel resumido de acompanhamento.

## Precedência

Em caso de divergência:

1. `../AGENTS.md`;
2. `05-progresso/registro-de-decisoes.md`;
3. documento específico vigente;
4. `01-produto/visao-geral.md`;
5. tarefa dentro das decisões vigentes;
6. histórico Git.

Proposta não aprovada não pode ser implementada como decisão.

## Leitura eficiente

Sempre:

- `AGENTS.md`;
- enunciado da tarefa;
- este índice;
- documentos específicos indicados.

Conforme impacto:

- `05-progresso/registro-de-decisoes.md`;
- `04-planejamento/plano-oficial-fase-1.md`;
- `03-arquitetura/arquitetura-vigente.md`;
- `00-governanca/contexto-ambientes.md`;
- documento técnico relacionado.

## Índice vigente

### Governança — `00-governanca`

- `contexto-ambientes.md` — ambientes;
- `metodo-padrao-trabalho-assistido.md` — processo PO + Assistente + Codex;
- `politica-capacidade-codex.md` — seleção de capacidade.

### Produto — `01-produto`

- `visao-geral.md` — propósito/requisitos/limites;
- `categorizacao-atendimentos-equipamentos.md` — domínio `Procedimento × Atendimento × Equipamento`.

### Telas — `02-telas`

- `README.md` — mapa do Bloco 8;
- `01-login.md` — Login;
- `02-shell-sidebar.md` — Shell;
- `03-dashboard.md` — Dashboard;
- `04-lista-pesquisa-processos.md` — Processos;
- `05-leitor-processo.md` — Reader, stepper e execução com checklist + observação de serviço por Etapa;
- `06-editor-processo.md` — Editor;
- `07-historico-revisoes.md` — Histórico/Revisões;
- `08-lista-pesquisa-atendimentos.md` — Lista de Atendimentos;
- `09-atendimento-execucao-equipamento.md` — Atendimento/Equipamento/checklist/observações/Ficha;
- `10-usuarios-permissoes.md` — Usuários/Permissões;
- `11-meu-perfil.md` — Meu perfil;
- `12-configuracoes-categorias.md` — Configurações/Categorias;
- `13-backup-restauracao.md` — Backup/Restauração UX;
- `14-exportacao-impressao-ficha.md` — Procedimentos + Ficha compacta/PDF/preview;
- `15-estados-transversais.md` — Estados transversais.

### Arquitetura — `03-arquitetura`

- `arquitetura-vigente.md` — visão consolidada, incluindo Bloco 10 / Etapas 1–6;
- `modelo-dados-schema-fase-1.md` — schema conceitual, checklist, observações por Etapa e snapshots;
- `concorrencia-fila-conflitos-eventos.md` — concorrência geral;
- `implantacao-pocket.md` — implantação/ciclo central;
- `compatibilidade-windows-client.md` — Windows/WebView2;
- `host-pocket.md` — Controller/Host;
- `launcher-distribuicao-client.md` — distribuição;
- `comunicacao-client-host.md` — HTTP/JSON/WebSocket;
- `autenticacao-sessao-autorizacao.md` — autenticação/autorização.

### Planejamento — `04-planejamento`

- `roadmap.md` — fases;
- `plano-oficial-fase-1.md` — estado/gates/pendências;
- `bloco-9-atendimentos-execucao-checklist.md` — contrato operacional;
- `bloco-10-exportacao-impressao-ficha.md` — mapa técnico do Bloco 10; **Etapas 1–6 consolidadas; Etapa 7 próxima e ainda não aberta**;
- `tarefas-codex/README.md` — tarefas Codex ativas.

### Progresso — `05-progresso`

- `registro-de-decisoes.md` — decisões/gates;
- `changelog-projeto.md` — marcos;
- `diario-de-progresso.md` — histórico;
- `revisao-cruzada-fase-0.md` — evidência histórica.

## Estado atual

**Fase 1 em andamento; Blocos 8 e 9 concluídos. Bloco 10 está em andamento com Etapas 1–6 consolidadas. Etapa 7 — Template físico A4 da Ficha é a próxima, ainda não aberta.**

### Bloco 9

Consolidado:

- lifecycle `Em andamento / Concluído / Cancelado`;
- criação no primeiro save;
- conclusão/cancelamento/reabertura;
- responsabilidade por preset;
- checklist persistente e progresso derivado;
- observação de serviço opcional por Etapa no Reader operacional;
- revisão exata usada;
- snapshot histórico de Equipamento/estado final aplicável;
- matriz operacional;
- códigos `AT-000001` e `EQP-000001`;
- lifecycle/capacidade da Ficha.

### Direção visual

- clareza com baixa densidade textual permanente;
- cor, forma, símbolo, posição e ícones podem substituir texto repetitivo quando claros;
- detalhes secundários sob demanda;
- cor nunca é o único meio para estado importante;
- `Visão geral` + uma página lógica por Etapa;
- stepper compacto de círculos/linhas navegável e separado do progresso operacional.

### Bloco 10 — Etapas 1–6

Fonte técnica: `04-planejamento/bloco-10-exportacao-impressao-ficha.md`.

- geração documental Host-side por `DocumentModel`;
- PDF de Procedimentos via Typst embutido;
- DOCX OOXML Transitional via pipeline Rust direto;
- impressão Windows do PDF oficial via WebView2 `ShowPrintUI(System)`;
- Procedimento físico A4 retrato multipágina;
- Reader sem geometria A4;
- PDF com Noto Sans/Noto Sans Mono; DOCX com Arial/Consolas;
- Ficha compacta = prestação de contas resumida ao cliente;
- Ficha usa dados confirmados/snapshot do Atendimento e Equipamento;
- observações de serviço por Etapa podem compor a Ficha;
- Ficha não imprime checklist/progresso/passos/timeline/revisões detalhadas por padrão;
- Ficha possui PDF canônico próprio e preview SVG derivados do mesmo `PagedDocument`;
- exatamente uma página; `2+` = `SHEET_OVERFLOW`;
- Salvar/Imprimir reutilizam os mesmos bytes PDF da prévia;
- preview fica preso à `source_version` e não muda silenciosamente.

### Gate antes da Etapa 7

```text
squash merge da Etapa 6
→ remover branch da Etapa 6
→ remoto somente com main
→ zero PRs abertos
```

### Pendências vigentes

- parâmetros finais de autenticação;
- Gerência × configuração da empresa;
- Gerência × Backup;
- regra editorial de categoria arquivada em nova revisão;
- Etapas 7–12 do Bloco 10;
- mecanismo técnico do Bloco 11;
- validações reais do ambiente corporativo.

Todo avanço consolidado de fase/bloco/tela/etapa deve atualizar o painel raiz no mesmo checkpoint.