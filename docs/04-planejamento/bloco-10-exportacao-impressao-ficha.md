# Bloco 10 — Exportação / Impressão + Ficha Compacta

**Status:** EM ANDAMENTO — ETAPAS 1–3 CONSOLIDADAS / ETAPA 4 PRÓXIMA  
**Fase:** Fase 1 — Fechamento arquitetural e especificação  
**Abertura:** 2026-08-25  
**Etapa 1 consolidada:** 2026-08-25  
**Etapa 2 consolidada:** 2026-08-26  
**Etapa 3 consolidada:** 2026-08-26

## 1. Objetivo do bloco

Fechar, uma etapa por vez, o contrato técnico de geração documental do StepFlow, preservando o caráter Pocket e a UX já aprovada no Bloco 8.

Este documento é o mapa técnico do Bloco 10. Uma etapa futura só entra em análise quando for explicitamente aberta. Não pertence a este bloco implementar código de produção, fechar Backup/Restore técnico ou abrir o Bloco 11.

## 2. Etapas do Bloco 10

| Ordem | Etapa | Estado |
|---|---|---|
| 1 | Arquitetura de geração documental | **CONSOLIDADO / APROVADO PELO PO** |
| 2 | PDF de Procedimentos | **CONSOLIDADO / APROVADO PELO PO** |
| 3 | DOCX de Procedimentos | **CONSOLIDADO / APROVADO PELO PO** |
| 4 | Impressão Windows de Procedimentos | **PRÓXIMA — AINDA NÃO EM ANÁLISE** |
| 5 | Template físico de Procedimentos | PENDENTE |
| 6 | PDF + preview da Ficha compacta | PENDENTE |
| 7 | Template físico A4 da Ficha | PENDENTE |
| 8 | Limites textuais e densidade da Ficha | PENDENTE |
| 9 | Múltiplos MACs / Procedimentos na Ficha | PENDENTE |
| 10 | Nomes de arquivo + artefatos temporários | PENDENTE |
| 11 | QR / barcode | PENDENTE |
| 12 | Validação técnica final do Bloco 10 | PENDENTE |

As Etapas 4–12 permanecem fora de análise neste checkpoint. A Etapa 4 está apenas marcada como próxima.

---

# Etapa 1 — Arquitetura de geração documental

**Status:** CONSOLIDADO / APROVADO PELO PO

## 3. Objetivo

Definir somente as fronteiras arquiteturais da geração documental:

- onde a geração acontece;
- como o Client solicita um documento;
- como o Host escolhe e congela a fonte da geração;
- como autorização e consistência são garantidas;
- como renderers futuros compartilham regras de negócio sem compartilhar HTML de tela;
- como o artefato volta ao Client;
- como concorrência e consumo de recursos são limitados;
- o que não vira persistência, fila ou serviço novo.

A Etapa 1 não escolhe biblioteca PDF, biblioteca DOCX, margens, fontes, preview, impressão Windows, limites de texto, quantidade de MACs ou QR/barcode.

## 4. Contratos herdados

Permanecem vigentes:

- PDF, DOCX e impressão são obrigatórios para Procedimentos;
- exportação usa documento próprio, nunca screenshot;
- a fonte de Procedimento é a revisão exata selecionada/autorizada;
- revisão nova não substitui silenciosamente a revisão escolhida;
- ficha pertence ao Atendimento e pode existir com ou sem Equipamento;
- ficha usa somente estado confirmado pelo Host;
- `Em andamento` pode gerar ficha de acompanhamento;
- `Concluído` reimprime estado histórico aplicável;
- `Cancelado` precisa ser identificado inequivocamente;
- ficha ocupa no máximo uma página A4;
- DOCX específico da ficha não é requisito inicial;
- autorização real permanece Host-side;
- Client nunca abre SQLite diretamente;
- evento WebSocket é sinal de mudança, não fonte oficial de estado.

## 5. Fronteira arquitetural consolidada

A geração documental pertence ao **Host**. O Client continua responsável pela experiência local do usuário.

```text
Client
  ↓ solicita geração com identidade da fonte
Host
  ↓ autentica e autoriza
  ↓ valida versão/revisão esperada
  ↓ captura snapshot consistente da fonte
  ↓ materializa DocumentModel semântico
  ↓ encerra leitura/transação SQLite
  ↓ renderiza o formato solicitado
  ↓ devolve o artefato pela conexão autenticada
Client
  ↓ recebe o artefato
  ├─→ salvar localmente
  └─→ preview/impressão quando as etapas correspondentes definirem
```

Consequências:

- Client não gera documento a partir do DOM/HTML da tela;
- Client não replica regras de negócio documentais;
- Host não recebe caminho arbitrário do computador remoto;
- Host não precisa acessar filesystem do usuário;
- não existe compartilhamento SMB obrigatório para exportar;
- geração usa somente dados que o Host pode autorizar e confirmar;
- engines futuras recebem um modelo semântico comum, não conteúdo de UI.

## 6. Requisição por identidade da fonte

O Client não envia o texto completo do documento para o Host gerar.

A solicitação contém conceitualmente:

```text
- tipo de documento;
- formato solicitado;
- identificador estável da fonte;
- revisão/versão esperada quando aplicável;
- contexto mínimo necessário à operação.
```

Para Procedimento, a revisão específica imutável identifica o snapshot documental. Para Atendimento mutável, a requisição preserva a revisão confirmada que o Client está usando.

Exemplo:

```text
Client mostra Atendimento revisão 42
→ solicita ficha para revisão esperada 42
→ Host já está na revisão 43
→ NÃO gerar silenciosamente a 43
→ informar estado obsoleto/conflito
→ Client reconsulta antes de nova tentativa
```

A forma exata de endpoint/payload fica para implementação da API; a semântica de **fonte esperada** está consolidada.

## 7. Captura consistente antes da renderização

A geração possui duas fases:

```text
FASE A — captura
Host abre leitura consistente
→ valida fonte/autorização
→ lê dados necessários
→ resolve snapshots/revisões
→ captura identidade corporativa necessária
→ materializa DocumentModel em memória
→ encerra leitura/transação

FASE B — renderização
DocumentModel já materializado
→ renderer produz artefato
→ nenhuma transação SQLite permanece aberta
```

A renderização não mantém uma transação de banco aberta por todo o tempo de geração. Isso evita segurar snapshot SQLite desnecessariamente, ampliar contenção com o writer e misturar dados de momentos diferentes.

## 8. Fonte por domínio

### Procedimento

A fonte documental é a revisão exata solicitada, já imutável. A identidade corporativa aplicável é capturada no início da geração.

### Atendimento em andamento

A fonte é um snapshot consistente do estado confirmado na revisão esperada, incluindo vínculos necessários ao documento.

### Atendimento concluído

A fonte usa o estado histórico consolidado aplicável, incluindo projeção congelada do Equipamento e revisões exatas dos Procedimentos utilizados.

### Atendimento cancelado

Usa o estado oficial aplicável e preserva identificação inequívoca do cancelamento.

## 9. DocumentModel semântico

Depois da captura, o Host cria uma representação intermediária tipada e imutável para aquela geração.

Conceitualmente:

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

O modelo é semântico, não layout PDF nem estrutura DOCX específica. Ele pode representar título, metadados, seções, parágrafos, passos/subpassos, checklist documental, notas/alertas, comandos/código e campos estruturados da ficha.

O `DocumentModel` não contém:

- HTML arbitrário;
- JavaScript;
- comandos executáveis;
- DOM da interface;
- classes CSS da aplicação;
- token de sessão;
- caminho escolhido pelo usuário;
- estado transitório do Client.

## 10. Um modelo de conteúdo, renderers separados

```text
fonte oficial
      ↓
DocumentModel semântico
      ├─→ renderer PDF        [Etapa 2]
      └─→ renderer DOCX       [Etapa 3]
```

Regras de negócio como revisão utilizada, equipamento histórico aplicável e existência de campos são resolvidas antes dos renderers. Os renderers não reconsultam o banco nem reconstruem o domínio.

## 11. Geração é leitura derivada

Gerar documento não deve:

- entrar na fila de mutações do writer;
- criar revisão de Procedimento;
- alterar Atendimento;
- marcar checklist;
- alterar `updated_at` funcional;
- emitir evento de domínio apenas porque um arquivo foi gerado;
- criar registro permanente de exportação por padrão.

## 12. Concorrência de renderização

Renderização pode consumir CPU e memória, portanto usa limite próprio de concorrência, separado da fila de mutações.

```text
requisições de geração
        ↓
limite bounded de renderização
        ├─ vaga disponível → gerar
        └─ saturado → backpressure / SERVER_BUSY
```

Regras consolidadas:

- não bloquear o writer esperando renderer;
- não criar fila persistente de exportações;
- não gravar jobs aceitos para executar depois;
- evitar renderização pesada no executor assíncrono principal quando isso puder bloquear o Host;
- quantidade exata de renderizações simultâneas será definida por implementação/medição.

## 13. Sem job persistente na primeira versão

Não criar inicialmente:

- tabela `export_jobs`;
- scheduler documental;
- estado persistente `queued/running/completed`;
- retenção de artefatos no servidor;
- recuperação de exportação após restart;
- histórico de downloads;
- fila offline de geração.

Fluxo inicial:

```text
request autenticado
→ captura consistente
→ renderização
→ resposta com artefato
```

Arquitetura assíncrona só será proposta futuramente se testes reais demonstrarem necessidade.

## 14. Transporte do artefato

O artefato retorna pela API autenticada, como bytes/stream adequado ao formato. O Client decide o destino local conforme as etapas futuras.

O Host não recebe path do computador remoto para escrever diretamente nele.

Se um renderer precisar de arquivo temporário interno durante a requisição, esse detalhe fica em área privada controlada pelo Host, inacessível por path fornecido pelo Client, limitado à vida da operação, removido após uso/falha e fora de backup/histórico funcional.

## 15. Runtime Pocket/autocontido

A geração documental não depende operacionalmente de:

- Microsoft Office instalado;
- automação COM do Office;
- LibreOffice;
- Adobe Reader;
- Chrome/Chromium externo headless;
- `wkhtmltopdf` ou conversor executável separado;
- serviço web/cloud de conversão.

Bibliotecas compiladas com o Host podem ser usadas. Crates/versões específicas ficam para as etapas/implementação correspondentes.

## 16. Responsabilidade do Client

O Client:

- inicia a ação a partir da UX aprovada;
- mantém a sessão autenticada;
- informa fonte/revisão esperada;
- apresenta preparação/erro/cancelamento;
- recebe o artefato;
- abre fluxos locais de salvar/preview/imprimir quando definidos.

O Client não reconstitui autorização, não consulta SQLite, não monta documento final pelo HTML da tela e não escolhe silenciosamente outra revisão.

## 17. Autorização e sessão

A autorização é validada no Host no momento da solicitação.

Procedimento exige conceitualmente capacidade de ler a revisão solicitada + capacidade de exportar/imprimir. Ficha exige capacidade operacional aplicável ao Atendimento acessível.

Na arquitetura síncrona inicial, a requisição é aceita somente com sessão válida e o artefato retorna pela mesma operação autenticada. Não existe URL pública permanente de download.

## 18. Cancelamento, desconexão e falhas

Como geração não altera domínio:

- fechar/cancelar o Client pode abandonar a resposta;
- o Host pode interromper trabalho quando tecnicamente seguro;
- conexão perdida não cria mutação de negócio para reconciliar;
- nova tentativa reconsulta a fonte oficial e usa a revisão atual confirmada.

Classes mínimas de falha:

- não autenticado;
- sem permissão;
- fonte inexistente/indisponível;
- revisão esperada obsoleta;
- fonte inválida para o documento;
- renderer indisponível/falhou;
- limite de recursos excedido;
- Host ocupado/backpressure;
- conexão interrompida.

## 19. Logs, persistência e backup

Log operacional mínimo pode registrar tipo do documento, formato, identificador técnico da fonte, duração e resultado/classe de erro.

Não registrar por padrão conteúdo integral, senha/token, bytes do artefato ou paths locais escolhidos pelo usuário.

Artefatos gerados não fazem parte do estado oficial apenas por terem sido produzidos. Por padrão:

- não entram no SQLite como histórico documental;
- não entram no backup;
- não precisam sobreviver a restart do Host;
- a cópia persistente é responsabilidade do usuário quando ele escolher salvar localmente.

Auditoria funcional permanente de cada exportação não é requisito consolidado nesta etapa.

## 20. Compatibilidade de API

A geração fica sob a API versionada do Host e respeita o handshake Client↔Host existente. Nomes finais de endpoints/payloads não são fechados nesta etapa.

## 21. Decisões consolidadas da Etapa 1

1. geração documental é responsabilidade do Host;
2. Client solicita por IDs/revisão esperada, nunca enviando documento montado;
3. Host captura snapshot consistente e encerra leitura antes de renderizar;
4. Atendimento mutável usa revisão esperada para impedir geração silenciosa de estado diferente;
5. `DocumentModel` semântico é a fronteira comum entre domínio e renderers;
6. renderers não reconsultam banco nem recebem HTML da UI;
7. geração é leitura derivada e não passa pela fila de mutações;
8. renderização tem limite próprio de concorrência/backpressure, sem fila persistente;
9. fluxo inicial é request → render → resposta, sem `export_jobs` persistentes;
10. artefato retorna pela API autenticada; Host não escreve em path arbitrário do Client;
11. runtime não depende de Office/LibreOffice/Adobe/Chrome externo/cloud/conversor auxiliar obrigatório;
12. artefatos gerados não viram histórico/backup por padrão;
13. Client permanece responsável apenas pela UX e pelo destino local do artefato;
14. detalhes de PDF, DOCX, impressão, templates, limites, MACs e QR permanecem nas respectivas etapas.

---

# Etapa 2 — PDF de Procedimentos

**Status:** CONSOLIDADO / APROVADO PELO PO

## 22. Objetivo da Etapa 2

Definir somente a base técnica do renderer PDF de Procedimentos e as capacidades mínimas que o PDF precisa preservar.

Esta etapa **não define** margens, tamanhos de fonte, cabeçalho/rodapé, paginação visual, densidade, layout final ou impressão Windows. Esses pontos permanecem nas Etapas 4 e 5.

## 23. Engine PDF consolidada

O StepFlow usa **Typst embutido como biblioteca Rust no Host**, sem executar `typst.exe`/CLI, browser ou processo conversor externo e sem depender de instalação externa.

A integração usa crates Rust oficiais do ecossistema Typst, encapsuladas por um **adaptador interno StepFlow ↔ Typst**. A arquitetura não depende da existência de um wrapper ou “camada oficial de embedding” específica.

Não se consolida versão exata de crate durante a Fase 1. Na implementação, versões serão fixadas em `Cargo.lock` e validadas pelo gate técnico correspondente.

### Por que essa direção

O StepFlow precisa de documento multipágina com fluxo de texto, quebras automáticas, Unicode, imagens, estrutura semântica e PDF final reproduzível sem navegador/conversor externo.

A engine escolhida entrega layout documental de alto nível. Não será construído no Host um motor próprio de paginação, medição de parágrafos e quebras apenas para escrever primitivas PDF.

## 24. Template interno confiável e fronteira de dados

O renderer PDF usa **template Typst interno, confiável e versionado com o Host**.

Conteúdo originado do domínio **nunca participa da construção textual do source Typst**, inclusive após escaping.

Fluxo conceitual:

```text
DocumentModel
→ valores/dados estruturados controlados
→ adaptador interno StepFlow
→ template Typst interno confiável
→ engine Typst embutida
→ PDF bytes
```

Regras de segurança:

- texto vindo do usuário é dado, nunca fonte Typst executável;
- não concatenar conteúdo do domínio para construir source Typst;
- não permitir `#eval`, imports, paths ou código arbitrário originado do conteúdo do Procedimento;
- não resolver pacotes Typst pela Internet em runtime;
- não permitir template escolhido pelo usuário na primeira versão;
- assets, templates e fontes necessários ficam empacotados/versionados ou resolvidos pelo Host;
- acesso a filesystem/imports do renderer fica restrito ao mundo/projeto virtual controlado pelo Host;
- o renderer não recebe path arbitrário, URL remota ou filesystem genérico originados do conteúdo;
- nenhuma URL remota é buscada durante a geração.

A ponte concreta pode usar inputs/valores tipados, JSON ou estrutura virtual equivalente, desde que preserve a separação entre **dados** e **source Typst**.

## 25. Contrato do PDF gerado

O renderer deve produzir PDF válido e autocontido com MIME `application/pdf`.

Baseline consolidado:

- **PDF 1.7 solicitado explicitamente ao exporter**;
- **Tagged PDF explicitamente habilitado** como baseline de acessibilidade;
- texto real, selecionável, pesquisável e copiável quando a origem é textual;
- fontes necessárias incorporadas/subsetadas no PDF;
- Unicode adequado para português e caracteres técnicos usados pelos Procedimentos;
- imagens/logo controlados podem ser incorporados sem dependência externa;
- documento multipágina com quebra automática;
- metadados básicos do documento podem ser gravados no PDF.

A primeira versão **não promete conformidade formal PDF/UA ou PDF/A** sem validação específica. Tagged PDF é baseline estrutural, não certificação.

Uma versão de engine que não consiga preservar PDF 1.7 e Tagged PDF conforme este contrato não atende ao renderer consolidado da Etapa 2.

## 26. Fontes

O PDF não deve depender de fontes instaladas no Windows da máquina central.

O pacote StepFlow deve carregar fontes licenciadas para redistribuição e suficientes para:

- texto normal em português;
- títulos/ênfases;
- caracteres técnicos comuns;
- símbolos usados por checklist/avisos;
- fonte monoespaçada para comandos/código.

A família visual exata, pesos e tamanhos ficam para a **Etapa 5 — Template físico de Procedimentos**.

Ausência de uma fonte do sistema não pode mudar silenciosamente paginação ou aparência do documento.

## 27. Blocos semânticos obrigatórios

O renderer PDF precisa representar todos os blocos documentais já suportados pelo `DocumentModel`:

- parágrafo;
- passos/subpassos numerados;
- checklist documental;
- nota;
- alerta;
- comando;
- bloco de código.

Regras:

- nenhum bloco conhecido pode ser descartado silenciosamente;
- comandos/código preservam whitespace relevante;
- código/comando é texto, não imagem rasterizada;
- conteúdo textual continua copiável;
- renderer desconhecendo um tipo de bloco deve falhar de forma explícita por incompatibilidade, não gerar PDF incompleto como se estivesse correto.

## 28. Imagens e identidade corporativa

A engine deve suportar a identidade corporativa capturada no `DocumentModel`, inclusive logo controlado pelo Host.

Capacidade mínima do renderer:

- PNG;
- JPEG;
- SVG vetorial controlado quando o formato estiver autorizado pela política de upload da empresa.

Esta etapa não altera a política de upload da Tela 12 nem decide seus limites de bytes/dimensões. O renderer recebe somente assets já aceitos e resolvidos pelo domínio/Host, nunca paths ou URLs arbitrários vindos do documento.

Não há necessidade inicial de incorporar PDF como imagem, anexos ou conteúdo remoto.

## 29. Paginação e layout nesta etapa

A Etapa 2 consolida somente que o motor deve suportar **fluxo multipágina e quebra automática**.

Não são definidos aqui:

- margens finais;
- tamanho A4 final do Procedimento;
- regras de widow/orphan;
- manter bloco inteiro na página;
- cabeçalho/rodapé;
- número de página;
- sumário;
- espaçamentos;
- tamanho mínimo/máximo de fonte.

Esses detalhes pertencem à Etapa 5.

Mesmo assim, o renderer nunca pode resolver overflow por truncamento silencioso de conteúdo.

## 30. Determinismo e versionamento

Com a mesma versão do Host, mesmo template, mesmas fontes/assets e mesmo `DocumentModel`, o layout do PDF deve ser estável.

Conteúdo visual **não pode depender implicitamente do relógio, locale, fontes do sistema ou outro estado ambiental da máquina central**. Data/hora que apareçam no documento devem vir explicitamente do `DocumentModel`/`generation_metadata`.

Não se exige identidade byte-a-byte se metadados técnicos, como timestamp de geração, forem diferentes. O requisito é estabilidade semântica/visual sob a mesma versão do renderer.

Template e engine fazem parte da versão do Host. Não existe atualização de template pela Internet em runtime.

## 31. Recursos e falhas

A Etapa 1 já definiu limite próprio de concorrência/backpressure. Para PDF:

- falha do renderer não devolve arquivo parcial como sucesso;
- erro de fonte/asset/template incompatível é erro de geração;
- conteúdo legítimo não pode ser truncado para contornar limite;
- limites numéricos de memória, tamanho e tempo serão definidos por implementação/medição e validados na Etapa 12;
- erro técnico deve gerar log diagnóstico sem registrar o conteúdo integral do Procedimento.

## 32. Recursos PDF fora da primeira versão

Não são requisitos iniciais:

- assinatura digital do PDF;
- criptografia/senha do PDF;
- formulários editáveis;
- anexos embutidos;
- JavaScript em PDF;
- multimídia;
- certificados;
- comentários/anotações gerados pelo StepFlow;
- PDF/A formal;
- PDF/UA formal.

Esses recursos não devem ser adicionados por inferência.

## 33. Alternativas avaliadas

### `printpdf`

É uma biblioteca Rust capaz de escrever PDF, fontes, imagens, SVG e texto. Entretanto, a camada de layout automático é recente/evolutiva e a alternativa de posicionamento manual transferiria paginação e composição documental demais para código próprio do StepFlow.

### `krilla`

É uma biblioteca Rust de alto nível para primitivas PDF, fontes, imagens, tagging e padrões PDF. O próprio projeto deixa layout textual, tabelas, page breaking e cabeçalhos/rodapés fora de seu escopo; portanto exigiria outra camada completa de layout.

### Typst embutido

Fornece motor documental e exportação PDF no ecossistema Rust, com suporte a padrões PDF, texto, imagens, layout multipágina e tagging, sem exigir um executável externo quando integrado por biblioteca.

Por isso é a direção consolidada para o renderer PDF de Procedimentos do StepFlow.

## 34. Decisões consolidadas — Etapa 2

1. renderer PDF de Procedimentos baseado em **Typst embutido como biblioteca Rust no Host**, com crates oficiais e adaptador interno StepFlow;
2. nenhuma execução de `typst.exe`/CLI, browser ou processo conversor externo;
3. template Typst interno, confiável, versionado e controlado pelo produto;
4. conteúdo originado do domínio entra somente como valores/dados estruturados e nunca participa da construção textual do source Typst, mesmo após escaping;
5. sem acesso à Internet/pacotes/recursos remotos durante geração;
6. filesystem/imports do renderer restritos ao mundo virtual, assets, fontes e templates controlados pelo Host;
7. PDF 1.7 é solicitado explicitamente ao exporter;
8. Tagged PDF é explicitamente habilitado como baseline, sem prometer PDF/UA/PDF-A formal;
9. texto textual permanece selecionável/pesquisável/copiável;
10. fontes necessárias são empacotadas/incorporadas, sem depender de fontes do Windows;
11. renderer suporta os blocos semânticos do Procedimento sem descarte silencioso e falha explicitamente em incompatibilidade;
12. comandos/código permanecem texto e preservam whitespace relevante;
13. renderer suporta fluxo multipágina e quebra automática, sem definir ainda o template físico;
14. capacidade para logo/imagens controladas PNG/JPEG e SVG quando autorizado, somente a partir de assets aceitos/resolvidos pelo Host;
15. conteúdo visual não depende implicitamente do relógio/ambiente da máquina central; data/hora visível vem de dados explícitos;
16. falha de renderer não produz artefato parcial tratado como sucesso;
17. estabilidade visual/semântica sob mesma versão do Host/template/fontes/assets/modelo, sem exigir bytes idênticos quando metadados técnicos variarem;
18. assinatura, senha, formulários, anexos, JavaScript, multimídia, PDF/A e PDF/UA formais ficam fora da primeira versão;
19. versão exata das crates e limites numéricos de memória/tamanho/tempo ficam para implementação/medição e validação técnica posterior.

## 35. Fechamento da Etapa 2

A Etapa 2 está **CONSOLIDADA / APROVADA PELO PO**.

Foram cumpridos os critérios de fechamento documental e a Etapa 3 foi posteriormente aberta, analisada e consolidada abaixo. O trabalho permaneceu documental, sem código funcional, migration ou scaffold.

---

# Etapa 3 — DOCX de Procedimentos

**Status:** CONSOLIDADO / APROVADO PELO PO

## 36. Objetivo da Etapa 3

Definir somente a base técnica do renderer **DOCX de Procedimentos** e as capacidades mínimas que o artefato editável precisa preservar.

Esta etapa não redefine a arquitetura documental das Etapas 1–2 e não define margens, tipografia final, cabeçalho/rodapé final, paginação visual, A4 ou densidade. Esses pontos permanecem reservados para a **Etapa 5 — Template físico de Procedimentos**.

Também não define a impressão Windows da Etapa 4.

## 37. Contratos herdados

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

## 38. Formato DOCX consolidado

O artefato é um **DOCX real baseado em Office Open XML / WordprocessingML**, empacotado segundo Open Packaging Conventions, com MIME:

```text
application/vnd.openxmlformats-officedocument.wordprocessingml.document
```

O baseline inicial de compatibilidade é **OOXML Transitional**. OOXML Strict não é baseline da primeira versão.

O Host produz o pacote diretamente em Rust. Não executar:

- Microsoft Word;
- automação COM;
- LibreOffice;
- conversor CLI externo;
- browser/headless;
- serviço web de conversão;
- pipeline PDF → DOCX;
- pipeline Typst → DOCX.

## 39. Biblioteca Rust consolidada

A direção consolidada é usar **`docx-rs` embutido como biblioteca Rust no Host**, encapsulado por um adaptador interno StepFlow.

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
- cobre estruturas necessárias como parágrafos, runs, estilos, numeração, tabelas, imagens, seções e headers/footers;
- permite empacotamento direto do documento;
- evita que o StepFlow implemente do zero toda a mecânica OPC, relações, content-types, numbering e WordprocessingML.

Não se consolida versão exata da crate durante a Fase 1. A versão será fixada em `Cargo.lock` no gate de implementação e validada contra a matriz real de compatibilidade.

A biblioteca é dependência de implementação; o contrato arquitetural pertence ao adaptador StepFlow. Se capacidade obrigatória não puder ser representada corretamente pela versão adotada, a geração deve falhar/ser corrigida — nunca descartar conteúdo silenciosamente.

## 40. Fronteira de dados e segurança

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

Se no futuro uma extensão OOXML interna for necessária por limitação da biblioteca, ela permanece **controlada pelo código StepFlow**, coberta por testes e sem aceitar XML arbitrário do domínio.

## 41. Editabilidade como requisito central

Diferentemente do PDF, o DOCX existe para continuar editável em um processador de texto compatível.

Portanto:

- texto textual permanece texto Word real;
- títulos, parágrafos, passos e notas não podem ser rasterizados;
- comandos/código permanecem texto;
- conteúdo deve ser selecionável, pesquisável, copiável e editável;
- imagens permanecem objetos de imagem incorporados;
- nenhuma etapa conhecida pode ser convertida para screenshot como solução de layout.

A edição posterior feita pelo usuário fora do StepFlow não altera a revisão oficial do Procedimento. O DOCX exportado é artefato derivado, não fonte de importação/sincronização.

## 42. Blocos semânticos obrigatórios

O renderer DOCX representa todos os blocos já suportados pelo `DocumentModel`:

- parágrafo;
- passos/subpassos numerados;
- checklist documental;
- nota;
- alerta;
- comando;
- bloco de código.

Regras:

- passos/subpassos usam numeração/lista Word real quando aplicável;
- checklist documental usa representação visual estável de caixa + texto, sem virar formulário interativo;
- nota e alerta mantêm distinção semântica por estilos/estrutura controlada;
- comando/código usam texto com preservação de whitespace relevante;
- tipo de bloco desconhecido/incompatível falha explicitamente em vez de produzir documento incompleto.

A aparência física final desses estilos pertence à Etapa 5.

## 43. Imagens e identidade corporativa

O renderer recebe somente assets já autorizados e resolvidos pelo Host.

Baseline DOCX:

- PNG;
- JPEG.

SVG não é requisito direto do DOCX v1 porque a compatibilidade varia entre versões/consumidores. Se a fonte controlada existir apenas em SVG, o Host deve fornecer uma representação compatível por pipeline interno validado ou falhar explicitamente; nunca remover a imagem silenciosamente nem depender de conversor externo.

A política de upload/armazenamento de logo não é alterada por esta etapa.

## 44. Estilos e template interno

O DOCX usa **estilos Word internos e versionados pelo StepFlow**, produzidos pelo renderer.

A primeira versão não depende de arquivo `.docx`/`.dotx` externo usado como template em runtime.

O renderer distingue conceitualmente estilos como:

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

## 45. Fontes e reflow

DOCX é formato refluível. O StepFlow **não promete paginação idêntica ao PDF**.

Mesmo com o mesmo conteúdo, a quebra de página pode variar por:

- versão do Microsoft Word/consumidor;
- fontes disponíveis;
- métricas de fonte/substituição;
- configurações de compatibilidade do processador de texto;
- configuração de impressão do ambiente.

A Etapa 3 consolida estabilidade **semântica e estrutural**, não identidade visual página a página com o PDF.

A família tipográfica final e eventual política de incorporação de fontes no DOCX ficam para a Etapa 5/gate de implementação, considerando licenciamento e compatibilidade. A política de fontes empacotadas do PDF não é herdada automaticamente pelo DOCX.

## 46. Whitespace de comandos e código

Comando/código deve preservar:

- espaços relevantes;
- tabs quando representadas pelo modelo;
- quebras de linha;
- indentação.

O renderer usa a semântica adequada de WordprocessingML para preservar espaço textual, sem substituir o bloco por imagem.

Quebras extremamente longas/linhas extensas serão tratadas pelo template/estilo da Etapa 5; truncamento silencioso continua proibido.

## 47. Relações externas e conteúdo ativo

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

URLs presentes no conteúdo podem permanecer como texto. Hyperlink ativo só existe se o domínio vier a consolidar esse tipo semântico; não deve ser inferido automaticamente a partir de texto.

## 48. Metadados

O pacote pode registrar metadados controlados, como:

- título;
- versão/revisão exibida;
- identidade técnica do documento;
- data de geração quando explicitamente fornecida por `generation_metadata`.

Não incluir token, senha, path local, conteúdo de sessão ou dado técnico interno desnecessário.

Metadados ambientais não podem alterar conteúdo visual semântico implicitamente.

## 49. Determinismo possível no DOCX

Com a mesma versão do Host, mesmo `DocumentModel`, mesmos assets e mesma definição de estilos, o **conteúdo e estrutura OOXML** devem permanecer estáveis.

Não se exige identidade byte-a-byte do ZIP nem paginação idêntica entre diferentes consumidores Word.

IDs/relationships gerados devem ser deterministicamente controlados quando razoável para facilitar testes e diagnóstico, mas diferenças técnicas sem efeito semântico não são falha por si só.

## 50. Validação do artefato

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

## 51. Recursos e concorrência

Aplicam-se os limites consolidados na Etapa 1:

- renderer DOCX usa o mesmo mecanismo geral de limite de concorrência documental;
- não entra na fila de mutações;
- não cria job persistente;
- falha não produz artefato parcial tratado como sucesso;
- limites numéricos de memória/tamanho/tempo ficam para medição/Etapa 12;
- temporários concretos continuam reservados para a Etapa 10.

A possibilidade de `docx-rs` empacotar diretamente o documento deve ser aproveitada quando adequada, evitando manter representações duplicadas grandes em memória sem necessidade.

## 52. Alternativas consideradas

### Construir OOXML manualmente

É tecnicamente possível gerar ZIP/OPC + WordprocessingML diretamente com crates de XML/ZIP. Entretanto, isso transferiria para o StepFlow responsabilidade por relações OPC, content-types, numbering, estilos, imagens e muitos detalhes de compatibilidade. Não é a direção preferida enquanto uma biblioteca Rust madura cobrir o contrato necessário.

### Automação do Microsoft Word/COM

Rejeitada como arquitetura de geração: acopla o Host à instalação/configuração do Office, acrescenta dependência operacional e conflita com o runtime Pocket/autocontido já consolidado.

### LibreOffice/conversores externos

Rejeitados pelo mesmo contrato autocontido das Etapas 1–2.

### Converter o PDF/Typst para DOCX

Rejeitado. PDF e DOCX têm objetivos diferentes; a conversão perderia estrutura/editabilidade e criaria acoplamento indevido entre renderers.

### `docx-rs`

É a direção consolidada porque fornece uma camada Rust própria para WordprocessingML/OOXML e empacotamento DOCX sem exigir processo externo, preservando editabilidade e reduzindo código estrutural próprio.

## 53. Decisões consolidadas — Etapa 3

1. DOCX de Procedimentos é gerado diretamente pelo Host Rust a partir do mesmo `DocumentModel`, sem converter PDF/Typst;
2. saída é `.docx` OOXML/WordprocessingML real com MIME oficial;
3. **OOXML Transitional** é o baseline inicial de compatibilidade; Strict não é baseline da primeira versão;
4. `docx-rs` é a biblioteca Rust preferida, encapsulada por adaptador interno StepFlow;
5. nenhuma dependência de Word/COM, LibreOffice, browser, CLI ou cloud para gerar;
6. conteúdo do domínio entra apenas como dados estruturados e nunca como XML/OOXML arbitrário;
7. template/estilos são internos e versionados; nenhum `.docx`/`.dotx` externo fornecido pelo usuário na primeira versão;
8. texto permanece real, selecionável, pesquisável, copiável e editável;
9. todos os blocos semânticos conhecidos são representados sem descarte silencioso;
10. passos usam numeração/lista real; checklist permanece documental e não vira formulário interativo;
11. comando/código preserva whitespace e permanece texto;
12. PNG/JPEG são baseline de imagem; SVG não é requisito direto do DOCX v1 e nunca pode ser omitido silenciosamente;
13. DOCX é refluível e não promete paginação idêntica ao PDF; layout físico final continua na Etapa 5;
14. política tipográfica/embedding de fontes do DOCX não é herdada automaticamente do PDF e permanece para Etapa 5/validação;
15. macros, ActiveX, OLE, remote templates, conteúdo externo, anexos, assinatura, senha/DRM e importação de DOCX editado ficam fora da primeira versão;
16. artefato incompleto/corrompido nunca é devolvido como sucesso;
17. versão exata da crate, limites numéricos e matriz real de compatibilidade ficam para implementação/Etapa 12.

## 54. Fechamento da Etapa 3

A Etapa 3 está **CONSOLIDADA / APROVADA PELO PO** neste checkpoint.

Foram cumpridos os critérios de fechamento documental:

- decisões aprovadas e refinadas com OOXML Transitional como baseline;
- README raiz atualizado para `✅ Consolidado`;
- `AGENTS.md`, arquitetura, registro de decisões, índice e plano oficial alinhados;
- proposta temporária da Etapa 3 deixa de ser fonte ativa após sua remoção desta branch;
- Etapa 4 permanece apenas como próxima até abertura explícita;
- trabalho permanece documental, sem código funcional, dependency, migration ou scaffold.

**Etapa 4 — Impressão Windows de Procedimentos** é a próxima etapa do Bloco 10, mas **ainda não está em análise**.