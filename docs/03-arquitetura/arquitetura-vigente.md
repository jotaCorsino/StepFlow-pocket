# Arquitetura Vigente — StepFlow Pocket

**Status:** CONSOLIDADA PARA A FASE 1 / EXTENSÃO OPERACIONAL EM PROPOSTA  
**Atualização:** 2026-08-21

## Visão geral

```text
Ponto de entrada interno
        ↓
StepFlowLauncher.exe (transitório)
        ↓
Client Tauri local por usuário
        ↓ HTTP/JSON + WebSocket
StepFlow Host Pocket na máquina central
        ↓
SQLite local + arquivos persistentes
```

## Client

Tecnologia: **Tauri 2 + HTML/CSS/JavaScript modular**.

Responsabilidades:

- UI, navegação e UX local;
- manter sessão apenas em memória;
- consumir API do Host;
- receber eventos e reconsultar estado;
- apresentar conflitos/erros;
- nunca abrir SQLite diretamente.

Baseline inicial: Windows 10/11 x64, com WebView2. Validação corporativa permanece pendente.

## Novo requisito funcional confirmado

A arquitetura deve passar a suportar:

- categorização de procedimentos;
- registro das informações reais de serviço/equipamento em cenários aplicáveis;
- busca operacional por cliente/OS/identificadores úteis;
- resumo dos procedimentos realizados;
- ficha compacta imprimível.

### Direção arquitetural proposta

Para manter baixo acoplamento e não misturar documentação com ocorrência real, recomenda-se separar futuramente:

- `Procedimento` — documentação/modelo oficial;
- `Atendimento/Execução` — registro real do serviço;
- `Equipamento` — ativo opcional ligado ao registro.

Essa separação, categorias múltiplas e os vínculos exatos estão **EM PROPOSTA** e não são contrato de implementação até aprovação do PO/Bloco 9.

## Launcher do Client

Launcher portátil/transitório em Rust:

1. lê manifesto/configuração;
2. compara versão;
3. copia artefatos para `%LOCALAPPDATA%\StepFlow\Client\versions\<versao>\`;
4. valida SHA-256;
5. inicia cópia local;
6. encerra.

Nenhum updater residente.

## Host Pocket

Tecnologia: Rust + Tokio/Axum + `rusqlite` bundled.

- Controller: ciclo de vida, paths/config, instância única, startup/readiness/shutdown;
- Host: autenticação, autorização, API, eventos, SQLite, writer/fila, revisões, auditoria, backup/restore e dados funcionais aprovados.

Sem Windows Service, auto-start, Task Scheduler ou daemon StepFlow como padrão. Encerrado o ciclo central, nenhum processo StepFlow permanece ativo.

## Persistência consolidada

```text
StepFlow\
├── app\
├── config\
├── data\
│   ├── stepflow.sqlite
│   ├── company\
│   └── avatars\
├── logs\
└── backups\
```

- SQLite local ao Host;
- foreign keys;
- WAL;
- migrations versionadas;
- revisões de procedimento imutáveis;
- versão exibida separada da revisão técnica;
- auditoria separada de logs;
- dados/config não são substituídos junto com binários.

A extensão de schema para categorias/registros de serviço/equipamentos está em proposta em `modelo-dados-schema-fase-1.md`.

## Comunicação

- HTTP/JSON em contratos versionados, inicialmente `/api/v1`;
- WebSocket autenticado para eventos;
- `deployment.json` sem segredos;
- handshake de compatibilidade antes do login;
- sem edição offline inicial;
- falha WebSocket → reconexão/reconsulta.

## Autenticação e autorização

- Argon2id;
- sessão opaca server-side;
- token em memória do Client;
- autorização Host-side;
- ADM/Gerência/Funcionário como presets;
- Gerência não administra ADM;
- bootstrap ADM local/controlado;
- permissões da nova área operacional ainda pendentes.

## Concorrência

- writer lógico coordenado;
- fila bounded/backpressure;
- revisão otimista por recurso quando necessário;
- `409 Conflict` para base obsoleta;
- constraints SQLite como última defesa;
- eventos pós-commit;
- sem soft/hard lock inicial;
- dois Hosts não usam o mesmo data dir.

Novos dados operacionais aprovados posteriormente devem seguir os mesmos princípios.

## Exportação e impressão

PDF, DOCX e impressão são requisitos da documentação de procedimentos.

Novo requisito confirmado: ficha compacta imprimível de serviço/equipamento. Estratégia, layout físico, PDF e eventuais identificadores visuais serão fechados no Bloco 10.

## Backup

Backup/restore é coordenado pelo Host e será especificado no Bloco 11. Quando a modelagem operacional for aprovada, os novos dados devem entrar no escopo do backup.

## Ambiente corporativo ainda pendente

- hostname/IP/paths reais;
- SMB/permissões;
- Windows/WebView2 reais;
- HTTP/HTTPS;
- antivírus/EDR/firewall;
- mecanismo real de start do Controller.

Essas pendências não autorizam hardcode de exemplos.

## Próximo trabalho

Bloco 8 continua em UI/UX. Antes de transformar a nova funcionalidade em telas/contratos definitivos, o PO deve aprovar ou ajustar a modelagem proposta em `docs/01-produto/categorizacao-atendimentos-equipamentos.md`. Lifecycle/checklist/permissões serão fechados no Bloco 9.
