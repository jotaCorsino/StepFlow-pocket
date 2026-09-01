# Changelog do Projeto — StepFlow Pocket

Este arquivo registra **marcos relevantes**, não cada commit ou conversa. O histórico operacional detalhado permanece no Git/PRs; decisões vigentes ficam em `registro-de-decisoes.md`.

## 2026-09-01

### Fase 1 concluída — Bloco 12

- Fase 1 encerrada documental e tecnicamente com **D12.1–D12.108**;
- source tree oficial planejada em `apps/` + `crates/`, sem abstrações vazias por antecipação;
- publicação Pocket consolidada com **`StepFlow.exe` na raiz** como único ponto de entrada normal e `_internal/` para a árvore técnica; esse contrato substitui o nome externo anterior do Launcher nas fontes vigentes;
- Rust 1.98.0, Edition 2024, resolver 3 e target Windows x64 MSVC definidos para a fundação executável;
- Client vanilla modular preservado sem Node/npm/Vite/bundler/framework no baseline;
- migrations Host-side imutáveis/embutidas, runner com checksum, testes em SQLite temporário real e fixtures sintéticas definidos;
- parâmetros finais de autenticação/sessão, Empresa/Categorias, Backup/Restore, comunicação e logs fechados;
- plano executável da Fase 2 dividido em F2-T01…F2-T08, cada tarefa com branch/PR próprios e pré-flight separado;
- `deployment.json` real exige input explícito de implantação e packaging implantável falha sem configuração obrigatória;
- sincronização local futura definida por fast-forward seguro, sem reset/clean/stash/rebase corretivo;
- gates corporativos permanecem reservados às etapas executáveis correspondentes e nunca recebem PASS presumido fora do ambiente aplicável;
- nenhum scaffold/runtime oficial, migration SQL ou código de negócio foi criado durante a Fase 1;
- transição para F2-T01 permanece condicionada a gate Git limpo, sincronização local segura e autorização explícita do PO.

### Bloco 11 — Backup / Restauração técnico

- contrato técnico consolidado em **D11.1–D11.116**;
- estado recuperável definido como `stepflow.sqlite + company/** + avatars/**`;
- pacote único `.stepflow-backup`, ZIP `Stored`, manifesto versionado, hashes SHA-256 e Online Backup API;
- consistência conjunta de SQLite + arquivos administrados com barrier de mutações;
- catálogo reconstruível, retenção sem scheduler/por quantidade e coordinator administrativo exclusivo;
- Restore com revalidação integral, migrations somente forward, safety backup obrigatório e troca lógica de `data/`;
- safety barrier `pre_restore` mantido desde a captura até o primeiro rename;
- journal fora de `data/`, fresh Host, rollback conhecido/`uncertain` e invalidação de sessões após fase destrutiva;
- disaster recovery local/transitório pelo Controller, sem listener normal da LAN;
- Backup permitido a ADM/Gerência e Restore restrito a ADM;
- auditoria administrativa estruturada fora de `data/`;
- paths Windows endurecidos e provenance por `source_deployment_id`;
- parser/extração bounded;
- baseline inicial sem criptografia ou assinatura application-level; SHA-256 permanece integridade, não autenticidade;
- gates Win32/filesystem/ACL/EDR/long paths/performance/crash permanecem antes de produção;
- validação técnica final concluiu sem bloqueador arquitetural conhecido.

## 2026-08-29

### Bloco 10 — geração documental, impressão e Ficha compacta

- Etapas 1–11 consolidadas e aprovadas;
- geração documental Host-side por snapshot consistente + `DocumentModel`;
- PDF de Procedimentos via Typst embutido;
- DOCX OOXML direto em Rust sob adaptador;
- impressão Windows pelo mesmo PDF oficial via WebView2 + `ShowPrintUI(System)`;
- Procedimento físico A4 multipágina;
- Ficha compacta PDF + preview do mesmo `PagedDocument`, exatamente uma A4;
- `SHEET_OVERFLOW` sem truncamento, segunda página ou compactação automática;
- soft limits orientativos 600/400/300/280;
- naming persistente e política de temporários consolidados;
- validação técnica final sem bloqueador arquitetural conhecido;
- Word, impressoras, SMB, Windows/WebView2 e EDR mantidos como gates de ambiente real.

### Contrato Pocket reforçado

- pasta pronta publicada no servidor Windows;
- entrada do usuário então documentada pelo Launcher no compartilhamento;
- Client preparado automaticamente em `%LOCALAPPDATA%`;
- zero instalador tradicional, preparação manual ou elevação por estação no uso normal;
- Client operacional não roda permanentemente do SMB;
- WebView2 Fixed Version não é executado de UNC/SMB;
- fallback WebView2 autocontido só entra após PoC provar preparação local sem instalação/admin/manualidade.

### Higiene documental pós-Bloco 10

- ownership documental reforçado;
- referências históricas consumidas removidas das fontes técnicas ativas;
- histórico detalhado mantido no Git em vez de repetido em documentos estáveis.

## 2026-08-25 a 2026-08-28

### Blocos 8 e 9 — UI/UX e operação

- Telas 01–15 consolidadas;
- Reader definido como manual/livro, com `Visão geral` e uma Etapa por página lógica;
- stepper definido como navegação, não conclusão operacional;
- domínio `Procedimento × Atendimento/Execução × Equipamento` consolidado;
- lifecycle `Em andamento / Concluído / Cancelado` com reabertura explícita;
- checklist persistente somente em Atendimento;
- `Observação do serviço` opcional por Etapa;
- códigos `AT-000001` e `EQP-000001`;
- permissões operacionais e reprodução histórica consolidadas;
- Ficha definida como prestação de contas resumida ao cliente.

## 2026-08-21

### Expansão do domínio

- categorias de Procedimentos incorporadas;
- Atendimentos e Equipamentos separados do Procedimento oficial;
- busca documental e operacional separadas;
- manutenção de computadores/notebooks passou a suportar dados específicos de Equipamento;
- requisito de Ficha compacta incorporado ao produto.

### Hardening de governança

- proteção explícita do working tree do PO;
- branch/base/SHA obrigatórios em tarefas Codex que alterem arquivos;
- distinção entre sessão Windows normal e sandbox Codex;
- Fase 1 mantida documental, sem scaffold/runtime de produção prematuro.

## 2026-08-20

### Arquitetura da Fase 1

- Tauri 2 + HTML/CSS/JavaScript modular para o Client;
- Windows 10/11 x64 + WebView2 como baseline;
- Launcher transitório + Client local versionado;
- Host Rust + Tokio/Axum + SQLite bundled;
- Controller/Host sob demanda;
- HTTP/JSON + WebSocket;
- Argon2id, sessão opaca e autorização Host-side no núcleo;
- WAL, writer coordenado, fila bounded e revisão otimista;
- PoCs descartáveis de Client e Host validaram execução isolada sem toolchain no runtime.

## 2026-08-19

### Fundação documental e governança

- repositório e estrutura documental inicializados;
- `AGENTS.md`, método PO + Assistente + Codex e política de capacidade criados;
- visão de produto, arquitetura, roadmap, planejamento, decisões e templates criados;
- Fase 0 revisada e encerrada;
- Fase 1 aberta;
- modularidade, multiusuário, modelo Pocket e metáfora de livro estabelecidos.

### Código

Nenhuma implementação funcional oficial foi criada durante os marcos da Fase 1.