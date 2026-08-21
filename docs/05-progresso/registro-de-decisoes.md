# Registro de Decisões — StepFlow Pocket

**Atualização:** 2026-08-21

Este arquivo registra decisões vigentes e pendências atuais. Propostas não aprovadas ficam explicitamente identificadas como **PROPOSTA**, nunca como decisão.

## Produto e UX

### Nome e propósito — consolidado

- produto: **StepFlow**;
- aplicação interna para documentação e execução guiada de procedimentos técnicos;
- uso não restrito à manutenção de computadores;
- deve acomodar manutenção, TI, Service Desk, Help Desk, infraestrutura/servidores, redes, guias e outros procedimentos internos;
- evitar burocracia e campos sem valor operacional;
- etapas funcionam como páginas de manual/livro;
- sidebar esquerda, logo pequeno no topo esquerdo;
- blocos copiáveis usam ícone discreto;
- PDF, DOCX e impressão são requisitos da documentação.

### Novo requisito de 2026-08-21 — consolidado

O PO confirmou:

- sistema de categorização de procedimentos;
- área para registrar dados específicos de computadores/notebooks quando o serviço exigir;
- campos como nome, processador, RAM, armazenamento, versão do sistema, MAC/identificador útil, saúde da bateria quando aplicável e observações;
- possibilidade de relacionar o registro a cliente e/ou ordem de serviço/referência útil para busca;
- busca facilitada pelos identificadores operacionais disponíveis;
- resumo do que foi feito/procedimentos realizados;
- ficha/relatório compacto extraível/imprimível para anexação física ao equipamento;
- essas funções não podem tornar o produto exclusivo de manutenção de PCs.

### Modelagem da nova funcionalidade — PROPOSTA

Ainda aguardam aprovação explícita do PO:

- separar `Procedimento`, `Atendimento/Execução` e `Equipamento` como entidades distintas;
- categorias configuráveis, simples e possivelmente múltiplas;
- equipamento reutilizável entre atendimentos;
- identificador interno/código StepFlow como identidade principal e MAC/serial/patrimônio como busca;
- um atendimento poder usar mais de um procedimento;
- atendimento preservar a revisão exata do procedimento utilizado;
- `Atendimentos` como nome/item próprio na sidebar.

Fonte da proposta: `docs/01-produto/categorizacao-atendimentos-equipamentos.md`.

### Procedimento documental

Campos principais atualmente consolidados continuam:

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

`Categorias` é requisito novo confirmado, mas sua cardinalidade/modelagem final ainda está em aprovação.

### Perfis

- ADM;
- Gerência;
- Funcionário.

Gerência não administra ADM. Funcionário é leitura/execução por padrão. Usuário pode editar avatar, nome de exibição, cargo e senha dentro das regras.

A matriz exata para categorias/serviços/equipamentos permanece pendente.

## Governança

- GitHub é a fonte principal de verdade;
- método PO + Assistente + Codex;
- uma tarefa por vez;
- toda tarefa Codex exige pré-flight separado do prompt;
- `AGENTS.md` é regra operacional superior;
- prompt autoriza escopo, mas não revoga decisão consolidada;
- mudança de decisão exige aprovação explícita do PO + atualização documental;
- tarefa Codex que altera arquivos declara branch/base + SHA;
- alteração preexistente no working tree pertence ao PO/outro fluxo;
- base divergente ou arquivo necessário já modificado faz Codex parar/reportar;
- Fase 1 em andamento; Bloco 8 em execução.

## Ambientes

- desenvolvimento atual fora da LAN corporativa;
- sessão normal do PO e sandbox Codex são contextos distintos;
- limitação do sandbox não vira requisito;
- Codex não altera ACL/Schannel/registro/PATH global/segurança para reparar ambiente;
- operações que exigem credenciais/elevação/Internet confiável/configuração global são reportadas para sessão normal do PO;
- infraestrutura real da empresa ainda não confirmada;
- exemplos históricos não são configuração;
- testes LAN/SMB fora do ambiente real não validam nem bloqueiam implantação.

## Pocket / máquina central

- implantação por pasta pronta;
- nenhuma toolchain/runtime de desenvolvimento exigida na máquina central;
- nenhum Windows Service StepFlow persistente;
- nenhum auto-start, Task Scheduler, watchdog, tray agent ou daemon como padrão;
- Controller/Host sob demanda;
- encerrado o ciclo central, nenhum processo StepFlow permanece ativo;
- dados/config/logs/backups separados dos binários.

## Client Windows

- Tauri 2 + HTML/CSS/JavaScript modular;
- alvo inicial Windows 10/11 x64;
- WebView2;
- executável isolado sem toolchain no runtime;
- Electron apenas contingência.

## Launcher

- launcher Rust x64 portátil/transitório;
- ponto de entrada interno → Client versionado em `%LOCALAPPDATA%` → execução local → launcher encerra;
- versões lado a lado + SHA-256;
- sem updater residente;
- launcher não inicia remotamente Host central.

## Host

- Rust + Tokio/Axum + `rusqlite` bundled;
- Controller inicia Host como filho;
- readiness, instância única e shutdown gracioso;
- ciclo central pertence ao Controller;
- fechar Client individual não encerra Host;
- auto-shutdown por último Client/timeout não consolidado;
- primeiro start central depende de ação local ou mecanismo corporativo existente/aprovado.

## Comunicação

- HTTP/JSON + WebSocket;
- contratos inicialmente `/api/v1`;
- `deployment.json` sem segredos;
- handshake de compatibilidade antes do login;
- primeira versão sem edição offline.

## Autenticação/autorização

Consolidado:

- Argon2id;
- sessão opaca server-side;
- token somente em memória do Client inicialmente;
- autorização Host-side;
- bootstrap ADM local/controlado;
- desativação preferida à exclusão quando houver histórico.

Pendentes:

- custo exato Argon2id;
- senha mínima final;
- expiração de sessão;
- permissão da Gerência para configuração/backup;
- permissões de categorização/registro operacional/equipamento.

## Dados e histórico

Consolidado:

- SQLite local ao Host;
- revisões de procedimento imutáveis;
- `revision_no` separado de `display_version`;
- migrations versionadas;
- auditoria append-only;
- arquivamento/desativação preferidos à exclusão destrutiva.

Nova extensão de schema para categorias/atendimentos/equipamentos está **PROPOSTA**, não implementável como contrato até aprovação da modelagem.

A antiga frase “nenhuma entidade formal de execução antes do Bloco 9” deixa de poder ser usada para negar o novo requisito de registro de serviço; porém o formato final dessa entidade ainda depende da aprovação da modelagem e do Bloco 9.

## Concorrência

- WAL;
- writer lógico coordenado;
- fila bounded/backpressure;
- revisão otimista quando houver risco de perda;
- conflitos não sobrescrevem automaticamente;
- eventos pós-commit;
- sem soft/hard lock inicial;
- dois Hosts não usam o mesmo data dir.

Novos dados operacionais, se aprovados na modelagem proposta, seguirão os mesmos princípios.

## Bloco 8 — UI/UX

- Login consolidado;
- núcleo do Shell consolidado;
- Shell reaberto somente para aprovar/rejeitar `Atendimentos` como nova entrada;
- Dashboard em análise;
- mapa de telas ampliado pelos requisitos novos;
- nenhuma UI de produção criada.

## Pendências vigentes

### Aprovação imediata da modelagem nova

1. `Procedimento × Atendimento/Execução × Equipamento` separados;
2. categorias simples/configuráveis e potencialmente múltiplas;
3. identificador interno/código StepFlow para equipamento;
4. `Atendimentos` como termo/área operacional;
5. múltiplos procedimentos por atendimento;
6. vínculo histórico à revisão utilizada;
7. necessidade de PDF da ficha além da impressão direta.

### Bloco 9

Fechar lifecycle, checklist/progresso, conclusão/reabertura, concorrência/histórico específicos e matriz operacional de permissões.

### Bloco 10

Fechar tecnologia/formato da ficha compacta, além de PDF/DOCX/impressão dos procedimentos.

### Bloco 11

Backup/restore incluindo novos dados aprovados.

### Bloco 12

Parâmetros finais, árvore oficial, scripts, contratos/testes e plano da Fase 2.

### Ambiente corporativo

Confirmar Windows/WebView2, SMB/permissões, hostname/porta, HTTP/HTTPS, antivírus/EDR/firewall e mecanismo real de start do Controller.
