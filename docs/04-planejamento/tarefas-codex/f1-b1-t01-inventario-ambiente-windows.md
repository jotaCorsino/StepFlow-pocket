# Tarefa Codex F1-B1-T01 — Inventário do Ambiente Windows e Pré-requisitos

**Fase:** 1 — Fechamento arquitetural e especificação

**Bloco:** 1 — Plataforma Windows, Client e distribuição

**Tipo:** investigação local, sem implementação de produto

**Status:** PRONTA PARA EXECUÇÃO

## Pré-requisito obrigatório

Concluído e validado em 2026-08-19:

`docs/04-planejamento/tarefas-codex/f1-b0-t01-bootstrap-repositorio-local.md`

O ambiente `C:\dev\StepFlow` foi confirmado como clone íntegro e limpo de `jotaCorsino/StepFlow-pocket`, branch `main`, com `AGENTS.md` lido pelo Codex.

Antes de iniciar esta tarefa, executar `git pull --ff-only` para incorporar atualizações documentais remotas posteriores ao bootstrap. Se o pull falhar ou houver alterações locais inesperadas, parar e reportar; não resolver por merge ou reset automático.

## 1. Objetivo

Levantar de forma mecânica o ambiente Windows disponível em `C:\dev\StepFlow` e os pré-requisitos relevantes para uma futura prova Tauri, sem instalar dependências, sem criar scaffold e sem implementar qualquer funcionalidade do StepFlow.

A tarefa deve responder com evidências:

- qual versão/edição/build do Windows está sendo usada;
- arquitetura do sistema operacional e do processador;
- se WebView2 Runtime está instalado e qual versão é detectável;
- se Node.js/npm já existem;
- se Rust/rustup/cargo já existem;
- se Microsoft C++ Build Tools/Visual Studio Build Tools são detectáveis;
- se Git está disponível e qual configuração básica do checkout;
- se o compartilhamento `\\192.168.5.7\Arquivos\StepFlow\` é acessível a partir desta máquina;
- se há algum bloqueio óbvio para a próxima prova técnica.

## 2. Contexto e fonte de verdade

Ler antes de executar:

1. `AGENTS.md`;
2. `docs/00-governanca/guia-mestre-desenvolvimento.md`;
3. `docs/04-planejamento/plano-oficial-fase-1.md`;
4. `docs/03-arquitetura/compatibilidade-windows-client.md`;
5. `docs/03-arquitetura/arquitetura-inicial.md`;
6. `docs/05-progresso/registro-de-decisoes.md`.

## 3. Estado inicial esperado

- repositório clonado/localizado em `C:\dev\StepFlow`;
- bootstrap local concluído e validado;
- branch `main` ativa;
- working tree limpo;
- checkout atualizado por `git pull --ff-only` antes da inspeção;
- nenhuma aplicação StepFlow funcional existe;
- nenhuma stack definitiva deve ser presumida como instalada;
- Tauri 2 é a direção arquitetural recomendada para prova, mas esta tarefa não cria projeto Tauri.

## 4. Escopo incluído

- atualizar o clone somente por fast-forward com `git pull --ff-only`;
- inspecionar sistema operacional;
- inspecionar arquitetura CPU/SO;
- inspecionar WebView2 por meios locais confiáveis, como registro, arquivos/runtime ou mecanismos do Windows;
- inspecionar versões existentes de Git, Node/npm, Rust/rustup/cargo;
- detectar Visual Studio/Build Tools/C++ quando possível sem instalar nada;
- verificar acesso de leitura ao compartilhamento `\\192.168.5.7\Arquivos\StepFlow\`;
- verificar estado Git local do repositório;
- registrar resultados e limitações;
- atualizar somente documentação de progresso necessária.

## 5. Fora do escopo

É proibido nesta tarefa:

- instalar Node.js;
- instalar Rust;
- instalar Visual Studio Build Tools;
- instalar WebView2;
- executar `npm create tauri-app` ou equivalente;
- criar `package.json`;
- criar `src-tauri`;
- criar HTML/CSS/JavaScript de produto;
- criar banco SQLite;
- criar Host;
- criar launcher;
- alterar arquitetura ou roadmap por conta própria;
- fazer commit/push sem instrução explícita separada do fluxo de trabalho vigente;
- resolver divergência Git por merge, rebase, reset ou force.

## 6. Regras e restrições

- executar somente comandos de leitura/inspeção ou testes de acesso sem alteração persistente relevante;
- `git pull --ff-only` é a única atualização Git autorizada antes da inspeção;
- não alterar políticas do Windows;
- não habilitar features opcionais;
- não alterar PATH;
- não mapear unidade de rede de forma persistente;
- não gravar no compartilhamento de rede;
- se determinado mecanismo de detecção falhar, tentar alternativa de leitura segura e registrar o método;
- não mascarar ausência de requisito como sucesso;
- não considerar Windows 7/8/8.1 como alvo oficial da primeira versão;
- não concluir que a máquina representa todas as estações da empresa; ela é apenas uma amostra.

## 7. Arquivos/áreas esperadas

A tarefa não deve criar código.

Documentação que pode ser alterada:

- `docs/05-progresso/diario-de-progresso.md`;
- opcionalmente um relatório de evidências em `docs/05-progresso/` se o volume justificar.

Não alterar `compatibilidade-windows-client.md` automaticamente apenas para registrar dados de uma máquina. Primeiro reportar os resultados.

## 8. Critérios de aceite

- [ ] bootstrap local anterior foi validado;
- [ ] `git pull --ff-only` foi executado com sucesso antes da inspeção;
- [ ] versão, edição e build do Windows foram identificados;
- [ ] arquitetura x64/x86/ARM foi identificada;
- [ ] presença/ausência e versão do WebView2 foram verificadas com evidência;
- [ ] Git foi verificado;
- [ ] Node/npm foram verificados;
- [ ] Rust/rustup/cargo foram verificados;
- [ ] Build Tools/C++ foram verificados na medida possível;
- [ ] acesso ao caminho `\\192.168.5.7\Arquivos\StepFlow\` foi testado sem escrita;
- [ ] `git status`, branch e remote do checkout foram registrados;
- [ ] nenhum pacote ou dependência foi instalado;
- [ ] nenhum scaffold/código de produto foi criado;
- [ ] limitações da inspeção foram explicitadas.

## 9. Validações obrigatórias

Primeiro:

```powershell
Set-Location C:\dev\StepFlow
git status --short --branch
git pull --ff-only
```

Se houver alterações locais inesperadas ou o pull não puder ser fast-forward, parar.

Depois executar equivalentes seguros no PowerShell, adaptando apenas quando necessário:

```powershell
Get-ComputerInfo | Select-Object WindowsProductName, WindowsVersion, OsBuildNumber, OsArchitecture
[Environment]::Is64BitOperatingSystem
[Environment]::Is64BitProcess
Get-CimInstance Win32_Processor | Select-Object Name, AddressWidth, DataWidth
```

Verificar ferramentas:

```powershell
git --version
node --version
npm --version
rustc --version
rustup --version
cargo --version
```

A ausência de uma ferramenta é um resultado válido; não instalar.

Tentar detectar Visual Studio/Build Tools com mecanismo disponível, preferindo `vswhere.exe` se existir.

Verificar WebView2 por pelo menos um método confiável e registrar exatamente qual método foi usado. Pode incluir consulta às chaves de registro oficiais do EdgeUpdate ou inspeção da instalação do runtime.

Verificar somente leitura do compartilhamento:

```powershell
Test-Path '\\192.168.5.7\Arquivos\StepFlow\'
Get-ChildItem '\\192.168.5.7\Arquivos\StepFlow\' -ErrorAction SilentlyContinue | Select-Object -First 20 Name, Length, LastWriteTime
```

Não criar, editar nem remover arquivos nessa pasta.

Verificar o checkout ao final:

```powershell
Set-Location C:\dev\StepFlow
git status --short --branch
git remote -v
git branch --show-current
git log -1 --oneline
```

## 10. Documentação a atualizar

Atualizar `docs/05-progresso/diario-de-progresso.md` com uma entrada curta contendo:

- objetivo da inspeção;
- ambiente encontrado;
- pré-requisitos presentes;
- pré-requisitos ausentes;
- resultado do acesso ao compartilhamento;
- bloqueios para a próxima prova.

Não registrar uma nova decisão arquitetural sem que ela decorra diretamente de evidência e esteja prevista na tarefa.

Como commit/push ainda não estão autorizados nesta tarefa, a alteração documental deve permanecer no working tree e ser incluída no relatório final para revisão posterior.

## 11. Relatório final obrigatório

Responder com:

1. objetivo executado;
2. resultado de `git pull --ff-only` e HEAD inspecionado;
3. Windows/arquitetura identificados;
4. WebView2 detectado e versão, ou ausência;
5. ferramentas encontradas e versões;
6. Build Tools detectados ou não;
7. resultado do acesso a `\\192.168.5.7\Arquivos\StepFlow\`;
8. estado Git do checkout;
9. arquivos alterados;
10. validações executadas;
11. bloqueios/pendências;
12. recomendação objetiva para a próxima tarefa, sem executá-la.

## 12. Regra de parada

Se o bootstrap local não estiver concluído, não executar o inventário.

Se `git pull --ff-only` falhar, se `C:\dev\StepFlow` não for o repositório correto, se houver alterações locais inesperadas relevantes ou se a inspeção exigir instalação/modificação do sistema para prosseguir, parar essa parte e reportar o bloqueio. Não corrigir silenciosamente o ambiente nesta tarefa.