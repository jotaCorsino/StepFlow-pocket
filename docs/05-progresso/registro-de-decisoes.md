# Registro de Decisões — StepFlow Pocket

**Atualização:** 2026-08-21

Este arquivo registra apenas decisões vigentes e pendências atuais. Discussões, troubleshooting, provas descartáveis e propostas superadas permanecem no histórico Git.

## Produto e UX

### Nome e propósito

- produto: **StepFlow**;
- aplicação interna para documentação e execução guiada de processos técnicos;
- evitar burocracia e campos sem valor operacional;
- etapas funcionam como páginas de manual/livro;
- sidebar esquerda, logo pequeno no topo esquerdo;
- blocos copiáveis usam ícone discreto;
- PDF, DOCX e impressão são requisitos obrigatórios.

### Processo documental

Campos principais:

- Código;
- Título;
- Área/Departamento;
- Responsável;
- Status;
- Versão;
- Objetivo;
- Observações;
- Pré-requisitos;
- Etapas;
- Histórico.

### Perfis

- ADM;
- Gerência;
- Funcionário.

Gerência não administra ADM. Funcionário é leitura/execução por padrão. Usuário pode editar avatar, nome de exibição, cargo e senha dentro das regras.

## Governança

- GitHub é a fonte principal de verdade;
- método PO + Assistente + Codex;
- uma tarefa por vez;
- toda tarefa Codex exige pré-flight de modelo/raciocínio separado do prompt;
- `AGENTS.md` é a regra operacional superior para agentes;
- o prompt da tarefa autoriza escopo, mas não revoga silenciosamente decisão consolidada;
- mudança de decisão exige indicação explícita de aprovação do PO e atualização dos documentos vigentes afetados;
- toda tarefa Codex que altere arquivos deve declarar branch/base e commit SHA esperado;
- alteração preexistente no working tree pertence ao PO/outro fluxo e não pode ser resetada, stashada, limpa, sobrescrita nem incorporada à tarefa;
- se a base Git estiver diferente ou um arquivo necessário já estiver modificado, o Codex para e reporta;
- Fase 1 está em andamento; Blocos 0–7 concluídos no núcleo arquitetural; Bloco 8 é o próximo.

## Ambientes

- desenvolvimento atual ocorre fora da LAN corporativa;
- sessão Windows normal do PO e sandbox Codex são contextos distintos;
- limitação específica do sandbox Codex não vira requisito do StepFlow;
- Codex não deve alterar ACL, Schannel, registro, PATH global ou segurança para reparar seu ambiente;
- operações que realmente dependam de credenciais, elevação, Internet confiável ou configuração global devem ser reportadas para execução controlada na sessão normal do PO;
- IP, hostname, share e paths reais da empresa ainda não foram confirmados;
- exemplos históricos de infraestrutura não são configuração;
- testes de LAN/SMB feitos fora do ambiente real não validam nem bloqueiam a implantação corporativa.

## Pocket / máquina central

**Consolidado:** implantação por pasta pronta, com mínimo impacto no Windows.

- nenhuma toolchain/runtime de desenvolvimento exigida na máquina central;
- nenhum Windows Service StepFlow persistente;
- nenhum auto-start, Task Scheduler, watchdog, tray agent ou daemon StepFlow como padrão;
- Controller/Host iniciam sob demanda;
- encerrado o ciclo central, nenhum processo StepFlow deve permanecer ativo;
- dados/config/logs/backups separados dos binários substituíveis.

## Client Windows

- **Tauri 2 + HTML/CSS/JavaScript modular**;
- alvo inicial Windows 10/11 x64;
- WebView2 como renderer;
- prova técnica confirmou executável isolado sem Node/npm/Rust/Cargo em runtime;
- Electron é contingência, não alternativa em avaliação ativa.

## Launcher

- launcher Rust x64 portátil/transitório;
- ponto de entrada interno → cópia Client versionada em `%LOCALAPPDATA%` → execução local → launcher encerra;
- versões lado a lado e SHA-256;
- sem instalador obrigatório/updater residente;
- launcher não inicia remotamente o Host central.

## Host

- **Rust + Tokio/Axum + `rusqlite` bundled**;
- Controller portátil na máquina central inicia Host como processo-filho;
- readiness, instância única e shutdown gracioso obrigatórios;
- ciclo de vida central pertence ao Controller;
- fechar um Client individual não encerra o Host central;
- encerrar o Controller/ciclo central solicita shutdown gracioso do Host;
- auto-shutdown por último Client ou timeout **não está consolidado** e não pode ser implementado por suposição;
- primeiro start central depende de ação na máquina central ou mecanismo corporativo já existente/aprovado.

## Comunicação

- HTTP/JSON para API;
- WebSocket para eventos;
- contratos versionados inicialmente em `/api/v1`;
- endpoint/configuração via `deployment.json` sem segredos;
- handshake de compatibilidade antes do login;
- primeira versão sem edição offline.

## Autenticação e autorização

Consolidado:

- Argon2id para senhas;
- sessão opaca server-side;
- token somente em memória do Client inicialmente;
- autorização por capacidade sempre no Host;
- bootstrap do ADM principal somente em fluxo local/controlado;
- desativar usuário em vez de excluir quando houver histórico.

Ainda não são contratos definitivos de implementação:

- custo exato do Argon2id;
- tamanho mínimo final de senha;
- tempos finais de expiração da sessão;
- permissão da Gerência para configuração da empresa;
- permissão da Gerência para backup.

Esses itens precisam de decisão explícita antes da implementação correspondente.

## Dados e histórico

- SQLite local ao Host;
- revisões de processo imutáveis;
- `revision_no` técnico separado de `display_version`;
- etapas/blocos estruturados por revisão;
- migrations numeradas/versionadas;
- auditoria append-only separada de logs;
- arquivamento/desativação preferidos à exclusão destrutiva normal.

## Concorrência

- WAL;
- um writer lógico coordenado;
- fila bounded/backpressure;
- revisão otimista obrigatória onde houver risco de perda concorrente;
- conflitos não sobrescrevem automaticamente;
- eventos somente pós-commit;
- sem soft/hard lock inicial de edição;
- dois Hosts não podem usar o mesmo data dir.

## Fase 1 / limites de implementação

- não criar scaffold oficial, árvore runtime definitiva ou código de negócio antes do Bloco 12/Fase 2;
- PoC na Fase 1 só quando explicitamente autorizada e descartável;
- Bloco 8 fecha UX/fluxos/estados, mas não escolhe tecnologia de exportação nem mecanismo técnico de backup;
- Dashboard faz parte da sequência obrigatória de análise do Bloco 8;
- parâmetros marcados como `PROPOSTA`, `PENDENTE` ou equivalentes não podem ser transformados em implementação definitiva pelo executor.

## Pendências vigentes

### Bloco 8 — UI/UX

Especificar e aprovar telas críticas, incluindo a UX de encerramento central quando houver Clients conectados.

### Bloco 9 — checklist durante execução

Definir se as marcações são temporárias, locais, persistidas por usuário ou entidade formal de execução.

### Bloco 10 — exportação

Escolher arquitetura/bibliotecas para PDF, DOCX e impressão offline.

### Bloco 11 — backup/restore

Fechar formato, retenção, validação e restauração segura.

### Bloco 12 — estrutura/Fase 2

Fechar parâmetros operacionais ainda necessários à implementação, definir árvore oficial, scripts, contratos/testes e plano da fundação executável.

### Ambiente corporativo

Confirmar Windows/WebView2, paths/SMB/permissões, hostname/porta, transporte HTTP/HTTPS, antivírus/EDR/firewall e mecanismo real de start do Controller central.
