# Arquitetura Vigente — StepFlow Pocket

**Status:** CONSOLIDADA PARA A FASE 1, INCLUINDO EXTENSÃO OPERACIONAL CONCEITUAL  
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

## Domínio funcional consolidado

A arquitetura deve suportar:

- procedimentos/documentação oficiais;
- categorização configurável e múltipla;
- `Atendimentos` como ocorrências reais de serviço/execução;
- `Equipamento` opcional/reutilizável quando o atendimento envolver ativo físico;
- busca documental separada da busca operacional;
- resumo do trabalho realizado;
- vínculo do atendimento com uma ou mais revisões de procedimentos utilizadas;
- ficha compacta imprimível de atendimento/equipamento.

Separação consolidada:

```text
Procedimento
   ↓ usado em
Atendimento/Execução
   ↓ opcionalmente relacionado a
Equipamento
```

Essa estrutura não transforma o StepFlow em CRM, estoque, RMM ou sistema completo de chamados.

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

## Persistência

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
- dados/config não são substituídos junto com binários;
- categorias, equipamentos e atendimentos fazem parte da extensão conceitual aprovada do schema.

Detalhes: `modelo-dados-schema-fase-1.md`.

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
- matriz operacional de permissões de Atendimentos/equipamentos/categorias permanece pendente do Bloco 9.

## Concorrência

- writer lógico coordenado;
- fila bounded/backpressure;
- revisão otimista por recurso quando necessário;
- `409 Conflict` para base obsoleta;
- constraints SQLite como última defesa;
- eventos pós-commit;
- sem soft/hard lock inicial;
- dois Hosts não usam o mesmo data dir;
- categorias/equipamentos/atendimentos seguem controle otimista equivalente quando houver risco de perda.

## Exportação e impressão

PDF, DOCX e impressão são requisitos da documentação de procedimentos.

A ficha compacta imprimível de atendimento/equipamento também é requisito. Estratégia, layout físico, PDF específico e eventuais identificadores visuais serão fechados no Bloco 10.

## Backup

Backup/restore é coordenado pelo Host e será especificado no Bloco 11, incluindo categorias, equipamentos e atendimentos.

## Ambiente corporativo ainda pendente

- hostname/IP/paths reais;
- SMB/permissões;
- Windows/WebView2 reais;
- HTTP/HTTPS;
- antivírus/EDR/firewall;
- mecanismo real de start do Controller.

Essas pendências não autorizam hardcode de exemplos.

## Próximo trabalho

Bloco 8 continua em UI/UX. Telas 01–04 estão consolidadas; próxima superfície: **Tela 05 — Leitor em formato livro**.

Lifecycle/checklist/permissões operacionais serão fechados no Bloco 9; tecnologia/formato final da ficha, no Bloco 10.
