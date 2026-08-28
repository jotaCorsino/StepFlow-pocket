# Bloco 10 — Exportação / Impressão + Ficha Compacta

**Status:** EM ANDAMENTO — ETAPAS 1–6 CONSOLIDADAS / ETAPA 7 PRÓXIMA  
**Fase:** Fase 1 — Fechamento arquitetural e especificação  
**Abertura:** 2026-08-25  
**Etapa 1 consolidada:** 2026-08-25  
**Etapa 2 consolidada:** 2026-08-26  
**Etapa 3 consolidada:** 2026-08-26  
**Etapa 4 consolidada:** 2026-08-26  
**Etapa 5 consolidada:** 2026-08-27  
**Etapa 6 consolidada:** 2026-08-28

## 1. Objetivo

Fechar, uma etapa por vez, o contrato de geração documental, exportação, impressão e Ficha compacta do StepFlow, preservando o caráter Pocket e a UX aprovada.

Este arquivo é o **mapa técnico do Bloco 10**, não uma duplicação integral das fontes canônicas. Detalhes permanecem em:

- `../../AGENTS.md` — governança operacional;
- `../03-arquitetura/arquitetura-vigente.md` — arquitetura vigente;
- `../03-arquitetura/modelo-dados-schema-fase-1.md` — modelo conceitual;
- `../05-progresso/registro-de-decisoes.md` — decisões/gates;
- `../02-telas/05-leitor-processo.md` — Reader;
- `../02-telas/09-atendimento-execucao-equipamento.md` — execução;
- `../02-telas/14-exportacao-impressao-ficha.md` — UX documental.

Não pertence a este bloco implementar código de produção, fechar Backup/Restore técnico ou abrir o Bloco 11.

## 2. Etapas

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

A Etapa 7 permanece fechada até o squash merge da Etapa 6, remoção da branch e verificação do remoto limpo.

---

# Etapa 1 — Arquitetura de geração documental

**Status:** CONSOLIDADO / APROVADO PELO PO

```text
Client
→ solicita identidade da fonte + revisão esperada
Host
→ autentica/autoriza
→ captura snapshot consistente
→ materializa DocumentModel semântico
→ encerra leitura/transação SQLite
→ renderiza fora da fila de mutações
→ devolve artefato pela API autenticada
Client
→ salva / preview / imprime conforme contrato específico
```

Contrato:

- geração pertence ao Host;
- Client não envia documento montado e DOM/HTML/CSS não são fonte documental;
- fonte mutável usa revisão esperada;
- renderers recebem `DocumentModel` imutável e não reconsultam SQLite;
- renderização ocorre fora da fila de mutações e usa limite próprio bounded;
- sem `export_jobs`/scheduler/fila persistente inicialmente;
- geração não cria mutação funcional;
- Host não grava em path arbitrário da workstation;
- artefatos não viram histórico/backup por padrão;
- runtime não depende de Office, LibreOffice, Adobe Reader, browser externo/headless, `wkhtmltopdf` ou cloud obrigatória.

Fronteira:

```text
DocumentModel
├── document_kind
├── source_identity
├── source_version
├── company_identity
├── metadata
├── sections[] / semantic_blocks[]
└── generation_metadata
```

---

# Etapa 2 — PDF de Procedimentos

**Status:** CONSOLIDADO / APROVADO PELO PO

- Typst embutido como biblioteca Rust no Host, via crates oficiais e adaptador interno;
- sem `typst.exe`, browser ou conversor externo;
- template interno/confiável/versionado;
- conteúdo do domínio entra somente como dados estruturados;
- filesystem/imports/fontes/assets controlados e sem recursos remotos;
- PDF 1.7 + Tagged PDF como baseline, sem promessa formal PDF/A ou PDF/UA;
- texto real selecionável/pesquisável;
- fontes incorporadas/subsetadas;
- parágrafo, passos/subpassos, checklist, nota, alerta, comando e código precisam ser representados ou falhar explicitamente;
- multipágina automático, sem truncamento;
- PNG/JPEG/SVG controlados;
- falha nunca devolve artefato parcial como sucesso.

Versões exatas das crates e limites numéricos ficam para o gate de implementação/Etapa 12.

---

# Etapa 3 — DOCX de Procedimentos

**Status:** CONSOLIDADO / APROVADO PELO PO

- `.docx` real em OOXML/WordprocessingML/OPC, baseline OOXML Transitional;
- geração direta Rust a partir do mesmo `DocumentModel`, sem converter PDF/Typst;
- `docx-rs` preferido sob adaptador interno;
- sem Word/COM, LibreOffice, browser/headless, CLI conversor ou cloud;
- sem XML/OOXML/relationships/paths/URLs arbitrários originados do usuário;
- template/estilos internos; sem `.docx/.dotx` externo como template v1;
- texto e numeração permanecem Word reais/editáveis;
- checklist é documental, não formulário;
- comando/código preserva whitespace;
- PNG/JPEG baseline; SVG direto não obrigatório;
- DOCX é refluível e não promete paginação idêntica ao PDF;
- Arial + Consolas referenciadas, sem embedding v1;
- pacote incompleto/corrompido nunca é sucesso.

---

# Etapa 4 — Impressão Windows de Procedimentos

**Status:** CONSOLIDADO / APROVADO PELO PO

```text
PDF oficial da revisão exata
→ Client Windows
→ recurso local transitório controlado
→ WebView2 dedicada/transitória
→ ShowPrintUI(System)
→ diálogo do Windows
```

- impressão física acontece no Client, não no Host;
- usa o mesmo PDF da Etapa 2;
- não imprime HTML da UI nem DOCX;
- webview principal permanece intacta;
- sem software externo, seletor próprio ou impressão silenciosa como baseline;
- StepFlow não gerencia drivers/spooler nem persiste impressoras no Host;
- sucesso é entrega do fluxo ao Windows, não confirmação física de papel;
- detalhes do recurso temporário ficam para Etapa 10.

---

# Etapa 5 — Template físico de Procedimentos

**Status:** CONSOLIDADO / APROVADO PELO PO

## Reader ≠ documento físico

- Reader usa páginas lógicas, sem geometria A4;
- `Visão geral` precede Etapa 1;
- cada Etapa permanece página lógica própria;
- stepper compacto de círculos/linhas é navegação, não progresso operacional.

## Procedimento exportado

- A4 retrato multipágina;
- margens-base 18 mm;
- sem capa exclusiva;
- sem sumário físico obrigatório por padrão;
- sem header repetitivo;
- rodapé compacto;
- títulos de Etapa não forçam nova folha automaticamente;
- paginação automática sem truncamento/redução silenciosa;
- PDF usa Noto Sans/Noto Sans Mono incorporadas;
- DOCX referencia Arial/Consolas;
- PDF é referência física; DOCX é refluível.

O limite de uma A4 não se aplica ao Procedimento completo.

---

# Etapa 6 — PDF + preview da Ficha compacta

**Status:** CONSOLIDADO / APROVADO PELO PO

## 3. Finalidade da Ficha

A Ficha é uma **prestação de contas resumida ao cliente**, não relatório técnico completo.

Prioriza:

- identificação do Atendimento/serviço;
- identificação do computador/dispositivo;
- características relevantes do dispositivo;
- `Resumo do trabalho`;
- observações gerais relevantes;
- observações de serviço registradas pelo técnico durante as Etapas.

Características do Equipamento podem incluir, quando aplicáveis e úteis:

- processador;
- RAM;
- armazenamento HD/SSD;
- sistema operacional;
- serial/patrimônio para identificação;
- saúde da bateria;
- observações do Equipamento.

Dados vêm do cadastro/snapshot aplicável; não há redigitação para gerar a Ficha.

Não imprimir por padrão:

- checklist completo;
- percentual/progresso;
- passos/subpassos;
- comandos/código;
- timeline/auditoria;
- IDs técnicos internos;
- lista detalhada de revisões.

## 4. Observação do serviço por Etapa

Correção transversal consolidada nesta etapa:

```text
Atendimento
+ revisão vinculada
+ Etapa
→ Observação do serviço opcional
```

- existe somente no contexto operacional;
- não altera o Procedimento oficial;
- é persistida pelo Host;
- Reader standalone não cria esse estado;
- editável somente enquanto o Atendimento estiver editável/autorizado;
- Concluído/Cancelado = somente leitura até reabertura;
- concorrência por Etapa/equivalente;
- observações preenchidas podem compor a Ficha;
- estado final aplicável precisa ser historicamente reproduzível após reabertura;
- não introduzir autosave por inferência.

## 5. PDF canônico e preview

```text
Atendimento confirmado + source_version esperada
→ DocumentModel document_kind = service_sheet
→ template Typst próprio da Ficha
→ PagedDocument
→ validar quantidade de páginas

1 página
→ PDF canônico
→ SVG de preview da mesma página

0 páginas
→ erro

2+ páginas
→ SHEET_OVERFLOW
```

- Ficha reutiliza a infraestrutura Typst, com template próprio;
- PDF 1.7 + Tagged PDF como baseline;
- PDF e SVG derivam do **mesmo PagedDocument**, sem segundo layout em HTML;
- não retornar apenas primeira página;
- não cortar conteúdo;
- não criar segunda folha;
- não reduzir fonte silenciosamente para caber;
- preview SVG é tratado como imagem/documento visual controlado, sem script/navegação externa.

## 6. Superfície do preview

```text
Tela 09
→ Ficha / Imprimir
→ modal/overlay grande
→ folha A4 centralizada

Ficha AT-000142               [ salvar ] [ imprimir ] [ × ]
```

- sem nova sidebar;
- sem toolbar textual extensa;
- página A4 escalada proporcionalmente;
- controles compactos/icon-only quando inequívocos, sempre acessíveis;
- preview permanece preso à `source_version` usada na geração;
- alteração remota não troca a folha silenciosamente;
- prévia desatualizada exige regeneração antes de nova saída atual.

## 7. Salvar e imprimir

`Salvar PDF` e `Imprimir` reutilizam os **mesmos bytes PDF** correspondentes à prévia aberta.

- não regenerar silenciosamente ao clicar em uma ação;
- destino de salvar é local e escolhido no Client;
- Host não recebe path arbitrário da workstation;
- impressão reutiliza WebView2 transitória + `ShowPrintUI(System)`;
- resultado PDF + SVG é transitório, sem job/histórico/backup automático;
- naming e materialização/limpeza concreta de temporários ficam para Etapa 10.

## 8. Lifecycle/histórico

- `Em andamento`: usa estado confirmado atual;
- `Concluído`: reimprime estado histórico aplicável;
- `Cancelado`: identifica inequivocamente o status;
- alterações locais não salvas/conflitos bloqueiam geração;
- reabertura não pode reescrever silenciosamente a Ficha histórica anterior.

---

# Etapas seguintes

## Etapa 7 — Template físico A4 da Ficha

**Status:** PRÓXIMA — AINDA NÃO EM ANÁLISE

Fechará geometria física final, hierarquia, seções, espaçamentos, tipografia e apresentação da única A4, sem reabrir PDF/preview já consolidados.

## Etapa 8 — Limites textuais e densidade

Definirá limites/priorização para resumo, observações do Atendimento, Equipamento e Etapas, incluindo diagnóstico de overflow.

## Etapa 9 — Múltiplos dados excepcionais

Definirá comportamento com muitos MACs, muitos Procedimentos/observações e outros casos que pressionem a única A4.

## Etapa 10 — Nomes + temporários

Definirá naming, materialização local, paths controlados, lifecycle e limpeza de artefatos transitórios.

## Etapa 11 — QR/barcode

Só entra se houver benefício operacional aprovado; não é requisito por padrão.

## Etapa 12 — Validação técnica final

Fechará matriz real de Windows/WebView2/Office compatível, impressão, limites de recursos, casos de erro e critérios técnicos antes da implementação.

## 9. Gate atual

A Etapa 6 está documentalmente consolidada nesta branch, mas **não está operacionalmente encerrada** até:

```text
PR da Etapa 6
→ validação
→ ready
→ squash merge em main
→ apagar branch
→ verificar somente main + zero PRs abertos
```

Somente depois disso a Etapa 7 pode ser aberta.
