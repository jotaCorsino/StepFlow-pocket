# Changelog do Projeto — StepFlow Pocket

## 2026-08-20

### Arquitetura da Fase 1

- Tauri 2 validado como direção do Client Windows;
- baseline inicial Windows 10/11 x64 e WebView2;
- launcher transitório + Client local versionado em `%LOCALAPPDATA%` consolidado;
- Host consolidado em Rust + Tokio/Axum + SQLite bundled;
- requisito Pocket reforçado: nenhum serviço/processo StepFlow residente quando o produto está fechado;
- Controller central sob demanda consolidado;
- HTTP/JSON + WebSocket e `deployment.json` consolidados;
- Argon2id, sessão opaca e autorização Host-side consolidados;
- modelo de dados com revisões imutáveis e migrations consolidado;
- concorrência com WAL, writer coordenado, fila bounded e revisão otimista consolidada;
- Blocos 0–7 da Fase 1 encerrados em nível arquitetural;
- Bloco 8 — UI/UX definido como próximo trabalho.

### Provas técnicas úteis

- PoC Tauri comprovou build/execução isolada sem toolchain de desenvolvimento no runtime;
- PoC Host comprovou HTTP + SQLite bundled + execução isolada;
- artefatos/PoCs locais descartáveis foram removidos após os testes.

### Limpeza documental

- removidos da árvore ativa troubleshooting de sandbox, tarefas Codex concluídas, gates descartáveis e revisões granulares já consumidas;
- consolidados documentos duplicados do Host em `docs/03-arquitetura/host-pocket.md`;
- `arquitetura-inicial.md` substituída por `arquitetura-vigente.md`;
- README, índice documental, produto, planejamento, roadmap e decisões atualizados ao estado atual;
- histórico removido continua recuperável pelo Git;
- diário de progresso preservado sem alteração para não conflitar com edição local existente;
- varredura final dos Markdown ativos concluída, removendo referências operacionais obsoletas e alinhando os critérios da Fase 1.

## 2026-08-19

### Fundação documental e governança

- repositório e estrutura documental inicializados;
- `AGENTS.md`, método PO + Assistente + Codex e política de capacidade criados;
- visão de produto, arquitetura inicial, roadmap, planejamento, decisões, diário e templates criados;
- Fase 0 revisada e encerrada;
- Fase 1 aberta;
- multiusuário, modelo enxuto de processo, metáfora de livro, perfis e requisitos de exportação consolidados.

### Código

Nenhuma implementação funcional oficial foi criada.
