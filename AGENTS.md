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
- Bloco 10 está **EM ANDAMENTO**; **Etapa 1 — Arquitetura de geração documental está CONSOLIDADA**; **Etapa 2 — PDF de Procedimentos é a próxima, ainda não aberta**.
- A modelagem `Procedimento × Atendimento/Execução × Equipamento`, categorias múltiplas, lifecycle operacional, checklist persistente, matriz operacional, códigos `AT/EQP`, snapshot histórico de Equipamento e arquitetura-base de geração documental estão consolidados.
- Bloco 10 ainda fecha as Etapas 2–12; Bloco 11 fecha Backup/Restore técnico; Bloco 12 fecha estrutura oficial, parâmetros finais e plano da Fase 2.

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

## Arquitetura de geração documental consolidada — Bloco 10 / Etapa 1

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
- artefatos gerados não viram histórico/backup por padrão;
- engines PDF/DOCX, impressão, templates, limites, preview, MACs, temporários concretos e QR/barcode continuam nas Etapas 2–12.

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

- engine/tecnologia final PDF/DOCX/impressão;
- template físico final da ficha;
- margens/tipografia/densidade;
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