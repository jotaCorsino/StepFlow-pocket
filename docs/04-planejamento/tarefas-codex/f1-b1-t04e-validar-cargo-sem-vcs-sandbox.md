# Tarefa Codex F1-B1-T04E — Validar Cargo sem VCS no Sandbox

**Fase:** 1 — Fechamento arquitetural e especificação  
**Bloco:** 1 — Plataforma Windows, Client e distribuição  
**Tipo:** prova técnica mínima, local e descartável  
**Status:** PRONTA PARA EXECUÇÃO  
**Relação:** continuação da F1-B1-T04D; último gate de toolchain antes de retomar F1-B1-T04

## 1. Objetivo

Confirmar que o Cargo consegue criar e validar um projeto Rust mínimo no sandbox quando a inicialização de VCS é explicitamente desativada com `--vcs none`, usando apenas:

- diretório temporário gravável;
- `CARGO_HOME` transitório;
- PATH transitório apontando para o toolchain existente;
- modo completamente offline.

A tarefa não cria a PoC Tauri.

## 2. Fundamentação

O Cargo documenta a opção `--vcs none` para impedir a criação de repositório de controle de versão em `cargo new`/`cargo init`. Isso permite separar a capacidade de compilação Rust da restrição do sandbox sobre diretórios `.git`.

Fontes primárias:

- https://doc.rust-lang.org/cargo/commands/cargo-init.html
- https://doc.rust-lang.org/cargo/commands/cargo-new.html
- https://doc.rust-lang.org/cargo/reference/config.html

## 3. Estado conhecido

- checkout local autorizado: `main`, HEAD `97c5f4a`;
- única modificação local autorizada: `docs/05-progresso/diario-de-progresso.md`;
- `C:\Users\Estudos\AppData\Local\Temp` é gravável no sandbox;
- toolchain existente em `C:\Users\Estudos\.rustup\toolchains\stable-x86_64-pc-windows-msvc\bin` é executável;
- `rustc 1.97.1`, `cargo 1.97.1`, host `x86_64-pc-windows-msvc`;
- `CARGO_HOME` transitório funciona;
- falha anterior ocorreu somente quando `cargo init` tentou criar `.git`.

## 4. Procedimento obrigatório

Executar em nova sessão PowerShell não elevada.

### 4.1 Gate local

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

### 4.2 Preparar diretório temporário

```powershell
$BaseTemp = [System.IO.Path]::GetTempPath()
$ProbeRoot = Join-Path $BaseTemp 'StepFlow-PoC\F1-B1-T04E'
$ProbeCargoHome = Join-Path $ProbeRoot 'cargo-home'
$CargoProbe = Join-Path $ProbeRoot 'cargo-probe'

New-Item -ItemType Directory -Force -Path $ProbeCargoHome | Out-Null
New-Item -ItemType Directory -Force -Path $CargoProbe | Out-Null
```

Não usar `C:\dev\StepFlow-PoC`.

### 4.3 Configuração transitória

```powershell
$ToolchainBin = 'C:\Users\Estudos\.rustup\toolchains\stable-x86_64-pc-windows-msvc\bin'

$OriginalPath = $env:PATH
$OriginalCargoHome = $env:CARGO_HOME
$OriginalRustupHome = $env:RUSTUP_HOME

$env:CARGO_HOME = $ProbeCargoHome
$env:PATH = "$ToolchainBin;$OriginalPath"
```

Não alterar `RUSTUP_HOME`.

Validar:

```powershell
rustc --version
rustc -Vv
cargo --version
```

Confirmar host `x86_64-pc-windows-msvc`.

### 4.4 Criar projeto sem VCS

```powershell
Set-Location $CargoProbe
cargo init --bin --name stepflow_cargo_probe --vcs none .
```

Depois verificar explicitamente que não existe `.git`:

```powershell
Test-Path (Join-Path $CargoProbe '.git')
Get-ChildItem -Force $CargoProbe
```

O resultado esperado para `Test-Path ...\.git` é `False`.

Se o comando ainda tentar criar VCS ou retornar erro relacionado a `.git`, parar e reportar.

### 4.5 Validar build offline

```powershell
cargo check --offline
```

Se passar, executar também:

```powershell
cargo build --offline
```

Registrar o caminho do executável gerado no `target\debug` e confirmar sua existência. Não é necessário executá-lo nesta tarefa.

Nenhuma dependência externa deve ser adicionada.

### 4.6 Restaurar variáveis

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

- não alterar ACL/owner;
- não alterar PATH User/Machine;
- não usar `setx`;
- não instalar/reinstalar Rust;
- não instalar crates;
- não usar rede Cargo/npm/Git;
- não criar Tauri;
- não executar npm;
- não modificar o repo oficial/diário;
- não commit/push;
- não executar F1-B1-T04.

## 6. Critérios de aceite

- `cargo init --vcs none` concluído;
- nenhum `.git` criado;
- `cargo check --offline` concluído;
- `cargo build --offline` concluído;
- executável Rust mínimo gerado;
- nenhuma rede, elevação ou mudança permanente necessária.

## 7. Relatório final obrigatório

Responder com:

1. objetivo executado;
2. usuário/contexto;
3. HEAD/branch/working tree;
4. caminho temporário usado;
5. CARGO_HOME transitório;
6. versões rustc/cargo e host;
7. comando exato de `cargo init`;
8. resultado de `cargo init --vcs none`;
9. confirmação de ausência de `.git`;
10. arquivos principais criados pelo Cargo;
11. resultado de `cargo check --offline`;
12. resultado de `cargo build --offline`;
13. caminho do executável Rust mínimo, se gerado;
14. confirmação de ausência de dependências externas;
15. confirmação de ausência de rede;
16. confirmação de ausência de elevação;
17. restauração das variáveis;
18. estado Git final;
19. conclusão objetiva sobre o bloqueio de toolchain local;
20. recomendação para retomar ou adaptar F1-B1-T04, sem executá-la.

## 8. Regra de parada

Parar sem ampliar escopo se `cargo init --vcs none` ainda tentar usar VCS, se `cargo check --offline`/`cargo build --offline` exigirem rede ou componente externo, ou se houver necessidade de elevação/ACL/configuração permanente.
