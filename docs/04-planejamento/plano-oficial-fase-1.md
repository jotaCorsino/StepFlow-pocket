# Plano Oficial — Fase 1: Fechamento Arquitetural e Especificação

**Status:** EM ANDAMENTO  
**Início:** 2026-08-19  
**Atualização:** 2026-08-28

## Objetivo

Transformar requisitos e arquitetura conceitual em decisões implementáveis antes da fundação executável do StepFlow.

A Fase 1 autoriza documentação, decisões técnicas e provas descartáveis quando necessárias. Não autoriza scaffold/runtime oficial nem código de negócio definitivo antes do gate correspondente do Bloco 12/Fase 2.

## Estado dos blocos

| Bloco | Tema | Status | Fonte vigente |
|---|---|---|---|
| 0 | Bootstrap do ambiente | CONCLUÍDO | contexto/repositório validado |
| 1 | Client Windows/Tauri | CONCLUÍDO | `03-arquitetura/compatibilidade-windows-client.md` |
| 2 | Host Pocket | CONCLUÍDO | `03-arquitetura/host-pocket.md` |
| 3 | Launcher/distribuição | CONCLUÍDO | `03-arquitetura/launcher-distribuicao-client.md` |
| 4 | Comunicação Client↔Host | CONCLUÍDO | `03-arquitetura/comunicacao-client-host.md` |
| 5 | Autenticação/autorização | NÚCLEO CONCLUÍDO / PARÂMETROS FINAIS PENDENTES | `03-arquitetura/autenticacao-sessao-autorizacao.md` |
| 6 | Dados/schema/migrations | NÚCLEO + EXTENSÃO OPERACIONAL CONSOLIDADOS CONCEITUALMENTE | `03-arquitetura/modelo-dados-schema-fase-1.md` |
| 7 | Concorrência/eventos | NÚCLEO CONCLUÍDO | `03-arquitetura/concorrencia-fila-conflitos-eventos.md` + Bloco 9 |
| 8 | UI/UX | CONCLUÍDO | `02-telas/README.md` |
| 9 | Execução operacional/Atendimentos | **CONCLUÍDO** | `04-planejamento/bloco-9-atendimentos-execucao-checklist.md` |
| 10 | Exportação/impressão + ficha compacta | **EM ANDAMENTO — ETAPAS 1–6 CONSOLIDADAS / ETAPA 7 PRÓXIMA** | `04-planejamento/bloco-10-exportacao-impressao-ficha.md` |
| 11 | Backup/restauração | PENDENTE | política técnica/operacional |
| 12 | Estrutura oficial + Fase 2 | PENDENTE | fundação do repositório |

## Extensão de produto consolidada

Fazem parte da Fase 1:

- categorias configuráveis/múltiplas;
- domínio `Procedimento × Atendimento/Execução × Equipamento`;
- Atendimentos como área operacional própria;
- Equipamento opcional/reutilizável;
- múltiplos Procedimentos por Atendimento;
- revisão exata utilizada preservada;
- checklist persistente em contexto de execução;
- `Observação do serviço` opcional por Etapa no Reader operacional;
- estado final historicamente reproduzível após conclusão/reabertura;
- Ficha compacta como prestação de contas resumida ao cliente, com ou sem Equipamento;
- identidade central da empresa;
- PDF/DOCX/impressão contextual de Procedimentos;
- PDF próprio + preview da Ficha;
- estados transversais;
- matriz operacional de capacidades;
- códigos `AT-000001` / `EQP-000001`;
- princípio visual de clareza com baixa densidade textual permanente, usando cor/forma/símbolo/posição quando apropriado sem sacrificar acessibilidade.

## Bloco 8 — UI/UX — concluído

Telas 01–15 estão consolidadas/aprovadas. Nenhuma UI de produção foi criada.

Direção transversal:

- Reader em formato livro/manual;
- `Visão geral` como primeira página lógica;
- uma Etapa por página lógica;
- stepper horizontal compacto de círculos/linhas, navegável diretamente;
- stepper representa navegação, não conclusão operacional;
- informação secundária sob demanda e baixa densidade textual quando possível.

A Tela 05 foi posteriormente atualizada para incorporar `Observação do serviço` por Etapa somente no contexto operacional de Atendimento. A Tela 14 foi atualizada para refletir a Ficha como prestação de contas resumida e o preview consolidado no Bloco 10.

## Bloco 9 — Execução operacional / Atendimentos — concluído

Fonte: `bloco-9-atendimentos-execucao-checklist.md`.

### Lifecycle

```text
rascunho local
→ primeiro save aceito
→ Em andamento
   ├─→ Concluído
   └─→ Cancelado

Concluído/Cancelado
→ Reabrir
→ Em andamento
```

- `Resumo do trabalho` + responsável são obrigatórios para concluir;
- checklist incompleto avisa, não bloqueia automaticamente;
- cancelamento exige motivo;
- concluído/cancelado são read-only até reabertura;
- ADM/Gerência reabrem por preset.

### Procedimentos, checklist e observações

- Funcionário usa revisão publicada por padrão;
- ADM/Gerência podem selecionar explicitamente outras revisões autorizadas;
- Reader standalone não persiste execução;
- Reader operacional persiste checklist;
- progresso deriva somente de itens marcados/total;
- cada Etapa pode receber `Observação do serviço` opcional;
- observação pertence ao Atendimento + revisão vinculada + Etapa e não altera o Procedimento;
- checklist usa concorrência granular por item;
- observação usa concorrência granular por Etapa/equivalente;
- eventos remotos não sobrescrevem edição local silenciosamente;
- não há autosave por inferência.

### Equipamento e histórico

- Funcionário cria/edita Equipamento;
- ADM/Gerência arquivam/reativam;
- não arquivar Equipamento ligado a Atendimento em andamento;
- conclusão congela projeção histórica relevante do Equipamento;
- conclusão também preserva o estado final aplicável das observações de serviço;
- reabertura + nova conclusão não reescreve silenciosamente a prestação de contas anterior.

### Códigos e Ficha

```text
AT-000001
EQP-000001
```

Host-only, seis dígitos, gaps permitidos.

Ficha:

- capacidade por preset para ADM/Gerência/Funcionário em Atendimento acessível;
- `Em andamento`: geração a partir do estado confirmado;
- `Concluído`: reimpressão histórica aplicável;
- `Cancelado`: identificação inequívoca;
- finalidade: prestação de contas resumida ao cliente.

## Bloco 10 — Exportação e impressão

**Status: EM ANDAMENTO — Etapas 1–6 consolidadas; Etapa 7 próxima e ainda não aberta.**

Fonte: `bloco-10-exportacao-impressao-ficha.md`.

| Ordem | Etapa | Estado |
|---|---|---|
| 1 | Arquitetura de geração documental | **CONSOLIDADO / APROVADO PELO PO** |
| 2 | PDF de Procedimentos | **CONSOLIDADO / APROVADO PELO PO** |
| 3 | DOCX de Procedimentos | **CONSOLIDADO / APROVADO PELO PO** |
| 4 | Impressão Windows de Procedimentos | **CONSOLIDADO / APROVADO PELO PO** |
| 5 | Template físico de Procedimentos | **CONSOLIDADO / APROVADO PELO PO** |
| 6 | PDF + preview da Ficha compacta | **CONSOLIDADO / APROVADO PELO PO** |
| 7 | Template físico A4 da Ficha | **PRÓXIMA — AINDA NÃO EM ANÁLISE** |
| 8 | Limites textuais e densidade da Ficha | PENDENTE |
| 9 | Múltiplos MACs / Procedimentos / dados excepcionais | PENDENTE |
| 10 | Nomes de arquivo + artefatos temporários | PENDENTE |
| 11 | QR / barcode | PENDENTE |
| 12 | Validação técnica final do Bloco 10 | PENDENTE |

### Etapa 1 — arquitetura documental

- geração Host-side;
- Client solicita identidade da fonte + revisão esperada;
- snapshot consistente → `DocumentModel` imutável → encerra leitura/transação → renderiza;
- renderers não usam DOM/HTML nem reconsultam banco;
- geração é leitura derivada, fora da fila de mutações;
- renderização tem limite próprio bounded;
- sem job/fila persistente inicialmente;
- Host não grava em path arbitrário do Client;
- runtime autocontido, sem Office/LibreOffice/Adobe/browser externo/cloud obrigatória.

### Etapa 2 — PDF de Procedimentos

- Typst embutido como biblioteca Rust;
- template interno confiável e dados estruturados;
- sem recursos remotos;
- PDF 1.7 + Tagged PDF baseline;
- fontes incorporadas/subsetadas;
- blocos semânticos conhecidos são representados ou falham explicitamente;
- multipágina automático, sem truncamento;
- PNG/JPEG/SVG controlados.

### Etapa 3 — DOCX de Procedimentos

- DOCX real OOXML/WordprocessingML/OPC, baseline Transitional;
- geração Rust direta pelo mesmo `DocumentModel`, sem PDF → DOCX;
- `docx-rs` preferido sob adaptador interno;
- texto/listas editáveis reais;
- sem Word/COM, LibreOffice, browser/headless ou cloud;
- Arial/Consolas referenciadas sem embedding v1;
- pacote incompleto/corrompido nunca é sucesso.

### Etapa 4 — impressão Windows

- impressão física no Client Windows;
- usa o mesmo PDF oficial da revisão;
- WebView2 transitória/dedicada + `ShowPrintUI(System)`;
- diálogo padrão do Windows;
- sem impressão silenciosa/seletor próprio/software externo como baseline;
- sucesso significa fluxo entregue ao Windows, não confirmação física de papel;
- temporários concretos ficam para Etapa 10.

### Etapa 5 — template físico de Procedimentos

- Reader não possui geometria A4;
- Procedimento exportado usa A4 retrato multipágina, margens-base 18 mm;
- sem capa exclusiva/sumário físico obrigatório/header repetitivo por padrão;
- rodapé compacto;
- paginação automática sem truncamento/redução silenciosa;
- PDF usa Noto Sans/Noto Sans Mono incorporadas;
- DOCX usa Arial/Consolas referenciadas;
- limite de uma A4 é exclusivo da Ficha.

### Etapa 6 — PDF + preview da Ficha compacta

Finalidade:

- Ficha é prestação de contas resumida ao cliente;
- prioriza identificação do serviço/dispositivo, características relevantes, `Resumo do trabalho` e observações;
- processador, RAM, HD/SSD, SO quando útil, bateria quando aplicável e observações do Equipamento podem compor a folha;
- observações de serviço por Etapa entram quando preenchidas;
- checklist, percentual/progresso, passos, comandos, timeline, IDs internos e lista detalhada de revisões não aparecem por padrão.

Geração:

```text
Atendimento confirmado + source_version
→ DocumentModel service_sheet
→ template Typst próprio
→ PagedDocument
→ exigir exatamente 1 página
   ├─→ PDF canônico
   └─→ SVG de preview
```

- PDF e SVG derivam do mesmo layout;
- 2+ páginas = `SHEET_OVERFLOW`, sem segunda folha/corte/redução silenciosa;
- preview aparece em modal/overlay simples com A4 centralizada;
- `Salvar PDF` e `Imprimir` usam os mesmos bytes PDF da prévia;
- impressão reutiliza WebView2 + diálogo Windows;
- prévia fica presa à `source_version` e não muda silenciosamente;
- resultado PDF/SVG é transitório, sem job/histórico/backup automático;
- reimpressão de concluído usa estado histórico aplicável.

## Próximas etapas do Bloco 10

### Etapa 7 — Template físico A4 da Ficha

Fechar geometria, hierarquia, seções, espaçamentos, tipografia e apresentação da única folha, sem reabrir PDF/preview.

### Etapa 8 — Limites textuais e densidade

Fechar limites/priorização de resumo e observações, incluindo diagnóstico de overflow.

### Etapa 9 — Casos de muitos dados

Fechar múltiplos MACs, muitos Procedimentos/observações e demais situações que pressionem a única A4.

### Etapa 10 — Nomes + temporários

Fechar naming, materialização local, paths controlados e limpeza.

### Etapa 11 — QR/barcode

Somente se houver benefício operacional aprovado.

### Etapa 12 — validação técnica final

Fechar matriz corporativa, limites de recursos, erros e critérios técnicos antes da implementação.

## Gate para abrir a Etapa 7

A Etapa 7 só pode iniciar depois de:

```text
Etapa 6 consolidada
→ PR validado
→ squash merge em main
→ branch da Etapa 6 removida
→ remoto somente com main
→ zero PRs abertos
```

## Bloco 11 — Backup/Restore

Permanece pendente para fechar pacote, atomicidade, checksums, retenção, restart/reconexão, sessões e disaster recovery local.

## Bloco 12 / antes da implementação

Permanecem pendentes:

- parâmetros finais de autenticação;
- Gerência × configuração da empresa;
- Gerência × Backup;
- regra editorial de categoria arquivada em nova revisão;
- forma física final dos snapshots históricos necessários;
- árvore oficial/migrations/scripts/testes;
- plano da Fase 2;
- sincronização explícita de `C:\dev\StepFlow` com o remoto, preservando alterações preexistentes do PO.

Não criar scaffold, runtime definitivo ou código de negócio durante a Fase 1.

Toda tarefa Codex futura que altere arquivos deve trazer base Git esperada, pré-flight de capacidade e obedecer `AGENTS.md`.