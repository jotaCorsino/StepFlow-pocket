# Plano de Adaptação — Ambiente de Build e Execução Assistida da Fase 1

**Data:** 2026-08-20  
**Status:** PLANO RECOMENDADO  
**Escopo:** Fase 1 / Bloco 1 — Client Windows e critério Pocket

## 1. Problema que motivou a adaptação

A sequência de provas F1-B1-T04A até F1-B1-T04G demonstrou que parte dos bloqueios observados não pertence à arquitetura do StepFlow, mas ao contexto `EARTH\CodexSandboxOffline` e à preparação incompleta do ambiente de build Windows.

Misturar essas duas categorias gerou diagnósticos repetitivos e baixo avanço operacional.

O plano adapta o fluxo de trabalho para evitar novos ciclos desse tipo.

## 2. Conclusões técnicas consolidadas

### 2.1. O que já está validado

- Node.js e npm existem no computador de desenvolvimento;
- Rust 1.97.1 e Cargo 1.97.1 estão instalados no perfil `EARTH\Estudos`;
- target/host Rust: `x86_64-pc-windows-msvc`;
- o sandbox consegue ler e executar diretamente esse toolchain;
- `CARGO_HOME` pode ser isolado em diretório temporário gravável;
- `cargo init --vcs none` funciona no sandbox;
- `cargo check --offline` funciona sem dependências externas;
- WebView2 está presente no computador de desenvolvimento;
- Visual Studio Community 2026 está instalado e é detectável;
- o checkout Git pode ser sincronizado normalmente pela sessão Windows `EARTH\Estudos`.

### 2.2. Bloqueios artificiais do sandbox

Não devem ser tratados como requisitos do produto:

- Git/HTTPS via Schannel não é confiável em `CodexSandboxOffline`;
- criação/manipulação de `.git` pode ser restringida;
- `C:\dev\StepFlow-PoC` não é gravável pela identidade sandbox;
- PATH e homes do sandbox diferem do usuário real de desenvolvimento;
- acesso à rede para obtenção de dependências não deve ser presumido.

### 2.3. Lacuna real do ambiente de desenvolvimento

A instalação atual do Visual Studio Community 2026 não satisfaz o componente MSVC x64/x86 consultado por `vswhere`, e `cl.exe`, `link.exe` e `VsDevCmd.bat` não foram disponibilizados nas provas.

Para desenvolvimento Tauri no Windows, o ambiente precisa do workload **Desktop development with C++** / Microsoft C++ Build Tools. Essa é uma dependência de build do computador de desenvolvimento, não do servidor ou dos clientes finais.

## 3. Princípio de adaptação

Separar definitivamente dois papéis operacionais:

### Ambiente Windows normal do PO (`EARTH\Estudos`)

Responsável por operações que exigem uma ou mais destas capacidades:

- instalação/modificação de software global;
- elevação administrativa;
- acesso confiável à internet;
- autenticação/credenciais Windows/Git;
- sincronização Git remota;
- preparação inicial de caches/dependências externas;
- comandos que precisam reproduzir o ambiente real de desenvolvimento fora do sandbox.

### Codex (`EARTH\CodexSandboxOffline`)

Responsável por:

- leitura e alteração dos arquivos autorizados do projeto;
- análise de código e documentação;
- geração de scripts/comandos reproduzíveis;
- testes locais que não dependam de capacidades bloqueadas do sandbox;
- builds offline quando toolchain e dependências já estiverem disponíveis;
- inspeção de artefatos e evidências;
- elaboração de patches e relatórios.

O Codex não deve tentar reparar o sandbox para fazê-lo se comportar como a conta real do desenvolvedor.

## 4. Caminho principal recomendado

### Gate A — preparar uma única vez o C++ Build Environment

Executar pelo PO em sessão Windows normal.

Na instalação existente do Visual Studio Community 2026, usar o Visual Studio Installer para adicionar o workload:

**Desktop development with C++**

Manter os componentes padrão/recomendados do workload necessários a desenvolvimento x64 Windows, incluindo o MSVC Build Tools x64/x86 e Windows SDK aplicável.

Não instalar componentes preview sem necessidade.

Essa alteração é permitida no computador de desenvolvimento e não viola Pocket, pois não será repetida no servidor ou nos computadores clientes.

### Gate B — validação única fora do sandbox

Depois da modificação, validar na sessão normal do PO:

1. `vswhere` reconhece `Microsoft.VisualStudio.Component.VC.Tools.x86.x64`;
2. `VsDevCmd.bat` existe;
3. Developer Command Prompt x64 encontra `cl.exe` e `link.exe`;
4. um projeto Rust mínimo sem dependências conclui `cargo build`;
5. o executável gerado inicia.

Se esse gate falhar mesmo após o workload oficialmente exigido estar instalado, interromper Tauri e iniciar avaliação de alternativa. Não abrir nova cadeia longa de probes.

### Gate C — preparar a PoC Tauri na sessão normal

Como a rede do sandbox não é confiável, a criação inicial e obtenção de dependências externas devem ocorrer na sessão `EARTH\Estudos`.

Criar a PoC descartável fora do repositório oficial, em caminho gravável pelo PO, com:

- Tauri 2;
- Vanilla JavaScript;
- npm;
- identifier descartável;
- sem código oficial StepFlow.

Executar a instalação das dependências e o primeiro build usando a rede normal do PO.

### Gate D — prova Pocket

Executar a prova originalmente desejada:

1. `tauri build --no-bundle`;
2. registrar o `.exe`, tamanho e SHA-256;
3. copiar somente o artefato necessário para pasta isolada;
4. iniciar a cópia fora da árvore de build;
5. repetir o runtime com Node/npm/Rust/Cargo removidos apenas do PATH da sessão de teste;
6. confirmar que nenhuma toolchain é necessária em runtime;
7. registrar dependências de sistema restantes, especialmente WebView2.

Esse gate pode ser executado pelo PO com roteiro gerado pelo Assistente/Codex; o Codex analisa as evidências e consolida a decisão arquitetural.

## 5. Critério para manter Tauri

Tauri permanece a opção preferida se todos forem verdadeiros:

- build x64 funciona com ambiente de desenvolvimento oficialmente suportado;
- build `--no-bundle` produz artefato executável útil;
- cópia isolada inicia sem Node/npm/Rust/Cargo;
- nenhuma instalação administrativa é necessária no runtime do artefato testado além de dependências de sistema aceitas;
- tamanho/distribuição continuam compatíveis com Pocket;
- WebView2 presente no Windows alvo ou tratável sem transformar o servidor em ambiente de desenvolvimento.

## 6. Critério para abandonar Tauri

Não abandonar por limitações do sandbox.

Abrir avaliação comparativa apenas se, após o Gate A corretamente preparado, ocorrer uma destas situações reais:

- build Tauri continuar incompatível com Windows alvo suportado;
- artefato não puder ser executado de forma compatível com a estratégia Pocket;
- runtime exigir instalação pesada/global não aceitável;
- distribuição ou manutenção introduzir complexidade incompatível com o requisito central;
- dependências obrigatórias não puderem ser atendidas no ambiente corporativo previsto.

## 7. Alternativas se Tauri falhar por motivo real

### Alternativa 1 — Cliente web/browser-first

Host serve a aplicação web interna via HTTP(S); usuário acessa por navegador ou atalho.

Vantagens:

- distribuição de Client praticamente zero;
- maior compatibilidade com PCs antigos que possuam navegador compatível;
- atualização central imediata;
- forte aderência ao acesso por endereço interno.

Custos:

- experiência menos "aplicativo desktop";
- algumas integrações locais exigiriam abordagem web;
- launcher de duplo clique seria atalho/pequeno bootstrapper opcional.

### Alternativa 2 — Electron

Mantém desktop JS, mas empacota Chromium/Node.

Vantagens:

- toolchain frontend mais familiar;
- menor dependência de WebView do sistema.

Custos:

- artefato significativamente mais pesado;
- maior volume para distribuição/atualização;
- pior alinhamento inicial com Pocket.

### Alternativa 3 — shell Windows nativo/alternativa menor

Somente considerar após medir complexidade de manutenção, compatibilidade e impacto no time. Não é recomendada como troca imediata sem evidência de falha real do Tauri.

## 8. Estratégia de eficiência

A partir deste plano:

- no máximo uma validação por gate;
- nenhuma sequência A/B/C de microdiagnósticos para limitações já classificadas do sandbox;
- se um gate falhar, analisar o conjunto de evidências antes de criar nova tarefa;
- instalações e rede ficam no ambiente real do PO;
- o sandbox não é usado como referência de requisitos de implantação;
- exemplos e falhas do ambiente pessoal não viram requisitos corporativos.

## 9. Relação com Pocket

A preparação pesada é aceita somente no computador de desenvolvimento.

O princípio Pocket continua exigindo que o servidor/host de produção receba artefatos compilados/autocontidos na maior medida prática e não dependa de:

- Rust;
- Cargo;
- Node.js;
- npm;
- Visual Studio;
- MSVC Build Tools;
- SDKs/compiladores de desenvolvimento.

Nada neste plano autoriza instalar o toolchain de desenvolvimento no servidor da empresa.

## 10. Próxima ação

A próxima ação não é Codex.

O PO deve executar o **Gate A** uma única vez no Visual Studio Installer da máquina de desenvolvimento e adicionar `Desktop development with C++` à instalação existente do Visual Studio Community 2026.

Depois disso será preparado um único roteiro de Gate B. Se o Gate B passar, seguir diretamente para a PoC Tauri; se falhar, aplicar o critério de saída e avaliar alternativas, sem nova cadeia de probes incrementais.
