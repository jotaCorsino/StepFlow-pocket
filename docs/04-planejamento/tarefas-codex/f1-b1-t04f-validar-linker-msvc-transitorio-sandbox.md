# Tarefa Codex F1-B1-T04F — Validar Linker MSVC Transitório no Sandbox

**Fase:** 1 — Fechamento arquitetural e especificação  
**Bloco:** 1 — Plataforma Windows, Client e distribuição  
**Tipo:** prova técnica local descartável  
**Status:** PRONTA PARA EXECUÇÃO  
**Relação:** continuação da F1-B1-T04E; pré-condição para retomar F1-B1-T04

## 1. Objetivo

Validar, sem instalar ou alterar componentes do Visual Studio e sem modificar configuração permanente do Windows, se a sessão não elevada `EARTH\CodexSandboxOffline` consegue carregar temporariamente o ambiente MSVC x64 já instalado e concluir o link de um projeto Rust mínimo com `cargo build --offline`.

A tarefa não cria Tauri.

## 2. Estado conhecido

- checkout local autorizado em `main`, HEAD `97c5f4a`;
- única alteração local autorizada: `docs/05-progresso/diario-de-progresso.md`;
- `rustc 1.97.1` e `cargo 1.97.1` funcionam via toolchain existente;
- host `x86_64-pc-windows-msvc`;
- `%TEMP%` é gravável;
- `cargo init --vcs none` e `cargo check --offline` funcionam;
- `cargo build --offline` falha apenas porque `link.exe` não é localizado;
- Visual Studio/MSVC já foram inventariados anteriormente no computador de desenvolvimento.

## 3. Fundamentação

A Microsoft documenta que o Developer Command Prompt/Developer PowerShell configura as variáveis necessárias às ferramentas C++ e que `VsDevCmd.bat` aceita arquitetura de target/host.

Fontes:

- https://learn.microsoft.com/visualstudio/ide/reference/command-prompt-powershell
- https://learn.microsoft.com/visualstudio/install/workload-component-id-vs-community

## 4. Procedimento obrigatório

Executar em nova sessão PowerShell não elevada.

### 4.1. Gate local

Em `C:\dev\StepFlow`:

```powershell
git status --short --branch
git diff --check
git branch --show-current
git log -1 --oneline
```

Aceitar somente:

- branch `main`;
- HEAD `97c5f4a`;
- única modificação local no diário autorizado.

Não executar rede Git.

### 4.2. Localizar Visual Studio/MSVC sem modificar nada

Primeiro testar o `vswhere.exe` padrão do Visual Studio Installer:

```powershell
$VsWhere = "${env:ProgramFiles(x86)}\Microsoft Visual Studio\Installer\vswhere.exe"
Test-Path $VsWhere
```

Se existir, consultar:

```powershell
& $VsWhere -latest -products * -requires Microsoft.VisualStudio.Component.VC.Tools.x86.x64 -property installationPath
```

Registrar o `installationPath` retornado.

Se o `vswhere.exe` não existir, não baixar nem instalar ferramenta; parar e reportar.

Definir:

```powershell
$VsInstallPath = (& $VsWhere -latest -products * -requires Microsoft.VisualStudio.Component.VC.Tools.x86.x64 -property installationPath | Select-Object -First 1)
$VsDevCmd = Join-Path $VsInstallPath 'Common7\Tools\VsDevCmd.bat'
```

Validar:

```powershell
$VsInstallPath
Test-Path $VsDevCmd
Get-Item $VsDevCmd | Select-Object FullName,Length,LastWriteTime
```

Se não houver instalação com o componente MSVC x64/x86 ou `VsDevCmd.bat` não existir, parar e reportar sem abrir o Visual Studio Installer.

### 4.3. Confirmar que link.exe não está no ambiente normal

Antes de carregar ambiente MSVC:

```powershell
where.exe link
Get-Command link.exe -ErrorAction SilentlyContinue
```

Registrar o resultado.

### 4.4. Criar novo probe temporário Rust sem VCS

Usar:

```powershell
$BaseTemp = [System.IO.Path]::GetTempPath()
$ProbeRoot = Join-Path $BaseTemp 'StepFlow-PoC\F1-B1-T04F'
$ProbeCargoHome = Join-Path $ProbeRoot 'cargo-home'
$CargoProbe = Join-Path $ProbeRoot 'cargo-probe'

New-Item -ItemType Directory -Force -Path $ProbeCargoHome | Out-Null
New-Item -ItemType Directory -Force -Path $CargoProbe | Out-Null
```

Toolchain:

```powershell
$ToolchainBin = 'C:\Users\Estudos\.rustup\toolchains\stable-x86_64-pc-windows-msvc\bin'
```

Criar o projeto usando diretamente Cargo com configuração temporária da sessão atual:

```powershell
$OriginalPath = $env:PATH
$OriginalCargoHome = $env:CARGO_HOME
$OriginalRustupHome = $env:RUSTUP_HOME

$env:CARGO_HOME = $ProbeCargoHome
$env:PATH = "$ToolchainBin;$OriginalPath"

Set-Location $CargoProbe
cargo init --bin --name stepflow_msvc_probe --vcs none .
cargo check --offline
```

Se `cargo init` ou `cargo check --offline` falhar, parar e reportar.

### 4.5. Testar ambiente MSVC em processo filho

Não tentar importar permanentemente variáveis para Windows.

Executar um processo `cmd.exe` filho que:

1. chama `VsDevCmd.bat` com host/target x64;
2. adiciona o toolchain Rust ao PATH somente desse processo;
3. define `CARGO_HOME` somente nesse processo;
4. confirma `where link`;
5. confirma `where rustc` e `where cargo`;
6. executa `cargo build --offline` no probe.

Construir o comando de forma equivalente a:

```powershell
$CmdLine = "call `"$VsDevCmd`" -arch=x64 -host_arch=x64 && " +
           "set `"CARGO_HOME=$ProbeCargoHome`" && " +
           "set `"PATH=$ToolchainBin;%PATH%`" && " +
           "where link && where rustc && where cargo && " +
           "cd /d `"$CargoProbe`" && " +
           "cargo build --offline"

cmd.exe /d /s /c $CmdLine
$BuildExitCode = $LASTEXITCODE
$BuildExitCode
```

Registrar a saída relevante do `VsDevCmd`, caminhos retornados por `where` e exit code.

Não usar Developer Command Prompt elevado.

Se `VsDevCmd.bat` solicitar instalação, alteração de componentes ou falhar por ausência de SDK/toolset, parar e reportar.

### 4.6. Validar artefato se build passar

Se exit code for `0`, localizar:

```powershell
$Exe = Join-Path $CargoProbe 'target\debug\stepflow_msvc_probe.exe'
Test-Path $Exe
Get-Item $Exe | Select-Object FullName,Length,LastWriteTime
Get-FileHash $Exe -Algorithm SHA256
```

Executar smoke test simples:

```powershell
& $Exe
$LASTEXITCODE
```

Esperado: programa Rust padrão executa e encerra normalmente.

### 4.7. Restaurar ambiente PowerShell atual

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

## 5. Proibições

- não instalar/modificar Visual Studio ou Build Tools;
- não abrir Visual Studio Installer para corrigir componentes;
- não baixar `vswhere`;
- não alterar PATH User/Machine;
- não usar `setx`;
- não alterar ACL/owner;
- não executar elevado;
- não usar rede Cargo/npm/Git;
- não adicionar crates;
- não criar Tauri;
- não modificar o repositório oficial/diário;
- não commit/push;
- não executar F1-B1-T04.

## 6. Critérios de aceite

- Visual Studio/MSVC instalado localizado via `vswhere`;
- `VsDevCmd.bat` existente;
- `link.exe` ausente antes e localizado após carregar ambiente MSVC no processo filho;
- Cargo/Rust permanecem utilizáveis no mesmo processo;
- `cargo build --offline` conclui com exit code 0;
- executável Rust mínimo é gerado e executa;
- nenhuma instalação, elevação ou mudança permanente é realizada.

## 7. Relatório final obrigatório

Responder com:

1. objetivo executado;
2. usuário/contexto;
3. HEAD/branch/working tree;
4. caminho e resultado do `vswhere`;
5. installationPath Visual Studio detectado;
6. caminho/estado de `VsDevCmd.bat`;
7. resultado de `where link` no ambiente normal;
8. caminho temporário do probe;
9. resultado de `cargo init --vcs none`;
10. resultado de `cargo check --offline`;
11. parâmetros usados no `VsDevCmd.bat`;
12. resultado de `where link` no processo Developer Command Prompt;
13. resultado de `where rustc` e `where cargo` nesse processo;
14. resultado/exit code de `cargo build --offline`;
15. caminho/tamanho/SHA-256 do executável, se gerado;
16. resultado do smoke test do executável;
17. confirmação de ausência de rede/elevação/instalação;
18. restauração das variáveis e estado Git final;
19. conclusão sobre disponibilidade do linker MSVC para builds Rust no sandbox;
20. recomendação para retomar/adaptar F1-B1-T04.

## 8. Regra de parada

Parar sem ampliar escopo se `vswhere` não existir, não detectar instalação com MSVC x64/x86, `VsDevCmd.bat` não existir, o ambiente Developer Command Prompt não localizar `link.exe`, o build exigir componente adicional, rede, elevação ou alteração persistente.
