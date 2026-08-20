# Tarefa Codex F1-B1-T04G — Detectar MSVC inclusive pré-release e validar linker

**Fase:** 1 — Fechamento arquitetural e especificação  
**Bloco:** 1 — Plataforma Windows, Client e distribuição  
**Tipo:** diagnóstico operacional local e prova de build Rust mínima  
**Status:** PRONTA PARA EXECUÇÃO  
**Relação:** continuação da F1-B1-T04F; pré-condição para retomar F1-B1-T04

## 1. Objetivo

Determinar se a instalação Visual Studio 2026/MSVC existente está registrada como pré-release e, se estiver íntegra, carregar temporariamente seu ambiente de desenvolvedor e concluir um build Rust mínimo offline sem instalar ou alterar componentes.

A tarefa não cria a PoC Tauri.

## 2. Contexto

A F1-B1-T04F encontrou `vswhere.exe`, mas a consulta sem `-prerelease` usando `-requires Microsoft.VisualStudio.Component.VC.Tools.x86.x64` não retornou instalação. Inventários anteriores registraram Visual Studio Community 2026 e MSVC no computador de desenvolvimento.

A documentação Microsoft informa que `vswhere` não pesquisa pré-releases por padrão e que `-prerelease` deve ser usado para incluí-las. A documentação atual de componentes do Visual Studio mantém `Microsoft.VisualStudio.Component.VC.Tools.x86.x64` como ID das ferramentas MSVC x64/x86.

Fontes:
- https://learn.microsoft.com/visualstudio/install/tools-for-managing-visual-studio-instances
- https://learn.microsoft.com/visualstudio/install/workload-component-id-vs-community
- https://learn.microsoft.com/visualstudio/ide/reference/command-prompt-powershell

## 3. Estado local aceito

- branch `main`;
- HEAD local autorizado: `97c5f4a`;
- única modificação local autorizada: `docs/05-progresso/diario-de-progresso.md`.

Não executar `git pull` ou `git fetch`.

## 4. Procedimento

Executar em PowerShell não elevado.

### 4.1 Gate Git local

```powershell
Set-Location C:\dev\StepFlow
git status --short --branch
git diff --check
git branch --show-current
git log -1 --oneline
```

Parar se branch/HEAD/working tree divergirem do estado autorizado.

### 4.2 Inventário completo do vswhere

```powershell
$VsWhere = "${env:ProgramFiles(x86)}\Microsoft Visual Studio\Installer\vswhere.exe"

& $VsWhere -all -products * -format json
& $VsWhere -all -prerelease -products * -format json
```

Registrar as instâncias retornadas, especialmente `installationName`, `installationVersion`, `productId`, `installationPath`, `isPrerelease` e estado relevante.

Depois comparar:

```powershell
& $VsWhere -all -products * -requires Microsoft.VisualStudio.Component.VC.Tools.x86.x64 -property installationPath

& $VsWhere -all -prerelease -products * -requires Microsoft.VisualStudio.Component.VC.Tools.x86.x64 -property installationPath
```

Se somente a segunda consulta retornar a instalação, registrar explicitamente que a omissão de `-prerelease` explica o resultado da F1-B1-T04F.

### 4.3 Localizar ferramentas sem inventar caminhos

Usar `vswhere` com `-prerelease` para localizar diretamente:

```powershell
$VsDevCmd = (& $VsWhere -latest -prerelease -products * -find Common7\Tools\VsDevCmd.bat | Select-Object -First 1)
$Link = (& $VsWhere -latest -prerelease -products * -find VC\Tools\MSVC\**\bin\Hostx64\x64\link.exe | Select-Object -First 1)
$Cl = (& $VsWhere -latest -prerelease -products * -find VC\Tools\MSVC\**\bin\Hostx64\x64\cl.exe | Select-Object -First 1)

$VsDevCmd
$Link
$Cl
```

Verificar `Test-Path` nos resultados.

Se `VsDevCmd.bat`, `link.exe` ou `cl.exe` não forem encontrados, parar e reportar. Não instalar componentes.

### 4.4 Preparar probe temporário

Usar somente `[System.IO.Path]::GetTempPath()`:

```powershell
$BaseTemp = [System.IO.Path]::GetTempPath()
$ProbeRoot = Join-Path $BaseTemp 'StepFlow-PoC\F1-B1-T04G'
$ProbeCargoHome = Join-Path $ProbeRoot 'cargo-home'
$CargoProbe = Join-Path $ProbeRoot 'cargo-probe'

New-Item -ItemType Directory -Force -Path $ProbeCargoHome | Out-Null
New-Item -ItemType Directory -Force -Path $CargoProbe | Out-Null
```

### 4.5 Rust/Cargo transitórios

```powershell
$ToolchainBin = 'C:\Users\Estudos\.rustup\toolchains\stable-x86_64-pc-windows-msvc\bin'
$OriginalPath = $env:PATH
$OriginalCargoHome = $env:CARGO_HOME
$OriginalRustupHome = $env:RUSTUP_HOME

$env:CARGO_HOME = $ProbeCargoHome
$env:PATH = "$ToolchainBin;$OriginalPath"

rustc --version
rustc -Vv
cargo --version
```

Confirmar host `x86_64-pc-windows-msvc`.

### 4.6 Criar projeto sem VCS e validar check

```powershell
Set-Location $CargoProbe
cargo init --bin --name stepflow_msvc_probe --vcs none .
cargo check --offline
```

Parar se qualquer etapa falhar.

### 4.7 Build em processo filho com VsDevCmd

```powershell
$CmdLine =
  "call `"$VsDevCmd`" -arch=amd64 -host_arch=amd64 && " +
  "set `"CARGO_HOME=$ProbeCargoHome`" && " +
  "set `"PATH=$ToolchainBin;%PATH%`" && " +
  "where link && " +
  "where cl && " +
  "where rustc && " +
  "where cargo && " +
  "cd /d `"$CargoProbe`" && " +
  "cargo build --offline"

cmd.exe /d /s /c $CmdLine
$BuildExitCode = $LASTEXITCODE
$BuildExitCode
```

Critério: `link`, `cl`, `rustc` e `cargo` localizados; `cargo build --offline` concluído; exit code 0.

Se faltar SDK/toolset ou for solicitado componente adicional, parar e reportar sem instalar nada.

### 4.8 Validar executável

Somente com exit code 0:

```powershell
$Exe = Join-Path $CargoProbe 'target\debug\stepflow_msvc_probe.exe'
Test-Path $Exe
Get-Item $Exe | Select-Object FullName,Length,LastWriteTime
Get-FileHash $Exe -Algorithm SHA256
& $Exe
$LASTEXITCODE
```

### 4.9 Restaurar sessão

Restaurar `PATH`, `CARGO_HOME` e `RUSTUP_HOME` aos valores originais. Nenhuma alteração permanente é permitida.

## 5. Proibições

- não instalar/modificar Visual Studio ou Build Tools;
- não abrir Visual Studio Installer para corrigir componentes;
- não alterar ACL/owner;
- não usar elevação;
- não alterar PATH User/Machine;
- não usar rede Cargo/npm/Git;
- não criar Tauri;
- não alterar o repositório oficial ou o diário;
- não commit/push;
- não executar F1-B1-T04.

## 6. Relatório final obrigatório

Responder com:

1. objetivo;
2. usuário/contexto;
3. HEAD/working tree;
4. resultado do vswhere sem `-prerelease`;
5. resultado do vswhere com `-prerelease`;
6. metadados da instalação encontrada;
7. resultado das duas consultas com `-requires`;
8. caminho de `VsDevCmd.bat`;
9. caminho de `link.exe`;
10. caminho de `cl.exe`;
11. versões/host Rust e Cargo;
12. resultado de `cargo init --vcs none`;
13. resultado de `cargo check --offline`;
14. parâmetros do `VsDevCmd.bat`;
15. resultado de `where link/cl/rustc/cargo` no processo filho;
16. resultado/exit code de `cargo build --offline`;
17. caminho/tamanho/SHA-256 do executável, se gerado;
18. smoke test, ausência de rede/elevação/instalação e restauração das variáveis;
19. conclusão sobre a causa da F1-B1-T04F e disponibilidade real do linker MSVC;
20. recomendação para retomar/adaptar F1-B1-T04.

## 7. Regra de parada

Parar sem ampliar escopo se a instalação não aparecer nem com `-prerelease`, se as ferramentas não existirem, se o ambiente Developer Command Prompt não localizar linker/compiler, se faltar SDK/toolset, se o build exigir componente adicional, elevação ou alteração persistente.
