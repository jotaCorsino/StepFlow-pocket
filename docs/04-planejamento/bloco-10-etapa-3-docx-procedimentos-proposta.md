# Bloco 10 — Etapa 3 — DOCX de Procedimentos — Proposta para análise

**Status:** EM ANÁLISE / PROPOSTA PARA APROVAÇÃO DO PO  
**Fase:** Fase 1 — Fechamento arquitetural e especificação  
**Abertura:** 2026-08-26  
**Base consolidada:** Bloco 10 / Etapas 1 e 2

## 1. Objetivo

Definir somente a base técnica do renderer **DOCX de Procedimentos** e as capacidades mínimas que o artefato editável precisa preservar.

Esta etapa não redefine a arquitetura documental das Etapas 1–2 e não define margens, tipografia final, cabeçalho/rodapé final, paginação visual, A4 ou densidade. Esses pontos permanecem reservados para a **Etapa 5 — Template físico de Procedimentos**.

Também não define a impressão Windows da Etapa 4.

## 2. Contratos herdados

Permanecem vigentes:

- geração documental pertence ao Host;
- Client solicita por identidade da fonte/revisão esperada e não envia documento montado;
- Host captura snapshot consistente e materializa `DocumentModel` antes da renderização;
- renderer não consulta SQLite nem recebe DOM/HTML da UI;
- geração é leitura derivada e fica fora da fila de mutações;
- renderização usa limite próprio de concorrência/backpressure;
- artefato retorna pela API autenticada;
- Host não grava em path arbitrário do Client;
- runtime documental não depende de Office, LibreOffice, browser ou serviço cloud;
- PDF e DOCX são renderers independentes do mesmo `DocumentModel` — DOCX não é produzido por conversão do PDF.

## 3. Formato proposto

O artefato deve ser um **DOCX real baseado em Office Open XML / WordprocessingML**, empacotado segundo Open Packaging Conventions, com MIME:

```text
application/vnd.openxmlformats-officedocument.wordprocessingml.document
```

O Host deve produzir o pacote diretamente em Rust. Não executar:

- Microsoft Word;
- automação COM;
- LibreOffice;
- conversor CLI externo;
- browser/headless;
- serviço web de conversão;
- pipeline PDF → DOCX.

## 4. Biblioteca Rust proposta

A direção preferida é usar **`docx-rs` embutido como biblioteca Rust no Host**, encapsulado por um adaptador interno StepFlow.

Fluxo conceitual:

```text
DocumentModel
→ DocxRenderer / adaptador StepFlow
→ estruturas WordprocessingML/OOXML
→ docx-rs
→ pacote OPC/ZIP
→ DOCX bytes
```

Motivos:

- gera `.docx` diretamente em Rust;
- não depende de Office instalado;
- suporta parágrafos, runs, estilos, numeração, tabelas, imagens, seções e headers/footers;
- permite empacotamento direto do documento;
- evita que o StepFlow implemente do zero toda a mecânica OPC, relações, content-types, numbering e WordprocessingML.

Não consolidar versão exata da crate durante a Fase 1. A versão será fixada em `Cargo.lock` no gate de implementação e validada contra a matriz real de compatibilidade.

A biblioteca é uma dependência de implementação; o contrato arquitetural pertence ao adaptador StepFlow. Se uma capacidade obrigatória não puder ser representada corretamente pela versão adotada, a geração deve falhar/ser corrigida — nunca descartar conteúdo silenciosamente.

## 5. Fronteira de dados e segurança

Conteúdo originado do domínio entra somente como **valores/dados estruturados**.

Regras:

- nenhum texto do usuário vira XML/OOXML por concatenação não controlada;
- nenhum conteúdo do Procedimento pode injetar relationships, partes OPC, XML arbitrário ou instruções executáveis;
- não aceitar `.docx`/`.dotx` de template fornecido pelo usuário na primeira versão;
- não carregar remote template;
- não criar relationships externas para imagens/objetos;
- assets são resolvidos pelo Host antes do renderer;
- renderer não recebe path arbitrário ou URL originados do conteúdo;
- nenhum recurso é baixado da Internet durante geração.

Se no futuro uma extensão OOXML interna for necessária por limitação da biblioteca, ela deve permanecer **controlada pelo código StepFlow**, coberta por testes e sem aceitar XML arbitrário do domínio.

## 6. Editabilidade é requisito central

Diferentemente do PDF, o DOCX existe para continuar editável em um processador de texto compatível.

Portanto:

- texto textual deve permanecer texto Word real;
- títulos, parágrafos, passos e notas não podem ser rasterizados;
- comandos/código permanecem texto;
- conteúdo deve ser selecionável, pesquisável, copiável e editável;
- imagens permanecem objetos de imagem incorporados;
- nenhuma etapa conhecida pode ser convertida para screenshot como solução de layout.

A edição posterior feita pelo usuário fora do StepFlow não altera a revisão oficial do Procedimento. O DOCX exportado é um artefato derivado, não uma fonte de importação/sincronização.

## 7. Blocos semânticos obrigatórios

O renderer DOCX precisa representar todos os blocos já suportados pelo `DocumentModel`:

- parágrafo;
- passos/subpassos numerados;
- checklist documental;
- nota;
- alerta;
- comando;
- bloco de código.

Regras propostas:

- passos/subpassos usam numeração/lista Word real quando aplicável;
- checklist documental usa representação visual estável de caixa + texto, sem virar formulário interativo;
- nota e alerta mantêm distinção semântica por estilos/estrutura controlada;
- comando/código usam texto com preservação de whitespace relevante;
- tipo de bloco desconhecido/incompatível falha explicitamente em vez de produzir documento incompleto.

A aparência física final desses estilos pertence à Etapa 5.

## 8. Imagens e identidade corporativa

O renderer recebe somente assets já autorizados e resolvidos pelo Host.

Baseline proposto para DOCX:

- PNG;
- JPEG.

SVG não é requisito direto do DOCX v1 porque a compatibilidade varia entre versões/consumidores. Se a fonte controlada existir apenas em SVG, o Host deve fornecer uma representação compatível por pipeline interno validado ou falhar explicitamente; nunca remover a imagem silenciosamente nem depender de conversor externo.

A política de upload/armazenamento de logo não é alterada por esta etapa.

## 9. Estilos e template interno

O DOCX deve usar **estilos Word internos e versionados pelo StepFlow**, produzidos pelo renderer.

A primeira versão não depende de um arquivo `.docx`/`.dotx` externo usado como template em runtime.

O renderer deve distinguir conceitualmente estilos como:

- título do documento;
- metadados;
- título de seção/etapa;
- corpo;
- passo numerado;
- checklist;
- nota;
- alerta;
- comando;
- código.

Cores, fontes, tamanhos, margens e composição visual final desses estilos permanecem na Etapa 5.

## 10. Fontes e reflow

DOCX é um formato refluível. O StepFlow **não deve prometer paginação idêntica ao PDF**.

Mesmo com o mesmo conteúdo, a quebra de página pode variar por:

- versão do Microsoft Word/consumidor;
- fontes disponíveis;
- métricas de fonte/substituição;
- configurações de compatibilidade do processador de texto;
- configuração de impressão do ambiente.

A Etapa 3 consolida estabilidade **semântica e estrutural**, não identidade visual página a página com o PDF.

A família tipográfica final e eventual política de incorporação de fontes no DOCX ficam para a Etapa 5/gate de implementação, considerando licenciamento e compatibilidade. Não importar automaticamente a política de fontes empacotadas do PDF para o DOCX.

## 11. Whitespace de comandos e código

Comando/código deve preservar:

- espaços relevantes;
- tabs quando representadas pelo modelo;
- quebras de linha;
- indentação.

O renderer deve usar a semântica adequada de WordprocessingML para preservar espaço textual, sem substituir o bloco por imagem.

Quebras extremamente longas/linhas extensas serão tratadas pelo template/estilo da Etapa 5; truncamento silencioso continua proibido.

## 12. Relações externas e conteúdo ativo

Ficam fora da primeira versão:

- macros/VBA e `.docm`;
- ActiveX;
- OLE/objetos incorporados;
- scripts;
- links de imagem externos;
- remote templates;
- `altChunk` com HTML externo;
- anexos incorporados;
- assinatura digital do documento;
- proteção/senha/DRM;
- formulários/content controls interativos como requisito de checklist;
- importação de um DOCX editado de volta ao StepFlow.

URLs presentes no conteúdo podem permanecer como texto. Hyperlink ativo só deve existir se o domínio vier a consolidar esse tipo semântico; não deve ser inferido automaticamente a partir de texto.

## 13. Metadados

O pacote pode registrar metadados controlados, como:

- título;
- versão/revisão exibida;
- identidade técnica do documento;
- data de geração quando explicitamente fornecida por `generation_metadata`.

Não incluir token, senha, path local, conteúdo de sessão ou dado técnico interno desnecessário.

Metadados ambientais não podem alterar conteúdo visual semântico implicitamente.

## 14. Determinismo possível no DOCX

Com a mesma versão do Host, mesmo `DocumentModel`, mesmos assets e mesma definição de estilos, o **conteúdo e estrutura OOXML** devem permanecer estáveis.

Não se exige identidade byte-a-byte do ZIP nem paginação idêntica entre diferentes consumidores Word.

IDs/relationships gerados devem ser deterministicamente controlados quando razoável para facilitar testes e diagnóstico, mas diferenças técnicas sem efeito semântico não são falha por si só.

## 15. Validação do artefato

A geração só é sucesso quando o pacote DOCX estiver completo e fechável.

Validações esperadas no gate técnico posterior:

- ZIP/OPC íntegro;
- partes obrigatórias e relationships coerentes;
- XML bem formado;
- reabertura/parse por biblioteca de validação quando aplicável;
- abertura sem diálogo de reparo no Microsoft Word da matriz corporativa validada;
- todos os blocos semânticos presentes;
- imagens/numeração preservadas;
- comandos/código com whitespace esperado.

A matriz concreta de versões do Word/Office e demais consumidores pertence à validação técnica posterior e ao ambiente corporativo real.

## 16. Recursos e concorrência

Aplicam-se os limites consolidados na Etapa 1:

- renderer DOCX usa o mesmo mecanismo geral de limite de concorrência documental;
- não entra na fila de mutações;
- não cria job persistente;
- falha não produz artefato parcial tratado como sucesso;
- limites numéricos de memória/tamanho/tempo ficam para medição/Etapa 12;
- temporários concretos continuam reservados para a Etapa 10.

A possibilidade de `docx-rs` empacotar diretamente o documento deve ser aproveitada quando adequada, evitando manter representações duplicadas grandes em memória sem necessidade.

## 17. Alternativas consideradas

### Construir OOXML manualmente

É tecnicamente possível gerar ZIP/OPC + WordprocessingML diretamente com crates de XML/ZIP. Entretanto, isso transfere para o StepFlow responsabilidade por relações OPC, content-types, numbering, estilos, imagens e muitos detalhes de compatibilidade. Não é a direção preferida enquanto uma biblioteca Rust madura cobrir o contrato necessário.

### Automação do Microsoft Word/COM

Rejeitada como arquitetura de geração: acopla o Host à instalação/configuração do Office, acrescenta dependência operacional e conflita com o runtime Pocket/autocontido já consolidado.

### LibreOffice/conversores externos

Rejeitados pelo mesmo contrato autocontido das Etapas 1–2.

### Converter o PDF/Typst para DOCX

Rejeitado. PDF e DOCX têm objetivos diferentes; a conversão perderia estrutura/editabilidade e criaria acoplamento indevido entre renderers.

### `docx-rs`

É a direção proposta porque fornece uma camada Rust própria para WordprocessingML/OOXML e empacotamento DOCX sem exigir processo externo, preservando editabilidade e reduzindo código estrutural próprio.

## 18. Decisões propostas ao PO — Etapa 3

A Etapa 3 propõe consolidar somente:

1. DOCX de Procedimentos é gerado diretamente pelo Host Rust a partir do mesmo `DocumentModel` sem converter PDF;
2. saída é `.docx` OOXML/WordprocessingML real com MIME oficial;
3. `docx-rs` é a biblioteca Rust preferida, encapsulada por adaptador interno StepFlow;
4. nenhuma dependência de Word/COM, LibreOffice, browser, CLI ou cloud para gerar;
5. conteúdo do domínio entra apenas como dados estruturados e nunca como XML/OOXML arbitrário;
6. template/estilos são internos e versionados; nenhum `.docx`/`.dotx` externo fornecido pelo usuário na primeira versão;
7. texto permanece real, selecionável, pesquisável, copiável e editável;
8. todos os blocos semânticos conhecidos são representados sem descarte silencioso;
9. passos usam numeração/lista real; checklist permanece documental e não vira formulário interativo;
10. comando/código preserva whitespace e permanece texto;
11. PNG/JPEG são baseline de imagem; SVG não é requisito direto do DOCX v1 e nunca pode ser omitido silenciosamente;
12. DOCX é refluível e não promete paginação idêntica ao PDF; layout físico final continua na Etapa 5;
13. política tipográfica/embedding de fontes do DOCX não é herdada automaticamente do PDF e permanece para Etapa 5/validação;
14. macros, ActiveX, OLE, remote templates, conteúdo externo, anexos, assinatura, senha/DRM e importação de DOCX editado ficam fora da primeira versão;
15. artefato incompleto/corrompido nunca é devolvido como sucesso;
16. versão exata da crate, limites numéricos e matriz real de compatibilidade ficam para implementação/Etapa 12.

## 19. Critério de fechamento

Esta proposta só deve ser promovida para a fonte canônica `bloco-10-exportacao-impressao-ficha.md` e para `AGENTS.md`, README, arquitetura, registro e plano oficial após **aprovação explícita do PO**.

Até lá:

- Etapa 2 permanece consolidada;
- Etapa 3 permanece em análise nesta branch/PR;
- Etapa 4 continua pendente e não deve ser aberta;
- nenhum código funcional, dependency, scaffold ou migration é autorizado por este documento.
