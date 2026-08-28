# Bloco 10 — Exportação / Impressão + Ficha Compacta

**Status:** EM ANDAMENTO — ETAPAS 1–10 CONSOLIDADAS / ETAPA 11 PRÓXIMA  
**Fase:** Fase 1 — Fechamento arquitetural e especificação  
**Abertura:** 2026-08-25  
**Etapa 1 consolidada:** 2026-08-25  
**Etapa 2 consolidada:** 2026-08-26  
**Etapa 3 consolidada:** 2026-08-26  
**Etapa 4 consolidada:** 2026-08-26  
**Etapa 5 consolidada:** 2026-08-27  
**Etapa 6 consolidada:** 2026-08-28  
**Etapa 7 consolidada:** 2026-08-28  
**Etapa 8 consolidada:** 2026-08-28  
**Etapa 9 consolidada:** 2026-08-28  
**Etapa 10 consolidada:** 2026-08-28

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
| 8 | Limites textuais e densidade da Ficha | **CONSOLIDADO / APROVADO PELO PO** |
| 9 | Múltiplos MACs / Procedimentos / dados excepcionais | **CONSOLIDADO / APROVADO PELO PO** |
| 10 | Nomes de arquivo + artefatos temporários | **CONSOLIDADO / APROVADO PELO PO** |
| 11 | QR / barcode | **PRÓXIMA — AINDA NÃO EM ANÁLISE** |
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
- recurso local transitório segue o contrato consolidado na Etapa 10.

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
- MAC segue a projeção compacta consolidada na Etapa 9.

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

---

# Etapa 8 — Limites textuais e densidade da Ficha

**Status:** CONSOLIDADO / APROVADO PELO PO

## 14. Princípio central

A restrição de uma A4 pertence ao artefato client-facing e **não pode destruir o dado operacional**.

```text
Atendimento
→ preserva dados reais

Ficha
→ usa os mesmos dados aplicáveis
→ Typst tenta diagramar

1 página
→ válido

2+ páginas
→ SHEET_OVERFLOW
→ técnico revisa conscientemente os campos reais
```

Não criar na v1:

- campos paralelos exclusivos para impressão;
- versão resumida duplicada de observações;
- resumo automático/IA;
- truncamento ou reticências automáticas;
- deduplicação semântica;
- editor paralelo da Ficha.

## 15. Soft limits recomendados

| Campo | Faixa recomendada |
|---|---:|
| `Resumo do trabalho` | até **600 caracteres** |
| Observação geral do Atendimento | até **400 caracteres** |
| Observação do Equipamento | até **300 caracteres** |
| Observação do serviço por Etapa | até **280 caracteres por Etapa** |

Essas faixas são **soft limits**:

- orientam escrita curta compatível com prestação de contas;
- não bloqueiam save;
- não bloqueiam conclusão;
- não truncam nem alteram o dado;
- não garantem encaixe físico sozinhas;
- podem ser refinadas futuramente por evidência da validação real da Etapa 12.

Contador/aviso aparece somente próximo de aproximadamente **80%** da faixa recomendada para não poluir a UI.

## 16. Autoridade de encaixe e overflow

A contagem de caracteres não é autoridade física. O resultado real do Typst decide se a Ficha cabe.

Quando o layout produzir mais de uma página:

```text
SHEET_OVERFLOW
→ nenhum PDF final confirmado
→ nenhum preview final válido
→ nenhuma impressão
→ Atendimento permanece íntegro
```

O bloqueio pertence somente à geração da Ficha.

O Host devolve diagnóstico semântico suficiente para orientar revisão, por exemplo:

```text
SHEET_OVERFLOW
contributors:
- work_summary
- service_general_note
- device_note
- stage_note:<stage_id>
```

Não é necessário prometer percentual exato de contribuição visual. O diagnóstico pode considerar comprimento, quebras/linhas, quantidade de observações, faixas recomendadas ultrapassadas e blocos variáveis presentes.

## 17. UX de correção

```text
Ficha / Imprimir
→ SHEET_OVERFLOW
→ mensagem curta
→ Revisar atendimento
→ destacar discretamente os campos mais prováveis
```

A edição acontece sempre nos dados reais do Atendimento/Equipamento/Etapa. Após salvar estado confirmado, a Ficha é gerada novamente.

Ordem normal das observações:

```text
1. observação geral do Atendimento
2. observação do Equipamento
3. observações das Etapas na ordem executada
```

Textos semelhantes não são deduplicados automaticamente; podem representar fatos distintos.

## 18. Densidade e normalização segura

Existe um único template físico consolidado na Etapa 7.

Não criar fallback automático por:

- modo compacto;
- redução de fonte;
- redução de margem;
- compressão automática de espaçamento;
- omissão de conteúdo legítimo.

Espaço em branco é aceitável e não deve ser preenchido artificialmente.

Normalização de apresentação pode somente remover ruído sem mudar significado, como trim, quebras de linha consistentes, colapso de linhas vazias repetidas sem significado e omissão de campos/labels vazios.

Hard limits técnicos de storage/API não derivam do tamanho da A4; ficam para os gates técnicos/implementação correspondentes.

---

# Etapa 9 — Múltiplos MACs / Procedimentos / dados excepcionais

**Status:** CONSOLIDADO / APROVADO PELO PO

## 19. Princípio de projeção

A Ficha é uma **projeção client-facing resumida**, não uma serialização completa do Atendimento.

```text
dado operacional completo
→ permanece preservado no domínio/histórico

Ficha
→ projeta apenas o que ajuda o cliente a entender o serviço
```

Multiplicidade não autoriza truncar o domínio nem obriga imprimir toda a estrutura interna.

Não introduzir por causa desta Etapa:

- `include_in_sheet` ou `sheet_priority` no domínio;
- editor paralelo da Ficha;
- seleção manual transitória de itens apenas para impressão;
- segunda página;
- modo compacto alternativo;
- redução automática de fonte, margem ou espaçamento.

## 20. Procedimentos vinculados

Procedimentos/revisões vinculados permanecem como **proveniência operacional e histórica** e não entram na Ficha por padrão, independentemente da quantidade.

O cliente recebe a síntese pelo `Resumo do trabalho` e pelas observações client-facing aplicáveis. Não imprimir lista de Procedimentos apenas porque existem vários vínculos.

Essa regra não apaga nem reduz os vínculos internos usados para histórico, auditoria e reprodutibilidade.

## 21. Múltiplos MACs

A projeção de identificadores de rede é determinística:

```text
0 MACs
→ omitir

1 MAC
→ exibir valor compactamente

2 MACs
→ exibir ambos compactamente

3+ MACs
→ exibir somente a quantidade
   "MACs: N identificadores cadastrados"
```

- labels existentes, como `LAN` ou `Wi-Fi`, podem contextualizar os valores exibidos;
- não inventar `MAC principal` se o domínio não possui essa semântica;
- não escolher arbitrariamente dois MACs entre três ou mais como se fossem prioritários;
- a lista completa continua preservada no domínio e nas superfícies operacionais apropriadas.

## 22. Muitas observações e dados excepcionais

Observações legítimas de serviço por Etapa continuam candidatas à Ficha na ordem consolidada. Não existe cap automático do tipo “mostrar somente as primeiras N”.

```text
muitas observações legítimas
→ renderizar normalmente
→ se couber: Ficha válida
→ se não couber: SHEET_OVERFLOW
```

Isso é aceitável: alguns Atendimentos podem exigir revisão humana consciente dos textos reais antes de produzir uma Ficha de uma página.

Campos estruturados excepcionalmente longos — como nome, SO, serial, patrimônio ou label — quebram linha quando a diagramação permitir. Não usar truncamento, reticências ou abreviação inventada para fazê-los caber.

## 23. Diagnóstico de multiplicidade

O diagnóstico semântico de `SHEET_OVERFLOW` pode indicar pressão causada por quantidade ou por campo estruturado longo, por exemplo:

```text
contributors:
- stage_notes_count
- stage_note:<stage_id>
- long_structured_field:<field>
```

Não incluir `process_count` como contribuinte visual enquanto Procedimentos não forem renderizados na Ficha.

A projeção compacta de 3+ MACs reduz a multiplicidade antes do layout; o Host não precisa fingir que cada identificador oculto consumiu espaço físico.

Limites técnicos finais de quantidade/payload/recursos permanecem para a Etapa 12 e não são inferidos da geometria da A4.

---

# Etapa 10 — Nomes de arquivo + artefatos temporários

**Status:** CONSOLIDADO / APROVADO PELO PO

## 24. Arquivo persistente e naming

Arquivo salvo pelo usuário e artefato transitório interno são conceitos distintos.

```text
artefato gerado
→ Salvar
→ diálogo nativo do Windows
→ usuário escolhe pasta/nome
→ arquivo persistente fora do cleanup do StepFlow
```

Nome sugerido para Procedimento:

```text
{codigo} - {titulo} - v{display_version} - r{revision_no}.{ext}
```

Sem `display_version`:

```text
{codigo} - {titulo} - r{revision_no}.{ext}
```

Nome sugerido para Ficha:

```text
{service_code} - Ficha.pdf
```

- a revisão técnica permanece no nome do Procedimento;
- o usuário pode editar o nome no diálogo nativo;
- Ficha não inclui por padrão cliente, equipamento, serial/patrimônio/MAC, resumo, técnico, status ou timestamp no filename;
- arquivo persistente salvo pelo usuário não é apagado posteriormente pelo StepFlow.

## 25. Sanitização e escrita segura

Sanitização afeta somente o filename, nunca o conteúdo documental.

- bloquear caracteres/controles inválidos no Windows, segmentos `.`/`..`, nomes reservados e injeção de caminho;
- preservar Unicode/acentos válidos;
- extensão deriva do tipo de artefato e não de texto livre;
- segmento variável do título pode ser limitado para manter nome manejável;
- conflito com arquivo existente não gera overwrite silencioso: o diálogo do sistema conduz confirmação/substituição ou outro nome;
- save só é sucesso após gravação integral;
- quando a plataforma/filesystem suportar, preferir auxiliar opaco no mesmo diretório e promoção/replace seguro ao final;
- falha tenta remover somente o parcial criado pela própria operação, sem apagar arquivo preexistente do usuário.

## 26. Materialização transitória no Client

Regra principal: se o consumidor consegue usar bytes/memória com segurança, não criar temporário apenas por conveniência.

```text
bytes canônicos
→ materializar no Client somente quando integração local exige filesystem
→ consumo local
→ descarte best-effort
```

- Host não cria temporário para simular acesso à workstation;
- preview pode permanecer em memória/protocolo controlado quando a implementação permitir;
- PDF para WebView2/impressão pode usar recurso local transitório;
- PDF/DOCX final escolhido pelo usuário não é temporário;
- raiz temporária é por usuário, resolvida por API do sistema/Tauri, sob namespace StepFlow;
- cada instância do Client usa subdiretório opaco próprio;
- temporários usam nomes opacos, sem cliente, título, equipamento, serial/MAC, resumo/observações ou técnico;
- não usar instalação, diretório corrente, SMB central, dados SQLite, backup, Documents/Desktop/Downloads ou destino final de exportação como raiz normal de temporários.

## 27. Lifecycle, lock e scavenging

Lifecycle normal:

```text
criar
→ concluir escrita
→ disponibilizar ao consumidor
→ manter enquanto houver uso local
→ fechar consumidor
→ remover best-effort
```

- não apagar enquanto WebView2/Windows ainda puder precisar do recurso;
- encerramento normal tenta limpar somente o diretório da própria instância;
- lock por Windows/WebView2/antivírus não autoriza unlock forçado, kill de processo ou alteração de ACL;
- retry pode ocorrer apenas em momento seguro do lifecycle;
- crash pode deixar órfãos;
- scavenging posterior atua somente no namespace transitório StepFlow, não segue symlink/reparse point para fora e não apaga item que possa pertencer a instância ativa;
- item bloqueado é ignorado e não impede uso normal;
- não criar Windows Service, Task Scheduler, daemon ou watchdog para limpeza;
- falha de cleanup não altera retroativamente o resultado funcional de save/preview/print já concluído.

## 28. Privacidade, persistência e não-objetivos

Logs técnicos podem registrar tipo de artefato, operação, resultado, erro técnico e ID opaco quando necessário, sem conteúdo documental por padrão.

Evitar por padrão em log:

- resumo/observações;
- cliente;
- serial/MAC;
- path completo escolhido pelo usuário quando desnecessário ao diagnóstico.

Não criar nesta etapa:

- pasta global permanente `Exports` mantida pelo Host;
- histórico de exportações no SQLite;
- `export_jobs` persistente;
- cache permanente de PDFs/Fichas por Atendimento;
- retenção automática de cópia de todo arquivo salvo;
- temporários no compartilhamento central por conveniência;
- regeneração silenciosa apenas para salvar/imprimir.

Temporários e exportações não entram em SQLite, histórico ou backup por padrão.

## 29. Validação reservada para a Etapa 12

Validar concretamente antes da implementação:

- API Tauri/Windows para diretório temporário por usuário;
- comportamento WebView2 com PDF local e momento seguro de remoção;
- escrita/promote/replace em NTFS e SMB;
- proteções de symlink/reparse point;
- múltiplas instâncias do Client;
- limites de memória/tamanho/concurrency;
- antivírus/EDR mantendo handle;
- path longo e Unicode no Windows 10/11 alvo;
- política final de retry/scavenging sem daemon.

---

# Etapas seguintes

## Etapa 11 — QR/barcode

**Status:** PRÓXIMA — AINDA NÃO EM ANÁLISE

Só entra se houver benefício operacional aprovado; não é requisito por padrão.

## Etapa 12 — Validação técnica final

Fechará matriz real de Windows/WebView2/Office compatível, impressão, limites de recursos, casos de erro e critérios técnicos antes da implementação.

## 30. Gate atual

A Etapa 10 está documentalmente consolidada nesta branch, mas **não está operacionalmente encerrada** até:

```text
PR da Etapa 10
→ validação
→ ready
→ squash merge em main
→ apagar branch
→ verificar somente main + zero PRs abertos
```

Somente depois disso a Etapa 11 pode ser aberta.
