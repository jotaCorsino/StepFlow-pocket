# Tela 14 — Exportação / Impressão + Ficha Compacta — UX

## 1. Identificação

- código/nome da tela: Tela 14 — Exportação / Impressão + Ficha Compacta — UX;
- status: **CONSOLIDADO / APROVADO PELO PO**;
- bloco original: Fase 1 — Bloco 8 (UI/UX);
- atualização técnica/funcional: Bloco 10 / Etapas 1–6;
- última consolidação: 2026-08-28.

## 2. Objetivo

Definir duas famílias distintas de saída sem transformar telas em documentos:

1. **Procedimento** — exportar/imprimir uma revisão específica como documento técnico completo;
2. **Ficha compacta de Atendimento** — prestação de contas resumida ao cliente, com no máximo uma folha A4.

As duas superfícies compartilham a arquitetura documental do Host, mas possuem templates e objetivos diferentes.

## 3. Princípios consolidados

- documento nunca é screenshot da interface;
- geração pertence ao Host e usa estado/revisão autorizados;
- identidade da empresa vem da configuração central vigente;
- exportar/imprimir não altera dados funcionais;
- revisão/estado selecionado não é substituído silenciosamente por versão mais nova;
- campos vazios ou não aplicáveis são omitidos;
- Client mantém a experiência local de salvar/preview/imprimir;
- autorização real permanece no Host;
- baixa densidade textual também orienta preview e documento quando compatível com clareza.

## 4. Dois fluxos separados

```text
Leitor de Processo
→ Exportar / Imprimir
→ PDF | DOCX | Imprimir
→ documento completo da revisão selecionada
```

```text
Atendimento
→ Ficha / Imprimir
→ PDF + preview da prestação de contas
→ Salvar PDF | Imprimir
```

Não criar item global `Exportações` na sidebar.

---

# Procedimentos

## 5. Permissão e fonte

A matriz vigente permite `Exportar/imprimir` por padrão para ADM, Gerência e Funcionário, desde que a sessão também possa ler a revisão selecionada.

A revisão aberta é a fonte exata:

```text
r17 aberta
→ solicitar exportação de r17
→ r18 surge/publica
→ documento continua sendo r17
```

Se a revisão deixar de estar autorizada antes da geração ser aceita, o Host rejeita a operação.

## 6. Entrada e painel

```text
Leitor
→ menu contextual
→ Exportar / Imprimir
```

```text
PR-014 · Configuração de VLAN
Versão 2.0 · revisão r18 · Publicada

[ Exportar PDF ] [ Exportar DOCX ] [ Imprimir ]
```

Revisão histórica/draft autorizado deve ser identificada inequivocamente.

## 7. Escopo do Procedimento exportado

A primeira versão exporta o **Procedimento completo** da revisão selecionada.

Quando aplicável, contém:

- identidade da empresa;
- código/título;
- Área/Departamento;
- responsável documental;
- categorias;
- versão/revisão e estado editorial necessário;
- objetivo;
- pré-requisitos;
- observações gerais;
- Etapas em ordem;
- parágrafos;
- passos/subpassos;
- checklist documental;
- notas/alertas;
- comandos/código.

Não entram sidebar, stepper, botões, toasts, ícones de copiar ou demais chrome da UI.

## 8. Saídas e layout do Procedimento

Baseline consolidado do Bloco 10:

- PDF próprio via Typst embutido no Host;
- DOCX real via pipeline Rust próprio, sem PDF → DOCX;
- impressão usa o mesmo PDF oficial no Client Windows;
- A4 retrato multipágina;
- sem capa exclusiva por padrão;
- sem sumário físico obrigatório por padrão;
- margens-base 18 mm;
- sem header repetitivo;
- rodapé compacto de identificação/paginação;
- PDF usa Noto Sans/Noto Sans Mono incorporadas;
- DOCX referencia Arial/Consolas sem embedding v1;
- PDF é a referência física de impressão; DOCX é refluível.

Procedimentos podem ocupar várias páginas. O limite de uma A4 nunca se aplica ao Procedimento completo.

## 9. Estados de geração documental

A UX suporta:

- preparando;
- pronto para salvar/imprimir;
- concluído;
- cancelado pelo usuário;
- falha de geração;
- Host indisponível;
- sem permissão;
- revisão indisponível/obsoleta.

Não inventar percentual sem progresso real.

Exportação é leitura/derivação: não cria revisão, não publica, não altera checklist/progresso nem muda `updated_at` funcional apenas pela geração.

---

# Ficha compacta de Atendimento

## 10. Finalidade

A Ficha é uma **prestação de contas resumida para o cliente**.

Ela deve responder rapidamente:

```text
qual serviço foi realizado?
em qual computador/dispositivo?
quais características principais ele possui?
qual foi o panorama geral do trabalho?
houve alguma observação relevante durante o serviço?
```

Não é relatório técnico detalhado, checklist impresso, reprodução do Procedimento nem timeline operacional.

## 11. Entrada, capacidade e lifecycle

Ponto de entrada:

```text
Tela 09 — Atendimento
→ Ficha / Imprimir
```

Preset consolidado:

- ADM: sim;
- Gerência: sim;
- Funcionário: sim para Atendimento acessível.

Lifecycle:

- `Em andamento`: pode gerar ficha de acompanhamento do estado confirmado;
- `Concluído`: pode reimprimir o estado histórico aplicável;
- `Cancelado`: pode gerar/reimprimir quando autorizado, identificando claramente `Cancelado`.

Alterações locais não salvas ou conflito pendente bloqueiam a geração.

## 12. Fonte confirmada

A ficha nunca mistura rascunho local com estado oficial.

```text
Ficha / Imprimir
→ existem alterações não salvas/conflito?
   → sim: salvar/reconciliar primeiro
   → não: Host captura estado/revisão esperada
→ gerar
```

Uma prévia já aberta permanece ligada à `source_version` usada. Evento remoto não troca a folha silenciosamente; o usuário precisa regenerar para produzir uma nova saída atual.

## 13. Conteúdo essencial da Ficha

### Serviço

Quando aplicável:

- código do Atendimento;
- OS/referência externa;
- cliente/solicitante;
- técnico/responsável;
- data/status necessário para contexto;
- `Resumo do trabalho`;
- observações gerais relevantes.

### Equipamento/dispositivo — quando houver

O documento usa dados já cadastrados/snapshot aplicável; não exige redigitação para imprimir.

Prioriza:

- identificação/nome/tipo;
- processador;
- RAM;
- armazenamento HD/SSD;
- sistema operacional quando útil;
- serial/patrimônio quando úteis à identificação;
- MAC somente conforme regra futura de densidade;
- saúde da bateria quando aplicável e informada;
- observações gerais/específicas do Equipamento.

Campos vazios/não aplicáveis são omitidos.

### Observações de serviço por Etapa

Durante a execução vinculada, cada Etapa pode receber `Observação do serviço` opcional no Reader.

Quando houver conteúdo, essas observações entram na prestação de contas de forma resumida e legível.

```text
Etapa: Validação do SSD
Observação: unidade antiga apresentou setores defeituosos.
```

A ficha não precisa reproduzir o texto completo da Etapa para contextualizar a observação. A forma física exata de apresentação será fechada nas Etapas 7–8 do Bloco 10.

## 14. O que não aparece por padrão na Ficha

Para preservar uma folha limpa e útil ao cliente, não imprimir por padrão:

- checklist completo;
- percentual/progresso;
- lista de passos/subpassos;
- comandos/código;
- timeline/auditoria;
- IDs internos;
- detalhes de concorrência;
- lista detalhada de revisões técnicas utilizadas;
- metadados editoriais que não ajudam o cliente a entender o serviço.

Os vínculos/revisões continuam preservados internamente para histórico e consistência.

## 15. Atendimento sem Equipamento

A Ficha também pode existir sem Equipamento.

Nesse cenário:

- seção de Equipamento é omitida;
- não se reserva grande área vazia;
- identificação do serviço, resumo e observações continuam válidos.

Isso atende rede, infraestrutura, Help Desk e outras execuções sem ativo físico vinculado.

## 16. Regra rígida de uma A4

A Ficha possui **uma única página A4**.

```text
1 página → válida
2+ páginas → overflow → geração bloqueada
```

Não:

- criar segunda folha;
- retornar só a primeira página;
- truncar silenciosamente informação importante;
- reduzir fonte indefinidamente;
- omitir conteúdo conhecido de forma silenciosa apenas para caber.

Mensagem funcional base:

`A ficha possui conteúdo demais para uma página A4. Revise os campos indicados antes de imprimir.`

Template físico final pertence à Etapa 7; limites/priorização textual à Etapa 8; múltiplos MACs/Procedimentos excepcionais à Etapa 9.

## 17. PDF canônico da Ficha

A Ficha possui PDF próprio.

Fluxo:

```text
Atendimento confirmado
→ DocumentModel document_kind = service_sheet
→ template Typst interno da Ficha
→ PagedDocument de exatamente 1 página
→ PDF canônico
```

Baseline:

- PDF 1.7;
- Tagged PDF como baseline estrutural;
- texto real selecionável/pesquisável;
- assets/fontes controlados;
- sem HTML → PDF;
- sem screenshot/canvas;
- sem renderer externo;
- falha nunca retorna artefato parcial como sucesso.

DOCX específico da Ficha não é requisito inicial.

## 18. Preview da Ficha

O preview representa **o mesmo layout que originou o PDF**.

```text
mesmo DocumentModel
→ mesmo template Typst
→ mesmo PagedDocument de 1 página
   ├─→ PDF
   └─→ SVG de preview
```

O SVG é somente representação visual da mesma página diagramada; o PDF permanece artefato canônico.

Superfície:

```text
Tela 09
→ Ficha / Imprimir
→ modal/overlay grande
→ folha A4 centralizada

Ficha AT-000142                  [ salvar ] [ imprimir ] [ × ]
```

Regras:

- sem nova sidebar;
- sem toolbar extensa de navegador;
- página escalada proporcionalmente;
- controles compactos, com ícones quando inequívocos e nomes acessíveis;
- PDF e preview pertencem à mesma `source_version`;
- mudança remota não substitui a prévia silenciosamente;
- preview desatualizado pede regeneração antes de nova saída atual.

## 19. Salvar PDF e imprimir

`Salvar PDF` e `Imprimir` reutilizam os **mesmos bytes PDF** correspondentes à prévia aberta; não regeneram silenciosamente outro documento.

### Salvar

- usuário escolhe destino local;
- Host não recebe path arbitrário da workstation;
- cancelar diálogo é ação voluntária, não erro;
- naming final fica para Etapa 10.

### Imprimir

Reutiliza o fluxo Windows consolidado:

```text
PDF canônico
→ recurso local transitório controlado
→ WebView2 dedicada/transitória
→ ShowPrintUI(System)
→ diálogo Windows
```

O StepFlow informa entrega do fluxo ao Windows, não afirma que papel foi fisicamente impresso quando a API não fornece essa confirmação.

## 20. Estados da Ficha

Estados mínimos:

- preparando ficha;
- preparando prévia;
- pronta;
- desatualizada por mudança confirmada do Atendimento;
- cancelada pelo usuário;
- sem permissão;
- fonte/revisão indisponível;
- `SHEET_OVERFLOW`;
- falha de renderer/preview;
- Host indisponível/`SERVER_BUSY`.

Não exibir percentual fictício.

## 21. Acessibilidade e baixa densidade

- ações operáveis por teclado;
- icon-only apenas quando inequívoco, sempre com nome acessível;
- foco gerenciado no modal;
- status/cancelamento/desatualização não dependem apenas de cor;
- folha permanece A4 independentemente da janela;
- interface ao redor da folha mostra apenas o necessário para entender e agir.

## 22. Pendências restantes do Bloco 10

- Etapa 7 — template físico A4 final da Ficha;
- Etapa 8 — limites textuais, priorização e densidade;
- Etapa 9 — múltiplos MACs/Procedimentos e casos excepcionais;
- Etapa 10 — nomes de arquivo + temporários;
- Etapa 11 — QR/barcode somente se houver benefício aprovado;
- Etapa 12 — validação técnica final/matriz/limites de recursos.

## 23. Fora do escopo inicial

- assinatura digital;
- envio por e-mail;
- nuvem/compartilhamento externo;
- exportação em lote/ZIP;
- editor visual de templates;
- armazenamento permanente de toda exportação;
- DOCX da Ficha;
- QR/barcode sem aprovação;
- implementação funcional nesta fase.

## 24. Decisões consolidadas pelo PO

1. exportação/impressão permanece contextual;
2. Procedimento exportado usa revisão exata e documento completo;
3. PDF, DOCX e impressão de Procedimento são documentos próprios;
4. Ficha é prestação de contas resumida ao cliente;
5. Ficha usa estado confirmado/histórico aplicável;
6. Ficha pode existir sem Equipamento;
7. dados do Equipamento vêm do cadastro/snapshot já existente;
8. observações de serviço por Etapa são persistentes no Atendimento e podem entrar na Ficha;
9. checklist/progresso/revisões detalhadas não poluem a Ficha por padrão;
10. Ficha nunca ultrapassa uma A4;
11. Ficha possui PDF canônico próprio;
12. preview SVG e PDF derivam do mesmo PagedDocument;
13. Salvar/Imprimir reutilizam o PDF correspondente à prévia;
14. impressão usa o fluxo Windows já consolidado;
15. PDF/preview não alteram dados funcionais;
16. DOCX da Ficha não é requisito inicial;
17. QR/barcode permanece pendente de benefício operacional explícito.