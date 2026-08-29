# Tela 14 — Exportação / Impressão + Ficha Compacta — UX

## 1. Identificação

- código/nome da tela: Tela 14 — Exportação / Impressão + Ficha Compacta — UX;
- status: **CONSOLIDADO / APROVADO PELO PO**;
- bloco original: Fase 1 — Bloco 8 (UI/UX);
- atualização técnica/funcional: Bloco 10 / Etapas 1–10;
- última consolidação: 2026-08-29.

## 2. Objetivo

Definir duas famílias distintas de saída sem transformar telas em documentos:

1. **Procedimento** — exportar/imprimir uma revisão específica como documento técnico completo;
2. **Ficha compacta de Atendimento** — prestação de contas resumida ao cliente, com exatamente uma folha A4 quando válida.

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
- baixa densidade textual orienta preview e documento quando compatível com clareza;
- a Ficha usa espaço apenas para informação existente e útil ao cliente;
- arquivo persistente escolhido pelo usuário e artefato transitório interno possuem lifecycles distintos.

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

Nome persistente sugerido:

```text
{codigo} - {titulo} - v{display_version} - r{revision_no}.{ext}
```

Sem `display_version`, omitir somente esse segmento. O usuário pode editar o nome no diálogo nativo antes de salvar. A sanitização do filename nunca altera o conteúdo documental.

## 9. Estados de geração documental

A UX suporta preparando, pronto para salvar/imprimir, concluído, cancelado pelo usuário, falha de geração, Host indisponível, sem permissão e revisão indisponível/obsoleta.

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
- `Cancelado`: pode gerar/reimprimir quando autorizado, identificando claramente `CANCELADO`.

Alterações locais não salvas ou conflito pendente bloqueiam a geração.

## 12. Fonte confirmada

A Ficha nunca mistura rascunho local com estado oficial.

```text
Ficha / Imprimir
→ existem alterações não salvas/conflito?
   → sim: salvar/reconciliar primeiro
   → não: Host captura estado/revisão esperada
→ gerar
```

Uma prévia já aberta permanece ligada à `source_version` usada. Evento remoto não troca a folha silenciosamente; o usuário precisa regenerar para produzir uma nova saída atual.

## 13. Conteúdo essencial

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

Prioriza identificação/nome/tipo, processador, RAM, armazenamento HD/SSD, sistema operacional quando útil, serial/patrimônio quando úteis, saúde da bateria quando aplicável e observações relevantes.

MACs usam projeção compacta e determinística:

```text
0 MACs → omitir
1 MAC  → exibir valor
2 MACs → exibir ambos
3+     → "MACs: N identificadores cadastrados"
```

Labels existentes podem contextualizar os valores exibidos. Não inventar `MAC principal` nem escolher arbitrariamente dois identificadores entre três ou mais.

Campos vazios/não aplicáveis são omitidos.

### Observações de serviço por Etapa

Durante a execução vinculada, cada Etapa pode receber `Observação do serviço` opcional no Reader.

Quando houver conteúdo, essas observações entram na prestação de contas de forma resumida e legível. O texto completo da Etapa não precisa ser reproduzido; um nome curto da Etapa só aparece quando necessário para contextualizar a observação.

Observações legítimas não recebem cap automático de quantidade nem descarte apenas para caber na Ficha.

## 14. O que não aparece por padrão

Para preservar uma folha limpa e útil ao cliente, não imprimir por padrão:

- checklist completo;
- percentual/progresso;
- lista de passos/subpassos;
- comandos/código;
- timeline/auditoria;
- IDs internos;
- detalhes de concorrência;
- lista detalhada de revisões técnicas utilizadas;
- lista de Procedimentos vinculados;
- metadados editoriais que não ajudam o cliente a entender o serviço.

Os vínculos/revisões continuam preservados internamente para histórico e consistência. A quantidade de Procedimentos não muda essa regra: eles são proveniência operacional/histórica, não conteúdo client-facing por padrão.

## 15. Atendimento sem Equipamento

A Ficha também pode existir sem Equipamento.

Nesse cenário:

- seção de Equipamento é omitida;
- não se reserva área vazia;
- identificação do serviço, resumo e observações continuam válidos.

Não imprimir `Nenhum equipamento vinculado` na prestação de contas.

## 16. Regra rígida de uma A4

A Ficha válida possui **exatamente uma página A4**.

```text
1 página → válida
2+ páginas → SHEET_OVERFLOW → geração bloqueada
```

Não criar segunda folha, retornar apenas a primeira página, truncar informação importante, reduzir fonte dinamicamente ou omitir conteúdo silenciosamente para caber.

Mensagem funcional base:

`A ficha ficou extensa demais para uma página A4. Revise os textos indicados e gere novamente.`

### Limites textuais como orientação

A contagem de caracteres não decide se a Ficha cabe. O layout real do Typst permanece a autoridade final.

Soft limits recomendados:

| Campo | Faixa recomendada |
|---|---:|
| `Resumo do trabalho` | até **600 caracteres** |
| Observação geral do Atendimento | até **400 caracteres** |
| Observação do Equipamento | até **300 caracteres** |
| Observação do serviço por Etapa | até **280 caracteres por Etapa** |

- os limites são orientativos;
- ultrapassar a faixa não bloqueia save nem conclusão;
- o dado operacional nunca é truncado ou substituído para satisfazer a A4;
- contador/aviso aparece somente próximo de aproximadamente **80%** da faixa recomendada;
- hard limits técnicos de storage/API são independentes da geometria A4.

Exemplo de orientação discreta:

```text
Observação do serviço
[ texto ]
326 / 280 recomendado
Texto extenso pode dificultar a Ficha de uma página.
```

### Overflow e correção

Quando o Typst produzir 2+ páginas:

```text
SHEET_OVERFLOW
→ nenhum PDF final válido
→ nenhum preview final válido
→ nenhuma impressão
→ Atendimento permanece salvo e íntegro
```

O Host devolve diagnóstico semântico indicando os principais campos que pressionam a Ficha. Não precisa atribuir percentual exato de contribuição visual.

O Client oferece ação de revisão e leva o técnico aos **campos reais** do Atendimento/Equipamento/Etapa. Não existe editor paralelo exclusivo da Ficha.

Não usar para “resolver” overflow:

- IA/resumo automático;
- truncamento/reticências;
- deduplicação semântica;
- modo compacto alternativo;
- redução automática de fonte, margem ou espaçamento;
- omissão silenciosa de conteúdo legítimo.

Ordem normal das observações: Atendimento → Equipamento → Etapas na ordem executada. Textos semelhantes não são deduplicados automaticamente.

### Multiplicidade e dados excepcionais

A Ficha é uma projeção client-facing, não dump completo do domínio.

- Procedimentos vinculados permanecem fora da Ficha por padrão, seja um ou vários;
- 1–2 MACs aparecem compactamente; 3+ MACs são representados apenas pela quantidade cadastrada;
- muitas observações legítimas continuam candidatas à Ficha; se não couberem, o resultado correto é `SHEET_OVERFLOW`;
- campos estruturados excepcionalmente longos quebram linha quando possível, sem truncamento, reticências ou abreviação inventada;
- o diagnóstico pode apontar `stage_notes_count`, `stage_note:<stage_id>` ou `long_structured_field:<field>`;
- não criar `include_in_sheet`, `sheet_priority`, seleção transitória de itens ou editor paralelo apenas para resolver multiplicidade.

É aceitável que um Atendimento real exija revisão humana consciente dos textos antes de produzir uma Ficha de uma página.

## 17. Template físico consolidado

### Geometria

```text
papel:       A4
orientação:  retrato
páginas:     exatamente 1
margens:     15 mm em todos os lados
bleed:       nenhum
```

A composição é predominantemente vertical/uma coluna. Pequenos grupos horizontais são permitidos para dados curtos do serviço e do Equipamento.

### Ordem da folha

```text
1. Identidade da empresa + Atendimento
2. Identificação curta do serviço
3. Equipamento/dispositivo, quando houver
4. Serviço realizado
5. Observações, quando houver
```

### Cabeçalho

- logo opcional preservando proporção;
- nome da empresa como principal elemento institucional;
- contato/site/e-mail compactos quando configurados;
- código e data do Atendimento facilmente localizáveis;
- sem título gigante `FICHA DE ATENDIMENTO`;
- sem footer obrigatório e sem paginação;
- `Em andamento`/acompanhamento é discreto;
- `CANCELADO` deve ser textual e inequívoco, sem depender de cor;
- sem watermark grande.

### Identificação do serviço

Preferir linha curta, omitindo campos vazios:

```text
João Silva · OS-4587 · Técnico: Maria Souza
```

Não usar tabela de formulário apenas para distribuir esses campos.

### Equipamento

Usar **ficha técnica resumida sem grade/tabela pesada**:

```text
NOTE-15 · Notebook · EQP-0031
CPU i5-1135G7 · RAM 16 GB · SSD NVMe 512 GB
Windows 11 Pro 24H2 · Bateria 82%
Serial ABC123 · Patrimônio PAT-884
```

Armazenamento não assume sempre SSD; o valor real do domínio é apresentado. Campos ausentes desaparecem e a seção inteira colapsa quando não houver Equipamento. MACs seguem a projeção compacta da Etapa 9.

### Serviço realizado

`SERVIÇO REALIZADO` contém o `Resumo do trabalho` como texto corrido legível. Não repetir checklist, passos ou revisão técnica.

### Observações

Usar uma única seção `OBSERVAÇÕES`, reunindo quando aplicável:

- observação geral do Atendimento;
- observação relevante do Equipamento;
- observações de serviço por Etapa.

Apresentação preferencial: lista simples, sem subcards. Nome curto da Etapa entra somente quando melhora a compreensão. Se não houver observações, a seção inteira é omitida; não imprimir `Sem observações`.

## 18. Tipografia, contraste e espaçamento

PDF da Ficha usa **Noto Sans**.

Baseline:

```text
identificação principal:   14 pt
seção:                     10,5 pt semibold
corpo/resumo:               10 pt
ficha técnica:               9 pt
metadados institucionais:  8,5 pt
```

Regras:

- não reduzir dinamicamente abaixo do baseline para caber;
- divisórias horizontais finas apenas entre grandes grupos;
- contraste neutro e legível em monocromático;
- sem grandes fundos preenchidos;
- não congelar hex/RGB nesta fase;
- seções não têm altura fixa;
- seções vazias colapsam;
- Observações usam a maior área variável restante;
- não reservar caixas para escrita manual.

## 19. Exclusões do template

Não adicionar por inferência:

- assinatura do cliente/técnico;
- campo manual de data;
- termos jurídicos/garantia;
- valores/custos/financeiro;
- peças/estoque;
- SLA/prioridade;
- checklist/progresso/timeline;
- lista detalhada de Procedimentos;
- página 2;
- footer promocional do StepFlow.

## 20. PDF canônico e preview

```text
Atendimento confirmado
→ DocumentModel document_kind = service_sheet
→ template Typst interno da Ficha
→ mesmo PagedDocument de 1 página
   ├─→ PDF canônico
   └─→ SVG de preview
```

Baseline:

- PDF 1.7;
- Tagged PDF como baseline estrutural;
- texto real selecionável/pesquisável;
- assets/fontes controlados;
- sem HTML → PDF;
- sem screenshot/canvas;
- sem renderer externo;
- DOCX específico da Ficha não é requisito inicial.

## 21. Preview, salvar e imprimir

Preview:

```text
Tela 09
→ Ficha / Imprimir
→ modal/overlay grande
→ folha A4 centralizada

Ficha AT-000142                  [ salvar ] [ imprimir ] [ × ]
```

- sem nova sidebar ou toolbar extensa;
- página escalada proporcionalmente;
- controles compactos e acessíveis;
- PDF e preview pertencem à mesma `source_version`;
- mudança remota não substitui a prévia silenciosamente.

`Salvar PDF` e `Imprimir` reutilizam os **mesmos bytes PDF** da prévia aberta; não regeneram silenciosamente outro documento.

Impressão reutiliza WebView2 dedicada/transitória + `ShowPrintUI(System)`. O StepFlow informa entrega do fluxo ao Windows, não confirmação física de papel impresso.

### Nome persistente e save

A Ficha sugere:

```text
{service_code} - Ficha.pdf
```

Não incluir por padrão cliente, equipamento, serial/patrimônio/MAC, resumo, técnico, status ou timestamp no filename. O usuário pode editar o nome no diálogo nativo.

Conflito de nome nunca sobrescreve silenciosamente arquivo existente. O save só é sucesso após a gravação integral do arquivo aceito pelo usuário; quando suportado pela API/filesystem, preferir auxiliar opaco no mesmo diretório de destino e promoção/replace seguro ao final.

### Artefatos transitórios internos

```text
bytes canônicos
→ manter em memória quando suficiente
→ materializar no Client somente se uma integração local exigir filesystem
→ consumir
→ remover best-effort após liberação
```

- temporários ficam em diretório temporário por usuário resolvido por API do sistema/Tauri, sob namespace StepFlow e subdiretório opaco por instância;
- nomes temporários são opacos, como `print-<opaque-id>.pdf`, sem dados de negócio;
- não usar pasta de instalação, compartilhamento central, dados SQLite, backup, Documents/Desktop/Downloads ou pasta final de exportação como raiz normal de temporários;
- cada instância do Client isola seus temporários;
- não apagar enquanto WebView2/Windows ainda puder usar o recurso;
- lock de arquivo gera cleanup/retry seguro, nunca kill, alteração de ACL ou unlock forçado;
- crash pode deixar órfãos; scavenging posterior é best-effort e restrito ao namespace transitório do StepFlow;
- não criar Windows Service, Task Scheduler, daemon ou watchdog para limpeza;
- falha de cleanup não altera retroativamente save/preview/print já concluído;
- temporários/exportações não entram em SQLite, histórico ou backup por padrão.

## 22. Estados e acessibilidade

Estados mínimos incluem preparando ficha, preparando prévia, pronta, desatualizada, cancelada pelo usuário, sem permissão, fonte indisponível, `SHEET_OVERFLOW`, falha de renderer/preview, Host indisponível e `SERVER_BUSY`.

Para save/temporários, também distinguir falha de gravação, conflito de nome conduzido pelo diálogo do sistema e falha de cleanup não bloqueante.

- não exibir percentual fictício;
- ações operáveis por teclado;
- icon-only somente quando inequívoco, com nome acessível;
- foco gerenciado no modal;
- status/cancelamento/desatualização não dependem apenas de cor;
- folha permanece A4 independentemente da janela.

## 23. Pendência restante do Bloco 10

- Etapa 11 — validação técnica final/matriz/limites de recursos.

## 24. Fora do escopo inicial

- assinatura digital;
- envio por e-mail;
- nuvem/compartilhamento externo;
- exportação em lote/ZIP;
- editor visual de templates;
- armazenamento permanente de toda exportação;
- DOCX da Ficha;
- implementação funcional nesta fase.

## 25. Decisões consolidadas pelo PO

1. exportação/impressão permanece contextual;
2. Procedimento exportado usa revisão exata e documento completo;
3. PDF, DOCX e impressão de Procedimento são documentos próprios;
4. Ficha é prestação de contas resumida ao cliente;
5. Ficha usa estado confirmado/histórico aplicável;
6. Ficha pode existir sem Equipamento;
7. dados do Equipamento vêm do cadastro/snapshot existente;
8. observações de serviço por Etapa podem entrar na Ficha;
9. checklist/progresso/revisões detalhadas não poluem a Ficha por padrão;
10. Ficha válida possui exatamente uma A4 e overflow falha explicitamente;
11. Ficha possui PDF canônico próprio e preview SVG do mesmo PagedDocument;
12. template físico usa A4 retrato, margens 15 mm, composição vertical e baixa densidade;
13. cabeçalho é compacto, Equipamento é ficha técnica sem grade e `SERVIÇO REALIZADO` é a área narrativa principal;
14. `OBSERVAÇÕES` unifica observações relevantes sem subcards;
15. Noto Sans usa escala 14 / 10,5 / 10 / 9 / 8,5 pt conforme hierarquia;
16. Salvar/Imprimir reutilizam o PDF correspondente à prévia;
17. impressão usa o fluxo Windows consolidado;
18. soft limits são orientação: Resumo 600, Atendimento 400, Equipamento 300 e observação por Etapa 280 caracteres;
19. contador/aviso aparece apenas próximo de aproximadamente 80% da faixa recomendada;
20. `SHEET_OVERFLOW` bloqueia somente a geração da Ficha e preserva integralmente o Atendimento;
21. correção do overflow ocorre nos dados reais, guiada por diagnóstico semântico do Host, sem editor paralelo;
22. não há IA/resumo automático, truncamento, deduplicação semântica ou compactação automática para caber;
23. Procedimentos vinculados não são listados na Ficha por padrão, independentemente da quantidade;
24. MACs usam projeção determinística: 0 omite, 1–2 exibem valores, 3+ exibem apenas a quantidade cadastrada;
25. não existe `MAC principal` por inferência;
26. observações legítimas não sofrem cap/descarte automático e multiplicidade pode causar `SHEET_OVERFLOW`;
27. campos estruturados longos quebram linha quando possível, sem truncamento/reticências/abreviação inventada;
28. não existem `include_in_sheet`, `sheet_priority`, seleção transitória ou editor paralelo para resolver multiplicidade;
29. arquivo persistente escolhido pelo usuário é separado do lifecycle de temporários;
30. Procedimento sugere filename com código, título, versão editorial quando houver e revisão técnica; Ficha sugere `{service_code} - Ficha.pdf`;
31. sanitização de filename segue Windows e não altera conteúdo documental;
32. conflito de nome não gera overwrite silencioso;
33. temporário só é materializado no Client quando integração local precisa de filesystem;
34. temporários usam raiz do sistema por usuário, isolamento por instância e nomes opacos sem dados de domínio;
35. cleanup e scavenging são best-effort e restritos ao namespace StepFlow, sem serviço/daemon/tarefa agendada;
36. save só é sucesso após gravação integral e promoção segura é preferida quando suportada;
37. temporários/exportações não entram em SQLite, histórico ou backup por padrão;
38. DOCX da Ficha não é requisito inicial.
