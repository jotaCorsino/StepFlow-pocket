# Bloco 10 — Etapa 5 — Template físico de Procedimentos — Proposta para análise

**Status:** EM ANÁLISE / PROPOSTA PARA APROVAÇÃO DO PO  
**Fase:** Fase 1 — Fechamento arquitetural e especificação  
**Abertura:** 2026-08-27  
**Base consolidada:** Bloco 10 / Etapas 1–4

## 1. Objetivo

Definir o **template físico de Procedimentos** usado pelos renderers PDF e DOCX já consolidados, fechando:

- papel e orientação;
- margens;
- hierarquia tipográfica;
- cabeçalho e rodapé;
- composição da primeira página;
- sumário documental;
- composição das Etapas;
- espaçamentos;
- paginação e regras de quebra;
- aparência dos blocos semânticos;
- política visual de imagens/logo;
- estratégia tipográfica específica de PDF e DOCX.

Esta etapa não altera:

- conteúdo/semântica do `DocumentModel`;
- engine PDF Typst da Etapa 2;
- engine DOCX/OOXML da Etapa 3;
- mecanismo de impressão Windows da Etapa 4;
- nomes de arquivo ou temporários da Etapa 10;
- limites numéricos de recurso da Etapa 12;
- template físico da Ficha compacta, reservado à Etapa 7.

## 2. Princípio de design

O Procedimento físico deve parecer um **manual técnico compacto e profissional**, não uma captura do Reader e não um relatório burocrático.

Direção:

```text
identidade clara
+ hierarquia forte
+ leitura confortável
+ blocos técnicos inequívocos
+ uso eficiente da folha
+ comportamento previsível em impressão
```

Consequências:

- não criar capa exclusiva apenas para título/logo;
- não reproduzir sidebar, botões, ícones, chips interativos ou controles do Client;
- não forçar uma página física nova para cada Etapa;
- não usar decoração sem função documental;
- não depender de cor para transmitir significado;
- manter boa leitura também em impressão preto e branco.

## 3. Formato físico baseline

Procedimentos usam:

```text
papel: A4
orientação: retrato
largura: 210 mm
altura: 297 mm
```

Primeira versão não usa automaticamente:

- Letter;
- Legal;
- orientação paisagem;
- páginas de tamanho misto;
- sangria/full bleed.

Conteúdo largo deve se adaptar à área útil dentro das regras desta etapa. Não alterar orientação de páginas silenciosamente para acomodar um bloco excepcional.

## 4. Margens

Baseline proposto:

```text
topo:     20 mm
inferior: 20 mm
esquerda: 18 mm
direita:  18 mm
```

Objetivos:

- preservar área útil adequada para texto técnico e comandos;
- manter margem segura para impressoras corporativas comuns;
- reservar espaço limpo para header/footer;
- evitar aspecto de relatório excessivamente espaçado.

Não existe gutter/binding extra na primeira versão.

## 5. Sistema tipográfico

### PDF

Famílias canônicas propostas:

```text
texto/interface documental: Noto Sans
comando/código:             Noto Sans Mono
```

As fontes usadas pelo PDF permanecem empacotadas com o Host e incorporadas/subsetadas no artefato, conforme Etapa 2.

Noto é licenciado sob Open Font License e permite redistribuição/bundle, sendo adequado ao runtime autocontido do StepFlow.

### DOCX

Para maximizar compatibilidade em Windows/Word sem exigir instalação de fonte adicional no computador que abrirá o arquivo:

```text
texto documental: Arial
comando/código:    Consolas
```

A primeira versão do DOCX **não exige embedding de fontes**.

Isso é uma decisão deliberada e não uma herança do PDF:

- PDF prioriza estabilidade visual absoluta e usa fontes empacotadas;
- DOCX prioriza editabilidade/compatibilidade no ambiente Windows e continua refluível;
- PDF e DOCX preservam a mesma hierarquia, escala e semântica visual, mas não prometem métricas/paginação idênticas.

Se a matriz corporativa futura exigir font embedding no DOCX, isso pode ser reavaliado no gate técnico sem alterar o contrato semântico.

## 6. Escala tipográfica

Baseline:

| Uso | Tamanho | Peso/ênfase |
|---|---:|---|
| Título do Procedimento | 18 pt | semibold/bold |
| Código/contexto acima do título | 9 pt | semibold |
| Título `Visão geral` | 14 pt | semibold |
| Título de Etapa | 14 pt | semibold |
| Subtítulo estrutural | 11 pt | semibold |
| Corpo | 10 pt | regular |
| Metadados/labels | 8.5–9 pt | regular/semibold |
| Nota/alerta | 9.5 pt | regular |
| Comando/código | 8.5 pt | regular/semibold no label |
| Header/footer | 8 pt | regular |

Regras:

- corpo nunca é reduzido dinamicamente para fazer conteúdo caber;
- comando/código não é reduzido abaixo de 8.5 pt como workaround de overflow;
- não usar primeira linha recuada em parágrafos;
- texto normal usa alinhamento à esquerda, não justificação completa;
- hifenização automática não é requisito inicial;
- preservar palavras/identificadores técnicos sempre que possível.

## 7. Ritmo e espaçamento

Baseline conceitual:

- corpo com entrelinha aproximada de `1.30–1.35`;
- código/comando com entrelinha aproximada de `1.20–1.25`;
- 4–6 pt após parágrafo normal;
- 10–14 pt antes de novo título/subtítulo;
- 6–8 pt entre título e primeiro bloco relacionado;
- blocos de nota/alerta/comando/código usam inset interno suficiente para separar texto da borda;
- evitar linhas horizontais repetitivas entre todos os parágrafos.

A implementação pode converter esses valores às unidades nativas de Typst/OOXML sem mudar a intenção visual.

## 8. Paleta documental

O template é neutro e independente de uma cor corporativa que ainda não existe como campo consolidado da identidade da empresa.

Tokens visuais propostos:

```text
ink:       #111827
muted:     #6B7280
line:      #D1D5DB
surface:   #F3F4F6
paper:     #FFFFFF
```

Regras:

- texto principal sempre possui contraste alto;
- significado não depende apenas de cor;
- nota/alerta possuem rótulo textual explícito;
- o documento permanece inteligível em escala de cinza;
- logo preserva suas cores originais quando disponível.

Não aplicar automaticamente cores extraídas do logo.

## 9. Primeira página — sem capa exclusiva

A primeira página começa diretamente pelo conteúdo útil.

Composição conceitual:

```text
┌──────────────────────────────────────────────────────────┐
│ [logo]  Nome da Empresa                                 │
│         contato · site · e-mail                         │
│                                                          │
│ PR-014                                                   │
│ CONFIGURAÇÃO DE VLAN                                     │
│ Versão 2.0 · revisão r18 · Publicada                    │
│                                                          │
│ Área / Departamento        Responsável                   │
│ Categorias                 ...                           │
│                                                          │
│ VISÃO GERAL                                               │
│ Objetivo...                                               │
│ Pré-requisitos...                                         │
│ Observações...                                            │
│                                                          │
│ SUMÁRIO DE ETAPAS                                         │
│ 01. Preparação                                            │
│ 02. Configuração                                          │
│ 03. Validação                                             │
└──────────────────────────────────────────────────────────┘
```

Não reservar uma folha inteira apenas para título/logo.

Se a primeira página não comportar todos os itens de `Visão geral` e o sumário, o fluxo continua normalmente para a página seguinte, sem compressão artificial.

## 10. Identidade da empresa

Na primeira página:

- logo aparece quando configurado;
- nome da empresa aparece quando disponível;
- contato/site/e-mail opcionais são omitidos quando vazios;
- logo preserva proporção;
- não mostrar placeholder ou espaço vazio artificial na ausência de logo.

Logo baseline:

- largura visual máxima aproximada: 28 mm;
- altura visual máxima aproximada: 12 mm;
- sem crop;
- sem deformação;
- nunca usado como watermark de fundo.

A identidade institucional fica no corpo da primeira página, não apenas em header/footer, para preservar significado fora de elementos repetitivos.

## 11. Cabeçalho das páginas seguintes

A primeira página não precisa repetir um cabeçalho institucional acima do bloco de identidade.

Da página 2 em diante, header compacto:

```text
Nome da Empresa                         PR-014 · v2.0 · r18
───────────────────────────────────────────────────────────
```

Regras:

- não repetir título completo se isso comprometer espaço/legibilidade;
- código + versão/revisão são suficientes como identificação curta;
- contexto editorial especial (`REVISÃO HISTÓRICA`, `NÃO PUBLICADA`, `ARQUIVADA`, quando aplicável) deve permanecer identificável também nas páginas internas;
- esse contexto também aparece no corpo da primeira página, nunca somente no header.

Não inserir dados críticos exclusivos em header/footer.

## 12. Rodapé e paginação

Rodapé baseline:

```text
                                              Página X de N
```

Regras:

- numeração aparece em todas as páginas, inclusive a primeira;
- formato textual `Página X de N`;
- não incluir timestamp visível de geração por padrão;
- não incluir hostname, usuário, path, token, versão do Windows ou dados técnicos internos;
- `generation_metadata` pode continuar existindo como metadado técnico conforme contratos anteriores, sem obrigatoriamente aparecer no layout.

## 13. Metadados do Procedimento

A primeira página apresenta metadados em estrutura compacta, preferencialmente em duas colunas quando o conteúdo permitir.

Elementos já consolidados podem incluir, conforme disponibilidade:

- código;
- título;
- Área/Departamento;
- responsável;
- categorias;
- versão editorial;
- revisão técnica;
- estado editorial necessário para evitar ambiguidade.

Regras:

- labels menores e discretos;
- valores com prioridade visual sobre labels;
- conteúdo longo quebra em linhas, sem truncamento;
- categorias são texto/lista compacta, não chips de UI;
- campos vazios não reservam células vazias artificiais.

A Etapa 5 não acrescenta novos campos ao `DocumentModel`.

## 14. Visão geral

`Visão geral` é a primeira seção documental, coerente com o Reader.

Pode conter os campos já presentes no modelo, como:

- objetivo;
- pré-requisitos;
- observações gerais;
- demais conteúdo consolidado para essa seção.

Não criar conteúdo editorial novo por inferência.

## 15. Sumário documental

Baseline: **Sumário de etapas compacto**.

Regras:

- aparece quando houver duas ou mais Etapas;
- contém número + título das Etapas na ordem oficial;
- não replica todos os passos/subpassos;
- primeira versão não exige números de página no sumário;
- não depende de atualização de campos pelo Microsoft Word;
- pode quebrar em mais de uma página em Procedimentos excepcionalmente longos;
- sumário não é uma segunda navegação interativa nem contém links obrigatórios.

Razão para não exigir página no sumário: DOCX é refluível e seus números de página podem mudar conforme consumidor/fontes/ambiente. O sumário sem paginação permanece correto nos dois formatos.

## 16. Etapas no documento físico

Cada Etapa usa título:

```text
ETAPA 01 · Preparação
```

ou equivalente tipograficamente coerente.

A numeração visual usa pelo menos dois dígitos enquanto estiver dentro do padrão atual (`01`, `02`, `03`...).

**Não forçar page break antes de cada Etapa.**

O documento físico usa fluxo contínuo para evitar páginas parcialmente vazias. A separação “cada Etapa como uma página do livro” permanece uma decisão da experiência do Reader, não uma obrigação de desperdiçar uma folha física por Etapa.

O título da Etapa deve permanecer junto ao primeiro bloco de conteúdo sempre que tecnicamente possível.

## 17. Parágrafos

Parágrafo normal:

- corpo 10 pt;
- alinhado à esquerda;
- sem indentação de primeira linha;
- espaçamento posterior consistente;
- não usar caixas/bordas decorativas;
- preservar quebras intencionais existentes no modelo quando semanticamente relevantes.

Widows/orphans devem ser evitados conforme capacidade do renderer.

## 18. Passos e subpassos

Visual proposto:

```text
1. Passo principal
   texto complementar quando houver...

   1.1. Subpasso
   1.2. Subpasso

2. Próximo passo
```

Regras:

- numeração estrutural clara;
- hanging indent/alinhamento consistente;
- número não fica isolado em linha separada do texto;
- DOCX usa numeração/lista Word real conforme Etapa 3;
- não colocar cada passo dentro de um card/borda;
- um passo curto deve permanecer junto do início do seu conteúdo;
- subpassos podem continuar na página seguinte quando o conjunto for grande.

## 19. Checklist documental

Visual baseline:

```text
□ Verificar conectividade
□ Registrar configuração
□ Validar acesso
```

Regras:

- caixa é representação documental estável, não controle interativo;
- texto usa corpo normal ou ligeiramente compacto;
- cada item evita quebra interna de página quando razoável;
- não incluir estado operacional de Atendimento no documento de Procedimento standalone;
- o símbolo deve continuar legível em preto e branco.

## 20. Nota

Visual:

```text
┌ NOTA ────────────────────────────────────────────────────┐
│ Texto da observação...                                  │
└──────────────────────────────────────────────────────────┘
```

Tratamento:

- fundo `surface` leve;
- borda fina `line`;
- label `NOTA` explícito;
- sem depender de ícone ou cor azul;
- texto selecionável/editável conforme formato.

## 21. Alerta

Visual:

```text
┌ ATENÇÃO ─────────────────────────────────────────────────┐
│ Texto do alerta...                                      │
└──────────────────────────────────────────────────────────┘
```

Tratamento:

- label textual forte `ATENÇÃO`;
- contraste/borda mais forte que `NOTA`;
- fundo ainda adequado para impressão em escala de cinza;
- não depender exclusivamente de vermelho/amarelo;
- alerta não pode se confundir semanticamente com nota comum.

## 22. Comando

Visual baseline:

```text
COMANDO
┌──────────────────────────────────────────────────────────┐
│ ipconfig /all                                            │
└──────────────────────────────────────────────────────────┘
```

Regras:

- fonte monoespaçada;
- fundo neutro leve;
- texto preservado como texto;
- whitespace relevante preservado;
- não imprimir botão/ícone de copiar;
- comando curto permanece inteiro quando couber na área disponível.

## 23. Bloco de código

Visual semelhante ao comando, com label `CÓDIGO` quando isso ajudar a distinção.

Regras:

- monoespaçado 8.5 pt;
- preservar espaços, tabs, indentação e quebras de linha do modelo;
- não inserir line numbers por inferência;
- não rasterizar;
- não truncar;
- bloco longo pode continuar em página seguinte **entre linhas existentes**;
- uma linha individual não deve ser cortada horizontalmente.

A primeira versão não reduz fonte dinamicamente nem muda a página para paisagem por causa de uma linha excepcional.

Se uma única sequência ininterrupta for fisicamente impossível de representar dentro da largura útil sem alterar o conteúdo, a geração deve falhar explicitamente por conteúdo incompatível em vez de truncar ou modificar silenciosamente o comando/código. O gate da Etapa 12 deve testar esse limite com conteúdo real.

## 24. Quebras e integridade visual

Regras gerais:

- título de Etapa não fica órfão no final da página;
- subtítulo permanece junto do primeiro conteúdo relacionado;
- evitar uma única linha de parágrafo isolada no topo/fundo da página;
- checklist curto evita dividir um item ao meio;
- comando de uma/few linhas permanece inteiro quando couber;
- imagem não é dividida entre páginas;
- bloco de código grande pode quebrar entre linhas;
- nota/alerta curto deve preferir permanência integral;
- nota/alerta muito longo pode quebrar para não criar grandes vazios nem overflow;
- conteúdo nunca é truncado para satisfazer regra de `keep`.

Typst pode usar `sticky`/`breakable` e sua prevenção padrão de widow/orphan. DOCX usa os equivalentes de `keep_next`, `keep_lines`, `widow_control` e page-break/section quando necessário.

## 25. Imagens dentro do Procedimento

Imagens autorizadas pelo modelo:

- preservam proporção;
- nunca são cropadas automaticamente;
- largura máxima = área útil de conteúdo;
- altura máxima deve preservar espaço de leitura e não ultrapassar a área útil da página;
- imagem grande é escalada proporcionalmente para caber;
- imagem atômica vai para a página seguinte quando não houver espaço suficiente;
- não criar legenda automática se o domínio não fornecer legenda;
- não buscar recurso remoto.

Raster de resolução insuficiente não deve ser artificialmente ampliado de forma que prejudique severamente a leitura. O limiar técnico de DPI/px pertence ao gate de implementação/Etapa 12.

## 26. PDF versus DOCX

O template possui **uma linguagem visual comum**, mas dois comportamentos honestos:

### PDF

- paginação canônica;
- fontes Noto empacotadas/incorporadas;
- layout visual estável;
- referência para impressão da Etapa 4.

### DOCX

- mesma hierarquia e composição conceitual;
- Arial/Consolas para compatibilidade Windows;
- reflow permitido;
- page breaks podem variar por consumidor;
- não exigir identidade de página com PDF;
- continua totalmente editável.

Não adicionar hacks no DOCX para imitar cada quebra física do PDF.

## 27. Elementos deliberadamente ausentes

Não entram no template inicial por inferência:

- capa exclusiva;
- watermark decorativo;
- QR/barcode — Etapa 11;
- assinatura eletrônica;
- campo de aprovação manual;
- classificação/confidencialidade não existente no domínio;
- linha de assinatura;
- histórico completo de alterações, se não fizer parte do conteúdo documental já consolidado;
- timestamp visível obrigatório;
- URL de download;
- caminho local;
- nome do usuário que exportou;
- elementos do Reader/Client.

## 28. Acessibilidade e semântica

- informação essencial permanece no corpo e não apenas em header/footer;
- heading real é usado onde o renderer/formato suporta semântica de heading;
- ordem visual segue ordem lógica;
- rótulos `NOTA` e `ATENÇÃO` tornam os blocos distinguíveis sem cor;
- texto continua texto;
- Tagged PDF da Etapa 2 permanece habilitado;
- esta etapa não passa a prometer conformidade formal PDF/UA.

## 29. Determinismo

Com mesma versão do Host/template, mesmo `DocumentModel`, mesmos assets e mesmos recursos tipográficos:

- PDF deve manter paginação/layout visual estáveis;
- DOCX deve manter estilos/estrutura estáveis, sem exigir mesma paginação em diferentes consumidores;
- não usar horário do sistema para alterar layout;
- não variar template conforme impressora escolhida;
- não variar margens conforme estação Client.

## 30. Mapeamento técnico validado

A direção proposta cabe nas capacidades atuais dos renderers:

### Typst

- `page` controla papel, margens, header/footer e numeração;
- paginação é automática;
- headings são estruturalmente próprios para outline/semântica;
- blocks suportam `breakable` e `sticky`;
- prevenção de widow/orphan existe por padrão.

### DOCX / docx-rs

- seção/documento suporta page size e page margins;
- suporta header/footer e paginação;
- paragraph suporta `keep_next`, `keep_lines`, `widow_control` e `page_break_before`;
- estilos internos permanecem a fronteira do template.

A implementação concreta continua sujeita à versão fixada das crates e à matriz corporativa da Etapa 12.

## 31. Decisões propostas ao PO — Etapa 5

1. Procedimento físico usa **A4 retrato** na primeira versão;
2. margens baseline: **20 mm topo/inferior e 18 mm esquerda/direita**;
3. não existe capa exclusiva;
4. primeira página combina identidade da empresa, título/metadados, `Visão geral` e início do sumário/conteúdo;
5. PDF usa **Noto Sans + Noto Sans Mono** empacotadas/incorporadas;
6. DOCX usa **Arial + Consolas**, sem font embedding obrigatório v1;
7. escala tipográfica: título 18 pt, Etapa 14 pt, corpo 10 pt, código 8.5 pt e metadados/footer compactos;
8. corpo é alinhado à esquerda, sem justificação total e sem redução dinâmica para caber;
9. paleta é neutra/grayscale-safe; cor corporativa não é inferida do logo;
10. primeira página contém identidade; páginas seguintes usam header compacto com empresa + código/versão/revisão;
11. footer usa `Página X de N` e não exibe timestamp técnico por padrão;
12. sumário aparece com duas ou mais Etapas e lista Etapas **sem números de página** no baseline cross-format;
13. uma Etapa **não força nova página física**; headings usam keep/sticky com o primeiro conteúdo;
14. passos usam hierarquia numerada clara; checklist é documental;
15. Nota, Alerta, Comando e Código têm estilos distintos e funcionais, sem decoração excessiva;
16. código/comando preserva texto/whitespace; bloco longo quebra entre linhas existentes, sem truncamento/rasterização/font shrink;
17. sequência única fisicamente impossível de representar gera erro explícito em vez de alteração silenciosa;
18. imagens preservam proporção, não são cropadas e se ajustam à área útil;
19. widow/orphan e keeps são aplicados sem provocar truncamento;
20. PDF é a referência de paginação/impressão; DOCX preserva linguagem visual e editabilidade sem prometer page parity;
21. nenhuma configuração de impressora/estação altera o template;
22. QR/barcode, ficha compacta, temporários e limites técnicos permanecem em suas Etapas próprias.

## 32. Referências técnicas consultadas

- Typst — Page Setup Guide: `https://typst.app/docs/guides/page-setup/`
- Typst — `page`: `https://typst.app/docs/reference/layout/page/`
- Typst — `block`: `https://typst.app/docs/reference/layout/block/`
- Typst — `pagebreak`: `https://typst.app/docs/reference/layout/pagebreak/`
- Typst — `heading`: `https://typst.app/docs/reference/model/heading/`
- Typst — `text` / widow-orphan costs: `https://typst.app/docs/reference/text/text/`
- `docx-rs` 0.4.22 — `Section`: `https://docs.rs/docx-rs/latest/docx_rs/struct.Section.html`
- `docx-rs` 0.4.22 — `Paragraph`: `https://docs.rs/docx-rs/latest/docx_rs/struct.Paragraph.html`
- `docx-rs` 0.4.22 — `Docx`: `https://docs.rs/docx-rs/latest/docx_rs/struct.Docx.html`
- Noto documentation — use/licensing: `https://notofonts.github.io/noto-docs/website/use/`

## 33. Critério de fechamento

Esta proposta só deve ser promovida para `bloco-10-exportacao-impressao-ficha.md`, `AGENTS.md`, README, arquitetura, registro, índice e plano oficial após **aprovação explícita do PO**.

Até lá:

- Etapas 1–4 permanecem consolidadas;
- Etapa 5 permanece em análise apenas nesta branch/PR;
- Etapa 6 continua pendente e não deve ser aberta;
- fontes canônicas não mudam de estado;
- nenhum código funcional, dependency, fonte binária, scaffold ou migration é autorizado por este documento.
