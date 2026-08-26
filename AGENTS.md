# AGENTS.md — StepFlow Pocket

## Finalidade

Regras obrigatórias para Codex e outros agentes que atuem neste repositório.

## Fonte de verdade e fase atual

- GitHub é a fonte principal de verdade.
- Branch principal: `main`.
- Checkout local previsto: `C:\dev\StepFlow`.
- Desenvolvimento atual: computador pessoal fora da LAN corporativa.
- Fase vigente: **Fase 1 — Fechamento arquitetural e especificação**.
- Blocos 0–4 estão concluídos; Bloco 5 está concluído no núcleo com parâmetros finais pendentes; Bloco 6 está consolidado conceitualmente; Bloco 7 está concluído no núcleo; Blocos 8 e 9 estão **CONCLUÍDOS**.
- Bloco 10 está **EM ANDAMENTO**; **Etapas 1 — Arquitetura de geração documental, 2 — PDF de Procedimentos, 3 — DOCX de Procedimentos e 4 — Impressão Windows de Procedimentos estão CONSOLIDADAS**; **Etapa 5 — Template físico de Procedimentos é a próxima, ainda não aberta**.
- A modelagem `Procedimento × Atendimento/Execução × Equipamento`, categorias múltiplas, lifecycle operacional, checklist persistente, matriz operacional, códigos `AT/EQP`, snapshot histórico de Equipamento, arquitetura-base de geração documental, renderers PDF/DOCX e impressão Windows de Procedimentos estão consolidados.
- Bloco 10 ainda fecha as Etapas 5–12; Bloco 11 fecha Backup/Restore técnico; Bloco 12 fecha estrutura oficial, parâmetros finais e plano da Fase 2.

## Precedência e autoridade da tarefa

O enunciado da tarefa define **o trabalho autorizado**, mas não revoga silenciosamente decisões vigentes.

Em caso de conflito:

1. `AGENTS.md`;
2. `docs/05-progresso/registro-de-decisoes.md`;
3. documento específico vigente;
4. `docs/01-produto/visao-geral.md`;
5. enunciado da tarefa, dentro das decisões vigentes;
6. histórico.

Se o enunciado contrariar decisão consolidada, só prosseguir quando declarar explicitamente nova decisão aprovada pelo PO e atualizar os documentos afetados. Ambiguidade nunca autoriza escolher a alternativa mais conveniente.

## Leitura do Codex por camadas

### Sempre

1. `AGENTS.md`;
2. enunciado da tarefa;
3. `docs/README.md`;
4. documentos específicos indicados.

### Quando houver impacto correspondente

- `docs/05-progresso/registro-de-decisoes.md`;
- `docs/04-planejamento/plano-oficial-fase-1.md`;
- `docs/03-arquitetura/arquitetura-vigente.md`;
- `docs/00-governanca/contexto-ambientes.md`;
- `docs/01-produto/categorizacao-atendimentos-equipamentos.md` para categorias/Atendimentos/Equipamentos/ficha;
- `docs/04-planejamento/bloco-9-atendimentos-execucao-checklist.md` para lifecycle/checklist/matriz operacional;
- `docs/04-planejamento/bloco-10-exportacao-impressao-ficha.md` para geração documental/exportação/impressão/ficha;
- demais documentos técnicos específicos.

`metodo-padrao-trabalho-assistido.md` e `politica-capacidade-codex.md` orientam principalmente PO/Assistente e não precisam ser relidos pelo Codex em toda tarefa.

## Papéis

- **PO:** define produto, prioridade, comportamento e aprovação visual/funcional.
- **Assistente:** analisa, arquiteta, documenta e transforma decisões aprovadas em tarefas.
- **Codex:** executa tecnicamente o escopo recebido, sem inventar produto ou ampliar tarefa.

## Pré-flight de capacidade

A seleção de modelo/raciocínio é responsabilidade do Assistente + PO antes do envio da tarefa ao Codex.

Antes de **cada nova tarefa Codex**, o Assistente deve fornecer separadamente:

1. `PRÉ-FLIGHT PARA O PO — NÃO ENVIAR AO CODEX`;
2. `PROMPT / ENUNCIADO PARA O CODEX`.

Usar o menor perfil de capacidade suficiente com margem de segurança, conforme `docs/00-governanca/politica-capacidade-codex.md`.

## Base Git obrigatória

Toda tarefa que permita alteração deve informar branch/base esperada e commit SHA esperado.

Antes de escrever:

```text
git rev-parse HEAD
git status --short --branch
```

Se `HEAD` divergir, não fazer `pull`, `merge`, `rebase`, `reset` ou checkout corretivo automaticamente. Parar e reportar.

## Proteção absoluta do working tree

Qualquer alteração preexistente pertence ao PO/outro fluxo.

Sem autorização explícita e específica do PO, é proibido:

- `git reset --hard`;
- `git clean`;
- `git stash`;
- descartar/restaurar alterações locais;
- sobrescrever arquivo modificado preexistente;
- trocar branch descartando trabalho;
- incluir alteração preexistente no commit da tarefa.

Se arquivo necessário já estiver modificado, parar e reportar.

## Disciplina de Git

Durante o fechamento documental restante da Fase 1:

```text
1 trabalho lógico
→ 1 branch ativa
→ 1 PR
→ revisão/aprovação
→ squash/merge em main
→ apagar branch encerrada
→ verificar remoto limpo
→ iniciar o próximo trabalho documental
```

Uma branch mergeada não é considerada encerrada operacionalmente enquanto permanecer no remoto. O remoto é a fonte operacional. A sincronização do checkout local fica adiada até antes do primeiro trabalho de implementação com Codex.

## Regras operacionais

- uma tarefa por vez;
- não declarar parcial como concluído;
- não criar funcionalidade/dependência/estrutura relevante fora do escopo;
- não alterar UX/visual aprovado sem autorização;
- não transformar proposta, exemplo ou parâmetro provisório em decisão;
- manter documentação e implementação sincronizadas;
- **todo avanço consolidado de fase, bloco, tela ou etapa do bloco atual deve atualizar o painel do `README.md` no mesmo checkpoint documental**;
- não considerar avanço documental encerrado se o README estiver atrasado;
- preservar modularidade e baixo acoplamento;
- não versionar credenciais, segredos, banco real ou dados pessoais da empresa;
- exemplos de IP/hostname/share/path nunca viram configuração;
- testes de LAN corporativa fora dela são `NÃO APLICÁVEIS NESTE AMBIENTE`;
- protótipos descartáveis não viram produção silenciosamente.

## Ambiente Codex versus sessão normal do PO

Limitação do sandbox não vira requisito do produto.

Codex não repara o próprio ambiente alterando:

- ACL;
- Schannel;
- registro Windows;
- PATH global;
- políticas de segurança;
- reinstalações abertas de ferramentas válidas.

Operações que exijam credenciais, Internet confiável, elevação ou configuração global são reportadas para sessão Windows normal do PO.

## Regras Pocket obrigatórias

- implantação central por pasta pronta;
- nenhuma toolchain de desenvolvimento na máquina central de produção;
- sem Windows Service persistente, auto-start, Task Scheduler, watchdog, tray agent ou daemon como padrão;
- Host/Controller sob demanda;
- Controller aberto representa ciclo central ativo; encerrado o ciclo, nenhum processo StepFlow permanece ativo;
- fechar Client não encerra Host;
- não inventar auto-shutdown por ausência de Clients/timeout;
- Client roda localmente, preparado por launcher transitório;
- launcher encerra após iniciar Client;
- workstation remota não inicia por si só processo na máquina central apenas por executar `.exe` via SMB;
- dados/config/logs/backups ficam separados dos binários substituíveis.

## Regras técnicas consolidadas

- Client: Tauri 2 + HTML/CSS/JavaScript modular;
- Host: Rust + Tokio/Axum + `rusqlite`/SQLite bundled;
- HTTP/JSON + WebSocket;
- SQLite somente pelo Host;
- writer coordenado + fila bounded + revisão otimista;
- nenhuma sobrescrita silenciosa;
- sessão opaca e autorização Host-side;
- Argon2id, com parâmetros finais ainda pendentes;
- Procedimentos usam revisões imutáveis;
- PDF, DOCX e impressão são requisitos documentais;
- categorias são configuráveis/múltiplas;
- `Processos` e `Atendimentos` são domínios distintos;
- Equipamento possui identidade interna própria;
- MAC/serial/patrimônio são atributos de busca;
- Atendimento preserva a revisão realmente utilizada;
- ficha compacta imprimível é requisito do produto.

## Arquitetura de geração documental consolidada — Bloco 10 / Etapas 1–4

Etapa 1:

- geração documental pertence ao Host;
- Client solicita por identidade da fonte/revisão esperada e não envia documento montado;
- fonte mutável não pode ser substituída silenciosamente por revisão mais nova;
- Host captura snapshot consistente, materializa `DocumentModel` semântico e encerra a leitura/transação SQLite antes da renderização;
- renderers não recebem DOM/HTML da UI e não reconsultam o banco;
- geração é leitura derivada e fica fora da fila de mutações;
- renderização usa limite próprio de concorrência/backpressure;
- primeira versão não cria `export_jobs`, scheduler ou fila persistente de exportação;
- artefato retorna pela API autenticada;
- Host não grava em path arbitrário do Client;
- runtime documental não depende operacionalmente de Office, LibreOffice, Adobe Reader, Chrome/Chromium externo headless, `wkhtmltopdf` ou serviço cloud;
- artefatos gerados não viram histórico/backup por padrão.

Etapa 2 — PDF de Procedimentos:

- renderer baseado em Typst embutido no Host Rust, usando crates oficiais com adaptador interno StepFlow;
- sem `typst.exe`/CLI, browser ou processo conversor externo;
- template Typst interno, confiável e versionado;
- conteúdo do domínio entra somente como valores/dados estruturados e nunca participa da construção textual do source Typst;
- sem resolução de pacotes/recursos remotos em runtime; filesystem/imports ficam limitados ao mundo virtual controlado pelo Host;
- PDF 1.7 e Tagged PDF são solicitados/habilitados explicitamente; Tagged PDF não implica conformidade formal PDF/UA ou PDF/A;
- fontes necessárias são empacotadas e incorporadas/subsetadas, sem depender das fontes instaladas no Windows;
- texto permanece selecionável/pesquisável/copiável; comandos/código permanecem texto e preservam whitespace relevante;
- todos os blocos semânticos conhecidos devem ser representados sem descarte silencioso;
- fluxo multipágina e quebra automática são obrigatórios, sem antecipar margens, A4, tipografia, cabeçalho/rodapé ou paginação visual da Etapa 5;
- PNG/JPEG e SVG controlado são suportados somente a partir de assets já aceitos e resolvidos pelo Host;
- conteúdo visual não pode depender implicitamente de relógio/ambiente da máquina central; data/hora exibida vem de dados explícitos do modelo;
- falha de renderer não produz artefato parcial tratado como sucesso;
- assinatura, senha, formulários, anexos, JavaScript, multimídia e conformidade formal PDF/A ou PDF/UA ficam fora da primeira versão.

Etapa 3 — DOCX de Procedimentos:

- DOCX é gerado diretamente pelo Host Rust a partir do mesmo `DocumentModel`, sem conversão de PDF/Typst;
- saída é `.docx` real em OOXML/WordprocessingML, com baseline de compatibilidade **OOXML Transitional**; Strict não é baseline da primeira versão;
- `docx-rs` é a biblioteca preferida, encapsulada por adaptador interno StepFlow;
- sem Microsoft Word/COM, LibreOffice, browser/headless, CLI conversor ou serviço cloud;
- conteúdo do domínio entra apenas como dados estruturados e nunca como XML/OOXML arbitrário;
- estilos/template são internos e versionados; nenhum `.docx`/`.dotx` fornecido pelo usuário em runtime na primeira versão;
- texto permanece real, selecionável, pesquisável, copiável e editável;
- todos os blocos semânticos conhecidos devem ser representados sem descarte silencioso;
- passos/subpassos usam numeração/lista Word real quando aplicável; checklist permanece documental, não formulário interativo;
- comandos/código permanecem texto e preservam whitespace relevante;
- PNG/JPEG são baseline de imagem; SVG não é requisito direto do DOCX v1 e não pode ser omitido silenciosamente;
- DOCX é refluível e não promete paginação idêntica ao PDF; layout físico final permanece reservado à Etapa 5;
- política tipográfica/embedding de fontes do DOCX não é herdada automaticamente do PDF e permanece para Etapa 5/gate técnico;
- relationships externos, macros/VBA, ActiveX, OLE, remote templates, conteúdo externo, anexos, assinatura, senha/DRM e importação de DOCX editado ficam fora da primeira versão;
- artefato incompleto/corrompido nunca é devolvido como sucesso;
- versão exata da crate, limites numéricos e matriz real de compatibilidade ficam para implementação/Etapa 12.

Etapa 4 — Impressão Windows de Procedimentos:

- impressão física acontece no Client Windows da estação do usuário, não no Host central;
- artefato canônico de impressão é o mesmo PDF produzido pelo renderer da Etapa 2 para a revisão exata selecionada;
- não existe renderer separado de impressão e o Client não imprime HTML da UI nem DOCX;
- Client usa WebView2 transitória/dedicada para carregar somente recurso PDF local controlado, sem substituir a webview principal;
- baseline usa WebView2 `ShowPrintUI(System)` por adaptador Windows isolado sob Tauri `with_webview`;
- diálogo padrão é o diálogo de impressão do Windows; impressão silenciosa e seletor próprio de impressoras ficam fora da primeira versão;
- StepFlow não enumera/persiste impressoras no Host e não gerencia drivers/spooler corporativo;
- não usar `ShellExecute`/handler PDF externo, Word/COM, LibreOffice, browser externo ou spool PDF bruto como fallback silencioso;
- recurso local de impressão é transitório; materialização/nome/path/limpeza concreta ficam para a Etapa 10;
- `ShowPrintUI` não confirma impressão física: a UI só pode afirmar que o fluxo foi entregue ao Windows e não grava `printed=true` por inferência;
- fechamento/cancelamento do diálogo não é erro funcional; falhas de geração, preparação local, compatibilidade WebView2 e abertura do diálogo permanecem distintas;
- duplicidade concorrente da mesma ação é controlada localmente sem criar fila/job persistente;
- gate técnico posterior valida Windows 10/11 x64, WebView2, PDF multipágina, impressoras locais/de rede e operação offline;
- versão mínima concreta do WebView2 fica para matriz corporativa/gate técnico;
- layout físico do Procedimento permanece integralmente reservado para a Etapa 5.

Templates físicos, limites, preview da ficha, MACs, temporários concretos e QR/barcode continuam nas Etapas 5–12.

## Regras operacionais consolidadas do Bloco 9

### Lifecycle

```text
Em andamento
Concluído
Cancelado
```

- primeiro save aceito cria o Atendimento;
- abrir tela não cria registro oficial;
- responsável + Resumo do trabalho são obrigatórios para conclusão;
- checklist incompleto gera confirmação, não bloqueio automático;
- cancelamento exige motivo;
- Concluído/Cancelado não recebem edição operacional direta;
- reabertura explícita volta a `Em andamento`;
- ADM/Gerência reabrem por preset; Funcionário não.

### Responsabilidade

- Funcionário cria Atendimento inicialmente para si;
- Funcionário padrão edita/conclui somente Atendimento do qual é responsável;
- ADM/Gerência podem atribuir/reatribuir e operar qualquer Atendimento acessível.

### Procedimentos e checklist

- Funcionário seleciona revisão publicada;
- ADM/Gerência podem selecionar explicitamente revisão histórica/não publicada já autorizada;
- Reader standalone = checklist documental;
- Reader no contexto de Atendimento = checklist persistente;
- progresso deriva apenas de checklist marcado/total;
- 100% não conclui Atendimento automaticamente;
- checklist usa concorrência granular por item/equivalente.

### Equipamento

- código `EQP-000001`;
- criar/editar: ADM/Gerência/Funcionário;
- arquivar/reativar: ADM/Gerência;
- não arquivar Equipamento vinculado a Atendimento `Em andamento`;
- conclusão congela projeção histórica relevante do Equipamento.

### Atendimento

- código `AT-000001`;
- códigos são Host-only, seis dígitos, gaps permitidos;
- Status entra na lista de Atendimentos;
- Data/Período usam `started_at`;
- ordenação default: mais recente primeiro.

### Categorias

- gerir categorias: ADM/Gerência;
- Funcionário não;
- Gerência × configuração da empresa continua PENDENTE;
- Gerência × Backup continua PENDENTE;
- regra editorial de nova revisão com categoria arquivada continua pendente.

### Ficha

- gerar/reimprimir: ADM/Gerência/Funcionário para Atendimento acessível;
- `Em andamento`: geração para acompanhamento;
- `Concluído`: reimpressão do estado histórico aplicável;
- `Cancelado`: saída identifica o estado;
- template/engine permanecem nas próximas etapas do Bloco 10.

## Pendências ainda não consolidáveis para implementação

Não inventar por suposição:

- template físico final de Procedimentos e da ficha;
- margens/tipografia/densidade;
- versão mínima concreta do WebView2 para impressão e detalhes de integração sujeitos ao gate técnico;
- nomes/paths/limpeza concretos do recurso temporário de impressão;
- limites numéricos finais dos textos destinados à ficha;
- necessidade de PDF específico da ficha;
- QR/barcode;
- mecanismo técnico final de Backup/Restore;
- retenção/disaster recovery;
- parâmetros finais de autenticação ainda marcados como pendentes;
- Gerência × configuração da empresa;
- Gerência × Backup;
- regra editorial de nova revisão ainda referenciando categoria arquivada;
- parâmetros reais do ambiente corporativo.

## Tarefa Codex

Toda tarefa declara:

- objetivo;
- base Git;
- fonte de verdade;
- escopo;
- fora do escopo;
- critérios de aceite;
- validações;
- documentação impactada.

O relatório final informa:

- base/estado inicial;
- arquivos alterados;
- decisões técnicas dentro do escopo;
- validações/resultados;
- riscos/pendências;
- próximos passos sugeridos.

## Gate de implementação da Fase 1

Na Fase 1, trabalho estrutural significa documentação, organização documental ou PoC explicitamente descartável autorizada.

Não criar scaffold oficial, módulos runtime definitivos, árvore final ou código de negócio antes do Bloco 12/Fase 2 autorizar.

Antes do primeiro trabalho de implementação com Codex, sincronizar explicitamente o checkout local com o remoto sem apagar/incorporar indevidamente alterações preexistentes do PO.

Se a fase/documentação não autorizar implementação definitiva, limitar-se ao trabalho documental/investigativo solicitado.