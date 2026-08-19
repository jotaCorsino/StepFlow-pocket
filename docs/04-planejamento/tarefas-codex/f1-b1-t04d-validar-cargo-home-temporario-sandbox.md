# Tarefa Codex F1-B1-T04D — Validar CARGO_HOME Temporário Gravável no Sandbox

**Fase:** 1 — Fechamento arquitetural e especificação  
**Bloco:** 1 — Plataforma Windows, Client e distribuição  
**Tipo:** diagnóstico operacional local e configuração transitória de sessão  
**Status:** PRONTA PARA EXECUÇÃO  
**Relação:** continuação da F1-B1-T04C; pré-condição para retomar F1-B1-T04

## 1. Objetivo

Validar, sem alterar ACLs nem configuração permanente do Windows, se a sessão `EARTH\CodexSandboxOffline` consegue usar o toolchain Rust já validado com um `CARGO_HOME` temporário em diretório já gravável pelo sandbox.

A tarefa não cria a PoC Tauri.

## 2. Estado conhecido

- checkout local autorizado em `main`, HEAD `97c5f4a`;
- única modificação local autorizada: `docs/05-progresso/diario-de-progresso.md`;
- `rustc.exe` e `cargo.exe` acessíveis diretamente em `C:\Users\Estudos\.rustup\toolchains\stable-x86_64-pc-windows-msvc\bin`;
- host `x86_64-pc-windows-msvc` confirmado;
- `C:\dev\StepFlow-PoC` não é gravável pela identidade sandbox;
- nenhuma ACL deve ser alterada para contornar isso.

## 3. Fundamentação

Cargo permite alterar seu diretório home/cache através da variável `CARGO_HOME`. Por padrão ele usa `%USERPROFILE%\.cargo` no Windows, mas o caminho pode ser redirecionado por variável de ambiente.

Fontes:

- https://doc.rust-lang.org/cargo/guide/cargo-home.html
- https://doc.rust-lang.org/cargo/reference/environment-variables.html

## 4. Procedimento

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

### 4.2. Registrar diretórios temporários disponíveis

```powershell
whoami
$env:TEMP
$env:TMP
[System.IO.Path]::GetTempPath()
```

Montar candidatos somente abaixo desses diretórios temporários. Não usar `C:\dev`.

Exemplo conceitual:

```powershell
$BaseTemp = [System.IO.Path]::GetTempPath()
$ProbeRoot = Join-Path $BaseTemp 'StepFlow-PoC\F1-B1-T04D'
$ProbeCargoHome = Join-Path $ProbeRoot 'cargo-home'
```

### 4.3. Validar escrita no temporário

```powershell
New-Item -ItemType Directory -Force -Path $ProbeCargoHome | Out-Null
$WriteProbe = Join-Path $ProbeCargoHome 'write-probe.tmp'
Set-Content -Path $WriteProbe -Value 'ok' -NoNewline
Get-Item $WriteProbe | Select-Object FullName,Length
Remove-Item $WriteProbe
```

Se o primeiro candidato falhar, testar no máximo os caminhos retornados por `$env:TEMP`, `$env:TMP` e `[System.IO.Path]::GetTempPath()`, sem inventar outros diretórios nem alterar permissões.

Se nenhum for gravável, parar e reportar.

### 4.4. Toolchain

Definir:

```powershell
$ToolchainBin = 'C:\Users\Estudos\.rustup\toolchains\stable-x86_64-pc-windows-msvc\bin'
```

Validar novamente por caminho absoluto:

```powershell
& (Join-Path $ToolchainBin 'rustc.exe') --version
& (Join-Path $ToolchainBin 'rustc.exe') -Vv
& (Join-Path $ToolchainBin 'cargo.exe') --version
```

### 4.5. Configuração somente da sessão

Salvar:

```powershell
$OriginalPath = $env:PATH
$OriginalCargoHome = $env:CARGO_HOME
$OriginalRustupHome = $env:RUSTUP_HOME
```

Depois, somente no processo atual:

```powershell
$env:CARGO_HOME = $ProbeCargoHome
$env:PATH = "$ToolchainBin;$OriginalPath"
```

Não alterar `RUSTUP_HOME`.

Validar:

```powershell
Get-Command rustc
Get-Command cargo
rustc --version
rustc -Vv
cargo --version
```

Confirmar host `x86_64-pc-windows-msvc`.

### 4.6. Smoke test local de Cargo sem rede

Criar projeto Rust mínimo apenas no mesmo diretório temporário gravável, sem dependências externas:

```powershell
$CargoProbe = Join-Path $ProbeRoot 'cargo-probe'
New-Item -ItemType Directory -Force -Path $CargoProbe | Out-Null
Set-Location $CargoProbe
cargo init --bin --name stepflow_cargo_probe .
cargo check --offline
```

O objetivo é confirmar que Cargo consegue criar/usar metadados e invocar rustc nesse ambiente temporário. Não adicionar crates/dependências.

Se `cargo check --offline` tentar buscar rede ou exigir componente adicional, parar e reportar.

### 4.7. Restaurar variáveis

Restaurar somente a sessão:

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

Não é necessário apagar a pasta temporária durante a tarefa; ela pode permanecer como evidência descartável.

## 5. Proibições

- não alterar ACL/owner;
- não usar `C:\dev\StepFlow-PoC`;
- não alterar PATH User/Machine;
- não usar `setx`;
- não instalar/reinstalar Rust;
- não instalar crates adicionais;
- não usar rede Cargo/npm/Git;
- não criar Tauri;
- não executar npm;
- não modificar o repo oficial/diário;
- não commit/push;
- não executar F1-B1-T04.

## 6. Critérios de aceite

- diretório temporário do sandbox identificado e gravável;
- CARGO_HOME temporário criado nesse diretório;
- toolchain existente executável;
- PATH temporário permite `rustc`/`cargo` por nome;
- host MSVC x64 confirmado;
- `cargo init` e `cargo check --offline` funcionam em projeto sem dependências;
- nenhuma mudança permanente de ambiente ou segurança.

## 7. Relatório final obrigatório

Responder com:

1. objetivo;
2. usuário/contexto;
3. HEAD/working tree;
4. valores TEMP/TMP/GetTempPath;
5. candidato temporário escolhido;
6. resultado do teste de escrita;
7. toolchain/versões absolutas;
8. CARGO_HOME transitório usado;
9. resultado de Get-Command rustc/cargo;
10. versões/host no PATH transitório;
11. caminho do cargo-probe;
12. resultado de cargo init;
13. resultado de cargo check --offline;
14. necessidade ou não de rede;
15. necessidade ou não de elevação;
16. restauração das variáveis;
17. arquivos/pastas temporários criados;
18. estado Git final;
19. conclusão sobre viabilidade do ambiente transitório;
20. recomendação para retomar/adaptar F1-B1-T04.

## 8. Regra de parada

Parar sem ampliar escopo se nenhum diretório temporário padrão for gravável, se Cargo/rustc não funcionarem com o ambiente transitório, se houver necessidade de elevação/ACL, ou se `cargo check --offline` exigir rede/componente não previsto.
