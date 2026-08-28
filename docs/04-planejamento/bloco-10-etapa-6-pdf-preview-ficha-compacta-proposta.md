# Bloco 10 — Etapa 6 — PDF + preview da Ficha compacta — Proposta para análise

**Status:** PROPOSTA CORRIGIDA / AGUARDANDO APROVAÇÃO DO PO  
**Fase:** Fase 1 — Fechamento arquitetural e especificação  
**Abertura:** 2026-08-28  
**Base consolidada:** Bloco 10 / Etapas 1–5  
**Base Git:** `main` em `80f9b53c7bb3610c60ae73c6599c57af2eb6951c`

## 1. Objetivo

Fechar o contrato técnico da **Ficha compacta de Atendimento em PDF** e sua **pré-visualização no Client**.

A Ficha é uma **prestação de contas resumida ao cliente**: mostra o panorama geral do equipamento e do serviço executado, sem reproduzir o Procedimento, checklist, histórico técnico completo ou detalhes internos desnecessários.

Permanecem fora desta etapa:

- template físico final A4 — Etapa 7;
- limites/priorização textual — Etapa 8;
- muitos MACs/Procedimentos — Etapa 9;
- nomes e temporários concretos — Etapa 10;
- QR/barcode — Etapa 11;
- limites e matriz técnica final — Etapa 12.

## 2. Contratos herdados

Permanecem vigentes:

- `Ficha / Imprimir` parte do Atendimento/Tela 09;
- a ficha usa apenas estado confirmado pelo Host;
- alterações locais não salvas ou conflito pendente bloqueiam geração;
- `Em andamento` pode gerar ficha de acompanhamento;
- `Concluído` pode reimprimir o estado histórico aplicável;
- `Cancelado` precisa ser identificado inequivocamente;
- ADM/Gerência/Funcionário podem gerar/reimprimir ficha de Atendimento acessível conforme matriz vigente;
- campos vazios/não aplicáveis são omitidos;
- Atendimento sem Equipamento continua podendo gerar ficha;
- a ficha nunca ultrapassa uma página A4;
- conteúdo excessivo não cria segunda página nem é truncado silenciosamente;
- DOCX específico da ficha não é requisito v1;
- geração documental pertence ao Host;
- Client não monta documento por HTML/DOM;
- renderização fica fora da fila de mutações e usa limite próprio bounded.

## 3. Correção funcional — finalidade da Ficha

A Ficha não é relatório técnico detalhado, nem espelho da Tela 09, nem extrato de auditoria.

O documento existe para responder de forma curta ao cliente:

```text
qual equipamento/dispositivo foi atendido?
como ele está identificado/configurado?
o que foi feito?
quais observações relevantes surgiram durante o serviço?
```

Consequência: informações internas úteis ao StepFlow podem permanecer no histórico do Atendimento sem aparecer na folha impressa.

Por padrão a Ficha **não precisa imprimir**:

- checklist item a item;
- percentual/progresso;
- passos executados;
- comandos/código;
- timeline operacional;
- IDs técnicos internos;
- `row_revision`;
- lista detalhada de revisões de Procedimento;
- conteúdo integral das Etapas;
- metadados que não ajudam o cliente a compreender o serviço.

A revisão exata dos Procedimentos continua preservada internamente para histórico e consistência, mesmo quando não aparece na Ficha.

## 4. Conteúdo principal da prestação de contas

A Ficha deve priorizar quatro grupos semânticos.

### 4.1 Identificação do serviço

Somente o necessário para reconhecer a prestação:

- código do Atendimento;
- data/período aplicável;
- responsável/técnico;
- cliente/solicitante ou referência externa quando informado e útil;
- status somente quando necessário para evitar ambiguidade, especialmente `Cancelado`.

A Etapa 7 decide a composição física desses dados.

### 4.2 Identificação do equipamento/dispositivo

Quando houver Equipamento vinculado, usar os dados previamente cadastrados/confirmados, sem solicitar redigitação para a Ficha.

Conceitualmente:

- código/nome do Equipamento;
- tipo;
- serial/patrimônio quando existentes e úteis à identificação;
- demais identificadores consolidados conforme regra física posterior.

MAC continua sujeito à Etapa 9, especialmente quando houver múltiplos endereços.

### 4.3 Características relevantes do dispositivo

Quando aplicáveis e informadas:

- processador;
- memória RAM;
- armazenamento HD/SSD e capacidade;
- sistema operacional/versão quando útil;
- saúde/vida da bateria para dispositivo aplicável;
- observação geral do Equipamento.

Esses dados vêm do cadastro/snapshot aplicável já mantido pelo Atendimento. A geração da Ficha não cria um segundo cadastro.

### 4.4 Serviço realizado e observações

É o núcleo da prestação de contas:

- `Resumo do trabalho` do Atendimento;
- Observações gerais do Atendimento, quando preenchidas;
- Observações do Equipamento, quando relevantes;
- **Observações específicas registradas pelo técnico durante a execução das Etapas**.

A Ficha não reproduz a instrução estática da Etapa. Ela leva apenas a observação operacional realmente criada durante aquele serviço.

## 5. Novo refinamento operacional — observação por Etapa

A documentação atual já persiste checklist no Reader vinculado a Atendimento, mas ainda não descreve a observação operacional específica da Etapa.

Fica proposta a seguinte correção transversal:

- durante a execução de um Procedimento em Atendimento, cada Etapa pode ter uma **Observação do serviço** opcional;
- essa observação pertence ao Atendimento/executado, nunca à revisão oficial do Procedimento;
- Reader standalone não cria nem persiste essa observação;
- somente Atendimento `Em andamento` + capacidade apropriada permite edição;
- `Concluído`/`Cancelado` tornam a observação somente leitura até eventual reabertura;
- observação vazia não ocupa espaço na Ficha;
- uma nova publicação do Procedimento não troca nem reatribui a observação de uma execução existente.

Conceitualmente, o dado precisa estar ligado a:

```text
Atendimento
+ vínculo Atendimento × Procedimento/revisão
+ Etapa da revisão executada
+ texto da observação
+ controle concorrente/auditoria mínima
```

O schema físico não é fechado nesta etapa.

## 6. Concorrência das observações por Etapa

Para preservar o modelo multiusuário sem criar conflitos globais desnecessários:

- a observação de uma Etapa deve possuir controle de revisão próprio ou equivalente;
- editar a observação da Etapa 2 não deve invalidar checklist/observação da Etapa 5 por conveniência de implementação;
- conflito no mesmo texto preserva a edição local e exige reconciliação;
- confirmação visual só acontece após estado aceito pelo Host.

Isso segue a mesma filosofia de granularidade já adotada para checklist.

A interação exata de salvar no Reader será consolidada nas fontes de UI quando esta decisão for aprovada; não se introduz autosave por inferência nesta proposta.

## 7. Observações na Ficha

Somente observações realmente preenchidas entram no documento.

Origens possíveis:

```text
Observação geral do Equipamento
Observação geral do Atendimento
Observação do serviço — Etapa A
Observação do serviço — Etapa B
...
```

As observações por Etapa devem manter associação suficiente para o leitor compreender o contexto, mas sem imprimir estrutura técnica pesada.

Exemplo conceitual enxuto:

```text
OBSERVAÇÕES DO SERVIÇO
• Limpeza interna realizada; excesso de poeira próximo ao cooler.
• Troca do SSD: unidade anterior apresentou setores defeituosos.
• Validação final: bateria em 68% de saúde, recomendado acompanhamento.
```

A Etapa 7 define se o título da Etapa aparece ou se a observação é apresentada em lista simples. A Etapa 8 define limites/priorização quando houver texto demais.

Não resumir, truncar ou remover automaticamente uma observação legítima sem regra consolidada. Se o conjunto ultrapassar uma A4, a geração falha por overflow e orienta revisão dos campos aplicáveis.

## 8. DocumentModel da Ficha

A Ficha reutiliza a arquitetura `DocumentModel` já consolidada, com `document_kind` próprio.

Conceitualmente:

```text
DocumentModel
├── document_kind = service_sheet
├── source_identity = Atendimento
├── source_version
├── company_identity
├── metadata
│   ├── atendimento
│   ├── data/status aplicável
│   ├── responsável
│   └── cliente/referência?
├── sections
│   ├── device_summary?
│   │   ├── identity
│   │   ├── cpu
│   │   ├── memory
│   │   ├── storage
│   │   ├── os?
│   │   ├── battery?
│   │   └── device_note?
│   ├── work_summary
│   ├── general_service_note?
│   └── stage_service_notes[]
└── generation_metadata
```

O modelo pode carregar informações internas adicionais necessárias à consistência, mas o template da Ficha só apresenta o subconjunto aprovado para o cliente.

## 9. PDF próprio e canônico

A Ficha possui PDF próprio, diferente do PDF de Procedimento.

Fluxo:

```text
Atendimento confirmado
→ snapshot/revisão esperada
→ DocumentModel da Ficha
→ template Typst interno específico
→ PagedDocument
→ validar exatamente 1 página
→ PDF canônico da Ficha
```

A Ficha reutiliza a infraestrutura Typst embutida do Host:

- PDF 1.7;
- Tagged PDF como baseline estrutural;
- texto real selecionável/pesquisável/copiável;
- fontes/assets controlados;
- nenhum recurso remoto em runtime;
- conteúdo do domínio apenas como dados estruturados;
- falha nunca produz PDF parcial tratado como sucesso.

Não criar HTML→PDF, screenshot, browser headless ou nova biblioteca PDF apenas para a Ficha.

## 10. Gate rígido de uma página

A validação ocorre depois do layout real.

```text
DocumentModel
→ Typst layout
→ PagedDocument

1 página
→ válido

0 páginas
→ erro de geração

2+ páginas
→ SHEET_OVERFLOW
→ nenhum PDF/preview confirmado
```

Proibido resolver overflow por:

- retornar somente a primeira página;
- criar segunda folha;
- cortar texto silenciosamente;
- remover observações sem regra;
- reduzir fonte dinamicamente até caber.

Template A4 final = Etapa 7. Limites/priorização de conteúdo = Etapa 8.

## 11. Preview usa exatamente o mesmo layout

O preview não possui um segundo template.

```text
mesmo DocumentModel
→ mesmo template Typst
→ mesmo PagedDocument de 1 página
   ├─→ PDF canônico
   └─→ SVG de preview
```

O SVG é uma representação vetorial da mesma página diagramada.

Benefícios:

- preview e impressão mostram o mesmo conteúdo;
- não existe divergência HTML × PDF;
- não é screenshot da Tela 09;
- não exige toolbar do visualizador PDF;
- encaixa no princípio de baixa densidade visual do Pocket.

O PDF permanece o artefato oficial; SVG é transitório e somente para visualização.

## 12. Superfície de preview no Client

Fluxo:

```text
Tela 09
→ Ficha / Imprimir
→ Host gera PDF + preview
→ modal/overlay grande
```

A superfície deve ser simples:

- uma folha A4 centralizada e escalada proporcionalmente;
- sem sidebar adicional;
- sem metadados repetidos fora da folha;
- sem toolbar textual extensa;
- ações mínimas para salvar PDF, imprimir e fechar;
- ícones podem ser usados quando inequívocos, com nome acessível/tooltip.

Salvar e imprimir reutilizam os mesmos bytes PDF correspondentes à prévia aberta.

## 13. Impressão

A impressão da Ficha reutiliza o mecanismo Windows já consolidado:

```text
PDF canônico da Ficha
→ recurso local transitório
→ WebView2 dedicada/transitória
→ ShowPrintUI(System)
→ diálogo de impressão do Windows
```

Não criar renderer separado para impressão.

Detalhes de nome/path/limpeza do recurso local permanecem na Etapa 10.

## 14. Fonte e estabilidade da geração

A prévia e o PDF ficam presos ao Atendimento + `source_version` usado na geração.

Se o Atendimento mudar depois:

- a folha aberta não muda silenciosamente;
- o Client indica de forma discreta que a ficha precisa ser atualizada;
- nova saída do estado atual exige regeneração;
- não existe regeneração automática em background substituindo a folha que está sendo conferida.

Em `Concluído`, usar o estado histórico congelado aplicável, incluindo características/observações preservadas. Em `Cancelado`, identificar o cancelamento claramente.

## 15. Resultado transitório

Conceitualmente:

```text
ServiceSheetGenerationResult
├── source_identity
├── source_version
├── pdf_bytes
└── preview_svg
```

- PDF e SVG derivam da mesma página;
- não há job persistente;
- não há URL pública permanente;
- artefatos não viram histórico/backup automaticamente;
- fechar o preview pode descartar o resultado local transitório.

## 16. Estado e erros

Estados mínimos:

```text
preparando ficha
→ preparando prévia
→ pronta
```

Falhas relevantes:

- alteração local não salva;
- conflito;
- sem permissão;
- Atendimento indisponível;
- revisão esperada obsoleta;
- `SHEET_OVERFLOW`;
- falha de renderer/template/asset;
- falha de preview;
- backpressure/limite de recursos;
- conexão interrompida.

Nenhum percentual fictício. Nenhum artefato parcial como sucesso.

## 17. Alterações canônicas necessárias após aprovação

Esta proposta introduz um refinamento que impacta decisões anteriores.

Se aprovada, a consolidação da Etapa 6 deverá atualizar minimamente:

- Tela 05 — Reader operacional: área de Observação do serviço por Etapa;
- Tela 09 — Atendimento: observações específicas das Etapas como parte do estado operacional e da Ficha;
- Tela 14 — Ficha: finalidade de prestação de contas resumida e conteúdo client-facing enxuto;
- modelo de dados conceitual: persistência da observação por Etapa/vínculo de execução;
- Bloco 9/registro de decisões: lifecycle, concorrência e snapshot dessas observações;
- arquitetura/Bloco 10: `DocumentModel` e PDF/preview da Ficha.

Nenhuma fonte canônica é alterada antes da aprovação do PO.

## 18. Decisões propostas da Etapa 6

1. a Ficha é uma prestação de contas resumida ao cliente;
2. o conteúdo impresso prioriza identificação do serviço, dispositivo, características, resumo do trabalho e observações;
3. checklist, progresso, passos, comandos, timeline e detalhes internos não aparecem por padrão;
4. cada Etapa executada em Atendimento pode receber uma Observação do serviço opcional e persistente;
5. observação por Etapa é estado do Atendimento, não conteúdo da revisão do Procedimento;
6. observações por Etapa vazias são omitidas da Ficha;
7. observações preenchidas entram no material impresso dentro das regras de uma A4;
8. equipamento usa dados previamente cadastrados/snapshotados, sem redigitação para gerar a Ficha;
9. PDF específico da Ficha passa a ser artefato canônico;
10. PDF e preview SVG derivam do mesmo `PagedDocument` Typst;
11. resultado precisa possuir exatamente uma página;
12. overflow falha explicitamente, sem segunda página/truncamento/redução arbitrária de fonte;
13. preview é modal/overlay simples dentro do Client;
14. salvar/imprimir usam os mesmos bytes PDF apresentados pela geração;
15. impressão reutiliza WebView2 + `ShowPrintUI(System)`;
16. prévia fica presa à `source_version` e nunca muda silenciosamente após evento remoto;
17. DOCX da Ficha continua fora da v1;
18. Etapas 7–12 permanecem fechadas até esta Etapa 6 ser aprovada/consolidada e operacionalmente encerrada.

## 19. Fora do escopo

- layout físico final da folha;
- margens e tipografia finais;
- quantidade máxima de caracteres;
- regra exata para muitos MACs/Procedimentos;
- mecanismo visual exato de salvar observação por Etapa;
- schema físico/migration;
- nomes de arquivos;
- temporários concretos;
- QR/barcode;
- implementação funcional.

## 20. Gate

Até aprovação explícita do PO:

- PR permanece draft;
- somente esta proposta pode ser alterada;
- nenhuma fonte canônica é promovida;
- Etapa 7 não é aberta.
