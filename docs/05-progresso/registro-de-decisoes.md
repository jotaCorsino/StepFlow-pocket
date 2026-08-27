# Registro de Decisões — StepFlow Pocket

**Atualização:** 2026-08-27

Este arquivo registra decisões vigentes e pendências atuais. Propostas não aprovadas não podem ser tratadas como contrato.

## 1. Governança

- GitHub é a fonte operacional de verdade durante o fechamento documental restante da Fase 1;
- checkout local `C:\dev\StepFlow` será sincronizado explicitamente antes do primeiro trabalho de implementação com Codex;
- alterações locais preexistentes do PO devem ser preservadas;
- uma tarefa lógica por vez;
- uma branch ativa por trabalho;
- um PR por trabalho;
- revisão/aprovação → squash/merge → apagar branch encerrada → verificar remoto limpo → próximo trabalho;
- branch mergeada não é considerada encerrada enquanto permanecer no remoto;
- `AGENTS.md` é a regra operacional superior;
- todo avanço consolidado de fase/bloco/tela/etapa atualiza o README raiz no mesmo checkpoint;
- toda tarefa Codex futura exige pré-flight de capacidade separado do prompt;
- Fase 1 não autoriza runtime/scaffold/código de negócio oficial antes do gate correspondente.

## 2. Papéis

- PO: requisitos, prioridade, regra de negócio e aprovação final;
- Assistente: análise, arquitetura, coerência documental, gates e tarefas fechadas;
- Codex: implementação de tarefa pequena/aprovada, sem inventar produto.

## 3. Produto

StepFlow é aplicação interna para documentar, consultar, versionar e executar procedimentos técnicos de forma guiada.

Uso amplo:

- manutenção;
- TI;
- Service Desk/Help Desk;
- infraestrutura/servidores;
- redes;
- procedimentos internos;
- guias técnicos.

Não transformar por inferência em:

- CRM;
- financeiro/faturamento;
- estoque;
- RMM;
- sistema completo de chamados/SLA.

## 4. Modelo de domínio

Consolidado:

```text
Procedimento
   ↓ usado em
Atendimento/Execução
   ↓ opcionalmente relacionado a
Equipamento
```

- Procedimento = documentação/modelo oficial;
- Atendimento = ocorrência real de trabalho;
- Equipamento = ativo físico opcional e reutilizável;
- Atendimento pode existir sem Equipamento;
- Atendimento pode usar zero, um ou vários Procedimentos;
- vínculo preserva revisão exata realmente utilizada;
- alteração futura do Procedimento não reescreve histórico do Atendimento.

## 5. Categorias

- configuráveis pela empresa;
- múltiplas por Procedimento;
- simples, sem árvore hierárquica inicial;
- pesquisáveis/filtráveis;
- arquivamento preserva histórico;
- nomes equivalentes normalizados devem ser evitados;
- gestão por preset: ADM e Gerência;
- Funcionário não gere categorias por preset;
- autorização real continua Host-side/capability-based.

Pendente:

- regra editorial de nova revisão de Procedimento ainda referenciando categoria arquivada.

## 6. Campos principais do Procedimento

- Código;
- Título;
- Área/Departamento;
- Responsável;
- Status;
- Versão;
- Objetivo;
- Observações;
- Pré-requisitos;
- Categorias;
- Etapas;
- Histórico.

## 7. Reader / manual

- experiência principal em formato livro/manual;
- `Visão geral` antes da Etapa 1, como primeira página lógica não numerada;
- uma Etapa = uma página lógica;
- `Sumário` temporário;
- `Anterior`/`Próxima`;
- indicador textual compacto `Etapa X de Y`;
- stepper horizontal compacto de círculos/linhas, navegável diretamente entre Etapas;
- stepper diferencia Etapas anteriores/atual/seguintes por preenchimento, contraste, forma/símbolo e cor, sem depender apenas de cor;
- nomes das Etapas não precisam ser repetidos permanentemente junto a todos os marcadores; permanecem no título da página e no Sumário;
- cada marcador possui acionamento por clique/teclado e nome acessível/tooltip quando necessário;
- estado anterior no stepper significa percurso de navegação, nunca conclusão operacional do Atendimento;
- blocos tipados: paragraph, numbered_steps, checklist, note, warning, command, code;
- sem HTML arbitrário;
- comando/código preserva whitespace e nunca é executado;
- copiar usa botão icon-only acessível + feedback curto;
- revisão aberta permanece estável quando nova revisão aparece;
- revisão histórica recebe identificação persistente;
- UI busca clareza com baixa densidade textual permanente: cor, forma, símbolo, posição e ícones podem substituir texto repetitivo quando o significado continuar claro; detalhes secundários podem aparecer sob demanda.

### Reader standalone

- checklist é documental;
- marcação não persiste execução;
- navegação/stepper não grava progresso operacional.

### Reader em Atendimento

- cabeçalho identifica `Executando no atendimento AT-...`;
- revisão fica presa ao vínculo;
- checklist é persistente;
- voltar retorna ao Atendimento;
- lifecycle do Atendimento controla editabilidade do checklist;
- stepper permanece navegação documental e não substitui a contagem operacional de checklist.

## 8. Editor e revisões de Procedimento

- Editor = `Informações` + `Etapas`;
- painel local `Estrutura`, sem segunda sidebar global;
- categorias existentes, sem criação inline;
- blocos tipados apenas;
- salvamento explícito, sem autosave inicial;
- cada save aceito cria revisão imutável;
- `base_revision`/controle otimista;
- `409` preserva alterações locais;
- sem merge automático;
- publicar é ação separada de salvar;
- revisão histórica nunca é alterada/destruída;
- `Criar nova revisão a partir desta` cria novo trabalho baseado no snapshot antigo;
- `revision_no` técnico separado de `display_version` editorial.

## 9. Atendimentos — lifecycle do Bloco 9

Estados iniciais:

```text
Em andamento
Concluído
Cancelado
```

Fluxo:

```text
rascunho Client
→ primeiro save aceito
→ Em andamento
   ├─→ Concluído
   └─→ Cancelado

Concluído/Cancelado
→ Reabrir
→ Em andamento
```

- abrir tela nova não persiste registro;
- primeiro save cria ID, código e `started_at`;
- não existe draft persistente inicial;
- Concluído/Cancelado são read-only operacionalmente;
- correção posterior exige reabertura;
- lifecycle é auditável/versionado.

## 10. Códigos legíveis

Consolidado:

```text
Atendimento: AT-000001
Equipamento: EQP-000001
```

- Host-only;
- seis dígitos;
- sequência simples por implantação/banco ativo;
- gaps permitidos;
- não editáveis;
- não substituem IDs internos estáveis;
- sem ano/departamento/hostname/dado pessoal inicialmente.

## 11. Responsável do Atendimento

- necessário para conclusão;
- Funcionário cria inicialmente para si;
- Funcionário padrão edita/conclui apenas Atendimento do qual é responsável;
- Funcionário não reatribui para outro usuário;
- ADM/Gerência podem atribuir/reatribuir;
- usuário desativado permanece no histórico, não em novas atribuições normais.

## 12. Conclusão

Para `Concluir atendimento`:

- estado `Em andamento`;
- capacidade apropriada;
- responsável definido;
- `Resumo do trabalho` obrigatório;
- alterações locais relevantes salvas;
- sem conflito/base obsoleta.

Não são obrigatórios por si só:

- OS;
- cliente;
- Equipamento;
- Procedimento vinculado.

Checklist incompleto:

- gera aviso/confirmação;
- não bloqueia automaticamente;
- não há semântica obrigatório/opcional nos itens iniciais.

Ao concluir:

- status `Concluído`;
- Host define `completed_at`;
- preserva revisões/checklist final;
- congela projeção relevante do Equipamento;
- grava evento de conclusão.

## 13. Cancelamento

- somente `Em andamento`;
- preset ADM/Gerência;
- Funcionário não por preset;
- exige motivo curto;
- não exclui registro;
- preserva código/histórico;
- bloqueia edição direta;
- pode ser reaberto quando autorizado.

## 14. Reabertura

- Concluído/Cancelado → Em andamento;
- preset ADM/Gerência;
- Funcionário não;
- explícita/auditável;
- preserva histórico anterior;
- nova conclusão gera novo estado final.

## 15. Procedimentos usados em Atendimento

- vínculo com revisão exata;
- snapshots de código/título/versão;
- nova publicação não altera vínculo existente;
- Funcionário seleciona revisão publicada que possa ler;
- ADM/Gerência podem selecionar explicitamente histórica/não publicada já autorizada;
- revisão histórica/não publicada nunca é escolhida silenciosamente;
- remoção só em Atendimento editável;
- remoção com checklist marcado exige confirmação.

## 16. Checklist operacional

Estado separado da revisão documental.

Cada item precisa preservar conceitualmente:

- identidade de execução;
- vínculo Atendimento × Procedimento/revisão;
- origem no item documental;
- texto snapshot quando necessário;
- marcado/desmarcado;
- data/usuário;
- revisão/controle concorrente próprio ou equivalente.

Regras:

- somente Atendimento `Em andamento` + capacidade permite marcar/desmarcar;
- Concluído/Cancelado = somente leitura;
- 100% não conclui automaticamente;
- Reader standalone não persiste estado.

## 17. Progresso operacional

Derivado exclusivamente de checklists:

```text
PR-001        4 de 6
PR-022        2 de 2
Atendimento   6 de 8
```

- etapas visitadas não contam;
- `Etapa X de Y` e stepper não são percentual operacional;
- revisão sem checklist não mostra `0%` artificial.

## 18. Equipamento

Opcional/reutilizável.

Campos para computadores conforme aplicável:

- nome;
- tipo;
- CPU;
- RAM;
- armazenamento;
- SO/versão;
- serial;
- patrimônio;
- múltiplos MACs;
- bateria para Notebook;
- cliente/responsável relacionado;
- observações curtas.

Tipos mínimos:

- Servidor;
- Desktop;
- Notebook.

Não virar enum global rígida.

Bateria:

- opcional/contextual;
- percentual 0–100 quando informado.

MAC:

- múltiplos;
- label opcional;
- normalizado pelo Host;
- não é identidade canônica.

## 19. Capacidades de Equipamento

Preset:

- criar/editar: ADM/Gerência/Funcionário;
- arquivar/reativar: ADM/Gerência;
- Funcionário vincula/troca/desvincula em Atendimento editável do qual é responsável.

Não arquivar Equipamento vinculado a Atendimento `Em andamento`.

## 20. Snapshot histórico do Equipamento

Alteração posterior do cadastro global não reescreve Atendimento concluído/ficha final.

Cada conclusão congela projeção relevante do Equipamento. Reabertura + nova conclusão pode gerar novo estado final; histórico anterior permanece.

## 21. Lista/Pesquisa de Atendimentos

- tabela compacta;
- busca por Atendimento/OS/cliente/Equipamento/serial/patrimônio/MAC;
- filtros Responsável + Status + Período;
- Status = Em andamento/Concluído/Cancelado;
- Data/Período usam `started_at`;
- ordenação padrão `started_at DESC`;
- linha abre Tela 09;
- retorno preserva busca/filtros/ordenação/página/scroll;
- busca de Atendimentos permanece separada de Processos.

## 22. Matriz operacional

| Capacidade | ADM | Gerência | Funcionário |
|---|---:|---:|---:|
| Consultar Atendimentos | sim | sim | sim |
| Criar Atendimento | sim | sim | sim |
| Editar Atendimento próprio em andamento | sim | sim | sim |
| Editar qualquer Atendimento em andamento | sim | sim | não |
| Concluir Atendimento próprio | sim | sim | sim |
| Concluir qualquer Atendimento | sim | sim | não |
| Cancelar | sim | sim | não |
| Reabrir | sim | sim | não |
| Vincular/trocar/desvincular Equipamento editável | sim | sim | sim, quando responsável |
| Criar/editar Equipamento | sim | sim | sim |
| Arquivar/reativar Equipamento | sim | sim | não |
| Adicionar/remover Procedimento | sim | sim | sim, quando responsável |
| Selecionar revisão não publicada/histórica | sim | sim | não |
| Marcar/desmarcar checklist | sim | sim | sim, quando responsável |
| Gerar/reimprimir ficha acessível | sim | sim | sim |
| Gerir categorias | sim | sim | não |

Presets são defaults. Autorização real é granular e Host-side.

## 23. Usuários/segurança

- Gerência não administra ADM;
- usuário não eleva a própria autoridade;
- pelo menos um ADM ativo;
- `is_primary_admin` não é toggle comum;
- senha Argon2id;
- sessão opaca server-side;
- token em memória;
- troca da própria senha mantém sessão corrente e revoga demais sessões da conta;
- sessão expirada exige nova autenticação.

Pendentes:

- custo final Argon2id;
- senha mínima;
- duração da sessão;
- entropia/tamanho final do token;
- Gerência × configuração da empresa;
- Gerência × Backup.

## 24. Concorrência

- WAL;
- writer lógico coordenado;
- fila bounded/backpressure;
- revisão otimista;
- `409` para estado obsoleto;
- eventos pós-commit;
- sem soft/hard lock inicial;
- Atendimento e Equipamento têm revisões independentes;
- checklist usa controle granular por item/equivalente;
- fila ordena, não valida edição obsoleta;
- mutação de resultado incerto exige reconciliação, não retry cego;
- evento remoto não sobrescreve formulário local;
- geração documental é leitura derivada, fora da fila de mutações;
- renderização documental usa limite próprio de concorrência/backpressure.

## 25. Estados transversais e densidade visual

- menor superfície: campo → seção → página → Shell;
- sem indicador permanente de conexão saudável;
- loading não mostra cache antigo como atual;
- `sem registros` distinto de `sem resultados`;
- Host indisponível separado de WebSocket degradado quando HTTP continua saudável;
- perda de permissão limpa conteúdo protegido;
- conflito preserva edição local;
- incompatibilidade Client↔Host bloqueia uso;
- sem offline queue/autosave/draft persistente;
- mostrar permanentemente somente o necessário para entender e agir;
- usar forma, cor, símbolo, posição e ícones reconhecíveis para estados/ações simples quando isso reduzir texto sem perder clareza;
- detalhes secundários podem aparecer sob demanda em tooltip/popover/expansão;
- evitar chips, badges, labels e caixas quando não acrescentarem leitura útil;
- cor nunca é o único meio para estado importante;
- quando remover texto criar ambiguidade, o texto permanece.

## 26. Ficha compacta e geração documental

### Ficha compacta

- pertence ao Atendimento;
- com ou sem Equipamento;
- estado confirmado do Host;
- Em andamento: pode gerar para acompanhamento;
- Concluído: reimprime estado histórico aplicável;
- Cancelado: saída identifica claramente o estado;
- capacidade padrão para ADM/Gerência/Funcionário em Atendimento acessível;
- máximo uma página A4;
- não gerar segunda página normal;
- conteúdo excessivo bloqueia saída em vez de truncamento silencioso;
- cabeçalho usa identidade da empresa;
- impressão é requisito;
- DOCX específico não é requisito inicial.

### Arquitetura de geração documental — Bloco 10 / Etapa 1

Consolidado:

1. geração documental é responsabilidade do Host;
2. Client solicita por IDs/revisão esperada e não envia documento montado;
3. Host captura snapshot consistente e encerra leitura/transação SQLite antes de renderizar;
4. fonte mutável usa revisão esperada para impedir substituição silenciosa por estado mais novo;
5. `DocumentModel` semântico separa domínio de renderers;
6. renderers não reconsultam banco nem recebem DOM/HTML da UI;
7. geração é leitura derivada e fica fora da fila de mutações;
8. renderização usa limite próprio de concorrência/backpressure, sem fila persistente;
9. fluxo inicial é request → renderização → resposta, sem `export_jobs` persistentes;
10. artefato retorna pela API autenticada; Host não escreve em path arbitrário do Client;
11. runtime documental não depende operacionalmente de Office, LibreOffice, Adobe Reader, Chrome/Chromium externo headless, `wkhtmltopdf` ou serviço cloud/conversor obrigatório;
12. artefatos gerados não viram histórico/backup por padrão;
13. Client permanece responsável pela UX e pelo destino local;
14. detalhes de PDF, DOCX, impressão, templates, limites, preview, MACs, temporários concretos e QR/barcode permanecem nas respectivas etapas.

### PDF de Procedimentos — Bloco 10 / Etapa 2

Consolidado / aprovado pelo PO:

1. renderer PDF baseado em Typst embutido como biblioteca Rust no Host, com crates oficiais e adaptador interno StepFlow;
2. nenhuma execução de `typst.exe`/CLI, browser ou processo conversor externo;
3. template Typst interno, confiável e versionado com o produto;
4. conteúdo do domínio entra somente como valores/dados estruturados e nunca participa da construção textual do source Typst;
5. nenhum pacote/recurso remoto é resolvido durante geração; filesystem/imports ficam restritos ao mundo virtual, templates, fontes e assets controlados pelo Host;
6. PDF 1.7 é solicitado explicitamente ao exporter;
7. Tagged PDF permanece explicitamente habilitado como baseline, sem promessa de conformidade formal PDF/UA/PDF-A;
8. texto textual permanece selecionável, pesquisável e copiável;
9. fontes necessárias são empacotadas e incorporadas/subsetadas, sem depender das fontes instaladas no Windows;
10. todos os blocos semânticos conhecidos são representados; incompatibilidade/tipo desconhecido falha explicitamente em vez de ser descartado;
11. comandos e código permanecem texto e preservam whitespace relevante;
12. fluxo multipágina e quebra automática são obrigatórios; formato/layout físico seguem a Etapa 5 consolidada;
13. PNG/JPEG e SVG controlado são suportados somente a partir de assets previamente aceitos/resolvidos pelo Host;
14. conteúdo visual não depende implicitamente de relógio/ambiente da máquina central; data/hora visível vem de dados explícitos do `DocumentModel`/`generation_metadata`;
15. estabilidade visual/semântica é exigida sob mesma versão do Host/template/fontes/assets/modelo, sem exigir identidade byte-a-byte quando metadados técnicos variarem;
16. falha do renderer não produz artefato parcial tratado como sucesso;
17. assinatura digital, senha, formulários, anexos, JavaScript, multimídia, PDF/A formal e PDF/UA formal ficam fora da primeira versão;
18. versão exata das crates e limites numéricos de memória/tamanho/tempo ficam para implementação/medição e validação técnica posterior.

### DOCX de Procedimentos — Bloco 10 / Etapa 3

Consolidado / aprovado pelo PO:

1. DOCX de Procedimentos é gerado diretamente pelo Host Rust a partir do mesmo `DocumentModel`, sem converter PDF/Typst;
2. saída é `.docx` real em OOXML/WordprocessingML, empacotada segundo OPC, com **OOXML Transitional** como baseline inicial de compatibilidade;
3. `docx-rs` é a biblioteca Rust preferida, encapsulada por adaptador interno StepFlow;
4. não há dependência de Microsoft Word/COM, LibreOffice, browser/headless, CLI conversor ou serviço cloud;
5. conteúdo do domínio entra somente como dados estruturados e nunca como XML/OOXML arbitrário, relationship, parte OPC, path ou URL controlados pelo usuário;
6. estilos/template são internos e versionados; nenhum `.docx`/`.dotx` fornecido pelo usuário é template de runtime na primeira versão;
7. texto permanece Word real, selecionável, pesquisável, copiável e editável;
8. todos os blocos semânticos conhecidos são representados; incompatibilidade/tipo desconhecido falha explicitamente em vez de ser descartado;
9. passos/subpassos usam numeração/listas Word reais quando aplicável; checklist permanece documental e não vira formulário interativo;
10. comandos/código permanecem texto e preservam whitespace relevante;
11. PNG/JPEG são baseline; SVG não é requisito direto do DOCX v1 e precisa de representação interna compatível ou falha explícita, nunca omissão silenciosa;
12. DOCX é formato refluível e não promete paginação idêntica ao PDF nem entre consumidores Word;
13. DOCX v1 referencia **Arial** para texto e **Consolas** para comando/código, sem embedding/redistribuição dessas fontes pelo StepFlow;
14. relationships externos, macros/VBA/`.docm`, ActiveX, OLE, remote templates, conteúdo externo, anexos, assinatura, senha/DRM e importação de DOCX editado ficam fora da primeira versão;
15. pacote incompleto/corrompido nunca é tratado como sucesso; validação posterior cobre OPC/ZIP, XML/relationships e abertura sem reparo na matriz corporativa;
16. mesma versão do Host/modelo/assets/estilos deve manter estrutura e conteúdo semânticos estáveis quando razoável, sem exigir ZIP byte-a-byte idêntico;
17. versão exata da crate, limites numéricos e matriz real de compatibilidade ficam para implementação/Etapa 12.

### Impressão Windows de Procedimentos — Bloco 10 / Etapa 4

Consolidado / aprovado pelo PO:

1. impressão física de Procedimentos acontece no **Client Windows** da estação do usuário, não no Host central;
2. o artefato canônico de impressão é o **PDF produzido pelo renderer da Etapa 2** para a revisão exata selecionada;
3. não existe renderer separado de impressão e não se imprime HTML da UI nem DOCX;
4. o Client usa WebView2 transitória/dedicada, mantendo a webview principal e o estado do Reader intactos;
5. a WebView2 recebe somente recurso PDF local controlado, sem Internet, URL/path arbitrário originado do conteúdo ou token em URL;
6. o mecanismo baseline usa WebView2 `ShowPrintUI(System)` por adaptador Windows isolado sob Tauri `with_webview`;
7. o diálogo padrão é o diálogo de impressão do Windows; impressão silenciosa e seletor próprio de impressoras não são requisitos iniciais;
8. StepFlow não enumera/persiste impressoras no Host e não gerencia drivers, descoberta ou spooler corporativo;
9. `ShellExecute`/handler `.pdf`, Word/COM, LibreOffice, browser/visualizador externo e spool PDF bruto não são baseline nem fallback silencioso;
10. o recurso local de impressão é transitório; estratégia, nome, path e limpeza concretos ficam para a Etapa 10;
11. `ShowPrintUI` não permite afirmar impressão física concluída: fluxo entregue ao Windows é sucesso da integração, não confirmação de papel impresso;
12. fechamento/cancelamento do diálogo não é erro funcional e não gera auditoria persistente `printed=true`;
13. falhas de geração PDF, preparação local, compatibilidade WebView2 e abertura do diálogo são classes distintas;
14. duplicidade concorrente acidental da mesma ação é impedida localmente sem criar fila/job persistente;
15. gate técnico posterior valida Windows 10/11 x64, WebView2, PDF multipágina, Unicode/logo, impressoras locais/de rede, opções do diálogo, cancelamento e operação offline;
16. incompatibilidade de WebView2 deve ser explícita; versão mínima concreta fica para matriz corporativa/gate de implementação;
17. impressão usa o template físico consolidado na Etapa 5.

### Template físico de Procedimentos — Bloco 10 / Etapa 5

Consolidado / aprovado pelo PO:

1. Reader diário e documento exportado são superfícies distintas; página lógica do app não possui geometria A4;
2. `Visão geral` é a primeira página lógica do Reader e cada Etapa mantém página lógica própria;
3. stepper horizontal compacto de círculos/linhas navega diretamente entre Etapas, com estados anteriores/atual/seguintes e sem rótulos permanentes repetitivos;
4. estado anterior no stepper representa percurso de navegação, nunca conclusão operacional/checklist;
5. Procedimento exportado usa **A4 retrato multipágina** com margens-base de **18 mm** em todos os lados;
6. não há capa exclusiva; identificação e conteúdo útil começam na primeira página;
7. v1 não exige sumário documental físico por padrão;
8. títulos de Etapa podem usar forma curta como `01 · Preparação`; uma Etapa não força nova folha automaticamente;
9. não há cabeçalho repetitivo nas páginas internas; rodapé compacto identifica código/revisão e paginação;
10. blocos físicos usam baixa densidade visual e hierarquia semântica, evitando cards/bordas desnecessários;
11. parágrafo = texto normal; passos = numeração/indentação; checklist = `□`; nota/alerta = distinção semântica que não depende apenas de cor; comando/código = texto monoespaçado real;
12. imagens preservam proporção, sem crop automático;
13. paginação automática evita títulos/linhas órfãs; blocos longos podem quebrar, mas conteúdo nunca é truncado ou tem fonte reduzida silenciosamente para caber;
14. PDF usa **Noto Sans + Noto Sans Mono** empacotadas com o Host e incorporadas/subsetadas, respeitando OFL 1.1;
15. DOCX usa **Arial + Consolas** referenciadas, sem embedding/redistribuição dessas fontes pelo StepFlow v1;
16. baseline tipográfico: título 18 pt, Etapa 14 pt, corpo 10,5 pt, comando/código 9 pt e rodapé 8 pt;
17. PDF é a referência física de impressão; DOCX compartilha hierarquia/ordem/semântica, mas permanece refluível sem promessa de paginação idêntica;
18. limite rígido de **uma A4** pertence à Ficha compacta de Atendimento, não ao Procedimento completo.

A Etapa 6 — PDF + preview da Ficha compacta é a próxima, mas ainda não está em análise.

## 27. Backup/Restore

- dentro de Configurações;
- Host coordena;
- Client não escolhe SQLite/path;
- Restore exige capacidade + backup elegível;
- confirmação reforçada;
- safety backup obrigatório antes da etapa destrutiva normal;
- disaster recovery sem Host funcional fica no Bloco 11.

## 28. Pocket / implantação

- máquina central recebe pasta pronta;
- sem instalação normal/toolchain;
- sem Windows Service/Task Scheduler/watchdog/tray/daemon;
- Controller inicia Host como filho;
- fechar Client individual não encerra Host;
- encerrado ciclo central, zero processo StepFlow residual;
- auto-shutdown por último Client/timeout não está consolidado;
- launcher em workstation não inicia remotamente Host central por si só.

## 29. Tecnologias consolidadas

Client:

- Tauri 2;
- HTML/CSS/JS modular;
- Windows 10/11 x64;
- WebView2.

Host:

- Rust;
- Tokio/Axum;
- `rusqlite` bundled;
- SQLite Host-only.

Comunicação:

- HTTP/JSON `/api/v1` inicialmente;
- WebSocket autenticado;
- handshake de compatibilidade;
- `deployment.json` sem segredos;
- sem edição offline.

## 30. Estado da Fase 1

- Bloco 0: concluído;
- Bloco 1: concluído;
- Bloco 2: concluído;
- Bloco 3: concluído;
- Bloco 4: concluído;
- Bloco 5: núcleo concluído / parâmetros finais pendentes;
- Bloco 6: núcleo + extensão operacional conceitual consolidados;
- Bloco 7: núcleo concluído;
- Bloco 8: concluído;
- Bloco 9: concluído;
- **Bloco 10: em andamento — Etapas 1–5 consolidadas; Etapa 6 próxima, ainda não aberta**;
- Bloco 11: pendente;
- Bloco 12: pendente.

## 31. Pendências vigentes

### Bloco 10

- Etapa 6: PDF + preview da ficha;
- Etapa 7: template físico A4 da ficha;
- Etapa 8: limites textuais/densidade;
- Etapa 9: muitos MACs/Procedimentos;
- Etapa 10: nomes de arquivo + temporários concretos, incluindo materialização/limpeza do recurso local de impressão;
- Etapa 11: QR/barcode;
- Etapa 12: validação técnica final, incluindo matriz real Windows/WebView2/impressoras.

### Bloco 11

- mecanismo/pacote de Backup/Restore;
- atomicidade/checksums;
- retenção;
- restart/reconexão/sessões;
- disaster recovery local.

### Bloco 12 / antes da implementação correspondente

- parâmetros finais de autenticação;
- Gerência × configuração da empresa;
- Gerência × Backup;
- regra editorial de categoria arquivada em nova revisão;
- árvore oficial/migrations/scripts/testes;
- plano Fase 2;
- sincronização do checkout local antes do primeiro Codex de implementação.

### Ambiente corporativo

- hostname/IP/path reais;
- SMB/permissões;
- WebView2/Windows reais;
- HTTP/HTTPS;
- EDR/firewall;
- mecanismo real de start central.

## 32. Regra de precedência

Em divergência:

1. `AGENTS.md`;
2. este registro de decisões, na versão mais recente;
3. documento específico vigente;
4. visão de produto;
5. tarefa dentro das decisões aprovadas;
6. histórico Git.

Nenhuma pendência pode ser convertida silenciosamente em decisão por executor.