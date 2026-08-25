# Bloco 10 — Exportação / Impressão + Ficha Compacta

**Status:** EM ANDAMENTO — ETAPA 1 CONSOLIDADA / ETAPA 2 EM ANÁLISE  
**Fase:** Fase 1 — Fechamento arquitetural e especificação  
**Abertura:** 2026-08-25  
**Etapa 1 consolidada:** 2026-08-25

## 1. Objetivo do bloco

Fechar, uma etapa por vez, o contrato técnico de geração documental do StepFlow, preservando o caráter Pocket e a UX já aprovada no Bloco 8.

Este documento é o mapa técnico do Bloco 10. Apenas a etapa explicitamente marcada como `EM ANÁLISE` pode receber decisões novas. Não pertence a este bloco implementar código de produção, fechar Backup/Restore técnico ou abrir o Bloco 11.

## 2. Etapas do Bloco 10

| Ordem | Etapa | Estado |
|---|---|---|
| 1 | Arquitetura de geração documental | **CONSOLIDADO / APROVADO PELO PO** |
| 2 | PDF de Procedimentos | **EM ANÁLISE / PROPOSTA PARA APROVAÇÃO DO PO** |
| 3 | DOCX de Procedimentos | PENDENTE |
| 4 | Impressão Windows de Procedimentos | PENDENTE |
| 5 | Template físico de Procedimentos | PENDENTE |
| 6 | PDF + preview da Ficha compacta | PENDENTE |
| 7 | Template físico A4 da Ficha | PENDENTE |
| 8 | Limites textuais e densidade da Ficha | PENDENTE |
| 9 | Múltiplos MACs / Procedimentos na Ficha | PENDENTE |
| 10 | Nomes de arquivo + artefatos temporários | PENDENTE |
| 11 | QR / barcode | PENDENTE |
| 12 | Validação técnica final do Bloco 10 | PENDENTE |

As Etapas 3–12 permanecem pendentes e não estão em análise neste trabalho.

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

**Status:** EM ANÁLISE / PROPOSTA PARA APROVAÇÃO DO PO

## 22. Objetivo da Etapa 2

Definir somente a base técnica do renderer PDF de Procedimentos e as capacidades mínimas que o PDF precisa preservar.

Esta etapa **não define** margens, tamanhos de fonte, cabeçalho/rodapé, paginação visual, densidade, layout final ou impressão Windows. Esses pontos permanecem nas Etapas 4 e 5.

## 23. Engine PDF proposta

A proposta é usar **Typst embutido como biblioteca Rust no Host**, sem executar `typst.exe`/CLI e sem depender de instalação externa.

A integração deve utilizar crates Rust do ecossistema oficial do Typst para compilar/renderizar o documento dentro do processo do Host. O wrapper concreto poderá usar a camada oficial de embedding disponível e estável no momento da implementação, desde que preserve este contrato.

Não consolidar uma versão exata de crate durante a Fase 1. Na implementação, versões serão fixadas em `Cargo.lock` e validadas pelo gate técnico correspondente.

### Por que essa direção

O StepFlow precisa de documento multipágina com fluxo de texto, quebras automáticas, Unicode, imagens, estrutura semântica e PDF final reproduzível sem navegador/conversor externo.

A engine escolhida deve entregar layout documental de alto nível. Não queremos construir no Host um motor próprio de paginação, medição de parágrafos e quebras apenas para escrever primitivas PDF.

## 24. Template interno confiável

O renderer PDF usa **template Typst interno, versionado com o Host**.

O conteúdo do Procedimento não é concatenado como código Typst.

Fluxo conceitual:

```text
DocumentModel
→ serialização estruturada controlada
→ template interno confiável
→ engine Typst embutida
→ PDF bytes
```

Regras de segurança:

- texto vindo de usuário é tratado como dado, nunca como fonte Typst executável;
- não permitir `#eval`, imports, paths ou código arbitrário originado do conteúdo do Procedimento;
- não resolver pacotes Typst pela Internet em runtime;
- não permitir template escolhido pelo usuário na primeira versão;
- assets e templates necessários ficam empacotados/versionados com o produto;
- acesso a filesystem do renderer fica restrito ao mundo/projeto virtual controlado pelo Host;
- nenhuma URL remota é buscada durante a geração.

A forma concreta de ponte estruturada pode ser JSON/estrutura virtual equivalente, desde que o conteúdo seja escapado e não seja transformado em código por concatenação de strings.

## 25. Contrato do PDF gerado

O renderer deve produzir PDF válido e autocontido com MIME `application/pdf`.

Baseline proposto:

- **PDF 1.7** para a primeira versão;
- texto real, selecionável, pesquisável e copiável quando a origem é textual;
- fontes necessárias incorporadas/subsetadas no PDF;
- Unicode adequado para português e caracteres técnicos usados pelos Procedimentos;
- imagens/logo controlados podem ser incorporados sem dependência externa;
- documento multipágina com quebra automática;
- metadados básicos do documento podem ser gravados no PDF;
- Tagged PDF permanece habilitado como baseline de acessibilidade quando suportado pela engine.

A primeira versão **não promete conformidade formal PDF/UA ou PDF/A** sem validação específica. Tagged PDF é baseline, não certificação.

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

Capacidade mínima desejada do renderer:

- PNG;
- JPEG;
- SVG vetorial controlado quando o formato estiver autorizado pela política de upload da empresa.

Esta etapa não altera a política de upload da Tela 12 nem decide seus limites de bytes/dimensões. O renderer recebe somente assets já aceitos pelo domínio/Host.

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

Não se exige identidade byte-a-byte se metadados como timestamp de geração forem diferentes. O requisito é estabilidade semântica/visual sob a mesma versão do renderer.

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

Por isso é a proposta preferida para o StepFlow.

## 34. Decisões propostas ao PO — Etapa 2

A Etapa 2 propõe consolidar somente:

1. renderer PDF de Procedimentos baseado em **Typst embutido como biblioteca Rust no Host**;
2. nenhuma execução de `typst.exe`/CLI ou processo conversor externo;
3. template Typst interno, versionado e controlado pelo produto;
4. conteúdo de usuário entra como dados estruturados/escapados e nunca como código Typst concatenado;
5. sem acesso à Internet/pacotes remotos durante geração;
6. filesystem/imports do renderer restritos aos assets/templates controlados pelo Host;
7. baseline PDF 1.7;
8. texto textual permanece selecionável/pesquisável/copiável;
9. fontes necessárias são empacotadas/incorporadas, sem depender de fontes do Windows;
10. renderer suporta os blocos semânticos do Procedimento sem descarte silencioso;
11. renderer suporta fluxo multipágina e quebra automática, sem definir ainda o template físico;
12. capacidade para logo/imagens controladas PNG/JPEG e SVG quando autorizado;
13. Tagged PDF fica habilitado como baseline quando suportado, sem prometer PDF/UA/PDF-A formal;
14. falha de renderer não produz artefato parcial tratado como sucesso;
15. estabilidade visual/semântica sob mesma versão do Host/template/fonts, sem exigir bytes idênticos;
16. assinatura, senha, formulários, anexos, JavaScript, multimídia, PDF/A e PDF/UA formais ficam fora da primeira versão.

## 35. Critério de fechamento da Etapa 2

A Etapa 2 só pode mudar para `CONSOLIDADO / APROVADO PELO PO` quando:

- as decisões propostas forem aprovadas ou ajustadas;
- README raiz marcar Etapa 2 como `✅ Consolidado` no mesmo checkpoint;
- arquitetura/registro/plano afetados forem alinhados;
- Etapa 3 continuar apenas como próxima até ser aberta explicitamente;
- o PR continuar documental, sem código funcional, migration ou scaffold.

Até lá, a Etapa 2 permanece proposta.