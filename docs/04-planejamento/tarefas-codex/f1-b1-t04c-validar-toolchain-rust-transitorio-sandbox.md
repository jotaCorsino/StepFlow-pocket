# Tarefa Codex F1-B1-T04C — Validar Toolchain Rust Transitório no Sandbox

**Fase:** 1 — Fechamento arquitetural e especificação  
**Bloco:** 1 — Plataforma Windows, Client e distribuição  
**Tipo:** diagnóstico operacional local e configuração transitória de sessão  
**Status:** PRONTA PARA EXECUÇÃO  
**Relação:** pré-condição corretiva para retomar `F1-B1-T04 — Prova Mínima Tauri 2 com Critério Pocket`

## 1. Objetivo

Validar, sem reinstalar Rust e sem alterar configuração permanente do Windows, se a sessão não elevada `EARTH\CodexSandboxOffline` consegue utilizar o toolchain Rust já instalado no perfil de desenvolvimento `EARTH\Estudos` por meio de caminhos explícitos e variáveis de ambiente temporárias da sessão.

A tarefa não cria a PoC Tauri.

## 2. Contexto

A F1-B1-T03 confirmou no perfil de desenvolvimento:

- Rustup home observado em `C:\Users\Estudos\.rustup`;
- toolchain `stable-x86_64-pc-windows-msvc`;
- `rustc 1.97.1`;
- `cargo 1.97.1`.

A F1-B1-T04 foi interrompida porque a conta `EARTH\CodexSandboxOffline` não encontra `rustup`, `rustc` e `cargo` no próprio `PATH`.

Esse resultado é compatível com o comportamento padrão do rustup no Windows, em que a instalação é vinculada ao perfil do usuário (`%USERPROFILE%\.cargo` e `%USERPROFILE%\.rustup`).

A meta agora é verificar se os binários reais do toolchain podem ser reutilizados pelo sandbox apenas durante a sessão.

## 3. Escopo incluído

- verificar identidade da sessão;
- verificar localmente o HEAD já conhecido, sem rede Git;
- inspecionar `%USERPROFILE%`, `PATH`, `CARGO_HOME` e `RUSTUP_HOME` da sessão sandbox;
- localizar o diretório `stable-x86_64-pc-windows-msvc` em `C:\Users\Estudos\.rustup\toolchains`;
- verificar existência e acesso aos binários `rustc.exe`, `cargo.exe` e `rustdoc.exe` do toolchain;
- executar os binários diretamente por caminho absoluto;
- criar uma pasta descartável e gravável para `CARGO_HOME` fora do repositório oficial;
- alterar `PATH` e `CARGO_HOME` somente no processo/sessão atual;
- validar `rustc`, `cargo`, host e target MSVC nesse ambiente transitório;
- encerrar/restaurar a sessão sem mudança persistente.

## 4. Fora do escopo

É proibido:

- alterar PATH de usuário ou sistema;
- usar `setx`;
- usar `[Environment]::SetEnvironmentVariable` com escopo User ou Machine;
- instalar/reinstalar Rust;
- executar rustup-init;
- alterar ACL/owner;
- copiar toolchain para outra pasta;
- modificar `C:\Users\Estudos\.rustup`;
- modificar `C:\Users\Estudos\.cargo`;
- executar PowerShell elevado;
- executar `git pull`, `git fetch` ou qualquer sincronização de rede Git;
- criar a PoC Tauri;
- executar npm create tauri-app;
- executar npm install;
- compilar Tauri;
- modificar o repositório oficial;
- alterar o diário;
- commit/push.

## 5. Estado Git esperado

O último HEAD confirmado pelo PO antes desta tarefa foi:

```text
97c5f4a docs: record sandbox Git diagnostics and manual sync gate
```

Como esta tarefa foi criada posteriormente apenas no remoto e a sincronização de rede do Codex é bloqueada, o arquivo desta tarefa pode não existir no checkout local.

Não tentar sincronizar.

Em `C:\dev\StepFlow`, verificar somente:

```powershell
git status --short --branch
git diff --check
git branch --show-current
git log -1 --oneline
```

Aceitar `97c5f4a` como HEAD local desta tarefa, desde que a única modificação seja o diário autorizado.

## 6. Procedimento obrigatório

Executar em nova sessão PowerShell **não elevada**.

### 6.1. Identidade e ambiente atual

```powershell
whoami
$env:USERPROFILE
$env:CARGO_HOME
$env:RUSTUP_HOME
$env:PATH -split ';'
```

Registrar os valores sem alterar ainda.

### 6.2. Localizar o toolchain existente

```powershell
$RustupRoot = 'C:\Users\Estudos\.rustup'
$ToolchainsRoot = Join-Path $RustupRoot 'toolchains'

Test-Path $RustupRoot
Test-Path $ToolchainsRoot
Get-ChildItem $ToolchainsRoot -Directory -ErrorAction Stop |
  Select-Object FullName,Name
```

Localizar especificamente:

```text
stable-x86_64-pc-windows-msvc
```

Definir:

```powershell
$ToolchainRoot = 'C:\Users\Estudos\.rustup\toolchains\stable-x86_64-pc-windows-msvc'
$ToolchainBin = Join-Path $ToolchainRoot 'bin'
```

Verificar:

```powershell
Test-Path (Join-Path $ToolchainBin 'rustc.exe')
Test-Path (Join-Path $ToolchainBin 'cargo.exe')
Test-Path (Join-Path $ToolchainBin 'rustdoc.exe')

Get-Item (Join-Path $ToolchainBin 'rustc.exe') |
  Select-Object FullName,Length,LastWriteTime
Get-Item (Join-Path $ToolchainBin 'cargo.exe') |
  Select-Object FullName,Length,LastWriteTime
```

Se houver `Access denied`, registrar e parar sem alterar permissões.

### 6.3. Executar diretamente por caminho absoluto

```powershell
& (Join-Path $ToolchainBin 'rustc.exe') --version
& (Join-Path $ToolchainBin 'rustc.exe') -Vv
& (Join-Path $ToolchainBin 'cargo.exe') --version
```

Se qualquer executável não puder ser iniciado pela conta sandbox, parar e reportar.

### 6.4. Preparar CARGO_HOME descartável

Usar fora do repositório oficial:

```text
C:\dev\StepFlow-PoC\F1-B1-T04C\cargo-home\
```

Criar somente essa estrutura descartável:

```powershell
$ProbeRoot = 'C:\dev\StepFlow-PoC\F1-B1-T04C'
$ProbeCargoHome = Join-Path $ProbeRoot 'cargo-home'
New-Item -ItemType Directory -Force -Path $ProbeCargoHome | Out-Null
```

Confirmar escrita local com um arquivo de prova que deve ser removido imediatamente:

```powershell
$WriteProbe = Join-Path $ProbeCargoHome 'write-probe.tmp'
Set-Content -Path $WriteProbe -Value 'ok' -NoNewline
Get-Item $WriteProbe | Select-Object FullName,Length
Remove-Item $WriteProbe
```

Nenhum arquivo deve ser criado dentro de `C:\dev\StepFlow`.

### 6.5. Configuração transitória da sessão

Salvar os valores atuais:

```powershell
$OriginalPath = $env:PATH
$OriginalCargoHome = $env:CARGO_HOME
$OriginalRustupHome = $env:RUSTUP_HOME
```

Na sessão atual apenas:

```powershell
$env:CARGO_HOME = $ProbeCargoHome
$env:PATH = "$ToolchainBin;$OriginalPath"
```

Não definir `RUSTUP_HOME` para a futura execução Tauri, pois a estratégia desta prova é usar os binários reais do toolchain diretamente e manter o cache Cargo descartável separado.

Confirmar:

```powershell
Get-Command rustc
Get-Command cargo
rustc --version
rustc -Vv
cargo --version
```

Confirmar especialmente:

```text
host: x86_64-pc-windows-msvc
```

### 6.6. Restauração lógica da sessão

Antes de finalizar, restaurar somente as variáveis do processo atual:

```powershell
$env:PATH = $OriginalPath

if ($null -eq $OriginalCargoHome) {
  Remove-Item Env:CARGO_HOME -ErrorAction SilentlyContinue
} else {
  $env:CARGO_HOME = $OriginalCargoHome
}

if ($null -eq $OriginalRustupHome) {
  Remove-Item Env:RUSTUP_HOME -ErrorAction SilentlyContinue
} else {
  $env:RUSTUP_HOME = $OriginalRustupHome
}
```

Confirmar que a alteração era apenas transitória:

```powershell
Get-Command rustc -ErrorAction SilentlyContinue
Get-Command cargo -ErrorAction SilentlyContinue
```

É esperado que voltem a não ser encontrados no PATH padrão da conta sandbox.

A pasta `C:\dev\StepFlow-PoC\F1-B1-T04C\` pode permanecer como evidência descartável e poderá ser removida em tarefa futura.

## 7. Critérios de aceite

- [ ] sessão `CodexSandboxOffline` confirmada como não elevada;
- [ ] HEAD/local tree verificados sem rede;
- [ ] toolchain `stable-x86_64-pc-windows-msvc` localizado no perfil `Estudos`;
- [ ] `rustc.exe` e `cargo.exe` acessíveis por caminho absoluto;
- [ ] host `x86_64-pc-windows-msvc` confirmado;
- [ ] CARGO_HOME descartável criado fora do repo e gravável;
- [ ] `PATH` temporário permite executar `rustc` e `cargo` por nome;
- [ ] nenhuma mudança persistente de PATH/ambiente realizada;
- [ ] nenhum arquivo do repo alterado;
- [ ] PoC Tauri não criada.

## 8. Relatório final obrigatório

Responder com:

1. objetivo executado;
2. usuário/contexto da sessão;
3. HEAD/branch e working tree local;
4. `USERPROFILE`, `CARGO_HOME` e `RUSTUP_HOME` originais;
5. diretórios de toolchain encontrados;
6. existência/acesso a `rustc.exe`, `cargo.exe` e `rustdoc.exe`;
7. resultado de `rustc --version` por caminho absoluto;
8. host retornado por `rustc -Vv` por caminho absoluto;
9. resultado de `cargo --version` por caminho absoluto;
10. caminho do CARGO_HOME descartável criado;
11. resultado do teste de escrita no CARGO_HOME descartável;
12. resultado de `Get-Command rustc` e `Get-Command cargo` após PATH temporário;
13. versões e host com o ambiente transitório;
14. confirmação de que nenhuma variável permanente foi alterada;
15. resultado após restaurar o ambiente da sessão;
16. arquivos/pastas criados fora do repo;
17. estado Git final;
18. erros/limitações encontrados;
19. conclusão objetiva sobre a viabilidade de usar o toolchain de `Estudos` pelo sandbox;
20. recomendação para retomar ou adaptar a F1-B1-T04, sem executá-la.

## 9. Regra de parada

Parar e reportar sem ampliar escopo se:

- a conta sandbox não puder ler/executar o toolchain existente;
- o diretório stable MSVC não existir;
- os binários retornarem host incompatível;
- o CARGO_HOME descartável não for gravável;
- for necessária elevação;
- for necessária alteração permanente de PATH, ACL ou perfil de usuário.

Não executar a F1-B1-T04 nesta tarefa.