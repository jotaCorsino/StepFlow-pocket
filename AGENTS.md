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
- Bloco 10 possui **Etapas 1–11 CONSOLIDADAS / APROVADAS PELO PO**; o fechamento operacional ainda depende do PR da Etapa 11, squash merge, remoção da branch e remoto limpo.
- Bloco 11 fecha Backup/Restore técnico; Bloco 12 fecha estrutura oficial, parâmetros finais e plano da Fase 2.
- Nenhuma implementação funcional oficial foi iniciada.

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
- `docs/04-planejamento/bloco-10-etapa-11-validacao-tecnica-final.md`;
- `docs/03-arquitetura/launcher-distribuicao-client.md`;
- `docs/03-arquitetura/compatibilidade-windows-client.md`.

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

**Gate atual obrigatório:** o Bloco 11 não pode ser aberto antes de o PR da Etapa 11 do Bloco 10 estar squash-mergeado, a branch correspondente removida do remoto e o remoto verificado com somente `main` e zero PRs abertos.

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
- protótipo/PoC descartável não vira produção silenciosamente.

## Ambiente Codex versus sessão normal do PO

Limitação do sandbox não vira requisito do produto. Codex não repara o próprio ambiente alterando ACL, Schannel, registro Windows, PATH global, políticas de segurança ou reinstalando ferramentas válidas. Operações que exijam credenciais, Internet confiável, elevação ou configuração global são reportadas para a sessão Windows normal do PO.

## Contrato Pocket obrigatório

O StepFlow deve funcionar como aplicação **Pocket**, no sentido operacional aprovado:

```text
pasta pronta publicada no servidor Windows
→ estação acessa o compartilhamento
→ usuário executa StepFlowLauncher.exe
→ Launcher prepara/valida o Client local automaticamente
→ Client abre de %LOCALAPPDATA%
→ Launcher encerra
```

É obrigatório:

- zero instalador tradicional obrigatório por estação;
- zero configuração manual de dependência por estação;
- zero privilégio administrativo no uso normal;
- zero toolchain de desenvolvimento na estação e máquina central de produção;
- nenhuma Internet obrigatória para uso normal;
- Client operacional local, não executado permanentemente do SMB;
- distribuição/atualização central pela pasta publicada e versões validadas pelo Launcher;
- Host/Controller sob demanda na máquina central;
- fechar Client individual não encerra Host;
- encerrado o ciclo central, nenhum processo StepFlow permanece ativo;
- sem Windows Service persistente, auto-start, Task Scheduler, watchdog, tray agent ou daemon como baseline.

Se uma dependência exigir instalação, elevação ou preparação manual por computador, a solução **não atende ao contrato Pocket** e deve ser redesenhada ou tratada como bloqueador técnico.

### WebView2 e Pocket

- Tauri usa WebView2 no Windows;
- Evergreen compatível já presente é preferível;
- Launcher/Client deve detectar disponibilidade real;
- não baixar/instalar runtime silenciosamente pela Internet em produção;
- Fixed Version não pode ser executado de localização de rede/UNC;
- eventual fallback Fixed/autocontido deve ser preparado localmente e só pode ser adotado após PoC provar funcionamento em `%LOCALAPPDATA%` sem instalação, elevação ou ação manual, inclusive no Windows 10 alvo;
- requisitos ACL/AppContainer de Fixed Runtime moderno no Windows 10 precisam ser automatizáveis sem enfraquecer o contrato Pocket;
- se a PoC falhar numa estação que deva ser suportada, o fallback é bloqueador até redesign; não autoriza instalador obrigatório.

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

- primeiro save cria Atendimento; abrir tela não cria registro oficial;
- responsável + `Resumo do trabalho` são obrigatórios para conclusão;
- checklist incompleto gera confirmação, não bloqueio automático;
- cancelamento exige motivo;
- Concluído/Cancelado são read-only até reabertura;
- ADM/Gerência reabrem por preset; Funcionário não;
- Funcionário cria Atendimento inicialmente para si e opera por responsabilidade;
- ADM/Gerência podem atribuir/reatribuir e operar qualquer Atendimento acessível;
- Funcionário seleciona revisão publicada;
- ADM/Gerência podem selecionar explicitamente revisão histórica/não publicada já autorizada;
- revisão vinculada nunca muda silenciosamente após nova publicação;
- checklist usa concorrência por item/equivalente;
- observação do serviço usa concorrência por Etapa/equivalente;
- evento remoto não sobrescreve edição local silenciosamente;
- não introduzir autosave por inferência;
- Equipamento usa `EQP-000001`, Atendimento usa `AT-000001`, códigos Host-only com seis dígitos e gaps permitidos;
- não arquivar Equipamento vinculado a Atendimento `Em andamento`;
- conclusão preserva estado histórico necessário à Ficha/reimpressão.

## Bloco 10 — arquitetura documental consolidada / Etapas 1–11

Fonte principal: `docs/04-planejamento/bloco-10-exportacao-impressao-ficha.md`.

### Etapa 1 — arquitetura

- geração pertence ao Host;
- Client solicita identidade da fonte + revisão esperada, sem documento montado;
- Host captura snapshot consistente e materializa `DocumentModel`;
- leitura/transação SQLite termina antes da renderização;
- renderers não recebem DOM/HTML nem reconsultam banco;
- geração é leitura derivada, fora da fila de mutações;
- renderização usa capacidade própria bounded;
- sem `export_jobs` persistente inicialmente;
- Host não grava em path arbitrário da workstation;
- artefatos não viram histórico/backup por padrão.

### Etapa 2 — PDF

- Typst embutido via crates oficiais + adaptador interno;
- `World` restrito a template/fontes/assets autorizados;
- PDF 1.7 + Tagged PDF baseline;
- texto selecionável/pesquisável;
- fontes incorporadas/subsetadas;
- multipágina automático;
- falha não produz parcial como sucesso.

### Etapa 3 — DOCX

- OOXML/WordprocessingML/OPC Transitional direto em Rust;
- `docx-rs` preferido sob adaptador;
- sem Word/COM, LibreOffice, browser/headless ou cloud;
- texto/listas editáveis;
- Arial/Consolas referenciadas sem embedding inicial;
- DOCX refluível e sem promessa de paginação do PDF.

### Etapa 4 — impressão Windows

- mesmo PDF oficial;
- Client Windows;
- recurso local transitório quando necessário;
- WebView2 dedicada/transitória;
- acesso nativo isolado + `ShowPrintUI(System)`;
- diálogo Windows, sem impressão silenciosa/seletor próprio baseline;
- sucesso = entrega ao fluxo Windows.

### Etapa 5 — Procedimento físico

- A4 retrato multipágina;
- margens-base 18 mm;
- sem capa exclusiva/sumário obrigatório/header repetitivo por padrão;
- rodapé compacto;
- sem truncamento/redução silenciosa;
- PDF é referência física; DOCX é refluível.

### Etapas 6–9 — Ficha

- prestação de contas resumida ao cliente;
- PDF + preview SVG do mesmo `PagedDocument`;
- exatamente uma página A4, margens 15 mm;
- `2+ páginas` = `SHEET_OVERFLOW`;
- Salvar/Imprimir reutilizam os mesmos bytes PDF da prévia;
- soft limits: Resumo 600, Atendimento 400, Equipamento 300, observação por Etapa 280;
- soft limit não bloqueia nem trunca;
- correção ocorre nos dados reais, sem editor paralelo/IA/resumo automático/compactação automática;
- Procedimentos vinculados não aparecem por padrão;
- MACs: 0 omite; 1–2 exibem valores; 3+ exibem quantidade;
- observações legítimas não recebem cap/descarte automático;
- multiplicidade pode produzir `SHEET_OVERFLOW`.

### Etapa 10 — nomes e temporários

- Procedimento: `{codigo} - {titulo} - v{display_version} - r{revision_no}.{ext}`; sem versão editorial, omite esse segmento;
- Ficha: `{service_code} - Ficha.pdf`;
- sanitização afeta somente filename e impede path injection;
- conflito não causa overwrite silencioso;
- save só é sucesso após gravação integral;
- temporário somente quando integração local exige filesystem;
- raiz temporária por usuário sob namespace StepFlow e diretório opaco por Client;
- cleanup/scavenging best-effort;
- reparse point nunca seguido para fora da raiz gerenciada;
- sem serviço/daemon/tarefa agendada de limpeza.

### Etapa 11 — validação técnica final

Fonte: `docs/04-planejamento/bloco-10-etapa-11-validacao-tecnica-final.md`.

- nenhum bloqueador arquitetural identificado;
- Typst/PDF/PagedDocument validados;
- DOCX direto validado com limite e teste de Word corporativo pendente;
- impressão Windows validada via WebView2 nativo + `ShowPrintUI(System)`;
- adapter Tauri/Wry/WebView2 deve usar família pinada/testada;
- save local/naming/temporários/scavenging viáveis com limites explícitos;
- SMB, Word, impressoras e EDR pendentes de ambiente real;
- memória/tamanho/concorrência/fila/timeout serão definidos por benchmark;
- Fixed WebView2 por UNC/SMB não utilizar;
- fallback WebView2 local exige PoC sem instalação/elevação/manualidade;
- requisito Pocket é gate superior.

## Pendências ainda não consolidáveis para implementação

- mecanismo técnico final de Backup/Restore;
- parâmetros finais de autenticação;
- Gerência × configuração da empresa;
- Gerência × Backup;
- regra editorial de categoria arquivada;
- validações reais de Windows/WebView2/Word/impressoras/SMB/EDR;
- parâmetros reais do ambiente corporativo;
- estrutura oficial/Fase 2 no Bloco 12.

## Gate de implementação da Fase 1

Na Fase 1, trabalho estrutural significa documentação, organização documental ou PoC explicitamente descartável autorizada. Não criar scaffold oficial, módulos runtime definitivos, árvore final ou código de negócio antes do Bloco 12/Fase 2 autorizar.

Antes do primeiro trabalho de implementação com Codex, sincronizar explicitamente o checkout local com o remoto sem apagar, sobrescrever, descartar ou incorporar indevidamente alterações preexistentes do PO.

Se a fase/documentação não autorizar implementação definitiva, limitar-se ao trabalho documental/investigativo solicitado.
