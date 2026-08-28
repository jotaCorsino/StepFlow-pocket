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
| 10 | Exportação/impressão + ficha compacta | **EM ANDAMENTO — ETAPAS 1–10 CONSOLIDADAS / ETAPA 11 PRÓXIMA** | `04-planejamento/bloco-10-exportacao-impressao-ficha.md` |
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
- limites textuais orientativos e política de overflow da Ficha;
- projeção compacta de MACs e regras para dados multiplicativos/excepcionais;
- naming persistente e política de artefatos transitórios do Client;
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

A Tela 05 incorpora `Observação do serviço` por Etapa somente no contexto operacional. A Tela 14 consolida a Ficha como prestação de contas resumida, PDF/preview, template físico A4, política de limites/overflow, regras de dados excepcionais, naming e temporários do Client.

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

**Status: EM ANDAMENTO — Etapas 1–10 consolidadas; Etapa 11 próxima, ainda não aberta.**

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
| 8 | Limites textuais e densidade da Ficha | **CONSOLIDADO / APROVADO PELO PO** |
| 9 | Múltiplos MACs / Procedimentos / dados excepcionais | **CONSOLIDADO / APROVADO PELO PO** |
| 10 | Nomes de arquivo + artefatos temporários | **CONSOLIDADO / APROVADO PELO PO** |
| 11 | QR / barcode | **PRÓXIMA — AINDA NÃO EM ANÁLISE** |
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
- overflow continua `SHEET_OVERFLOW`.

### Etapa 8 — limites textuais e densidade da Ficha

Contrato aprovado:

- limite de uma A4 não altera nem destrói o dado operacional;
- soft limits: `Resumo do trabalho` 600, observação geral do Atendimento 400, observação do Equipamento 300 e observação de serviço por Etapa 280 caracteres;
- contadores/avisos aparecem somente perto de aproximadamente 80% da faixa recomendada;
- soft limits não bloqueiam save nem conclusão;
- layout real Typst é a autoridade final para saber se a Ficha cabe;
- `2+ páginas` = `SHEET_OVERFLOW`, bloqueando somente PDF/preview/impressão da Ficha;
- Atendimento continua válido e íntegro após overflow;
- Host devolve diagnóstico semântico dos principais campos que pressionam a folha;
- correção ocorre nos dados reais, sem editor paralelo exclusivo da Ficha;
- observações seguem Atendimento → Equipamento → Etapas na ordem executada;
- sem IA, resumo automático, truncamento, reticências, deduplicação semântica, modo compacto ou redução automática de fonte/margem/espaçamento;
- hard limits técnicos de storage/API não são derivados da A4.

### Etapa 9 — dados excepcionais e multiplicidade

Contrato aprovado:

- Ficha permanece projeção client-facing resumida e não dump integral do domínio;
- Procedimentos vinculados ficam fora da Ficha por padrão, independentemente da quantidade;
- MACs: 0 omite; 1–2 exibem valores compactos; 3+ exibem apenas a quantidade cadastrada;
- labels existentes podem contextualizar MACs; não inventar `MAC principal`;
- observações legítimas não recebem cap de quantidade nem descarte automático;
- multiplicidade real pode produzir `SHEET_OVERFLOW` e exigir revisão humana consciente dos textos reais;
- campos estruturados longos quebram linha quando possível, sem truncamento, reticências ou abreviação inventada;
- diagnóstico pode indicar quantidade de observações, Etapa específica ou campo estruturado longo;
- não existem `include_in_sheet`, `sheet_priority`, seleção transitória, editor paralelo, segunda página ou modo compacto para resolver multiplicidade;
- limites técnicos finais permanecem para a Etapa 12.

### Etapa 10 — nomes de arquivo e artefatos temporários

Contrato aprovado:

- arquivo persistente escolhido pelo usuário e artefato transitório interno são lifecycles distintos;
- Procedimento sugere `{codigo} - {titulo} - v{display_version} - r{revision_no}.{ext}`; sem `display_version`, omite esse segmento;
- Ficha sugere `{service_code} - Ficha.pdf`, sem dados pessoais/operacionais adicionais no filename por padrão;
- sanitização segue regras de filename Windows, impede injeção de path e não altera conteúdo documental;
- conflito de nome não causa overwrite silencioso;
- save só é sucesso após gravação integral; auxiliar opaco no mesmo destino + promoção/replace seguro são preferidos quando suportados;
- temporário só existe no Client quando integração local precisa de filesystem;
- temporários usam raiz por usuário resolvida por API do sistema/Tauri, namespace StepFlow, subdiretório por instância e nomes opacos sem dados de negócio;
- cleanup/retry/scavenging são best-effort e restritos ao namespace StepFlow;
- lock não autoriza kill, unlock forçado ou alteração de ACL;
- não criar Windows Service, Task Scheduler, daemon ou watchdog para limpeza;
- temporários/exportações não entram em SQLite, histórico ou backup por padrão;
- API concreta, NTFS/SMB, WebView2, memória, paths Unicode/longos, concorrência e EDR ficam para a Etapa 12.

## Gate atual

A Etapa 11 só pode ser aberta após:

```text
squash merge da Etapa 10
→ apagar branch remota da Etapa 10
→ verificar remoto somente com main
→ verificar zero PRs abertos
```

## Pendências restantes da Fase 1

### Bloco 10

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
