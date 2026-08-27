# Bloco 10 — Exportação / Impressão + Ficha Compacta

**Status:** EM ANDAMENTO — ETAPAS 1–5 CONSOLIDADAS / ETAPA 6 PRÓXIMA  
**Fase:** Fase 1 — Fechamento arquitetural e especificação  
**Abertura:** 2026-08-25  
**Etapa 1 consolidada:** 2026-08-25  
**Etapa 2 consolidada:** 2026-08-26  
**Etapa 3 consolidada:** 2026-08-26  
**Etapa 4 consolidada:** 2026-08-26  
**Etapa 5 consolidada:** 2026-08-27

## 1. Objetivo do bloco

Fechar, uma etapa por vez, o contrato técnico de geração documental, exportação, impressão e Ficha compacta do StepFlow, preservando o caráter Pocket e a UX aprovada.

Este arquivo funciona como **mapa técnico consolidado do Bloco 10**. Detalhes transversais que já possuem fonte canônica própria permanecem em:

- `../../AGENTS.md` — regras operacionais superiores;
- `../03-arquitetura/arquitetura-vigente.md` — arquitetura técnica vigente;
- `../05-progresso/registro-de-decisoes.md` — decisões vigentes e pendências;
- `../02-telas/05-leitor-processo.md` — contrato do Reader;
- `../02-telas/14-exportacao-impressao-ficha.md` — UX de exportação/impressão/ficha.

O objetivo deste documento não é duplicar integralmente essas fontes, mas registrar os contratos específicos do Bloco 10 e a relação entre suas etapas.

Não pertence a este bloco implementar código de produção, fechar Backup/Restore técnico ou abrir o Bloco 11.

## 2. Etapas do Bloco 10

| Ordem | Etapa | Estado |
|---|---|---|
| 1 | Arquitetura de geração documental | **CONSOLIDADO / APROVADO PELO PO** |
| 2 | PDF de Procedimentos | **CONSOLIDADO / APROVADO PELO PO** |
| 3 | DOCX de Procedimentos | **CONSOLIDADO / APROVADO PELO PO** |
| 4 | Impressão Windows de Procedimentos | **CONSOLIDADO / APROVADO PELO PO** |
| 5 | Template físico de Procedimentos | **CONSOLIDADO / APROVADO PELO PO** |
| 6 | PDF + preview da Ficha compacta | **PRÓXIMA — AINDA NÃO EM ANÁLISE** |
| 7 | Template físico A4 da Ficha | PENDENTE |
| 8 | Limites textuais e densidade da Ficha | PENDENTE |
| 9 | Múltiplos MACs / Procedimentos na Ficha | PENDENTE |
| 10 | Nomes de arquivo + artefatos temporários | PENDENTE |
| 11 | QR / barcode | PENDENTE |
| 12 | Validação técnica final do Bloco 10 | PENDENTE |

As Etapas 6–12 permanecem fora de análise neste checkpoint. A Etapa 6 está apenas marcada como próxima.

---

# Etapa 1 — Arquitetura de geração documental

**Status:** CONSOLIDADO / APROVADO PELO PO

## 3. Fronteira arquitetural

A geração documental pertence ao **Host**. O Client permanece responsável pela experiência local e pelo destino do artefato.

```text
Client
  ↓ solicita por identidade da fonte + revisão esperada
Host
  ↓ autentica/autoriza
  ↓ valida revisão/versão esperada
  ↓ captura snapshot consistente
  ↓ materializa DocumentModel semântico
  ↓ encerra leitura/transação SQLite
  ↓ renderiza fora da fila de mutações
  ↓ devolve artefato pela API autenticada
Client
  ↓ recebe
  ├─→ salvar localmente
  └─→ preview/impressão conforme contrato específico
```

Consequências:

- Client não envia documento montado;
- DOM/HTML/CSS da UI não são fonte documental;
- Host não grava em path arbitrário da workstation;
- renderers não reconsultam SQLite nem reconstroem regras de negócio;
- fonte mutável usa revisão esperada para impedir substituição silenciosa por estado mais novo;
- artefato é derivado e não cria mutação funcional.

## 4. Captura consistente

A geração possui duas fases:

```text
captura consistente
→ autorização + leitura da fonte
→ snapshots/revisões/identidade corporativa
→ DocumentModel imutável em memória
→ encerra leitura/transação

renderização
→ usa apenas DocumentModel + assets controlados
→ produz o artefato
```

Nenhuma transação SQLite permanece aberta durante o trabalho pesado de renderização.

## 5. DocumentModel

Fronteira conceitual:

```text
DocumentModel
├── document_kind
├── source_identity
├── source_version
├── company_identity
├── metadata
├── sections[]
│   └── semantic_blocks[]
└── generation_metadata
```

Não contém:

- HTML/JavaScript arbitrário;
- DOM/classes CSS da aplicação;
- comandos executáveis;
- token de sessão;
- path escolhido pelo usuário;
- estado transitório do Client.

## 6. Concorrência e persistência

Geração é leitura derivada:

- fica fora da fila de mutações do writer;
- usa limite próprio bounded de concorrência/backpressure;
- não cria `export_jobs` persistentes na primeira versão;
- não cria scheduler documental;
- não mantém fila offline;
- não altera revisão, Atendimento, checklist ou `updated_at` funcional;
- não cria histórico de exportações por padrão;
- artefato não entra automaticamente em backup.

Fluxo inicial:

```text
request autenticado
→ captura
→ renderização
→ resposta
```

## 7. Runtime documental

O runtime normal não depende operacionalmente de:

- Microsoft Office/COM;
- LibreOffice;
- Adobe Reader;
- Chrome/Chromium externo headless;
- `wkhtmltopdf`;
- serviço cloud de conversão.

Bibliotecas compiladas/empacotadas com o Host podem ser usadas.

---

# Etapa 2 — PDF de Procedimentos

**Status:** CONSOLIDADO / APROVADO PELO PO

## 8. Renderer PDF

O PDF de Procedimentos usa **Typst embutido como biblioteca Rust no Host**, por crates oficiais do ecossistema Typst e adaptador interno StepFlow.

Não executar:

- `typst.exe`/CLI;
- browser;
- processo conversor externo.

A versão exata das crates será fixada no gate de implementação/Cargo.lock, não por suposição durante a Fase 1.

## 9. Template e segurança

- template Typst é interno, confiável e versionado;
- conteúdo do domínio entra apenas como dados/valores estruturados;
- conteúdo do usuário nunca participa da construção textual do source Typst;
- nenhum pacote/recurso remoto é resolvido em runtime;
- `World`/filesystem/imports ficam restritos a templates, fontes e assets controlados;
- nenhum path/URL arbitrário originado do Procedimento é fornecido ao renderer.

## 10. Contrato do PDF

Baseline:

- MIME `application/pdf`;
- PDF 1.7 solicitado explicitamente;
- Tagged PDF habilitado como baseline;
- Tagged PDF não implica promessa formal PDF/UA ou PDF/A;
- texto real selecionável/pesquisável/copiável;
- Unicode adequado para português e caracteres técnicos;
- fontes incorporadas/subsetadas;
- documento multipágina com quebra automática;
- imagens/logo controlados incorporáveis;
- falha do renderer nunca retorna artefato parcial como sucesso.

Blocos semânticos obrigatórios:

- parágrafo;
- passos/subpassos numerados;
- checklist documental;
- nota;
- alerta;
- comando;
- código.

Tipo conhecido incompatível/desconhecido para o renderer falha explicitamente; não há descarte silencioso.

Comando/código permanece texto e preserva whitespace relevante.

## 11. Assets e determinismo

PDF suporta assets controlados PNG/JPEG/SVG quando autorizados pela política do produto.

Conteúdo visual não depende implicitamente de:

- relógio da máquina;
- locale ambiental;
- fonte do sistema;
- recurso remoto.

Data/hora visível deve vir de `DocumentModel`/`generation_metadata` explícito.

Sob mesma versão do Host/template/fontes/assets/modelo, exige-se estabilidade visual/semântica; não é necessário byte-a-byte idêntico quando metadados técnicos legítimos variarem.

## 12. Recursos fora da v1

Não são requisitos iniciais:

- assinatura digital;
- criptografia/senha;
- formulários;
- anexos;
- JavaScript;
- multimídia;
- PDF/A formal;
- PDF/UA formal.

O layout físico final do PDF segue a Etapa 5 consolidada abaixo.

---

# Etapa 3 — DOCX de Procedimentos

**Status:** CONSOLIDADO / APROVADO PELO PO

## 13. Formato e renderer

DOCX é gerado diretamente pelo Host Rust a partir do mesmo `DocumentModel`, sem converter PDF/Typst.

Baseline:

- `.docx` real em OOXML/WordprocessingML/OPC;
- OOXML **Transitional** como compatibilidade inicial;
- `docx-rs` como biblioteca Rust preferida sob adaptador interno StepFlow;
- MIME oficial de WordprocessingML.

Não depende de:

- Word/COM;
- LibreOffice;
- browser/headless;
- CLI conversor;
- cloud;
- pipeline PDF → DOCX.

## 14. Segurança e template DOCX

- conteúdo do domínio entra somente como dados estruturados;
- usuário não injeta XML/OOXML, relationships, partes OPC, paths ou URLs arbitrários;
- estilos/template são internos e versionados pelo StepFlow;
- nenhum `.docx`/`.dotx` fornecido pelo usuário é carregado como template de runtime v1;
- nenhuma imagem/recurso remoto é baixado durante geração.

## 15. Editabilidade

DOCX existe como formato refluível/editável.

Portanto:

- títulos/parágrafos/passos/notas permanecem texto Word real;
- comandos/código permanecem texto;
- conteúdo é selecionável, pesquisável, copiável e editável;
- imagens permanecem objetos incorporados;
- nenhuma Etapa é convertida em screenshot para imitar PDF;
- edição externa do DOCX não altera a revisão oficial do StepFlow;
- não há import/sync de DOCX editado na v1.

## 16. Blocos e imagens

Todos os blocos semânticos conhecidos devem ser representados sem descarte silencioso.

- passos/subpassos usam numeração/lista Word real quando aplicável;
- checklist é documental, não formulário interativo;
- comando/código preserva espaços, tabs, quebras e indentação relevantes;
- PNG/JPEG são baseline de imagem;
- SVG não é requisito direto do DOCX v1 e exige representação interna compatível ou falha explícita — nunca omissão silenciosa.

## 17. Reflow e fontes

DOCX não promete paginação idêntica ao PDF nem entre diferentes consumidores Word.

A Etapa 5 consolidou a política tipográfica v1:

```text
texto:          Arial
comando/código: Consolas
embedding:      não
```

O StepFlow referencia essas famílias no DOCX, sem redistribuir/embutir os arquivos de fonte na v1. Compatibilidade real é validada no gate técnico.

## 18. Conteúdo ativo fora da v1

Não são requisitos:

- macros/VBA/`.docm`;
- ActiveX;
- OLE;
- remote templates;
- external relationships de conteúdo;
- anexos;
- assinatura digital;
- senha/DRM;
- formulário/content control interativo;
- importação de DOCX editado.

Artefato incompleto/corrompido nunca é tratado como sucesso. Validação técnica posterior cobre ZIP/OPC, XML/relationships e abertura sem reparo na matriz corporativa.

---

# Etapa 4 — Impressão Windows de Procedimentos

**Status:** CONSOLIDADO / APROVADO PELO PO

## 19. Local da impressão

A impressão física acontece no **Client Windows da estação do usuário**, não no Host central.

Motivos funcionais:

- impressoras pertencem ao contexto local do usuário/workstation;
- Windows/driver local controla disponibilidade e opções;
- múltiplos Clients podem imprimir em destinos diferentes;
- Host não precisa conhecer inventário de impressoras.

## 20. Artefato canônico

A impressão usa **o mesmo PDF oficial da Etapa 2** para a revisão exata selecionada.

Não existe:

- terceiro renderer de impressão;
- impressão de HTML do Reader;
- impressão via DOCX/Word;
- conversão local alternativa.

```text
Reader
→ Imprimir
→ Client solicita revisão esperada
→ Host autentica/autoriza
→ PdfRenderer oficial
→ bytes PDF
→ Client
→ recurso local transitório
→ WebView2 dedicada
→ diálogo Windows
```

## 21. WebView2 e diálogo Windows

O Client usa superfície WebView2 dedicada/transitória, separada da webview principal.

Baseline:

```text
Tauri WebviewWindow transitória
→ `with_webview`
→ CoreWebView2
→ ShowPrintUI(System)
→ diálogo de impressão do Windows
```

Regras:

- webview principal/estado do Reader permanecem intactos;
- a superfície recebe somente PDF local controlado;
- não busca Internet;
- não recebe token/senha em URL;
- não recebe path arbitrário do conteúdo;
- não vira janela funcional permanente;
- diálogo padrão é o diálogo do Windows;
- sem impressão silenciosa v1;
- sem seletor próprio de impressoras v1;
- StepFlow não enumera/persiste impressoras no Host;
- drivers/spooler/disponibilidade física pertencem ao Windows/ambiente corporativo.

## 22. Alternativas rejeitadas como baseline

Não usar silenciosamente:

- `ShellExecute`/handler `.pdf`;
- visualizador PDF externo;
- Word/COM;
- LibreOffice;
- browser externo;
- spool direto de PDF bruto;
- engine adicional de rasterização apenas para imprimir.

## 23. Recurso transitório

O PDF recebido vira recurso local privado e transitório de impressão.

Esta etapa não define mecanismo concreto, nome, diretório ou limpeza: isso permanece na **Etapa 10**.

O recurso não vira histórico, backup ou exportação persistente apenas por ser usado na impressão.

## 24. Verdade da UI

`ShowPrintUI(System)` abre/entrega o fluxo ao Windows, mas não fornece confirmação confiável de papel impresso versus cancelamento no diálogo.

Estados técnicos mínimos:

```text
preparando PDF
→ preparando impressão local
→ abrindo diálogo do Windows
→ fluxo entregue ao Windows
```

Regras:

- não mostrar `Impresso com sucesso` apenas porque o diálogo fechou;
- não persistir `printed=true` por inferência;
- cancelamento/fechamento do diálogo não é erro funcional;
- geração PDF, preparação local, compatibilidade WebView2 e abertura do diálogo são classes distintas de falha;
- duplicidade acidental da mesma ação é impedida localmente sem job/fila persistente.

## 25. Gate técnico posterior

Validar em Windows 10/11 x64 representativos:

- WebView2 compatível;
- PDF multipágina;
- Unicode/acentos;
- logo/imagens;
- `ShowPrintUI(System)`;
- impressoras locais/rede instaladas no Windows;
- opções do diálogo/driver;
- cancelamento;
- operação offline;
- retorno ao Reader sem perda de estado;
- ausência de dependência de visualizador externo.

Versão mínima concreta de WebView2 permanece para matriz corporativa/gate de implementação.

A impressão usa o template físico consolidado na Etapa 5 abaixo.

---

# Etapa 5 — Template físico de Procedimentos

**Status:** CONSOLIDADO / APROVADO PELO PO

## 26. Correção de escopo

A Etapa 5 define **somente o template físico do Procedimento exportado em PDF/DOCX**.

Ela não transforma o Reader em folha ou preview A4.

Três superfícies permanecem separadas:

```text
Reader do app
→ uso diário
→ páginas lógicas do manual
→ sem geometria A4

Procedimento exportado
→ PDF / DOCX / impressão
→ documento completo
→ multipágina

Ficha compacta de Atendimento
→ resumo do trabalho realizado
→ máximo 1 página A4
```

O limite rígido de uma A4 pertence à Ficha compacta, não ao Reader e não ao Procedimento completo.

## 27. Reader preservado

O Reader segue `../02-telas/05-leitor-processo.md`:

- `Visão geral` é a primeira página lógica, não uma Etapa numerada;
- cada `process_stage` corresponde a uma página lógica própria;
- Etapas não são fundidas para reduzir cliques;
- ao mudar de página, conteúdo começa no topo;
- conteúdo maior que viewport não é truncado nem mistura a próxima Etapa;
- `Anterior`, `Próxima`, Sumário e stepper navegam pelo mesmo conjunto de páginas.

### Stepper compacto

O stepper superior é horizontal, compacto e navegável.

Exemplo conceitual:

```text
●━━━━●━━━━◉────○────○────○────○
```

Usa prioritariamente:

- círculos;
- linhas;
- preenchimento;
- contraste/forma/símbolo;
- cor;
- interação por clique/teclado.

Estados:

- anteriores → visual de percorridas/concluídas na navegação;
- atual → destaque inequívoco;
- seguintes → estado neutro/futuro.

Regras:

- não repetir permanentemente nomes/rótulos por Etapa no stepper;
- nome completo permanece no título da página e no Sumário;
- tooltip/nome acessível pode complementar o marcador;
- stepper se adapta à largura disponível;
- estado anterior significa **percurso de navegação**, nunca conclusão operacional/checklist;
- `Etapa X de Y` permanece indicador textual compacto de posição.

## 28. Princípio transversal de baixa densidade

Direção aprovada para grande parte do Pocket:

```text
mostrar sempre
→ somente o necessário para entender e agir agora

mostrar visualmente
→ posição, progresso, estado e ações recorrentes

mostrar sob demanda
→ contexto secundário e detalhes

usar texto
→ quando forma, símbolo, cor ou posição não forem suficientes
```

Aplicação:

- preferir cor + forma + símbolo para estados simples;
- preferir ícones reconhecíveis para ações recorrentes quando claros;
- usar tooltip/popover/expansão para contexto secundário;
- evitar repetir em texto informação já comunicada claramente pelo componente;
- evitar chips/badges/labels/cards sem função real;
- preservar espaço para o conteúdo de trabalho.

Limites:

- minimalismo não pode criar ambiguidade;
- quando texto for necessário, ele permanece;
- cor nunca é o único meio de comunicar estado importante;
- acessibilidade e nome semântico continuam obrigatórios.

## 29. Formato físico do Procedimento

Baseline:

```text
papel:       A4
orientação:  retrato
páginas:     conforme necessário
margens:     18 mm em todos os lados
```

A4 aqui é o formato do **arquivo físico exportado**, sem relação com a geometria do Reader.

PDF e DOCX compartilham esse formato-base.

## 30. Primeira página

Não há capa exclusiva.

A primeira página começa diretamente com identificação e conteúdo útil, por exemplo:

```text
[logo] Empresa

PR-014
Configuração de VLAN
versão/revisão · estado editorial

Área/Departamento · Responsável

VISÃO GERAL
Objetivo...
Pré-requisitos...
Observações...

01 · Preparação
...
```

Direção:

- identidade compacta;
- código/título como hierarquia principal;
- somente metadados necessários;
- campos vazios não reservam espaço;
- sem tabela pesada apenas para metadados;
- conteúdo começa na mesma página.

## 31. Sumário físico

A v1 **não exige sumário documental físico por padrão**.

Razões:

- reduz densidade e páginas consumidas;
- evita repetir títulos desnecessariamente;
- sequência de Etapas já fornece hierarquia clara;
- evita depender de paginação rígida do DOCX.

Pode ser reavaliado futuramente se procedimentos extensos demonstrarem necessidade real.

## 32. Etapas no documento físico

Título recomendado de forma curta:

```text
01 · Preparação
02 · Configuração
03 · Validação
```

Não é necessário repetir `ETAPA` antes de cada título quando a hierarquia visual já for clara.

Regras:

- uma Etapa não força automaticamente nova folha;
- título permanece junto do primeiro bloco sempre que possível;
- sem espaço útil suficiente para título + início de conteúdo, ambos seguem para a próxima página;
- etapas curtas podem compartilhar a mesma folha física;
- separação usa espaço/hierarquia/forma discreta, não cards pesados.

## 33. Cabeçalho e rodapé

Baseline:

- **sem cabeçalho repetitivo** nas páginas internas;
- rodapé compacto com código/revisão e paginação.

Exemplo conceitual:

```text
PR-014 · r18                              3 / 8
```

Não mostrar no rodapé:

- token/sessão;
- hostname;
- path local;
- usuário técnico;
- timestamp ambiental desnecessário.

Informação essencial de identificação também aparece no corpo da primeira página, nunca apenas no rodapé.

## 34. Blocos físicos

A representação segue baixa densidade visual:

- `paragraph` → texto normal, sem card;
- `numbered_steps` → numeração + indentação;
- `checklist` → `□` + texto;
- `note` → forma/símbolo discreto + conteúdo;
- `warning` → símbolo/contraste mais forte e texto quando necessário para não depender só de cor;
- `command` → bloco monoespaçado compacto;
- `code` → bloco monoespaçado compacto;
- imagem → proporção preservada, sem crop automático.

Não envolver cada passo em borda/card.

Comando/código permanece texto real, selecionável/pesquisável e preserva whitespace relevante.

## 35. Paginação física

- paginação automática é o padrão;
- evitar widow/orphan em texto normal;
- título de Etapa não fica isolado no fim da página;
- subtítulo/label permanece com primeiro conteúdo relacionado quando possível;
- pequenos blocos de nota/alerta/comando/checklist ficam inteiros quando razoável;
- bloco excepcionalmente longo pode quebrar entre páginas;
- nunca truncar conteúdo;
- nunca reduzir fonte dinamicamente só para fazer conteúdo caber;
- nenhuma Etapa/bloco é omitida silenciosamente.

## 36. Tipografia do PDF

Famílias:

```text
texto:          Noto Sans
comando/código: Noto Sans Mono
```

Regras:

- fontes empacotadas com o Host;
- disponíveis somente pelo mundo controlado do renderer;
- incorporadas/subsetadas no PDF;
- sem dependência de fontes instaladas no Windows;
- arquivos/licença distribuídos conforme OFL 1.1.

## 37. Tipografia do DOCX

Famílias declaradas:

```text
texto:          Arial
comando/código: Consolas
```

Regras:

- não incorporar essas fontes no DOCX v1;
- não empacotar/redistribuir arquivos Arial/Consolas com StepFlow;
- DOCX apenas referencia as famílias;
- matriz Windows/Word real é validada posteriormente;
- substituição de fonte pelo consumidor, se ocorrer, não altera o conteúdo semântico.

A diferença de família entre PDF e DOCX é deliberada: PDF prioriza runtime autocontido/licença de redistribuição; DOCX prioriza compatibilidade Windows/Office sem exigir instalação extra.

## 38. Escala tipográfica

Baseline:

| Uso | Tamanho |
|---|---:|
| título do documento | 18 pt |
| título de Etapa | 14 pt |
| corpo | 10,5 pt |
| comando/código | 9 pt |
| rodapé | 8 pt |

Direção:

- espaçamento compacto, mas legível;
- hierarquia por peso/tamanho antes de caixas decorativas;
- sem redução dinâmica de fonte para acomodar conteúdo excessivo.

Pesos/pequenos refinamentos podem ser calibrados no gate visual de implementação sem alterar essa escala-base ou a hierarquia consolidada.

## 39. PDF × DOCX

Compartilham:

- A4 retrato;
- margens-base;
- ordem do conteúdo;
- identidade;
- hierarquia;
- semântica dos blocos.

Não compartilham promessa de paginação idêntica.

```text
PDF  → referência física de impressão
DOCX → artefato editável/refluível
```

Não forçar quebras artificiais no DOCX apenas para imitar o PDF.

## 40. Decisões consolidadas — Etapa 5

1. Reader diário e documento físico são superfícies distintas; Reader não possui geometria A4;
2. `Visão geral` é a primeira página lógica do Reader;
3. uma Etapa permanece uma página lógica própria do Reader;
4. stepper compacto de círculos/linhas é navegável e representa anterior/atual/seguinte;
5. estado anterior no stepper é navegação, não conclusão operacional;
6. baixa densidade textual/visual é princípio transversal do Pocket, sem sacrificar clareza/acessibilidade;
7. Procedimento exportado usa A4 retrato multipágina;
8. margens-base são 18 mm em todos os lados;
9. não há capa exclusiva;
10. v1 não exige sumário físico por padrão;
11. título físico de Etapa é enxuto e não força nova folha automaticamente;
12. sem cabeçalho repetitivo; rodapé compacto identifica documento/página;
13. blocos físicos evitam cards/bordas desnecessários e preservam semântica;
14. paginação evita títulos/linhas órfãs e nunca trunca/reduz fonte silenciosamente;
15. imagens preservam proporção, sem crop automático;
16. PDF usa Noto Sans + Noto Sans Mono empacotadas/incorporadas conforme OFL 1.1;
17. DOCX usa Arial + Consolas referenciadas, sem embedding/redistribuição v1;
18. escala-base: 18 pt título, 14 pt Etapa, 10,5 pt corpo, 9 pt código/comando, 8 pt rodapé;
19. PDF é referência física de impressão; DOCX permanece refluível sem paginação idêntica;
20. limite rígido de uma A4 pertence somente à Ficha compacta de Atendimento.

## 41. Fechamento da Etapa 5

A Etapa 5 está **CONSOLIDADA / APROVADA PELO PO**.

Foram fechados:

- separação inequívoca entre Reader, Procedimento exportado e Ficha compacta;
- correção do stepper do Reader e princípio transversal de baixa densidade visual;
- A4 retrato multipágina e margens do Procedimento;
- primeira página, ausência de capa/sumário obrigatório e composição enxuta;
- política de Etapas, header/footer e paginação;
- representação física dos blocos;
- tipografia PDF/DOCX e escala-base;
- relação PDF × DOCX.

O trabalho permaneceu documental, sem código funcional, dependency, migration ou scaffold.

---

# Próximas etapas

## 42. Etapa 6 — PDF + preview da Ficha compacta

**Status:** PRÓXIMA — AINDA NÃO EM ANÁLISE

Contrato herdado já consolidado para a Ficha:

- pertence ao Atendimento;
- usa estado confirmado pelo Host;
- pode existir com ou sem Equipamento;
- `Em andamento` pode gerar para acompanhamento;
- `Concluído` reimprime estado histórico aplicável;
- `Cancelado` deve aparecer inequivocamente;
- capacidade depende do Atendimento acessível/preset autorizado;
- máximo **uma página A4**;
- conteúdo excessivo não gera segunda página normal nem truncamento silencioso;
- impressão é requisito;
- DOCX específico não é requisito inicial.

A Etapa 6 ainda não decide nada além disso até ser explicitamente aberta após o fechamento operacional da Etapa 5.

## 43. Gate operacional antes da Etapa 6

A Etapa 5 só é encerrada operacionalmente após:

```text
consolidação documental
→ squash merge do PR
→ remoção da branch remota
→ verificação de remoto somente com main
→ zero PRs abertos
```

Antes desse gate, não iniciar pesquisa, proposta, branch ou análise da Etapa 6.

## 44. Etapas 7–12 ainda pendentes

- **Etapa 7:** Template físico A4 da Ficha;
- **Etapa 8:** Limites textuais e densidade da Ficha;
- **Etapa 9:** Múltiplos MACs / Procedimentos na Ficha;
- **Etapa 10:** Nomes de arquivo + artefatos temporários, incluindo materialização/limpeza concreta do recurso de impressão;
- **Etapa 11:** QR / barcode;
- **Etapa 12:** Validação técnica final do Bloco 10, incluindo matriz real de Windows/WebView2/Office/impressoras e limites medidos.

Nenhuma dessas etapas deve ser antecipada por inferência.

## 45. Pendências que permanecem fora deste fechamento

- versão mínima concreta do WebView2;
- versões exatas de crates/dependencies;
- limites numéricos de memória/tempo/tamanho/concorrência;
- paths/names/lifecycle concretos de temporários;
- detalhes finais da Ficha compacta das Etapas 6–9;
- QR/barcode;
- Backup/Restore técnico do Bloco 11;
- parâmetros reais do ambiente corporativo.

**Etapa 6 — PDF + preview da Ficha compacta é a próxima etapa do Bloco 10, mas ainda não está em análise.**