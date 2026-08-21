# Changelog do Projeto — StepFlow Pocket

## 2026-08-21

### Expansão de produto — categorização e registro de serviço/equipamento

Requisitos confirmados pelo PO:

- StepFlow passa a cobrir manutenção, TI, Service Desk, Help Desk, infraestrutura/servidores, redes, guias e outros procedimentos internos;
- categorização de procedimentos incorporada como requisito;
- manutenção de computadores/notebooks precisa permitir registrar dados específicos como nome, processador, RAM, armazenamento, SO/versão, MAC/identificador útil, saúde de bateria e observações;
- registro deve poder usar cliente e/ou OS/referência para facilitar busca;
- precisa haver resumo do que foi feito/procedimentos realizados;
- ficha/relatório compacto deve poder ser gerado para impressão e anexação física ao equipamento.

Direção de modelagem criada como **PROPOSTA**, ainda aguardando aprovação do PO:

- separar Procedimento × Atendimento/Execução × Equipamento;
- categorias configuráveis, simples e potencialmente múltiplas;
- identidade interna/código StepFlow para equipamentos, usando MAC/serial/patrimônio como busca;
- equipamento reutilizável entre serviços;
- múltiplos procedimentos por atendimento;
- vínculo do atendimento à revisão exata utilizada;
- item `Atendimentos` próprio na sidebar.

Documentação atualizada:

- criado `docs/01-produto/categorizacao-atendimentos-equipamentos.md` separando requisitos confirmados e propostas;
- modelo de dados ganhou extensão **proposta**, sem migration/código;
- arquitetura vigente registra o requisito e mantém a modelagem como pendente;
- Blocos 8, 9 e 10 foram ampliados no planejamento para acomodar o novo requisito;
- Shell reaberto apenas na decisão de navegação operacional;
- roadmap, índice, decisões e AGENTS sincronizados;
- nenhum código de produção criado.

### Hardening documental para execução Codex

- simulada leitura do repositório por um Codex sem memória de conversa;
- adicionada proteção explícita contra `reset --hard`, `git clean`, `stash`, descarte ou incorporação de alterações preexistentes;
- toda tarefa Codex que altere arquivos passa a exigir branch/base e commit SHA esperado;
- prompt autoriza escopo, mas não revoga silenciosamente decisão consolidada;
- leitura do Codex reduzida a camadas;
- restaurada distinção sessão Windows normal do PO × sandbox Codex;
- “trabalho estrutural” na Fase 1 não autoriza scaffold/runtime oficial antes do Bloco 12/Fase 2;
- Host esclarecido: Client individual não encerra Host; ciclo central pertence ao Controller; auto-shutdown por último Client/timeout não consolidado;
- autenticação separada entre núcleo consolidado e parâmetros pendentes;
- Bloco 8 inclui Dashboard e limita Backup/Exportação à UX até Blocos 10/11;
- precedência documental alinhada.

## 2026-08-20

### Arquitetura da Fase 1

- Tauri 2 validado como direção do Client Windows;
- baseline Windows 10/11 x64 e WebView2;
- launcher transitório + Client local versionado em `%LOCALAPPDATA%`;
- Host Rust + Tokio/Axum + SQLite bundled;
- requisito Pocket reforçado;
- Controller central sob demanda;
- HTTP/JSON + WebSocket e `deployment.json`;
- Argon2id, sessão opaca e autorização Host-side no núcleo;
- modelo de dados com revisões imutáveis/migrations;
- concorrência com WAL, writer coordenado, fila bounded e revisão otimista;
- Blocos 0–7 encerrados no núcleo arquitetural;
- Bloco 8 definido como próximo trabalho.

### Provas técnicas úteis

- PoC Tauri comprovou build/execução isolada sem toolchain no runtime;
- PoC Host comprovou HTTP + SQLite bundled + execução isolada;
- PoCs descartáveis removidas após os testes.

### Limpeza documental

- removidos troubleshooting, tarefas concluídas, gates descartáveis e revisões granulares consumidas;
- documentos do Host consolidados;
- `arquitetura-inicial.md` substituída por `arquitetura-vigente.md`;
- índices/produto/planejamento/roadmap/decisões atualizados;
- histórico removido permanece no Git;
- diário preservado sem alteração por existir edição local;
- varredura final dos Markdown ativos concluída.

## 2026-08-19

### Fundação documental e governança

- repositório e estrutura documental inicializados;
- `AGENTS.md`, método PO + Assistente + Codex e política de capacidade criados;
- visão de produto, arquitetura, roadmap, planejamento, decisões, diário e templates criados;
- Fase 0 revisada/encerrada;
- Fase 1 aberta;
- multiusuário, modelo enxuto, metáfora de livro, perfis e exportação consolidados.

### Código

Nenhuma implementação funcional oficial foi criada.
