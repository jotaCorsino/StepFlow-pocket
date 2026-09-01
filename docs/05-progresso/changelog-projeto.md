# Changelog do Projeto — StepFlow Pocket

Este arquivo registra **marcos relevantes**, não cada commit ou conversa. O histórico operacional detalhado permanece no Git/PRs; decisões vigentes ficam em `registro-de-decisoes.md`.

## 2026-09-01

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
- paths Windows endurecidos contra traversal/UNC/device/ADS/reserved names/case collision/reparse/non-regular;
- manifesto passou a incluir `source_deployment_id` para provenance da implantação;
- parser/extração definidos como bounded, com valores numéricos reservados ao Bloco 12;
- baseline inicial sem criptografia ou assinatura application-level; SHA-256 permanece integridade, não autenticidade;
- offsite/cópia corporativa de backups permanece responsabilidade operacional externa ao baseline;
- adapter Win32, filesystem, ACLs, EDR/antivírus, long paths, espaço, performance e crash injection permanecem gates antes de produção;
- validação técnica final concluiu **sem bloqueador arquitetural conhecido**.

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
- Word, impressoras, SMB, Windows/WebView2 e EDR mantidos como gates de ambiente real;
- limites de performance ficaram para medição na fase executável.

### Contrato Pocket reforçado

- pasta pronta publicada no servidor Windows;
- entrada do usuário pelo `StepFlowLauncher.exe` no compartilhamento;
- Client preparado automaticamente em `%LOCALAPPDATA%`;
- zero instalador tradicional, preparação manual ou elevação por estação no uso normal;
- Client operacional não roda permanentemente do SMB;
- WebView2 Fixed Version não é executado de UNC/SMB;
- fallback WebView2 autocontido só entra após PoC provar preparação local sem instalação/admin/manualidade.

### Higiene documental pós-Bloco 10

- iniciado checkpoint para remover estados de transição consumidos e duplicações;
- documentos de entrada passam a ter ownership mais estrito;
- referências históricas a blocos já encerrados deixam de permanecer como pendências ativas;
- histórico detalhado permanece no Git em vez de ser repetido em documentos técnicos.

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

Nenhuma implementação funcional oficial foi criada durante esses marcos da Fase 1.
