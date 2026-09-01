# Bloco 12 — Análise 2 — Workspace, build, dependências e configuração

**Status:** APROVADA PELO PO — D12.19–D12.34  
**Bloco:** 12 — Estrutura oficial + plano da Fase 2  
**Data:** 2026-09-01

## Objetivo

Fechar a fundação de build do futuro scaffold oficial sem criar ainda `Cargo.toml`, `rust-toolchain.toml`, binários, migrations ou código executável neste PR.

Esta análise parte da Análise 1 aprovada pelo PO e preserva o contrato Pocket, a separação `apps/ × crates/`, o Client HTML/CSS/JavaScript modular e a publicação com `StepFlow.exe` como único ponto de entrada normal.

## Princípio

A fundação deve ser reproduzível e simples:

- toolchain Rust explícita;
- `Cargo.lock` versionado;
- dependências diretas declaradas centralmente;
- builds oficiais com lockfile obrigatório;
- nenhuma dependência de Node/npm/Vite no baseline do Client;
- nenhum crate entra apenas por conveniência futura;
- versões de dependências são alteradas por mudança deliberada e revisada, nunca incidentalmente durante outra tarefa.

## Toolchain e workspace

Baseline aprovado:

```text
Rust 1.98.0
Edition 2024
Cargo resolver = 3
target oficial = x86_64-pc-windows-msvc
components dev = rustfmt + clippy
```

O repositório terá `rust-toolchain.toml` versionado com canal exato `1.98.0`, perfil `minimal`, componentes `rustfmt` e `clippy` e target Windows x64.

O `Cargo.toml` da raiz será um **virtual workspace**, sem package/binário na raiz. Packages reais entram somente como membros sob `apps/` ou `crates/` quando a etapa correspondente os criar.

`workspace.package` centraliza, quando aplicável:

- `edition = "2024"`;
- `rust-version = "1.98"`;
- versão inicial do produto/workspace;
- licença/metadados internos necessários.

O virtual workspace usa explicitamente `resolver = "3"`.

## Lockfile e política de atualização

`Cargo.lock` será versionado desde o primeiro scaffold oficial.

Build/test/package oficiais usam `--locked` sempre que a resolução de dependências não deve mudar. Falha porque o lockfile precisaria mudar é erro de build, não autorização para executar `cargo update` automaticamente.

Atualização de dependência exige mudança deliberada que:

1. altera o requisito direto quando necessário;
2. atualiza `Cargo.lock` conscientemente;
3. executa testes/builds relevantes;
4. registra incompatibilidade ou migration técnica necessária quando houver.

Não usar dependência por branch Git flutuante, `*`, `latest` ou referência não reprodutível no baseline oficial.

## Dependências compartilhadas do workspace

Dependências diretas que mais de um package realmente usar devem ser declaradas em `[workspace.dependencies]` e herdadas pelos packages. Isso evita versões divergentes sem criar um “Cargo.toml global” com crates ainda não utilizadas.

A regra de pinning aprovada é:

- requisito direto usa uma versão concreta compatível com SemVer;
- `Cargo.lock` registra a resolução exata;
- crates `0.x` não atravessam minor incompatível sem revisão explícita;
- bibliotecas com linha LTS documentada podem usar a linha LTS quando isso reduzir churn sem perder correções relevantes;
- versão de biblioteca de fase futura só é pinada quando entrar na primeira tarefa que realmente a utiliza.

Isso evita congelar em setembro de 2026 uma versão de Typst/ZIP/Argon2 que só será implementada várias fases depois, sem deixar o executor escolher silenciosamente: a adoção de cada dependência futura continua sendo mudança explícita e revisável.

## Baseline de dependências da fundação

Versões aprovadas para a fundação inicial:

| Responsabilidade | Baseline aprovado | Observação |
|---|---:|---|
| Tauri runtime | `tauri 2.11.x` | Client desktop |
| Tauri build | `tauri-build 2.6.x` | build script do Client |
| Tauri CLI dev | `tauri-cli 2.11.x` | ferramenta de desenvolvimento, não runtime de produção |
| Tokio | `~1.51` | linha LTS com suporte até março/2027 |
| Axum | `0.8.9` | HTTP/JSON/WebSocket Host |
| rusqlite | `0.40.2` | `bundled` obrigatório; `backup` quando Backup entrar |
| Serde | `1.0.229` | contratos/configuração |
| serde_json | `1.0.151` | JSON |
| tracing | `0.1.44` | telemetria/log estruturado |
| tower-http | `0.6.8` | middleware HTTP somente quando necessário |

Observações:

- o `Cargo.lock` fixa o patch/transitivo efetivamente validado;
- features devem ser mínimas e explícitas, evitando `full` quando não houver justificativa;
- `tokio` usa a linha `1.51.x` porque o próprio projeto a declara LTS e recomenda linha fixa/LTS quando se deseja estabilidade;
- `rusqlite` usa `bundled` para não depender de SQLite instalado na máquina; a feature `backup` entra no package proprietário quando o mecanismo de Backup for implementado;
- `zip`, `typst`, `argon2`, `uuid` e outras crates aprovadas conceitualmente em fases anteriores não entram no primeiro scaffold só para “reservar versão”.

## Client sem Node/bundler no baseline

O Client inicial usa diretamente:

```text
apps/client/web/
├── index.html
├── css/
└── js/
    └── módulos ES
```

Tauri funciona como host de assets estáticos e pode incorporar o diretório do frontend diretamente via `frontendDist`.

Baseline:

- sem `package.json`;
- sem `node_modules/`;
- sem npm/pnpm/yarn;
- sem Vite/Webpack/Rollup;
- sem framework SPA;
- sem transpilation obrigatória;
- JavaScript moderno compatível com WebView2 alvo + ES modules nativos.

Se futuramente um requisito concreto demonstrar necessidade de bundler/framework, isso é mudança arquitetural explícita; não entra silenciosamente durante implementação de tela.

Integrações Tauri locais que precisem ser chamadas do JavaScript devem usar a superfície mínima suportada pelo shell e pelas capabilities do Tauri. A forma exata (`window.__TAURI__`/invoke ou módulo empacotado localmente) será fechada na primeira tarefa do Client que realmente necessitar uma chamada nativa; isso não autoriza adicionar Node por conveniência.

## Tauri CLI e ferramentas de desenvolvimento

A CLI Tauri é ferramenta da máquina de desenvolvimento, não requisito do pacote Pocket.

A Fase 2 deverá fornecer um bootstrap/script verificável que:

- confira Rust/toolchain esperada;
- confira a versão da Tauri CLI;
- oriente instalação/ajuste apenas no ambiente de desenvolvimento quando ausente;
- nunca execute instalação de toolchain na máquina de produção como parte do StepFlow.

A versão da CLI deve permanecer compatível com a linha Tauri 2 usada pelo Client e ser explicitamente registrada no scaffold/tarefa que a introduzir.

## Configuração

Separar três classes:

### 1. Build/dev

Somente opções necessárias à compilação e desenvolvimento local. Não conter IP, hostname, share ou segredo corporativo oficial.

### 2. Deployment

`deployment.json` gerado/publicado contém somente localização/identidade não sensível necessária ao Client/Launcher, conforme contrato Pocket.

### 3. Runtime central

Configuração operacional do servidor fica em `_internal/server/config/` na publicação e não é compilada nos binários.

Regras:

- exemplos ficam explicitamente como exemplos;
- secrets/tokens/senhas nunca são versionados;
- variável de ambiente pode sobrescrever configuração em desenvolvimento/teste quando a fonte específica aprovar, mas não vira mecanismo obscuro obrigatório de produção;
- configuração inválida falha com diagnóstico explícito; não cai silenciosamente em endpoint de exemplo/default perigoso.

## Build e packaging

O build oficial de desenvolvimento produz artefatos em `target/`; packaging copia somente artefatos validados para `dist/` e monta a estrutura Pocket aprovada.

`target/` e `dist/` são descartáveis e ignorados pelo Git.

Não executar produção diretamente de `target/release` como procedimento de implantação. A implantação usa saída de packaging com manifesto/hash/layout aprovado.

Baseline de profile:

- `dev`: padrão Cargo, sem otimizações prematuras;
- `release`: otimizado para release, mas sem LTO/panic abort/codegen tuning obrigatório antes de benchmark;
- strip, LTO e ajustes de tamanho/performance entram somente se medição demonstrar benefício sem prejudicar diagnóstico/compatibilidade.

## Scripts

Como o alvo oficial é Windows, scripts operacionais de desenvolvimento/build/package podem ser PowerShell em `scripts/*.ps1`.

Regras:

- scripts são wrappers finos e auditáveis;
- falham em erro;
- não escondem decisões de versão fora dos manifests/toolchain;
- build/test/package usam lockfile;
- nenhum script de desenvolvimento vira requisito para o usuário final iniciar o StepFlow.

## Decisões D12.19–D12.34

- **D12.19:** toolchain oficial inicial = Rust `1.98.0`, target `x86_64-pc-windows-msvc`, perfil rustup `minimal`, com `rustfmt` e `clippy`;
- **D12.20:** workspace usa Edition 2024 e `resolver = "3"`, com manifest virtual na raiz e packages reais apenas sob `apps/`/`crates/`;
- **D12.21:** `rust-toolchain.toml` é versionado e fixa a toolchain usada pelo projeto;
- **D12.22:** `Cargo.lock` é versionado desde o primeiro scaffold e build/test/package oficiais usam `--locked` quando a resolução não pode mudar;
- **D12.23:** atualização de dependência é mudança explícita/revisada; `cargo update` incidental, wildcard e branch Git flutuante não são baseline;
- **D12.24:** dependências realmente compartilhadas ficam em `[workspace.dependencies]`; crate sem uso concreto não entra antecipadamente;
- **D12.25:** pinning usa requisito direto concreto + resolução exata no `Cargo.lock`; dependência de fase futura é pinada quando for efetivamente adotada em tarefa revisada;
- **D12.26:** fundação inicial adota Tauri `2.11.x`, `tauri-build 2.6.x`, Tauri CLI `2.11.x`, Tokio `~1.51`, Axum `0.8.9`, rusqlite `0.40.2`, Serde `1.0.229`, serde_json `1.0.151`, tracing `0.1.44` e tower-http `0.6.8`, somente onde houver uso real;
- **D12.27:** rusqlite usa feature `bundled`; features adicionais, inclusive `backup`, só entram no package proprietário quando o recurso correspondente for implementado;
- **D12.28:** Client vanilla modular não usa Node/npm/pnpm/yarn/Vite/bundler/framework no baseline; Tauri incorpora HTML/CSS/JS estático diretamente;
- **D12.29:** adicionar bundler/framework futuramente exige decisão explícita baseada em necessidade concreta, não preferência do executor;
- **D12.30:** Tauri CLI é dependência de desenvolvimento verificada pelo bootstrap de dev e nunca runtime/requisito de produção;
- **D12.31:** configuração separa build/dev, deployment e runtime central; nenhum endpoint/segredo corporativo fica hardcoded ou versionado;
- **D12.32:** `target/` e `dist/` são descartáveis; produção é montada por packaging, não executada diretamente de `target/release`;
- **D12.33:** profiles não recebem LTO/strip/panic/codegen tuning obrigatório antes de benchmark; otimização prematura não é baseline;
- **D12.34:** scripts oficiais de desenvolvimento/build/test/package podem ser PowerShell finos, lockfile-aware e sem criar dependência operacional para o usuário final.

## Evidência técnica consultada

- Rust 1.98.0 é o stable corrente em 2026-09-01;
- Cargo documenta Edition 2024 com resolver 3 e recomenda lockfile para reprodução determinística;
- rustup suporta `rust-toolchain.toml` versionado para fixar toolchain;
- Tauri documenta frontend estático e Node somente quando o ecossistema JS escolhido exigir;
- rusqlite 0.40.2 documenta `bundled` como opção apropriada para programas que controlam o próprio SQLite e expõe feature `backup` para a Online Backup API;
- Tokio 1.51.x é linha LTS vigente até março/2027.

## Fora do escopo desta análise

- criar manifests ou lockfile;
- instalar toolchains;
- gerar scaffold;
- criar pipeline CI;
- definir migrations/fixtures;
- definir parâmetros de autenticação/Backup finais;
- adotar dependências de fases futuras sem uso imediato.
