# Registro de Decisões — StepFlow Pocket

**Atualização:** 2026-08-21

Este arquivo registra apenas decisões vigentes e pendências atuais. Discussões, troubleshooting, provas descartáveis e propostas superadas permanecem no histórico Git.

## Produto e UX

### Nome e propósito

- produto: **StepFlow**;
- aplicação interna para documentação e execução guiada de processos técnicos;
- uso não restrito à manutenção de computadores;
- deve acomodar manutenção, TI, Service Desk, Help Desk, infraestrutura/servidores, redes, guias e outros procedimentos internos;
- evitar burocracia e campos sem valor operacional;
- etapas funcionam como páginas de manual/livro;
- sidebar esquerda, logo pequeno no topo esquerdo;
- blocos copiáveis usam ícone discreto;
- PDF, DOCX e impressão são requisitos obrigatórios para documentação.

### Procedimento documental

Campos principais vigentes:

- Código;
- Título;
- Área/Departamento;
- Categorias;
- Responsável;
- Status;
- Versão;
- Objetivo;
- Observações;
- Pré-requisitos;
- Etapas;
- Histórico.

### Categorização

- categorias são configuráveis, não hardcoded;
- um procedimento pode pertencer a uma ou mais categorias;
- categorias são pesquisáveis/filtráveis;
- exemplos de categoria não formam enumeração fixa;
- não criar hierarquia/taxonomia complexa sem necessidade comprovada.

### Atendimento/execução e equipamento

Nova decisão de produto consolidada em 2026-08-21:

- `Procedimento` é o modelo oficial reutilizável;
- `Atendimento/Execução` é uma ocorrência real de trabalho;
- `Equipamento` é entidade opcional associável a atendimentos;
- um atendimento pode usar um ou mais procedimentos;
- atendimento deve preservar a revisão do procedimento efetivamente utilizada;
- procedimentos podem existir sem atendimento formal;
- atendimentos podem existir sem equipamento.

Para computadores/notebooks, a ficha de equipamento suporta informações como nome, cliente/solicitante, processador, RAM, armazenamento, SO/versão, serial/patrimônio, MAC(s), saúde de bateria quando aplicável e observações.

A identidade canônica do equipamento é interna ao StepFlow. MAC/serial/patrimônio são atributos úteis para busca, não chave canônica obrigatória.

O sistema deve permitir busca operacional por código do atendimento/equipamento, OS/referência externa, cliente/solicitante, nome do equipamento, serial/patrimônio e MAC normalizado quando disponível.

### Ficha compacta

Atendimento com equipamento deve permitir gerar uma ficha/relatório compacto imprimível, com identidade, características principais, resumo do serviço/procedimentos e observações. A saída é documento próprio, não screenshot.

Tamanho físico, formatos finais e estratégia técnica pertencem ao Bloco 10.

### Perfis

- ADM;
- Gerência;
- Funcionário.

Gerência não administra ADM. Funcionário é leitura/execução por padrão. Usuário pode editar avatar, nome de exibição, cargo e senha dentro das regras.

A matriz exata de permissões para categorias, equipamentos e atendimentos permanece pendente para o Bloco 9.

## Governança

- GitHub é a fonte principal de verdade;
- método PO + Assistente + Codex;
- uma tarefa por vez;
- toda tarefa Codex exige pré-flight de modelo/raciocínio separado do prompt;
- `AGENTS.md` é a regra operacional superior para agentes;
- prompt autoriza escopo, mas não revoga silenciosamente decisão consolidada;
- mudança de decisão exige indicação explícita de aprovação do PO e atualização dos documentos vigentes;
- toda tarefa Codex que altere arquivos declara branch/base + SHA;
- alteração preexistente no working tree pertence ao PO/outro fluxo e não pode ser resetada, stashada, limpa, sobrescrita nem incorporada à tarefa;
- base divergente ou arquivo necessário já modificado faz o Codex parar/reportar;
- Fase 1 está em andamento; Bloco 8 em execução.

## Ambientes

- desenvolvimento atual ocorre fora da LAN corporativa;
- sessão Windows normal do PO e sandbox Codex são contextos distintos;
- limitação do sandbox não vira requisito do StepFlow;
- Codex não altera ACL, Schannel, registro, PATH global ou segurança para reparar ambiente;
- operações dependentes de credenciais/elevação/Internet confiável/configuração global são reportadas para sessão normal do PO;
- infraestrutura real da empresa ainda não está confirmada;
- exemplos históricos não são configuração;
- testes LAN/SMB fora do ambiente real não validam nem bloqueiam implantação.

## Pocket / máquina central

- implantação por pasta pronta;
- nenhuma toolchain/runtime de desenvolvimento exigida na máquina central;
- nenhum Windows Service StepFlow persistente;
- nenhum auto-start, Task Scheduler, watchdog, tray agent ou daemon StepFlow como padrão;
- Controller/Host iniciam sob demanda;
- encerrado o ciclo central, nenhum processo StepFlow permanece ativo;
- dados/config/logs/backups separados dos binários substituíveis.

## Client Windows

- Tauri 2 + HTML/CSS/JavaScript modular;
- alvo inicial Windows 10/11 x64;
- WebView2 como renderer;
- executável isolado sem Node/npm/Rust/Cargo no runtime;
- Electron é contingência, não alternativa ativa.

## Launcher

- launcher Rust x64 portátil/transitório;
- ponto de entrada interno → Client versionado em `%LOCALAPPDATA%` → execução local → launcher encerra;
- versões lado a lado e SHA-256;
- sem instalador obrigatório/updater residente;
- launcher não inicia remotamente Host central.

## Host

- Rust + Tokio/Axum + `rusqlite` bundled;
- Controller portátil inicia Host como filho;
- readiness, instância única e shutdown gracioso obrigatórios;
- ciclo central pertence ao Controller;
- fechar Client individual não encerra Host;
- auto-shutdown por último Client/timeout não está consolidado;
- primeiro start central depende de ação local ou mecanismo corporativo existente/aprovado.

## Comunicação

- HTTP/JSON para API;
- WebSocket para eventos;
- contratos inicialmente em `/api/v1`;
- `deployment.json` sem segredos;
- handshake de compatibilidade antes do login;
- primeira versão sem edição offline.

## Autenticação e autorização

Consolidado:

- Argon2id;
- sessão opaca server-side;
- token somente em memória do Client inicialmente;
- autorização sempre Host-side;
- bootstrap do ADM principal somente local/controlado;
- desativação preferida à exclusão quando houver histórico.

Pendentes antes da implementação correspondente:

- custo exato Argon2id;
- tamanho mínimo final de senha;
- expiração final de sessão;
- permissão da Gerência para configuração da empresa;
- permissão da Gerência para backup;
- permissões operacionais de categorias/equipamentos/atendimentos.

## Dados e histórico

- SQLite local ao Host;
- revisões de procedimento imutáveis;
- `revision_no` separado de `display_version`;
- categorias configuráveis com relação múltipla a procedimentos;
- equipamentos com identidade interna + código legível;
- atendimentos formais com equipamento opcional;
- atendimento referencia revisão de procedimento utilizada;
- migrations numeradas/versionadas;
- auditoria append-only separada de logs;
- arquivamento/desativação preferidos à exclusão destrutiva normal.

A antiga exclusão de “entidade formal de execução” está **superada pelo requisito de 2026-08-21** e não é mais válida.

## Concorrência

- WAL;
- writer lógico coordenado;
- fila bounded/backpressure;
- revisão otimista onde houver risco de perda concorrente;
- conflitos não sobrescrevem automaticamente;
- eventos somente pós-commit;
- sem soft/hard lock inicial de edição;
- dois Hosts não usam o mesmo data dir;
- equipamentos/atendimentos adotam controle otimista equivalente quando aplicável.

## Fase 1 / UI

- sem scaffold/código de produção antes do Bloco 12/Fase 2;
- PoC apenas quando autorizada e descartável;
- Bloco 8 fecha UX/fluxos/estados, sem escolher tecnologia de exportação/backup;
- Login consolidado;
- núcleo do Shell consolidado, reaberto apenas para decidir item `Atendimentos`;
- Dashboard em análise;
- mapa do Bloco 8 expandido para categorias, atendimentos e equipamento.

## Pendências vigentes

### Bloco 8 — UI/UX

- aprovar Dashboard;
- decidir `Atendimentos` como item próprio da sidebar;
- especificar lista/leitor/editor com categorias;
- especificar lista de atendimentos + ficha de equipamento;
- seguir demais telas previstas.

### Bloco 9 — execução operacional, atendimento e checklist

Definir:

- lifecycle do atendimento;
- marcações/progresso do checklist;
- conclusão/reabertura;
- vínculo operacional com revisão do procedimento;
- concorrência/histórico específicos;
- matriz de permissões de categorias/equipamentos/atendimentos.

### Bloco 10 — exportação

- arquitetura/bibliotecas para PDF, DOCX e impressão;
- formato/layout da ficha compacta de atendimento/equipamento;
- decidir se ficha exige PDF além de impressão direta;
- QR/barcode somente se houver valor aprovado.

### Bloco 11 — backup/restore

Fechar formato, retenção, validação e restauração segura incluindo os novos dados operacionais.

### Bloco 12 — estrutura/Fase 2

Fechar parâmetros necessários, árvore oficial, scripts, contratos/testes e plano executável.

### Ambiente corporativo

Confirmar Windows/WebView2, paths/SMB/permissões, hostname/porta, HTTP/HTTPS, antivírus/EDR/firewall e mecanismo real de start do Controller.
