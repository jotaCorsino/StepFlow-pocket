# Registro de Decisões — StepFlow Pocket

**Atualização:** 2026-08-25

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

### Categorização e domínio operacional — consolidado

Ficam aprovados:

- sistema de categorias configuráveis para procedimentos;
- um procedimento pode possuir múltiplas categorias simples;
- sem taxonomia hierárquica complexa na primeira versão;
- separação entre `Procedimento`, `Atendimento/Execução` e `Equipamento`;
- `Atendimentos` como área operacional própria no Client;
- equipamento é opcional e reutilizável entre atendimentos;
- equipamento possui identidade interna estável/código StepFlow próprio;
- MAC, serial, patrimônio, cliente e OS/referência são atributos de busca, não identidade canônica exclusiva;
- um atendimento pode utilizar múltiplos procedimentos;
- atendimento preserva vínculo com a revisão do procedimento realmente utilizada;
- para computadores, tipo suporta pelo menos `Servidor`, `Desktop` e `Notebook`;
- `Saúde da bateria` é contextual para Notebook;
- observações do equipamento são curtas e limitadas;
- atendimento pode ser relacionado a cliente/solicitante e ordem de serviço/referência externa;
- deve existir resumo do trabalho/procedimentos realizados;
- deve existir ficha compacta imprimível para anexação/acompanhamento físico do equipamento;
- ficha compacta deve ocupar no máximo uma página A4;
- cabeçalho da ficha suporta logo, nome da empresa, contato, site e e-mail;
- a nova capacidade não transforma o produto em CRM, estoque, RMM, financeiro ou sistema completo de chamados.

Fonte: `docs/01-produto/categorizacao-atendimentos-equipamentos.md`.

Pendentes:

- lifecycle/status exatos do atendimento;
- permissões operacionais;
- formato dos códigos legíveis;
- comportamento do checklist/progresso;
- regras de edição/reabertura após conclusão;
- template/layout físico final e tecnologia da ficha compacta;
- necessidade de PDF específico da ficha além da impressão direta.

### Procedimento documental

Campos principais continuam:

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

### Lista/Pesquisa de Processos — consolidado

- visualização padrão em lista/tabela compacta;
- busca por Código, Título ou termo documental;
- filtros principais por Categoria e Área;
- filtro Status somente para perfis que realmente precisem trabalhar com estados diferentes;
- categorias permitem seleção múltipla simples com semântica **OU** inicialmente;
- categorias aparecem como labels/chips discretos;
- ação principal abre o leitor, não o editor;
- ações administrativas ficam contextuais;
- `Arquivar` é a operação normal em vez de `Excluir`;
- retorno do leitor preserva busca/filtros;
- busca documental de `Processos` permanece separada da busca operacional de `Atendimentos`.

### Perfis

- ADM;
- Gerência;
- Funcionário.

Gerência não administra ADM. Funcionário é leitura/execução por padrão. Usuário pode editar avatar, nome de exibição, cargo e senha dentro das regras.

A autorização real é por capacidades Host-side; presets não substituem verificação granular.

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
- durante o fechamento documental restante da Fase 1, o remoto é a fonte operacional;
- sincronização do checkout local fica adiada até antes do primeiro trabalho de implementação com Codex;
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
- fechar Client individual não encerra Host;
- encerrado o ciclo central, nenhum processo StepFlow permanece ativo;
- dados/config/logs/backups separados dos binários;
- auto-shutdown por último Client/timeout não está consolidado.

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
- desativação preferida à exclusão quando houver histórico;
- Gerência nunca administra ADM;
- pelo menos um ADM ativo deve existir;
- após troca da própria senha, sessão corrente permanece e demais sessões da conta são revogadas.

Pendentes:

- custo exato Argon2id;
- senha mínima final;
- expiração de sessão;
- permissão da Gerência para configuração da empresa;
- permissão da Gerência para Backup;
- permissões operacionais de categorização/equipamentos/atendimentos.

## Dados e histórico

Consolidado:

- SQLite local ao Host;
- revisões de procedimento imutáveis;
- `revision_no` separado de `display_version`;
- migrations versionadas;
- auditoria append-only;
- arquivamento/desativação preferidos à exclusão destrutiva;
- categorias, equipamentos e atendimentos fazem parte da extensão conceitual aprovada do schema;
- atendimento preserva a revisão de procedimento utilizada;
- MAC não é chave canônica do equipamento;
- identidade da empresa é central e administrada pelo Host;
- logo/avatar são arquivos controlados pelo Host, não caminhos arbitrários persistidos pelo Client.

## Concorrência

- WAL;
- writer lógico coordenado;
- fila bounded/backpressure;
- revisão otimista quando houver risco de perda;
- conflitos não sobrescrevem automaticamente;
- eventos pós-commit;
- sem soft/hard lock inicial;
- dois Hosts não usam o mesmo data dir;
- categorias/equipamentos/atendimentos/configuração da empresa seguem o mesmo princípio de controle otimista quando necessário.

## Bloco 8 — UI/UX

### Telas 01–12 — consolidadas

- Login;
- Shell/sidebar, incluindo `Atendimentos`;
- Dashboard enxuto, sem KPIs/gráficos;
- Lista/Pesquisa de Processos;
- Leitor em formato livro;
- Editor de Processo + categorias;
- Histórico/Revisões;
- Lista/Pesquisa de Atendimentos;
- Atendimento/Execução + Equipamento;
- Usuários/Permissões;
- Meu perfil;
- Configurações + Categorias.

### Tela 12 — Configurações + Categorias — consolidada

- uma única superfície com navegação local `Empresa` + `Categorias`;
- seções aparecem conforme capacidades efetivas;
- autorização da Gerência para configuração da empresa permanece `PENDENTE`;
- identidade da empresa contém inicialmente logo, nome, contato, site e e-mail;
- logo pode ser escolhido, substituído/removido, com preview antes do save;
- identidade confirmada alimenta Shell e documentos/ficha sem duplicação manual;
- salvamento explícito, sem autosave;
- categorias são lista compacta com busca por nome e filtro por estado;
- categoria possui inicialmente nome + estado, sem hierarquia/cor/ícone;
- criação/edição acontece fora do Editor de Processo;
- arquivar/reativar substitui exclusão física normal;
- categoria arquivada deixa de ser opção normal para novas associações, preservando histórico;
- categorias duplicadas/visualmente equivalentes após normalização devem ser impedidas;
- regra de nova revisão de procedimento ainda referenciando categoria arquivada permanece pendente;
- Backup/Restore não foi antecipado;
- Exportação/Impressão e template A4 não foram antecipados.

Próxima superfície do Bloco 8: **Tela 13 — Backup/Restauração — UX**.

Nenhuma UI de produção foi criada.

## Pendências vigentes

### Bloco 8

- Tela 13 — Backup/Restauração — UX;
- Tela 14 — Exportação/Impressão + ficha compacta — UX;
- Tela 15 — estados transversais.

### Bloco 9

Fechar lifecycle, checklist/progresso, conclusão/reabertura, concorrência/histórico específicos, matriz operacional de permissões, formatos de códigos legíveis e regras operacionais ainda pendentes.

### Bloco 10

Fechar tecnologia/formato da ficha compacta, limite físico A4, limites textuais relacionados, além de PDF/DOCX/impressão dos procedimentos.

### Bloco 11

Backup/restore incluindo SQLite e arquivos administrados: categorias, equipamentos, atendimentos, identidade/logo da empresa, avatares e demais dados persistentes aplicáveis.

### Bloco 12

Parâmetros finais, árvore oficial, scripts, contratos/testes e plano da Fase 2.

### Ambiente corporativo

Confirmar Windows/WebView2, SMB/permissões, hostname/porta, HTTP/HTTPS, antivírus/EDR/firewall e mecanismo real de start do Controller.