# AGENTS.md — StepFlow Pocket

## Finalidade

Regras obrigatórias para agentes que atuem neste repositório. Este arquivo governa **como trabalhar**; detalhes de produto e arquitetura pertencem às fontes específicas.

## Fonte de verdade e estado

- GitHub é a fonte operacional principal de verdade.
- Branch principal: `main`.
- Checkout local previsto: `C:\dev\StepFlow`.
- Fase vigente: **Fase 1 — Fechamento arquitetural e especificação**.
- Blocos 0–11 estão encerrados.
- Bloco 12 está em análise; D12.1–D12.79 estão aprovadas e o plano da Fase 2 permanece em revisão.
- Nenhuma implementação funcional oficial foi iniciada.

Estado executivo corrente pertence ao `README.md` e ao plano da fase, não a documentos técnicos estáveis.

## Precedência

Em caso de conflito:

1. `AGENTS.md`;
2. `docs/05-progresso/registro-de-decisoes.md`;
3. documento específico vigente;
4. `docs/01-produto/visao-geral.md`;
5. tarefa, dentro das decisões vigentes;
6. histórico Git.

O enunciado autoriza trabalho, mas não revoga silenciosamente decisão consolidada. Se a tarefa contrariar decisão vigente, parar e retornar ao PO/Assistente. Ambiguidade nunca autoriza escolher a alternativa mais conveniente.

## Leitura por camadas

### Sempre

1. `AGENTS.md`;
2. enunciado da tarefa;
3. `docs/README.md`;
4. documento específico indicado.

### Conforme impacto

- `docs/05-progresso/registro-de-decisoes.md`;
- `docs/04-planejamento/plano-oficial-fase-1.md`;
- `docs/03-arquitetura/arquitetura-vigente.md`;
- `docs/03-arquitetura/modelo-dados-schema-fase-1.md`;
- `docs/00-governanca/contexto-ambientes.md`;
- documento de Produto/Tela/Arquitetura diretamente afetado.

## Papéis

- **PO:** autoridade de produto, prioridade, comportamento e aprovação visual/funcional.
- **Assistente:** análise, arquitetura, coerência documental, gates e preparação de tarefas.
- **Codex:** execução técnica do escopo aprovado, sem inventar produto ou ampliar tarefa.

## Pré-flight para Codex

Antes de **cada nova tarefa Codex**, inclusive PoC:

1. entregar `PRÉ-FLIGHT PARA O PO — NÃO ENVIAR AO CODEX`;
2. entregar `PROMPT / ENUNCIADO PARA O CODEX` separado;
3. usar o menor perfil de capacidade suficiente, conforme `docs/00-governanca/politica-capacidade-codex.md`.

## Base Git obrigatória

Toda tarefa que permita alteração informa branch/base e commit SHA esperado.

Antes de escrever:

```text
git rev-parse HEAD
git status --short --branch
```

Se `HEAD` divergir, não fazer `pull`, `merge`, `rebase`, `reset` ou checkout corretivo automaticamente. Parar e reportar.

## Proteção absoluta do working tree

Alteração preexistente pertence ao PO/outro fluxo.

Sem autorização explícita e específica do PO, é proibido:

- `git reset --hard`;
- `git clean`;
- `git stash`;
- descartar/restaurar alterações locais;
- sobrescrever arquivo modificado preexistente;
- trocar branch descartando trabalho;
- incluir alteração preexistente no commit da tarefa.

Se arquivo necessário já estiver modificado, parar e reportar.

## Disciplina Git

```text
1 trabalho lógico
→ 1 branch ativa
→ 1 PR
→ revisão/aprovação
→ squash merge em main
→ apagar branch encerrada
→ verificar remoto limpo
→ próximo trabalho
```

- branch mergeada não está encerrada enquanto permanecer no remoto;
- remoto é a fonte operacional;
- durante o fechamento da Fase 1, sincronização do checkout local permanece adiada até antes do primeiro trabalho de implementação;
- a proposta da Fase 2 mantém uma tarefa lógica por branch/PR; só vira contrato após aprovação do PO.

## Regras operacionais

- uma tarefa lógica por vez;
- não declarar parcial como concluído;
- não criar funcionalidade, dependência ou estrutura fora do escopo;
- não alterar UX/visual aprovado sem autorização;
- não transformar proposta/exemplo/valor provisório em decisão;
- manter documentação e implementação sincronizadas;
- atualizar painel do `README.md` quando houver mudança real de fase/bloco;
- preservar modularidade, responsabilidades claras e baixo acoplamento;
- evitar monólito HTML/JS;
- não versionar credenciais, segredos, banco real ou dados pessoais;
- exemplos de IP/hostname/share/path nunca viram configuração oficial;
- testes de LAN corporativa fora dela são `NÃO APLICÁVEIS NESTE AMBIENTE`;
- PoC descartável não vira produção silenciosamente;
- documento técnico estável não carrega “próximo bloco” nem gate Git histórico consumido.

## Ambiente Codex versus sessão normal do PO

Limitação do sandbox não vira requisito do produto. Codex não repara o próprio ambiente alterando ACL, Schannel, registro, PATH global, políticas de segurança ou reinstalando ferramentas válidas.

Operações que exijam credenciais, Internet confiável, elevação ou configuração global devem ser reportadas para execução apropriada na sessão Windows normal do PO.

## Contrato Pocket obrigatório

```text
pasta pronta publicada no servidor Windows
→ estação acessa compartilhamento
→ usuário executa StepFlow.exe na raiz
→ Launcher prepara/valida Client local automaticamente
→ Client abre de %LOCALAPPDATA%
→ Launcher encerra
```

`StepFlow.exe` é o Launcher com nome/ícone amigáveis e único ponto de entrada normal. Artefatos publicados ficam sob `_internal/`; `.lnk` não é requisito baseline.

É obrigatório no uso normal:

- zero instalador tradicional por estação;
- zero configuração manual de dependência;
- zero privilégio administrativo;
- zero toolchain de desenvolvimento em produção;
- nenhuma Internet obrigatória;
- Client operacional local, não permanentemente do SMB;
- atualização central pela pasta publicada + versões locais validadas;
- Host/Controller sob demanda;
- fechar Client individual não encerra Host;
- encerrado o ciclo central, nenhum processo StepFlow permanece;
- sem Windows Service persistente, auto-start, Task Scheduler, watchdog, tray agent ou daemon baseline.

Se dependência exigir instalação/elevação/preparação manual por computador, a solução não atende ao Pocket e deve ser redesenhada ou tratada como bloqueador.

### WebView2

- Tauri usa WebView2 no Windows;
- Evergreen compatível já presente é preferível;
- disponibilidade real deve ser detectada;
- não baixar/instalar runtime silenciosamente pela Internet em produção;
- Fixed Version não pode executar de UNC/SMB;
- fallback autocontido deve ser local e só entra após PoC provar `%LOCALAPPDATA%` sem instalação, elevação ou ação manual, inclusive no Windows 10 alvo;
- falha dessa PoC em estação suportada é bloqueador, não autorização para enfraquecer o Pocket.

Fontes: `docs/03-arquitetura/implantacao-pocket.md`, `launcher-distribuicao-client.md`, `compatibilidade-windows-client.md`.

## Invariantes técnicas consolidadas

- Client: Tauri 2 + HTML/CSS/JavaScript modular;
- Host: Rust + Tokio/Axum + `rusqlite` bundled;
- HTTP/JSON + WebSocket;
- SQLite somente pelo Host;
- WAL + writer lógico coordenado + fila bounded + revisão otimista;
- nenhuma sobrescrita silenciosa;
- sessão opaca e autorização Host-side;
- Procedimentos usam revisões imutáveis;
- `Procedimento`, `Atendimento/Execução` e `Equipamento` são domínios distintos;
- Atendimento preserva revisão usada;
- checklist persiste somente em Atendimento;
- `Observação do serviço` persiste opcionalmente por Etapa somente em Atendimento;
- histórico relevante precisa ser reproduzível;
- Reader usa experiência livro/manual;
- geração documental pertence ao Host e usa snapshot consistente + `DocumentModel`;
- PDF usa Typst embutido; DOCX usa OOXML direto em Rust; impressão usa o mesmo PDF via WebView2;
- Ficha válida possui exatamente uma A4; `2+` páginas = `SHEET_OVERFLOW`;
- Backup/Restore segue D11.1–D11.116;
- Backup = ADM/Gerência; Restore = ADM-only;
- disaster recovery é local/transitório pelo Controller;
- Restore destrutivo invalida sessões antigas;
- `uncertain` bloqueia readiness/mutações/cleanup destrutivo;
- safety backup `pre_restore` mantém barrier até o primeiro rename;
- paths Backup/Restore usam semântica Windows estrita e provenance por deployment;
- D12.1–D12.18 definem ownership e publicação `StepFlow.exe + _internal/`;
- D12.19–D12.34 definem toolchain/workspace/build/dependências;
- D12.35–D12.55 definem migrations/testes/fixtures;
- D12.56–D12.79 definem parâmetros iniciais de autenticação, empresa, categoria arquivada, Backup/Restore, reconexão e logs;
- aprovação dessas decisões ainda não autoriza scaffold antes do gate final da Fase 1.

### Parâmetros D12 relevantes

- Argon2id: 64 MiB / 3 passes / 4 lanes; senha 15–128 após NFKC;
- token opaco 32 bytes; sessão 30 min idle / 8 h absoluta;
- Gerência pode alterar configuração da empresa;
- categoria arquivada herdada permanece com aviso, mas não pode ser adicionada novamente enquanto arquivada;
- retenção de Backup default 20, faixa 5–100;
- Backup capture hard limit 10 s; pre_restore 120 s sem progresso / 10 min pré-destrutivo;
- readiness 30 s; relaunch Restore máximo 3 tentativas;
- connect 5 s / request comum 30 s; WebSocket backoff bounded;
- números ficam centralizados; configuração crítica inválida não cai silenciosamente em default inseguro.

Detalhes pertencem às fontes específicas.

## Pendências ainda abertas

- P12.80–P12.98: plano detalhado da Fase 2 e sequência de tarefas Codex;
- validação final da Fase 1;
- gate Git do Bloco 12;
- sincronização segura do checkout local antes do primeiro trabalho executável;
- gates corporativos de Windows/WebView2/Launcher/SMB/Word/impressoras/filesystem/ACL/EDR e adapters aplicáveis.

## Gate de implementação

Na Fase 1, trabalho estrutural significa documentação ou PoC explicitamente descartável autorizada. Não criar scaffold oficial, módulos runtime definitivos, árvore final ou código de negócio antes do Bloco 12/Fase 2 autorizar.

Antes do primeiro trabalho de implementação, sincronizar explicitamente o checkout local com o remoto **sem apagar, sobrescrever, descartar ou incorporar indevidamente** alterações preexistentes do PO.

## Regra final

Executar somente o escopo autorizado, preservar o consolidado e preferir referência à fonte específica em vez de duplicar contratos em governança.
