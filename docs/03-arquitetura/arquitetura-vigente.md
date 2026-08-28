# Arquitetura Vigente — StepFlow Pocket

**Status:** CONSOLIDADA PARA A FASE 1, INCLUINDO BLOCO 9 E BLOCO 10 / ETAPAS 1–6  
**Atualização:** 2026-08-28

## Visão geral

```text
Ponto de entrada interno
        ↓
StepFlowLauncher.exe (transitório)
        ↓
Client Tauri local por usuário
        ↓ HTTP/JSON + WebSocket
StepFlow Host Pocket na máquina central
        ↓
SQLite local + arquivos persistentes
```

## Client

Tecnologia: **Tauri 2 + HTML/CSS/JavaScript modular**.

Responsabilidades:

- UI/navegação;
- sessão em memória;
- consumir API do Host;
- receber eventos e reconsultar estado;
- apresentar conflitos/estados transversais;
- executar contexto operacional de Atendimento sem abrir SQLite diretamente;
- iniciar geração documental e receber artefatos produzidos pelo Host;
- encaminhar artefatos para fluxos locais de salvar/preview/impressão conforme os contratos específicos;
- realizar impressão física no contexto local da estação Windows usando o PDF oficial recebido do Host.

Direção visual transversal: privilegiar clareza com baixa densidade textual permanente, usando cor, forma, símbolo, posição e ícones reconhecíveis quando isso simplificar sem gerar ambiguidade; detalhes secundários podem aparecer sob demanda e cor nunca é o único meio para estado importante. O Reader usa stepper compacto navegável de círculos/linhas, separado do progresso operacional de checklist.

Baseline inicial: Windows 10/11 x64 + WebView2. Validação corporativa ainda pendente.

## Launcher

Launcher Rust x64 portátil/transitório:

1. lê manifesto/configuração;
2. compara versão;
3. prepara `%LOCALAPPDATA%\StepFlow\Client\versions\<versao>\`;
4. valida SHA-256;
5. inicia Client local;
6. encerra.

Sem updater residente.

## Host Pocket

Tecnologia: Rust + Tokio/Axum + `rusqlite` bundled.

- Controller: lifecycle central, paths/config, instância única, readiness/shutdown;
- Host: autenticação, autorização, API, eventos, SQLite, writer/fila, revisões, Atendimentos, Equipamentos, checklist, observações de serviço por Etapa, auditoria, backup/restore e geração documental.

Sem Windows Service, Task Scheduler, auto-start, watchdog ou daemon StepFlow como padrão.

Fechar um Client individual não encerra o Host. Encerrado o ciclo central, nenhum processo StepFlow permanece ativo.

## Domínio funcional

```text
Procedimento
   ↓ usado em
Atendimento/Execução
   ↓ opcionalmente relacionado a
Equipamento
```

Consolidado:

- Procedimentos oficiais/revisões imutáveis;
- categorias configuráveis/múltiplas;
- Atendimentos como ocorrências reais;
- Equipamento opcional/reutilizável;
- busca documental separada da operacional;
- múltiplas revisões de Procedimento por Atendimento;
- revisão exata usada preservada;
- checklist persistente somente em contexto de Atendimento;
- observação de serviço opcional por Etapa somente em contexto de Atendimento;
- ficha compacta com ou sem Equipamento;
- Ficha como prestação de contas resumida ao cliente;
- identidade da empresa centralizada;
- Backup/Restore administrativo;
- exportação contextual pela revisão/estado selecionado.

## Lifecycle operacional — Bloco 9

Estados iniciais:

```text
Em andamento
Concluído
Cancelado
```

Fluxo:

```text
rascunho Client
→ primeiro save aceito pelo Host
→ Em andamento
   ├─→ Concluído
   └─→ Cancelado

Concluído/Cancelado
→ Reabrir
→ Em andamento
```

Regras:

- rascunho inicial não persiste;
- Host gera `AT-000001` no primeiro save;
- conclusão exige responsável + resumo do trabalho;
- checklist incompleto gera confirmação, não bloqueio automático;
- cancelamento exige motivo curto;
- Concluído/Cancelado não aceitam edição direta;
- mudança posterior exige reabertura;
- lifecycle é auditável/versionado;
- estado final necessário para reimpressão histórica não pode ser reescrito silenciosamente após reabertura.

## Equipamento operacional

Código legível inicial:

```text
EQP-000001
```

- Host-only;
- ID interno continua canônico;
- serial/MAC/patrimônio são atributos de busca;
- múltiplos MACs;
- não arquivar Equipamento ligado a Atendimento `Em andamento`;
- Funcionário pode criar/editar Equipamento por preset;
- arquivar/reativar: ADM/Gerência.

Ao concluir Atendimento, o Host preserva uma projeção histórica relevante do Equipamento. Alteração futura do cadastro global não reescreve ficha/histórico concluído.

## Checklist e observações de serviço

A definição documental continua em `process_revision`/blocos imutáveis.

Estado de execução é separado e ligado a `service_record_process`.

### Checklist

- somente contexto de Atendimento persiste marcações;
- Reader standalone continua documental;
- progresso = itens marcados / total;
- etapa visitada não é progresso;
- 100% não conclui Atendimento automaticamente;
- checklist concluído/cancelado é somente leitura até reabertura;
- concorrência por item/equivalente evita conflito global desnecessário.

### Observação do serviço por Etapa

- cada Etapa da revisão vinculada pode receber texto opcional de execução;
- pertence ao Atendimento + vínculo da revisão + Etapa, não ao Procedimento oficial;
- Reader standalone não cria/persiste esse dado;
- fica editável somente enquanto o Atendimento estiver editável e autorizado;
- Concluído/Cancelado tornam a observação somente leitura até reabertura;
- concorrência usa granularidade por Etapa/equivalente;
- observações de Etapas independentes não devem conflitar globalmente;
- conclusão precisa preservar estado histórico suficiente para reimpressão da Ficha aplicável;
- não há autosave por inferência.

## Responsabilidade e permissões operacionais

Autorização continua Host-side e por capacidade.

Preset:

- ADM/Gerência: podem criar/editar/concluir/cancelar/reabrir Atendimentos acessíveis;
- Funcionário: cria e opera Atendimento do qual é responsável, podendo concluir o próprio;
- Funcionário não cancela/reabre por preset;
- Gerência gere categorias por preset;
- Funcionário seleciona revisão publicada para execução;
- ADM/Gerência podem selecionar explicitamente outras revisões autorizadas;
- gerar/reimprimir ficha: sim para os três presets em Atendimento acessível.

Continuam pendentes:

- Gerência × configuração da empresa;
- Gerência × Backup.

## Persistência

```text
StepFlow\
├── app\
├── config\
├── data\
│   ├── stepflow.sqlite
│   ├── company\
│   └── avatars\
├── logs\
└── backups\
```

Princípios:

- SQLite local ao Host;
- foreign keys;
- WAL;
- migrations versionadas;
- revisões de Procedimento imutáveis;
- `revision_no` separado de `display_version`;
- auditoria append-only;
- categorias/Equipamentos/Atendimentos no schema conceitual;
- checklist e observações de serviço operacionais separados do snapshot documental;
- logo/avatar como arquivos controlados pelo Host;
- dados/config não são substituídos com binários.

## Comunicação

- HTTP/JSON versionado, inicialmente `/api/v1`;
- WebSocket autenticado para eventos;
- `deployment.json` sem segredos;
- handshake de compatibilidade antes do login;
- sem edição offline;
- evento = sinal de mudança, não estado oficial completo;
- WebSocket degradado com HTTP saudável não implica Host totalmente indisponível;
- reconexão faz reconsulta/reconciliação.

## Concorrência

- WAL;
- writer lógico coordenado;
- fila bounded/backpressure;
- revisão otimista por recurso;
- `409` para base obsoleta;
- constraints SQLite como última defesa;
- eventos pós-commit;
- sem soft/hard lock inicial;
- dois Hosts não usam o mesmo data dir;
- checklist usa granularidade por item/equivalente;
- observação de serviço usa granularidade por Etapa/equivalente;
- Atendimento/Equipamento têm revisões separadas;
- timeout após mutação exige reconciliação, não retry cego;
- geração documental é leitura derivada e não passa pela fila de mutações;
- renderização documental usa limite próprio de concorrência/backpressure.

## Autenticação

- Argon2id;
- sessão opaca server-side;
- token em memória;
- capacidade Host-side;
- ADM/Gerência/Funcionário como presets;
- Gerência não administra ADM;
- bootstrap do primeiro ADM é local/controlado;
- sessão expirada exige nova autenticação;
- troca da própria senha mantém sessão atual e revoga demais sessões da conta.

Parâmetros numéricos finais ainda pendentes antes da implementação.

## Exportação e impressão

### Arquitetura de geração documental — Bloco 10 / Etapa 1

Consolidado:

```text
Client
  ↓ solicita por identidade da fonte + revisão esperada
Host
  ↓ autentica/autoriza
  ↓ captura snapshot consistente
  ↓ materializa DocumentModel semântico
  ↓ encerra leitura/transação SQLite
  ↓ renderiza fora da fila de mutações
  ↓ devolve artefato pela API autenticada
Client
  ↓ recebe
  └─→ destino local/preview/impressão conforme etapas específicas
```

Regras:

- geração é responsabilidade do Host;
- Client não envia documento montado nem usa DOM/HTML como fonte;
- fonte mutável usa revisão esperada; estado mais novo não substitui silenciosamente o confirmado pelo Client;
- `DocumentModel` semântico separa domínio de renderers;
- renderers não reconsultam SQLite nem reconstruem regras de negócio;
- captura consistente termina antes do trabalho pesado de renderização;
- geração não cria revisão, não altera Atendimento/checklist/observações de serviço e não muda `updated_at` funcional;
- limite de renderização é separado do writer;
- primeira versão não cria `export_jobs`, scheduler ou fila persistente documental;
- artefato retorna pela API autenticada; Host não escreve em path arbitrário do Client;
- artefatos gerados não viram histórico/backup por padrão;
- runtime documental não depende de Office, LibreOffice, Adobe Reader, Chrome/Chromium externo headless, `wkhtmltopdf` ou serviço cloud obrigatório;
- bibliotecas compiladas com o Host podem ser usadas;
- endpoints e parâmetros exatos pertencem às etapas/implementação correspondentes.

### PDF de Procedimentos — Bloco 10 / Etapa 2

Consolidado:

- renderer PDF usa **Typst embutido como biblioteca Rust no Host**, com crates oficiais e adaptador interno StepFlow;
- nenhuma execução de `typst.exe`/CLI, browser ou processo conversor externo;
- template Typst é interno, confiável e versionado com o produto;
- conteúdo do domínio entra no renderer somente como valores/dados estruturados e nunca participa da construção textual do source Typst;
- o mundo virtual do renderer restringe imports/filesystem aos templates, fontes e assets controlados pelo Host; não resolve pacotes, URLs ou recursos remotos em runtime;
- exporter solicita explicitamente **PDF 1.7** e mantém **Tagged PDF habilitado** como baseline;
- Tagged PDF não equivale nem promete conformidade formal PDF/UA ou PDF/A;
- texto originado como texto permanece selecionável, pesquisável e copiável;
- fontes necessárias são empacotadas e incorporadas/subsetadas, sem depender das fontes instaladas no Windows;
- todos os blocos semânticos conhecidos devem ser representados; tipo desconhecido/incompatível falha explicitamente em vez de ser descartado;
- comandos e código permanecem texto e preservam whitespace relevante;
- engine suporta fluxo multipágina e quebra automática; formato físico, margens, tipografia, cabeçalho/rodapé e paginação visual seguem a Etapa 5;
- PNG/JPEG e SVG controlado podem ser incorporados somente a partir de assets previamente aceitos/resolvidos pelo Host;
- conteúdo visual não pode depender implicitamente de relógio/ambiente da máquina central; data/hora visível vem de dados explícitos do `DocumentModel`/`generation_metadata`;
- mesma versão do Host/template/fontes/assets/modelo deve manter estabilidade visual/semântica, sem exigir identidade byte-a-byte quando metadados técnicos variarem;
- falha do renderer não devolve artefato parcial como sucesso;
- assinatura digital, senha, formulários, anexos, JavaScript, multimídia, PDF/A formal e PDF/UA formal não são requisitos da primeira versão.

A versão exata das crates não é consolidada na Fase 1; será fixada no `Cargo.lock` e validada no gate técnico de implementação. Limites numéricos de memória, tamanho, tempo e concorrência continuam dependentes de medição/validação posterior.

### DOCX de Procedimentos — Bloco 10 / Etapa 3

Consolidado:

- renderer DOCX usa o mesmo `DocumentModel` e gera `.docx` diretamente no Host Rust, sem converter PDF ou Typst;
- o artefato é OOXML/WordprocessingML real, empacotado segundo OPC, com **OOXML Transitional** como baseline de compatibilidade;
- `docx-rs` é a biblioteca preferida, encapsulada por adaptador interno StepFlow para evitar acoplamento do domínio;
- não executar Microsoft Word/COM, LibreOffice, browser/headless, CLI conversor ou serviço cloud;
- conteúdo do domínio entra somente como dados estruturados e não pode injetar XML/OOXML, relationships, partes OPC, paths ou URLs arbitrários;
- estilos/template são internos e versionados pelo StepFlow; a primeira versão não aceita `.docx`/`.dotx` externo como template em runtime;
- texto permanece texto Word real, selecionável, pesquisável, copiável e editável; imagens permanecem objetos incorporados;
- todos os blocos semânticos conhecidos devem ser representados; incompatibilidade falha explicitamente em vez de descartar conteúdo;
- passos/subpassos usam numeração/lista Word real quando aplicável;
- checklist exportado é documental e não vira formulário/content control interativo;
- comandos e código permanecem texto e preservam espaços, tabs, quebras e indentação relevantes;
- PNG/JPEG são baseline; SVG não é requisito direto do DOCX v1 e precisa de representação interna compatível ou falha explícita, nunca omissão silenciosa;
- DOCX é formato refluível: estabilidade exigida é semântica/estrutural, não paginação idêntica ao PDF ou entre consumidores Word;
- DOCX v1 referencia **Arial** para texto e **Consolas** para comando/código, sem embedding/redistribuição dessas fontes pelo StepFlow;
- relationships externos, macros/VBA/`.docm`, ActiveX, OLE, remote templates, conteúdo externo, anexos, assinatura digital, senha/DRM e importação de DOCX editado não são requisitos da primeira versão;
- mesmo modelo/assets/estilos/versão do Host devem manter conteúdo e estrutura OOXML estáveis quando razoável, sem exigir ZIP byte-a-byte idêntico;
- geração só é sucesso com pacote DOCX completo e coerente; validação posterior deve incluir OPC/ZIP, XML/relationships e abertura sem reparo na matriz corporativa real.

Versão exata da crate, limites numéricos e matriz de compatibilidade real permanecem para implementação/Etapa 12. Layout físico segue a Etapa 5 consolidada.

### Impressão Windows de Procedimentos — Bloco 10 / Etapa 4

Consolidado:

```text
Leitor
→ Imprimir
→ Client solicita a revisão esperada
→ Host autentica/autoriza e gera o PDF oficial
→ Client recebe os bytes
→ recurso local transitório controlado
→ WebView2 dedicada/transitória
→ ShowPrintUI(System)
→ diálogo de impressão do Windows
```

Regras:

- impressão física acontece no Client Windows da estação do usuário, não no Host central;
- o artefato canônico de impressão é exatamente o PDF produzido pelo renderer da Etapa 2 para a revisão exata selecionada;
- não existe renderer separado de impressão, não imprimir HTML da UI e não usar DOCX como fonte física;
- a webview principal não é navegada para o PDF; uma WebView2 transitória/dedicada recebe somente o recurso local controlado;
- baseline usa WebView2 `ShowPrintUI(System)` por adaptador Windows isolado sob Tauri `with_webview`;
- o diálogo de impressão do Windows é a superfície padrão; não criar seletor próprio de impressoras nem impressão silenciosa na primeira versão;
- StepFlow não enumera/persiste impressoras no Host e não gerencia drivers, descoberta ou spooler corporativo;
- não usar `ShellExecute`/handler `.pdf`, Word/COM, LibreOffice, browser externo, visualizador PDF externo ou spool PDF bruto como baseline/fallback silencioso;
- o recurso local é transitório; estratégia em memória/arquivo, nomes, paths e limpeza concreta pertencem à Etapa 10;
- abrir `ShowPrintUI` significa que o fluxo foi entregue ao Windows, não que houve impressão física; a UI não pode afirmar `Impresso com sucesso` nem gravar `printed=true` por inferência;
- fechamento/cancelamento do diálogo não é erro funcional; falhas de geração, preparação local, compatibilidade WebView2 e abertura do diálogo são classes distintas;
- a implementação impede duplicidade concorrente acidental local da mesma ação sem criar job/fila persistente;
- a webview de impressão não busca Internet, não recebe token/senha em URL nem path/HTML arbitrário originado do conteúdo;
- gate técnico posterior valida Windows 10/11 x64, runtime WebView2, PDF multipágina, Unicode/logo, impressoras locais/de rede, opções do diálogo e operação offline;
- incompatibilidade de WebView2 deve ser explícita, sem fallback silencioso para software externo;
- versão mínima concreta de WebView2 fica para matriz corporativa/gate de implementação;
- impressão usa o template físico consolidado na Etapa 5, sem criar layout alternativo.

### Template físico de Procedimentos — Bloco 10 / Etapa 5

Consolidado:

#### Separação Reader × documento físico

- a página lógica do Reader não possui geometria A4 e não é preview do documento exportado;
- `Visão geral` permanece a primeira página lógica do Reader;
- cada `process_stage` permanece uma página lógica própria;
- o stepper do Reader é horizontal, compacto e navegável, composto prioritariamente por círculos/linhas;
- estados anteriores/atual/seguintes usam preenchimento, contraste, forma/símbolo e cor sem depender apenas de cor;
- nomes das Etapas não precisam ser repetidos permanentemente no stepper;
- estado anterior no stepper representa percurso de navegação, nunca conclusão operacional/checklist.

#### Documento físico

- Procedimento completo usa **A4 retrato multipágina**;
- margens-base de **18 mm** em todos os lados;
- não há capa exclusiva: identificação e conteúdo útil começam na primeira página;
- v1 não exige sumário documental físico por padrão;
- títulos de Etapa são compactos, por exemplo `01 · Preparação`;
- uma Etapa não força automaticamente nova folha; título fica com o primeiro conteúdo quando possível;
- não há cabeçalho repetitivo nas páginas internas;
- rodapé curto identifica código/revisão e paginação;
- parágrafos não viram cards; passos usam numeração/indentação; checklist usa `□`; notas/alertas usam distinção semântica discreta; comando/código usam bloco monoespaçado compacto;
- imagens preservam proporção, sem crop automático;
- paginação é automática, evitando widow/orphan e títulos isolados; bloco longo pode quebrar, mas conteúdo nunca é truncado ou reduzido silenciosamente para caber.

#### Tipografia

- PDF usa **Noto Sans + Noto Sans Mono** empacotadas com o Host e incorporadas/subsetadas pelo renderer controlado, respeitando OFL 1.1;
- DOCX usa **Arial + Consolas** referenciadas, sem embedding ou redistribuição dessas fontes pelo StepFlow v1;
- baseline: título 18 pt, Etapa 14 pt, corpo 10,5 pt, comando/código 9 pt e rodapé 8 pt;
- PDF é a referência física de impressão; DOCX compartilha hierarquia/ordem/semântica, mas permanece refluível e não promete paginação idêntica.

O limite rígido de **uma página A4** pertence à Ficha compacta de Atendimento, não ao Procedimento completo.

### PDF + preview da Ficha compacta — Bloco 10 / Etapa 6

Consolidado:

#### Finalidade e conteúdo

- Ficha é prestação de contas resumida ao cliente;
- prioriza identificação do serviço, identificação/características do dispositivo, `Resumo do trabalho` e observações relevantes;
- características podem incluir processador, RAM, armazenamento, SO quando útil, bateria quando aplicável e observações do Equipamento;
- observações de serviço registradas nas Etapas entram quando preenchidas;
- checklist, progresso, passos, comandos, timeline, IDs internos e lista detalhada de revisões não aparecem por padrão;
- dados do Equipamento vêm do cadastro/snapshot histórico aplicável, sem redigitação para gerar a Ficha.

#### PDF e preview

```text
Atendimento confirmado + source_version esperada
→ DocumentModel document_kind = service_sheet
→ template Typst próprio da Ficha
→ PagedDocument
→ exigir exatamente 1 página
   ├─→ typst-pdf → PDF canônico
   └─→ typst-svg → preview vetorial
```

- mesma infraestrutura Typst embutida no Host, com template específico da Ficha;
- PDF 1.7 + Tagged PDF como baseline estrutural, sem promessa formal PDF/A ou PDF/UA;
- PDF e SVG derivam do mesmo `PagedDocument`, evitando segundo layout de preview;
- `2+ páginas` resulta em `SHEET_OVERFLOW`; não cortar, não retornar só a primeira página e não reduzir fonte silenciosamente;
- preview SVG é tratado como representação visual controlada, sem script/navegação externa;
- preview abre em modal/overlay grande no Client, folha A4 centralizada, sem nova sidebar ou toolbar textual extensa;
- `Salvar PDF` e `Imprimir` reutilizam os mesmos bytes PDF correspondentes à prévia;
- impressão reutiliza WebView2 transitória + `ShowPrintUI(System)`;
- resultado PDF + SVG é transitório; não cria job persistente nem histórico/backup automático;
- prévia fica presa à `source_version`; atualização remota não troca a folha silenciosamente e exige regeneração antes de nova saída atual.

#### Lifecycle histórico

- `Em andamento`: Ficha usa estado confirmado atual;
- `Concluído`: reimpressão usa estado histórico aplicável;
- `Cancelado`: saída identifica claramente o status;
- reabertura não pode reescrever silenciosamente Ficha histórica anterior;
- snapshot/projeção de Equipamento e observações de serviço precisam participar da reprodução histórica aplicável.

Etapas 7–12 continuam responsáveis por template A4 final da Ficha, limites/densidade, múltiplos MACs/Procedimentos, nomes/temporários, QR/barcode e validação técnica final. **Etapa 7 é a próxima e permanece fechada até o gate Git da Etapa 6.**

## Backup / Restore

UX normal já consolidada:

- dentro de Configurações;
- Client não copia SQLite nem escolhe path;
- backups conhecidos pelo Host;
- Restore exige autorização + backup elegível + confirmação reforçada;
- Restore normal exige safety backup do estado atual antes da etapa destrutiva;
- disaster recovery sem Host funcional fica fora da UI normal.

Bloco 11 ainda fecha pacote, atomicidade, checksums, retenção, restart/reconexão, sessões e recuperação local.

## Estados transversais

- menor superfície adequada: campo → seção → página → Shell;
- sem indicador permanente de conexão saudável;
- loading não apresenta cache antigo como atual;
- `sem registros` ≠ `sem resultados`;
- perda de permissão limpa conteúdo protegido;
- conflito preserva edição local;
- incompatibilidade Client↔Host bloqueia uso;
- sem offline queue/autosave/draft persistente;
- reduzir densidade textual permanente quando forma, símbolo, posição ou ícone comunicarem com clareza;
- não sacrificar acessibilidade/entendimento por minimalismo e não usar cor isoladamente como semântica crítica.

## Ambiente corporativo pendente

- hostname/IP/paths reais;
- SMB/permissões;
- Windows/WebView2 reais;
- HTTP/HTTPS;
- antivírus/EDR/firewall;
- mecanismo real de start do Controller.

Exemplos históricos não podem virar hardcode.

## Estado da Fase 1

- Blocos 0–4: concluídos;
- Bloco 5: núcleo concluído, parâmetros finais pendentes;
- Bloco 6: núcleo + extensão operacional conceitual consolidados;
- Bloco 7: núcleo concluído;
- Bloco 8: concluído;
- Bloco 9: concluído documentalmente;
- **Bloco 10: em andamento — Etapas 1–6 consolidadas; Etapa 7 próxima, ainda não aberta**;
- Blocos 11–12: pendentes.

Nenhum runtime/código funcional oficial foi criado durante esse fechamento documental.