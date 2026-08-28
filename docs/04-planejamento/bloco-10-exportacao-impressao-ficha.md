# Bloco 10 — Exportação / Impressão + Ficha Compacta

**Status:** EM ANDAMENTO — ETAPAS 1–7 CONSOLIDADAS / ETAPA 8 PRÓXIMA  
**Fase:** Fase 1 — Fechamento arquitetural e especificação  
**Abertura:** 2026-08-25  
**Etapa 1 consolidada:** 2026-08-25  
**Etapa 2 consolidada:** 2026-08-26  
**Etapa 3 consolidada:** 2026-08-26  
**Etapa 4 consolidada:** 2026-08-26  
**Etapa 5 consolidada:** 2026-08-27  
**Etapa 6 consolidada:** 2026-08-28  
**Etapa 7 consolidada:** 2026-08-28

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
| 7 | Template físico A4 da Ficha | **CONSOLIDADO / APROVADO PELO PO** |
| 8 | Limites textuais e densidade da Ficha | **PRÓXIMA — AINDA NÃO EM ANÁLISE** |
| 9 | Múltiplos MACs / Procedimentos / dados excepcionais | PENDENTE |
| 10 | Nomes de arquivo + artefatos temporários | PENDENTE |
| 11 | QR / barcode | PENDENTE |
| 12 | Validação técnica final do Bloco 10 | PENDENTE |

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

Dados vêm do cadastro/snapshot aplicável; não há redigitação para gerar a Ficha.

Não imprimir por padrão checklist completo, progresso, passos, comandos/código, timeline/auditoria, IDs internos ou lista detalhada de revisões.

## 4. Observação do serviço por Etapa

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
- estado final aplicável precisa ser historicamente reproduzível;
- não introduzir autosave por inferência.

## 5. PDF canônico e preview

```text
Atendimento confirmado + source_version esperada
→ DocumentModel document_kind = service_sheet
→ template Typst próprio da Ficha
→ PagedDocument

1 página
→ PDF canônico + SVG da mesma página

2+ páginas
→ SHEET_OVERFLOW
```

- PDF 1.7 + Tagged PDF como baseline;
- PDF e SVG derivam do mesmo `PagedDocument`, sem segundo layout HTML;
- sem corte, segunda folha ou redução silenciosa de fonte;
- preview em modal/overlay simples com folha A4 centralizada;
- `Salvar PDF` e `Imprimir` reutilizam os mesmos bytes da prévia;
- impressão reutiliza WebView2 transitória + `ShowPrintUI(System)`;
- resultado é transitório e preso à `source_version`.

---

# Etapa 7 — Template físico A4 da Ficha

**Status:** CONSOLIDADO / APROVADO PELO PO

## 6. Princípio

A folha comunica primeiro o que o cliente precisa entender:

```text
identificar
→ resumir o serviço
→ registrar observações relevantes
```

Não deve parecer relatório técnico completo, checklist ou formulário administrativo carregado.

## 7. Geometria

```text
papel:       A4
orientação:  retrato
páginas:     exatamente 1
margens:     15 mm em todos os lados
bleed:       nenhum
```

- área útil aproximada `180 × 267 mm`;
- nenhuma segunda página;
- nenhuma redução dinâmica de fonte;
- nenhum crop/truncamento silencioso;
- excesso continua `SHEET_OVERFLOW`.

## 8. Ordem da folha

```text
1. Identidade da empresa + Atendimento
2. Identificação curta do serviço
3. Equipamento/dispositivo, quando houver
4. Serviço realizado
5. Observações, quando houver
```

Composição predominantemente vertical/uma coluna. Microagrupamentos horizontais são permitidos para dados curtos.

## 9. Cabeçalho e status

- logo opcional preservando proporção;
- nome da empresa como elemento institucional principal;
- contato/site/e-mail compactos quando configurados;
- código/data do Atendimento facilmente localizáveis;
- sem título gigante `FICHA DE ATENDIMENTO`;
- sem footer obrigatório ou paginação;
- `Em andamento`/acompanhamento é discreto;
- `CANCELADO` é textual e inequívoco, sem depender de cor;
- sem watermark grande.

## 10. Identificação e Equipamento

Identificação do serviço em linha curta, omitindo campos vazios, por exemplo:

```text
João Silva · OS-4587 · Técnico: Maria Souza
```

Equipamento usa **ficha técnica resumida sem grade/tabela pesada**:

```text
NOTE-15 · Notebook · EQP-0031
CPU i5-1135G7 · RAM 16 GB · SSD NVMe 512 GB
Windows 11 Pro 24H2 · Bateria 82%
Serial ABC123 · Patrimônio PAT-884
```

- valores vêm do cadastro/snapshot aplicável;
- armazenamento não assume sempre SSD;
- campos não aplicáveis/vazios desaparecem;
- sem Equipamento, a seção inteira colapsa;
- MAC permanece para Etapa 9.

## 11. Serviço realizado e Observações

`SERVIÇO REALIZADO` contém o `Resumo do trabalho` como texto corrido legível, sem checklist/passos/revisões técnicas.

`OBSERVAÇÕES` é uma única seção client-facing que pode reunir:

- observação geral do Atendimento;
- observação relevante do Equipamento;
- observações do serviço por Etapa.

Apresentação preferencial: lista simples, sem subcards. Nome curto da Etapa aparece apenas quando necessário para contexto.

Se não houver observações, a seção desaparece completamente; não imprimir `Sem observações`.

## 12. Tipografia e contraste

PDF da Ficha usa **Noto Sans**.

Baseline:

```text
identificação principal:   14 pt
seção:                     10,5 pt semibold
corpo/resumo:               10 pt
ficha técnica:               9 pt
metadados institucionais:  8,5 pt
```

- não reduzir abaixo do baseline para caber;
- divisórias horizontais finas somente entre grandes grupos;
- contraste neutro e legível em monocromático;
- sem grandes fundos preenchidos;
- sem paleta RGB/hex congelada nesta fase;
- cor nunca é o único canal semântico.

## 13. Espaçamento e exclusões

- seções não têm altura fixa;
- seções vazias colapsam;
- não reservar caixas para escrita manual;
- observações usam a maior área variável restante;
- não adicionar assinatura, termos jurídicos, garantia, financeiro, peças/estoque, SLA, checklist, progresso, timeline, QR/barcode, lista detalhada de Procedimentos, página 2 ou footer promocional.

Overflow continua falha explícita; a Etapa 8 define limites/priorização textual e a Etapa 9 trata dados multiplicativos/excepcionais.

---

# Etapas seguintes

## Etapa 8 — Limites textuais e densidade

**Status:** PRÓXIMA — AINDA NÃO EM ANÁLISE

Definirá limites/priorização para resumo, observações do Atendimento, Equipamento e Etapas, além do diagnóstico de overflow.

## Etapa 9 — Múltiplos dados excepcionais

Definirá comportamento com muitos MACs, muitas observações/Procedimentos e outros casos que pressionem a única A4.

## Etapa 10 — Nomes + temporários

Definirá naming, materialização local, paths controlados, lifecycle e limpeza de artefatos transitórios.

## Etapa 11 — QR/barcode

Só entra se houver benefício operacional aprovado; não é requisito por padrão.

## Etapa 12 — Validação técnica final

Fechará matriz real de Windows/WebView2/Office compatível, impressão, limites de recursos, casos de erro e critérios técnicos antes da implementação.

## 14. Gate atual

A Etapa 7 está documentalmente consolidada nesta branch, mas **não está operacionalmente encerrada** até:

```text
PR da Etapa 7
→ validação
→ ready
→ squash merge em main
→ apagar branch
→ verificar somente main + zero PRs abertos
```

Somente depois disso a Etapa 8 pode ser aberta.
