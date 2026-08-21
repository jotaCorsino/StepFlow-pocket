# Arquitetura Vigente — StepFlow Pocket

**Status:** CONSOLIDADA PARA A FASE 1  
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

Baseline inicial: Windows 10/11 x64, com WebView2. Validação das máquinas corporativas permanece pendente.

## Domínios funcionais vigentes

O núcleo funcional passa a distinguir explicitamente:

- **Procedimentos** — documentação/modelos oficiais versionados;
- **Categorias** — classificação configurável e múltipla de procedimentos;
- **Atendimentos/Execuções** — ocorrências reais de trabalho quando rastreabilidade for necessária;
- **Equipamentos** — ativos físicos opcionais relacionados aos atendimentos;
- **Usuários/Permissões** — identidade e autorização;
- **Empresa/Exportação/Backup** — identidade e operação administrativa.

Procedimento e atendimento são entidades diferentes. Um atendimento preserva a revisão do procedimento efetivamente usada e pode ou não possuir equipamento associado.

## Launcher do Client

O ponto de entrada da rede é um launcher portátil/transitório em Rust.

Fluxo:

1. ler manifesto/configuração publicados;
2. comparar versão local;
3. copiar artefatos para `%LOCALAPPDATA%\StepFlow\Client\versions\<versao>\`;
4. validar SHA-256;
5. ativar/iniciar a cópia local;
6. encerrar o launcher.

Versões ficam lado a lado para atualização/rollback. Nenhum serviço/updater residente é usado.

## Host Pocket

Tecnologia: **Rust + Tokio/Axum + `rusqlite` com SQLite bundled**.

Na máquina central existem dois papéis executáveis:

- Controller: inicia sob demanda, valida paths/configuração, protege instância única, inicia/acompanha Host e coordena shutdown;
- Host: autenticação, autorização, API, eventos, SQLite, writer/fila, revisões, categorias, equipamentos, atendimentos, auditoria, backup/restore e arquivos administrados.

Não há Windows Service, auto-start, Task Scheduler ou daemon StepFlow como padrão. Encerrado o ciclo operacional, nenhum processo StepFlow deve permanecer ativo.

Detalhes: `host-pocket.md` e `implantacao-pocket.md`.

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
- foreign keys habilitadas;
- WAL para concorrência leitura/escrita;
- migrations versionadas;
- revisões de procedimento imutáveis;
- categorias configuráveis;
- equipamentos com identidade interna estável;
- atendimentos formais com equipamento opcional e vínculo à revisão utilizada;
- versão exibida separada da revisão técnica;
- auditoria separada de logs técnicos;
- dados/configuração não são substituídos junto com binários.

Detalhes: `modelo-dados-schema-fase-1.md` e `../01-produto/categorizacao-atendimentos-equipamentos.md`.

## Comunicação

- HTTP + JSON em contratos versionados, inicialmente `/api/v1`;
- WebSocket autenticado para eventos;
- `deployment.json` informa endpoint/contrato sem guardar segredo;
- handshake de compatibilidade antes do login;
- nenhuma edição offline na primeira versão;
- falha de WebSocket provoca reconexão e reconsulta, não replay obrigatório.

## Autenticação e autorização

- Argon2id para senhas;
- sessão opaca server-side;
- token mantido apenas em memória do Client inicialmente;
- autorização sempre no Host;
- ADM, Gerência e Funcionário são presets;
- Gerência não administra ADM;
- bootstrap do primeiro ADM ocorre localmente/controlado na máquina central;
- matriz específica de categorias/equipamentos/atendimentos ainda será fechada no Bloco 9.

## Concorrência

- um writer lógico coordenado no Host;
- fila bounded com backpressure;
- revisão otimista por recurso quando houver risco de perda concorrente;
- `409 Conflict` para base obsoleta;
- constraints SQLite como última defesa;
- eventos somente após commit;
- nenhum soft/hard lock de edição na primeira fundação;
- proteção contra dois Hosts usando o mesmo data dir;
- categorias, equipamentos e atendimentos seguem o mesmo princípio de coordenação/revisão quando houver mutações concorrentes.

## Exportação e impressão

PDF, DOCX e impressão são requisitos para documentação de procedimentos.

O novo requisito também exige ficha/relatório compacto de atendimento/equipamento para impressão física. Estratégia, tamanho/formato e eventual arquivo exportável serão fechados no Bloco 10.

## Backup

Backup/restore é coordenado pelo Host e será especificado no Bloco 11. Deve abranger os novos dados de categorias, equipamentos e atendimentos além do restante do banco/arquivos administrados.

## Ambiente corporativo ainda pendente

Somente no ambiente real serão consolidados/testados:

- hostname/IP e paths reais;
- compartilhamento SMB e permissões;
- versões reais de Windows/arquitetura;
- WebView2 nas estações;
- política de transporte HTTP/HTTPS;
- antivírus/EDR/firewall;
- mecanismo existente para iniciar o Controller na máquina central quando necessário.

Essas pendências não autorizam hardcode de exemplos.

## Próximo trabalho

Bloco 8 da Fase 1 continua em UI/UX, agora incorporando categorias e as superfícies de atendimento/equipamento. Lifecycle/checklist operacional será fechado no Bloco 9 antes da implementação.
