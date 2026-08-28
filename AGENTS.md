# AGENTS.md — StepFlow Pocket

## Finalidade

Regras obrigatórias para Codex e outros agentes que atuem neste repositório.

## Fonte de verdade e fase atual

- GitHub é a fonte operacional principal de verdade.
- Branch principal: `main`.
- Checkout local previsto: `C:\dev\StepFlow`.
- Desenvolvimento atual: computador pessoal fora da LAN corporativa.
- Fase vigente: **Fase 1 — Fechamento arquitetural e especificação**.
- Blocos 0–4 concluídos; Bloco 5 com núcleo concluído e parâmetros finais pendentes; Bloco 6 consolidado conceitualmente; Bloco 7 concluído no núcleo; Blocos 8 e 9 concluídos.
- Bloco 10 está **EM ANDAMENTO**; **Etapas 1–7 estão CONSOLIDADAS / APROVADAS PELO PO**; **Etapa 8 — Limites textuais e densidade da Ficha é a próxima e ainda não está aberta**.
- Bloco 11 fecha Backup/Restore técnico; Bloco 12 fecha estrutura oficial, parâmetros finais e plano da Fase 2.

## Precedência e autoridade

O enunciado da tarefa define o trabalho autorizado, mas não revoga silenciosamente decisões vigentes.

Em caso de conflito:

1. `AGENTS.md`;
2. `docs/05-progresso/registro-de-decisoes.md`;
3. documento específico vigente;
4. `docs/01-produto/visao-geral.md`;
5. enunciado da tarefa, dentro das decisões vigentes;
6. histórico Git.

Se a tarefa contrariar decisão consolidada, só prosseguir quando houver nova decisão explícita do PO e sincronização dos documentos afetados. Ambiguidade nunca autoriza escolher a alternativa mais conveniente.

## Leitura do Codex por camadas

### Sempre

1. `AGENTS.md`;
2. enunciado da tarefa;
3. `docs/README.md`;
4. documentos específicos indicados.

### Conforme impacto

- `docs/05-progresso/registro-de-decisoes.md`;
- `docs/04-planejamento/plano-oficial-fase-1.md`;
- `docs/03-arquitetura/arquitetura-vigente.md`;
- `docs/03-arquitetura/modelo-dados-schema-fase-1.md`;
- `docs/00-governanca/contexto-ambientes.md`;
- `docs/01-produto/categorizacao-atendimentos-equipamentos.md`;
- `docs/04-planejamento/bloco-9-atendimentos-execucao-checklist.md`;
- `docs/04-planejamento/bloco-10-exportacao-impressao-ficha.md`;
- documentos técnicos específicos da tarefa.

## Papéis

- **PO:** produto, prioridade, comportamento e aprovação visual/funcional.
- **Assistente:** análise, arquitetura, coerência documental, gates e preparação de tarefas.
- **Codex:** execução técnica do escopo aprovado, sem inventar produto ou ampliar tarefa.

## Pré-flight de capacidade

Antes de cada nova tarefa Codex, o Assistente entrega separadamente:

1. `PRÉ-FLIGHT PARA O PO — NÃO ENVIAR AO CODEX`;
2. `PROMPT / ENUNCIADO PARA O CODEX`.

Usar o menor perfil de capacidade suficiente com margem de segurança, conforme `docs/00-governanca/politica-capacidade-codex.md`.

## Base Git obrigatória

Toda tarefa que permita alteração informa branch/base e commit SHA esperado.

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
→ squash merge em main
→ apagar branch encerrada
→ verificar remoto limpo
→ iniciar próximo trabalho
```

- branch mergeada não está encerrada enquanto permanecer no remoto;
- remoto é a fonte operacional;
- sincronização do checkout local fica adiada até antes do primeiro trabalho de implementação com Codex.

**Gate atual obrigatório:** nenhuma pesquisa, branch, proposta ou análise da **Etapa 8** pode começar antes de a **Etapa 7** estar squash-mergeada, a branch da Etapa 7 removida do remoto e o remoto verificado com somente `main` e zero PRs abertos.

## Regras operacionais

- uma tarefa por vez;
- não declarar parcial como concluído;
- não criar funcionalidade/dependência/estrutura relevante fora do escopo;
- não alterar UX/visual aprovado sem autorização;
- não transformar proposta, exemplo ou parâmetro provisório em decisão;
- manter documentação e implementação sincronizadas;
- todo avanço consolidado de fase/bloco/tela/etapa atualiza o painel do `README.md` no mesmo checkpoint;
- não considerar avanço documental encerrado com painel atrasado;
- preservar modularidade, separação de responsabilidades e baixo acoplamento;
- não criar monolito HTML/JS quando módulos simples resolvem;
- não versionar credenciais, segredos, banco real ou dados pessoais da empresa;
- exemplos de IP/hostname/share/path nunca viram configuração;
- testes de LAN corporativa fora dela são `NÃO APLICÁVEIS NESTE AMBIENTE`;
- protótipo descartável não vira produção silenciosamente.

## Ambiente Codex versus sessão normal do PO

Limitação do sandbox não vira requisito do produto. Codex não repara o próprio ambiente alterando ACL, Schannel, registro Windows, PATH global, políticas de segurança ou reinstalando ferramentas válidas. Operações que exijam credenciais, Internet confiável, elevação ou configuração global são reportadas para a sessão Windows normal do PO.

## Regras Pocket obrigatórias

- implantação central por pasta pronta;
- nenhuma toolchain de desenvolvimento na máquina central de produção;
- sem Windows Service persistente, auto-start, Task Scheduler, watchdog, tray agent ou daemon como padrão;
- Host/Controller sob demanda;
- Controller aberto representa ciclo central ativo; encerrado o ciclo, nenhum processo StepFlow permanece ativo;
- fechar Client individual não encerra Host;
- não inventar auto-shutdown por ausência de Clients/timeout;
- Client roda localmente, preparado por launcher transitório;
- launcher encerra após iniciar Client;
- workstation remota não inicia sozinha processo na máquina central apenas por executar `.exe` via SMB;
- dados/config/logs/backups ficam separados dos binários substituíveis.

## Regras técnicas consolidadas

- Client: Tauri 2 + HTML/CSS/JavaScript modular;
- Host: Rust + Tokio/Axum + `rusqlite`/SQLite bundled;
- HTTP/JSON + WebSocket;
- SQLite somente pelo Host;
- writer coordenado + fila bounded + revisão otimista;
- nenhuma sobrescrita silenciosa;
- sessão opaca e autorização Host-side;
- Argon2id, parâmetros finais pendentes;
- Procedimentos usam revisões imutáveis;
- categorias configuráveis/múltiplas;
- `Processos` e `Atendimentos` são domínios distintos;
- Equipamento tem identidade interna própria; MAC/serial/patrimônio são atributos de busca;
- Atendimento preserva revisão efetivamente utilizada;
- checklist persiste somente em Atendimento;
- `Observação do serviço` persiste opcionalmente por Etapa somente em Atendimento;
- observação operacional não altera o Procedimento e usa concorrência granular por Etapa/equivalente;
- estado final relevante de Equipamento/observações deve ser historicamente reproduzível após conclusão/reabertura;
- Ficha compacta é prestação de contas resumida ao cliente;
- UI busca clareza com baixa densidade textual; cor nunca é o único meio para estado importante;
- Reader usa stepper compacto navegável de círculos/linhas; esse estado é navegação, nunca conclusão operacional.

## Bloco 9 — operação consolidada

Lifecycle:

```text
Em andamento
Concluído
Cancelado
```

- primeiro save cria Atendimento;
- abrir tela não cria registro oficial;
- responsável + `Resumo do trabalho` são obrigatórios para conclusão;
- checklist incompleto gera confirmação, não bloqueio automático;
- cancelamento exige motivo;
- Concluído/Cancelado são read-only até reabertura;
- ADM/Gerência reabrem por preset; Funcionário não.

Responsabilidade/revisões:

- Funcionário cria Atendimento inicialmente para si;
- Funcionário padrão edita/conclui apenas Atendimento do qual é responsável;
- ADM/Gerência podem atribuir/reatribuir e operar qualquer Atendimento acessível;
- Funcionário seleciona revisão publicada;
- ADM/Gerência podem selecionar explicitamente revisão histórica/não publicada já autorizada;
- revisão vinculada nunca muda silenciosamente após nova publicação.

Checklist/observação por Etapa:

- Reader standalone não persiste execução;
- Reader operacional persiste checklist;
- cada Etapa pode receber `Observação do serviço` opcional;
- observação pertence a Atendimento + vínculo da revisão + Etapa;
- progresso deriva somente do checklist;
- checklist usa concorrência por item/equivalente;
- observação usa concorrência por Etapa/equivalente;
- evento remoto não sobrescreve edição local silenciosamente;
- não introduzir autosave por inferência.

Equipamento:

- código `EQP-000001`;
- criar/editar: ADM/Gerência/Funcionário;
- arquivar/reativar: ADM/Gerência;
- não arquivar Equipamento vinculado a Atendimento `Em andamento`;
- conclusão congela projeção histórica relevante.

Atendimento/Ficha:

- código `AT-000001`;
- códigos Host-only, seis dígitos, gaps permitidos;
- gerar/reimprimir Ficha: ADM/Gerência/Funcionário para Atendimento acessível;
- `Em andamento`: estado confirmado atual;
- `Concluído`: estado histórico aplicável;
- `Cancelado`: saída identifica o estado;
- Ficha prioriza serviço/dispositivo/resumo/observações, não checklist/timeline detalhados.

## Bloco 10 — arquitetura documental consolidada / Etapas 1–7

### Etapa 1 — arquitetura

- geração pertence ao Host;
- Client solicita identidade da fonte + revisão esperada, sem documento montado;
- Host captura snapshot consistente e materializa `DocumentModel` semântico;
- leitura/transação SQLite termina antes da renderização;
- renderers não recebem DOM/HTML nem reconsultam banco;
- geração é leitura derivada, fora da fila de mutações;
- renderização usa limite próprio bounded;
- sem `export_jobs`, scheduler ou fila persistente inicialmente;
- Host não grava em path arbitrário do Client;
- artefatos não viram histórico/backup por padrão;
- runtime sem dependência operacional de Office, LibreOffice, Adobe Reader, browser externo/headless ou cloud obrigatória.

### Etapa 2 — PDF de Procedimentos

- Typst embutido no Host Rust via crates oficiais + adaptador interno;
- template interno confiável/versionado e domínio somente como dados estruturados;
- sem pacotes/recursos remotos; filesystem/fontes/assets controlados;
- PDF 1.7 + Tagged PDF baseline, sem promessa formal PDF/A ou PDF/UA;
- fontes incorporadas/subsetadas;
- texto real selecionável/pesquisável;
- todos os blocos conhecidos são representados ou falham explicitamente;
- multipágina automático, sem truncamento;
- PNG/JPEG/SVG controlados;
- falha não produz artefato parcial como sucesso.

### Etapa 3 — DOCX de Procedimentos

- DOCX real OOXML/WordprocessingML/OPC, baseline Transitional;
- geração Rust direta pelo mesmo `DocumentModel`;
- `docx-rs` preferido sob adaptador interno;
- sem Word/COM, LibreOffice, browser/headless, CLI ou cloud;
- template/estilos internos; sem `.docx/.dotx` externo em runtime v1;
- texto/listas permanecem Word reais/editáveis;
- DOCX refluível, sem promessa de paginação idêntica ao PDF;
- Arial + Consolas referenciadas sem embedding v1;
- pacote incompleto/corrompido nunca é sucesso.

### Etapa 4 — impressão Windows de Procedimentos

- impressão física no Client Windows;
- artefato canônico = mesmo PDF da Etapa 2;
- sem renderer separado, HTML da UI ou DOCX;
- WebView2 transitória/dedicada + `ShowPrintUI(System)`;
- diálogo padrão do Windows, sem impressão silenciosa/seletor próprio como baseline;
- StepFlow não gerencia drivers/spooler;
- recurso local é transitório; naming/path/limpeza ficam para Etapa 10;
- sucesso = fluxo entregue ao Windows, não confirmação física de papel;
- sem fallback silencioso para software externo.

### Etapa 5 — template físico de Procedimentos

- Reader e documento físico são superfícies independentes;
- Procedimento exportado = A4 retrato multipágina, margens-base 18 mm;
- sem capa exclusiva/sumário físico obrigatório/header repetitivo por padrão;
- rodapé compacto;
- paginação automática sem truncamento/redução silenciosa;
- PDF usa Noto Sans + Noto Sans Mono incorporadas/subsetadas;
- DOCX referencia Arial + Consolas sem embedding v1;
- PDF é referência física; DOCX é refluível;
- uma A4 é regra da Ficha, nunca do Procedimento completo.

### Etapa 6 — PDF + preview da Ficha

- Ficha é prestação de contas resumida ao cliente;
- prioriza identificação do serviço/dispositivo, características relevantes, `Resumo do trabalho` e observações;
- PDF próprio/canônico via template Typst da Ficha;
- PDF e preview SVG derivam do mesmo `PagedDocument`;
- resultado válido exige exatamente uma página;
- `2+ páginas` = `SHEET_OVERFLOW`, sem corte/segunda folha/redução silenciosa;
- preview em modal/overlay simples;
- Salvar/Imprimir reutilizam os mesmos bytes PDF;
- impressão reutiliza WebView2 + `ShowPrintUI(System)`;
- PDF/SVG são transitórios e presos à `source_version`.

### Etapa 7 — template físico A4 da Ficha

- A4 retrato, exatamente uma página, margens 15 mm e sem bleed;
- composição predominantemente vertical/uma coluna;
- cabeçalho institucional compacto, logo opcional, sem título gigante e sem footer obrigatório;
- `CANCELADO` textual/inequívoco; acompanhamento discreto;
- cliente/OS/técnico em linha curta, omitindo vazios;
- Equipamento em ficha técnica resumida sem grade/tabela pesada;
- `SERVIÇO REALIZADO` usa o `Resumo do trabalho` como área narrativa principal;
- uma única seção `OBSERVAÇÕES` reúne observações relevantes do Atendimento, Equipamento e Etapas;
- nome curto da Etapa só aparece quando necessário para contexto;
- seções vazias colapsam completamente;
- PDF usa Noto Sans: 14 pt identificação principal, 10,5 pt seção, 10 pt corpo, 9 pt ficha técnica e 8,5 pt metadados;
- divisórias discretas e contraste neutro legível em monocromático;
- sem caixas de escrita manual, assinatura, financeiro, garantia, checklist, progresso, timeline, QR/barcode, lista detalhada de Procedimentos, página 2 ou footer promocional;
- sem redução dinâmica de fonte; overflow continua `SHEET_OVERFLOW`;
- Etapa 8 define limites/priorização/densidade; Etapa 9 trata dados excepcionais.

## Pendências ainda não consolidáveis para implementação

- Etapa 8: limites textuais/priorização/densidade/diagnóstico de overflow da Ficha;
- Etapa 9: muitos MACs/Procedimentos/observações e dados excepcionais;
- Etapa 10: nomes/paths/limpeza concretos de artefatos temporários;
- Etapa 11: QR/barcode apenas se benefício aprovado;
- Etapa 12: matriz técnica final e limites de recursos;
- mecanismo técnico final de Backup/Restore;
- parâmetros finais de autenticação;
- Gerência × configuração da empresa;
- Gerência × Backup;
- regra editorial de categoria arquivada;
- parâmetros reais do ambiente corporativo.

## Gate de implementação da Fase 1

Na Fase 1, trabalho estrutural significa documentação, organização documental ou PoC explicitamente descartável autorizada. Não criar scaffold oficial, módulos runtime definitivos, árvore final ou código de negócio antes do Bloco 12/Fase 2 autorizar.

Antes do primeiro trabalho de implementação com Codex, sincronizar explicitamente o checkout local com o remoto sem apagar, sobrescrever, descartar ou incorporar indevidamente alterações preexistentes do PO.

Se a fase/documentação não autorizar implementação definitiva, limitar-se ao trabalho documental/investigativo solicitado.
