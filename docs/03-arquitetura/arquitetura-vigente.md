# Arquitetura Vigente — StepFlow Pocket

**Status:** CONSOLIDADA PARA A FASE 1, INCLUINDO EXTENSÃO OPERACIONAL CONCEITUAL  
**Atualização:** 2026-08-25

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
- ficha compacta de Atendimento, com ou sem equipamento vinculado;
- identidade da empresa centralizada e reutilizada pelo Shell e documentos;
- UX administrativa de Backup/Restauração coordenada pelo Host;
- exportação contextual de procedimentos baseada na revisão selecionada.

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
- categorias, equipamentos e atendimentos fazem parte da extensão conceitual aprovada do schema;
- logo e identidade da empresa são dados administrados pelo Host e não caminhos arbitrários fornecidos pelo Client.

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
- autorização da Gerência para alterar configuração da empresa permanece pendente;
- autorização da Gerência para Backup permanece pendente;
- Restore permanece não autorizado para Gerência;
- matriz operacional de permissões de Atendimentos/equipamentos/categorias permanece pendente do bloco correspondente;
- capacidade/lifecycle para gerar/reimprimir ficha operacional permanece pendente do Bloco 9.

## Concorrência

- writer lógico coordenado;
- fila bounded/backpressure;
- revisão otimista por recurso quando necessário;
- `409 Conflict` para base obsoleta;
- constraints SQLite como última defesa;
- eventos pós-commit;
- sem soft/hard lock inicial;
- dois Hosts não usam o mesmo data dir;
- categorias/equipamentos/atendimentos e identidade da empresa seguem controle otimista equivalente quando houver risco de perda.

## Exportação e impressão — UX consolidada

A Tela 14 consolidou o contrato funcional, mantendo a implementação técnica para o Bloco 10.

### Procedimentos

- PDF, DOCX e impressão são obrigatórios;
- `Exportar / Imprimir` é contextual no Leitor;
- a primeira versão gera o procedimento completo da revisão selecionada;
- documento é derivado da revisão explicitamente escolhida/autorizada;
- nova revisão não substitui silenciosamente a fonte de uma geração já iniciada;
- revisão histórica/draft é identificada de forma inequívoca;
- exportação/impressão gera documento próprio, nunca screenshot;
- gerar documento não publica, edita nem cria revisão;
- identidade da empresa usa configuração central vigente.

### Ficha de Atendimento

- documento próprio, derivado somente de estado confirmado pelo Host;
- pode existir com ou sem equipamento vinculado;
- quando não houver equipamento, a seção correspondente é omitida;
- campos vazios/não aplicáveis são omitidos;
- procedimentos utilizados preservam versão/revisão efetivamente utilizada;
- `Saúde da bateria` aparece somente quando aplicável/informada;
- limite rígido de no máximo uma página A4;
- conteúdo excepcional que não caiba bloqueia a saída em vez de gerar segunda página ou truncar silenciosamente;
- impressão da ficha é requisito;
- DOCX específico da ficha não é requisito inicial;
- PDF específico da ficha, preview, QR/barcode, margens, limites textuais e engine permanecem para o Bloco 10.

A identidade central da empresa administrada na Tela 12 fornece logo, nome, contato, site e e-mail aos templates.

## Backup / Restore — UX consolidada

A Tela 13 consolidou o contrato visual/funcional, sem fechar o mecanismo técnico do Bloco 11:

- Backup/Restauração fica dentro de `Configurações`;
- Client não copia SQLite nem escolhe paths internos;
- backups conhecidos pelo Host são listados com metadados administrativos;
- criação é coordenada pelo Host e não é cancelada silenciosamente ao fechar um Client;
- Restore só é oferecido para sessão autorizada e backup elegível;
- Restore exige confirmação reforçada;
- Restore normal pela UI exige safety backup do estado atual confirmado antes da etapa destrutiva;
- se o safety backup não puder ser confirmado, o Restore normal não prossegue;
- disaster recovery quando o Host não inicia fica fora da UX normal e pertence ao Bloco 11.

O Bloco 11 ainda fechará pacote, atomicidade, checksums, retenção, compressão/criptografia quando aplicável, coordenação de conexões/restart, sessões e recuperação local de desastre.

## Ambiente corporativo ainda pendente

- hostname/IP/paths reais;
- SMB/permissões;
- Windows/WebView2 reais;
- HTTP/HTTPS;
- antivírus/EDR/firewall;
- mecanismo real de start do Controller.

Essas pendências não autorizam hardcode de exemplos.

## Próximo trabalho

Bloco 8 continua em UI/UX. **Telas 01–14 estão consolidadas; próxima superfície: Tela 15 — Estados transversais.**

Lifecycle/checklist/permissões operacionais serão fechados no Bloco 9; engine/template final de exportação e ficha, no Bloco 10; política técnica de backup/restore, no Bloco 11.
