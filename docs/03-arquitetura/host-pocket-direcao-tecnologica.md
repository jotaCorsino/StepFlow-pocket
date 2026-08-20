# StepFlow Host — Direção Tecnológica Pocket

**Fase:** 1 — Fechamento arquitetural e especificação  
**Bloco:** 2 — StepFlow Host  
**Status:** DIREÇÃO RECOMENDADA / SUJEITA À PROVA POCKET DO HOST  
**Data:** 2026-08-20

## 1. Objetivo

Definir a direção tecnológica inicial do StepFlow Host antes de qualquer implementação definitiva, respeitando o requisito central de implantação Pocket.

O Host deverá coordenar autenticação, autorização, API, acesso exclusivo ao SQLite, transações, revisão otimista, serialização de escritas quando necessária, auditoria, eventos, backup/restauração e arquivos persistentes.

## 2. Restrições obrigatórias

A tecnologia do Host deve favorecer:

- implantação por pasta/copy-deploy;
- ausência de runtime global no servidor;
- ausência de Node.js, npm, Rust, Cargo, Visual Studio ou compiladores no servidor;
- ausência de mudanças permanentes em PATH;
- nenhuma reinicialização normal do Windows;
- configuração, dados, logs e backups isolados do restante do sistema;
- atualização de binários sem sobrescrever dados persistentes;
- execução offline da Internet depois da implantação;
- baixo impacto sobre outros serviços já existentes na máquina central.

A máquina central poderá ser um servidor Windows, notebook ou desktop comum. O produto deve evitar pressupor infraestrutura dedicada.

## 3. Opções consideradas

### 3.1. Rust + Tokio/Axum + rusqlite

Pontos favoráveis:

- produz executável nativo Windows;
- reutiliza o toolchain Rust/MSVC já validado para o Client Tauri no computador de desenvolvimento;
- target `x86_64-pc-windows-msvc` é Tier 1 do Rust;
- Axum fornece HTTP e suporte a WebSocket sobre o ecossistema Tokio/Tower;
- `rusqlite` permite compilar SQLite junto do aplicativo com a feature `bundled`, evitando exigir SQLite instalado globalmente no servidor;
- boa aderência a distribuição por pasta e isolamento de dependências de runtime.

Custos/riscos:

- maior rigor de implementação que uma stack dinâmica;
- dependências Rust precisam ser pinadas e auditadas no projeto;
- detalhes de shutdown, pool/uso de conexões SQLite, logging e serviço Windows exigem desenho explícito.

### 3.2. .NET / ASP.NET Core self-contained

Pontos favoráveis:

- publicação self-contained oficialmente suportada;
- publicação single-file disponível;
- ecossistema maduro para HTTP, background services, logging e Windows.

Custos/riscos:

- introduz uma segunda stack/toolchain principal no projeto;
- self-contained inclui runtime/framework no artefato e tende a gerar distribuição maior;
- native libraries e comportamento de single-file precisam ser avaliados caso bibliotecas externas sejam usadas.

Permanece alternativa forte de contingência.

### 3.3. Go

Pontos favoráveis:

- `go build` produz executável Windows simples;
- biblioteca padrão forte para HTTP;
- boa aderência conceitual a binário único.

Custos/riscos:

- introduz outro toolchain de desenvolvimento;
- SQLite exigiria escolher e validar driver/binding adicional;
- não há vantagem suficiente neste momento para superar a reutilização de Rust já validado.

Permanece alternativa secundária.

### 3.4. Node.js Single Executable Application

Pontos favoráveis:

- JavaScript familiar;
- Node atual pode produzir Single Executable Applications.

Custos/riscos:

- SEA permanece em desenvolvimento ativo;
- o Host central é o componente persistente mais crítico do sistema;
- empacotar um runtime Node inteiro não oferece vantagem clara sobre Rust para o requisito Pocket.

Não recomendado como direção principal.

## 4. Direção recomendada

Adotar para a próxima prova técnica:

**Rust + Axum + Tokio + rusqlite com SQLite bundled**.

Essa direção ainda não autoriza criar o Host definitivo. Ela autoriza somente uma PoC descartável da Fase 1 para validar empacotamento e runtime Pocket.

## 5. Escopo da prova técnica necessária

A primeira PoC do Host deverá ficar fora do repositório oficial e provar somente:

1. build release Windows x64;
2. processo HTTP local com endpoint `/health`;
3. criação/abertura de arquivo SQLite local ao diretório de dados da PoC;
4. escrita e leitura SQLite simples;
5. endpoint ou mecanismo mínimo que demonstre que Axum está operacional;
6. executável copiado para pasta isolada junto de configuração/dados necessários;
7. execução sem Rust/Cargo/Node/npm no PATH;
8. nenhuma instalação adicional no runtime;
9. registro do tamanho e SHA-256 do executável;
10. encerramento controlado do processo.

WebSocket pode ser incluído na PoC se não ampliar desnecessariamente o escopo; sua decisão de protocolo final pertence também ao Bloco 4.

## 6. Forma de execução do Host ainda não decidida

A tecnologia do Host e a forma de inicialização são decisões distintas.

Ainda não está aprovado se o Host será:

- processo normal iniciado por script/atalho administrativo;
- processo de background/tray;
- serviço Windows;
- outra forma controlada.

Registrar um serviço Windows altera estado do sistema e deve ser justificado frente ao princípio Pocket. A PoC inicial não deve instalar serviço.

## 7. Estrutura operacional conceitual

Continuar usando somente estrutura conceitual até o fechamento do Bloco 2:

```text
StepFlow\
├── app\
│   └── StepFlowHost.exe
├── config\
├── data\
├── logs\
└── backups\
```

Dados persistentes não devem ficar acoplados à pasta/versionamento dos binários de forma que uma atualização os sobrescreva.

## 8. Atualização e rollback

A direção deve permitir:

- parar o Host de forma controlada;
- substituir/alternar binários de `app`;
- preservar `config`, `data`, `logs` e `backups`;
- iniciar nova versão;
- retornar à versão anterior do binário sem restaurar banco automaticamente.

Migrations e compatibilidade de schema serão tratadas nos blocos posteriores e podem limitar rollback de binário; portanto rollback de aplicação e rollback de dados não devem ser confundidos.

## 9. Critério para abandonar Rust como direção do Host

Não abandonar por limitações do sandbox Codex.

Reavaliar a stack apenas se uma prova em ambiente Windows normal demonstrar problema real, como:

- impossibilidade de produzir artefato Pocket aceitável;
- dependência global inesperada no runtime;
- incompatibilidade com Windows alvo suportado;
- SQLite bundled ou rede HTTP introduzirem dependência operacional inadequada;
- complexidade de manutenção claramente superior às alternativas.

## 10. Próximas decisões do Bloco 2

Após a PoC de runtime/empacotamento:

1. forma de inicialização automática;
2. política de bind/endereço/porta;
3. paths finais/conceituais de config, data, logs e backups;
4. logging e diagnóstico de indisponibilidade;
5. shutdown controlado;
6. política de atualização do Host;
7. justificativa ou rejeição de serviço Windows.

Nenhuma dessas decisões deve hardcodar IP, hostname ou caminho da empresa ainda não confirmado.