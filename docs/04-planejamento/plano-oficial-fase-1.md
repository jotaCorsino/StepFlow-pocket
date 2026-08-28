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
| 10 | Exportação/impressão + ficha compacta | **EM ANDAMENTO — ETAPAS 1–7 CONSOLIDADAS / ETAPA 8 PRÓXIMA** | `04-planejamento/bloco-10-exportacao-impressao-ficha.md` |
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
- template físico A4 da Ficha;
- estados transversais;
- matriz operacional de capacidades;
- códigos `AT-000001` / `EQP-000001`;
- princípio visual de clareza com baixa densidade textual permanente.

## Bloco 8 — UI/UX

Telas 01–15 estão consolidadas/aprovadas. Nenhuma UI de produção foi criada.

Direção transversal:

- Reader em formato livro/manual;
- `Visão geral` como primeira página lógica;
- uma Etapa por página lógica;
- stepper horizontal compacto de círculos/linhas, navegável diretamente;
- stepper representa navegação, não conclusão operacional;
- informação secundária sob demanda e baixa densidade textual quando possível.

A Tela 05 incorpora `Observação do serviço` por Etapa somente no contexto operacional. A Tela 14 consolida a Ficha como prestação de contas resumida, PDF/preview e template físico A4.

## Bloco 9 — Execução operacional / Atendimentos

Fonte: `bloco-9-atendimentos-execucao-checklist.md`.

Consolidado:

- lifecycle `Em andamento / Concluído / Cancelado`, com reabertura explícita;
- primeiro save cria Atendimento; responsável + resumo são necessários para concluir;
- checklist incompleto avisa, não bloqueia automaticamente;
- Funcionário opera por responsabilidade;
- revisão exata do Procedimento é preservada;
- checklist persistente somente em Atendimento;
- observação de serviço opcional/persistente por Etapa;
- progresso deriva somente do checklist;
- Equipamento opcional/reutilizável;
- conclusão preserva estado histórico relevante;
- códigos `AT-000001` e `EQP-000001` Host-only;
- Ficha disponível conforme lifecycle/capacidade.

## Bloco 10 — Exportação e impressão

**Status: EM ANDAMENTO — Etapas 1–7 consolidadas; Etapa 8 próxima, ainda não aberta.**

Fonte ativa: `bloco-10-exportacao-impressao-ficha.md`.

### Etapas

| Ordem | Etapa | Estado |
|---|---|---|
| 1 | Arquitetura de geração documental | **CONSOLIDADO / APROVADO PELO PO** |
| 2 | PDF de Procedimentos | **CONSOLIDADO / APROVADO PELO PO** |
| 3 | DOCX de Procedimentos | **CONSOLIDADO / APROVADO PELO PO** |
| 4 | Impressão Windows de Procedimentos | **CONSOLIDADO / APROVADO PELO PO** |
| 5 | Template físico de Procedimentos | **CONSOLIDADO / APROVADO PELO PO** |
| 6 | PDF + preview da Ficha compacta | **CONSOLIDADO / APROVADO PELO PO** |
| 7 | Template físico A4 da Ficha | **CONSOLIDADO / APROVADO PELO PO** |
| 8 | Limites textuais e densidade da Ficha | **PRÓXIMA — AINDA NÃO EM ANÁLISE** |
| 9 | Múltiplos MACs / Procedimentos / dados excepcionais | PENDENTE |
| 10 | Nomes de arquivo + artefatos temporários | PENDENTE |
| 11 | QR / barcode | PENDENTE |
| 12 | Validação técnica final do Bloco 10 | PENDENTE |

### Etapas 1–5 — Procedimentos

- geração documental pertence ao Host e parte de snapshot consistente + `DocumentModel`;
- PDF de Procedimentos usa Typst embutido no Host;
- DOCX é OOXML Transitional gerado diretamente em Rust;
- impressão física ocorre no Client Windows usando o mesmo PDF oficial e WebView2 `ShowPrintUI(System)`;
- Procedimento físico usa A4 retrato multipágina, margens-base 18 mm;
- PDF usa Noto Sans/Noto Sans Mono incorporadas; DOCX referencia Arial/Consolas;
- nenhuma truncagem/redução silenciosa é permitida.

### Etapa 6 — PDF + preview da Ficha

- Ficha é prestação de contas resumida ao cliente;
- conteúdo prioritário: identificação do serviço/dispositivo, características relevantes, `Resumo do trabalho` e observações;
- PDF próprio/canônico via template Typst da Ficha;
- preview SVG e PDF derivam do mesmo `PagedDocument`;
- resultado válido exige exatamente uma página;
- `2+ páginas` = `SHEET_OVERFLOW`;
- Salvar/Imprimir reutilizam os mesmos bytes PDF;
- impressão reutiliza o fluxo Windows consolidado;
- PDF/SVG são transitórios e presos à `source_version`.

### Etapa 7 — template físico A4 da Ficha

Contrato aprovado:

- A4 retrato, exatamente uma página, margens **15 mm**, sem bleed;
- composição predominantemente vertical/uma coluna;
- cabeçalho institucional compacto, logo opcional, sem título gigante e sem footer obrigatório;
- `CANCELADO` textual/inequívoco e acompanhamento discreto;
- cliente/OS/técnico em linha curta, omitindo campos vazios;
- Equipamento em ficha técnica resumida sem tabela gradeada;
- `SERVIÇO REALIZADO` como área narrativa principal;
- uma única seção `OBSERVAÇÕES` reunindo observações relevantes do Atendimento, Equipamento e Etapas;
- nome curto da Etapa apenas quando necessário para contexto;
- seções vazias colapsam completamente;
- Noto Sans com baseline 14 / 10,5 / 10 / 9 / 8,5 pt;
- divisórias discretas, contraste neutro e legível em monocromático;
- sem assinatura, financeiro, garantia, checklist, progresso, timeline, QR/barcode, lista detalhada de Procedimentos, página 2 ou footer promocional;
- nenhuma redução dinâmica de fonte para caber;
- overflow continua `SHEET_OVERFLOW`;
- Etapa 8 fecha limites/priorização/densidade; Etapa 9 trata dados excepcionais.

## Gate atual

A Etapa 8 só pode ser aberta após:

```text
squash merge da Etapa 7
→ apagar branch remota da Etapa 7
→ verificar remoto somente com main
→ verificar zero PRs abertos
```

## Pendências restantes da Fase 1

### Bloco 10

- Etapa 8: limites textuais/priorização/densidade/diagnóstico de overflow;
- Etapa 9: muitos MACs/Procedimentos/observações e outros dados excepcionais;
- Etapa 10: nomes + temporários;
- Etapa 11: QR/barcode somente se aprovado;
- Etapa 12: validação técnica final/matriz real/limites.

### Segurança/configuração

- Argon2id exato;
- senha mínima final;
- duração de sessão;
- entropia/tamanho final do token;
- Gerência × configuração da empresa;
- Gerência × Backup;
- regra editorial de categoria arquivada em nova revisão.

### Bloco 11

- mecanismo/pacote de Backup/Restore;
- atomicidade/checksums/retenção;
- restart/reconexão/sessões;
- disaster recovery local.

### Bloco 12

- estrutura oficial do repositório;
- migrations/scripts/testes;
- plano da Fase 2;
- sincronização do checkout local antes do primeiro Codex de implementação.

## Regras finais

- não criar scaffold, runtime definitivo ou código de negócio durante a Fase 1 sem gate explícito;
- toda tarefa Codex futura que altere arquivos informa base Git esperada e pré-flight;
- preservar alterações locais preexistentes do PO;
- nenhuma pendência pode virar decisão por inferência.
