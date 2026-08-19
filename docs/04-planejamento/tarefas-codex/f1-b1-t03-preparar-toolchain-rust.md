# Tarefa Codex F1-B1-T03 — Preparar Toolchain Rust para Prova Tauri

**Fase:** 1 — Fechamento arquitetural e especificação  
**Bloco:** 1 — Plataforma Windows, Client e distribuição  
**Tipo:** preparação controlada de ambiente de desenvolvimento  
**Status:** CONCLUÍDA EM 2026-08-19

## Fechamento

A execução foi revisada e aprovada em `docs/05-progresso/revisao-f1-b1-t03-toolchain-rust.md`.

O toolchain `stable-x86_64-pc-windows-msvc` foi instalado e validado. A próxima prova deve iniciar em uma nova sessão PowerShell não elevada e confirmar o acesso normal a `rustup`, `rustc` e `cargo` antes de qualquer scaffold.

## 1. Objetivo

Instalar e validar somente o toolchain Rust necessário para uma futura prova Tauri no computador pessoal usado como ambiente de desenvolvimento do StepFlow.

Esta tarefa não cria projeto Tauri, não cria código e não transforma a preparação do ambiente em implementação de produto.

## 2. Fundamentação técnica

A investigação anterior confirmou que esta estação já possui:

- Windows 11 Pro 25H2 x64;
- WebView2;
- Microsoft C++/MSVC;
- Git;
- Node.js/npm.

Rust/rustup/cargo não estavam instalados.

A documentação oficial atual do Tauri 2 exige Rust para desenvolvimento e recomenda `rustup`. No Windows, o Tauri recomenda o host/toolchain MSVC e documenta `winget install --id Rustlang.Rustup` como método de instalação suportado.

Fontes primárias:

- https://v2.tauri.app/start/prerequisites/
- https://www.rust-lang.org/tools/install

## 3. Contexto e documentos obrigatórios

Antes de executar, ler:

1. `AGENTS.md`;
2. `docs/00-governanca/contexto-ambientes.md`;
3. `docs/00-governanca/politica-capacidade-codex.md`;
4. `docs/04-planejamento/plano-oficial-fase-1.md`;
5. `docs/03-arquitetura/compatibilidade-windows-client.md`;
6. `docs/05-progresso/revisao-f1-b1-t01-inventario-ambiente.md`;
7. `docs/04-planejamento/tarefas-codex/f1-b1-t02-confirmar-identidade-windows.md`.

## 4. Estado inicial esperado

- repositório oficial em `C:\dev\StepFlow`;
- branch `main`;
- existe uma alteração local autorizada em `docs/05-progresso/diario-de-progresso.md`, proveniente de tarefa anterior;
- Rust/rustup/cargo ausentes;
- Visual Studio/MSVC já presentes;
- nenhum `package.json`;
- nenhum `src-tauri`;
- nenhum código de produto.

A alteração local do diário deve ser preservada integralmente.

## 5. Escopo incluído

- atualizar documentação local por `git pull --ff-only` somente se o Git puder preservar a alteração local existente sem intervenção adicional;
- verificar se `winget` está disponível;
- instalar `Rustlang.Rustup` pelo `winget`, preferencialmente usando a fonte oficial configurada no Windows;
- permitir que `rustup` instale/configure o toolchain Rust stable para MSVC;
- garantir que o host/toolchain usado seja compatível com `x86_64-pc-windows-msvc`;
- abrir/usar uma nova sessão de terminal se necessário para atualização de PATH;
- validar `rustup`, `rustc` e `cargo`;
- registrar versões e host/target detectados;
- validar que nenhum scaffold ou código foi criado.

## 6. Método de instalação preferido

Primeiro verificar:

```powershell
winget --version
```

Se disponível, usar:

```powershell
winget install --id Rustlang.Rustup --exact
```

Aceitar somente prompts normais necessários à instalação do pacote oficial.

Depois da instalação, garantir o toolchain stable MSVC conforme a documentação do Tauri:

```powershell
rustup default stable-msvc
```

Se a sessão atual ainda não reconhecer os binários após a instalação, abrir uma nova sessão de PowerShell ou atualizar apenas o ambiente da sessão de forma segura. Não alterar PATH manualmente de forma permanente além do que o instalador oficial do rustup fizer.

### Fallback

Se `winget` não estiver disponível, a instalação **não deve ser improvisada**.

Parar essa parte e reportar que o método preferido não está disponível. O Assistente decidirá separadamente se será autorizado usar `rustup-init.exe` obtido de `rust-lang.org`.

## 7. Validações obrigatórias

Após a instalação:

```powershell
rustup --version
rustc --version
cargo --version
rustup show
rustc -Vv
```

Confirmar especialmente:

- toolchain `stable` instalado;
- host compatível com `x86_64-pc-windows-msvc`;
- `rustc` acessível;
- `cargo` acessível.

Se o host não for MSVC, não criar projeto e não tentar contornar silenciosamente. Reportar o problema.

## 8. Estado Git

Antes de qualquer instalação:

```powershell
Set-Location C:\dev\StepFlow
git status --short --branch
git diff --check
git pull --ff-only
```

O `git pull --ff-only` só pode prosseguir se preservar a alteração local autorizada no diário.

Se o Git exigir stash, merge, rebase, reset, checkout destrutivo ou qualquer resolução manual, **parar e reportar**.

Ao final:

```powershell
Set-Location C:\dev\StepFlow
git status --short --branch
git diff --check
```

A única alteração esperada no repositório continua sendo a alteração local já existente em `docs/05-progresso/diario-de-progresso.md`.

## 9. Fora do escopo

É proibido nesta tarefa:

- executar `npm create tauri-app`;
- executar `cargo create-tauri-app`;
- instalar Tauri CLI;
- instalar pacotes npm do StepFlow;
- criar `package.json`;
- criar `package-lock.json`;
- criar `src-tauri`;
- criar arquivos HTML/CSS/JavaScript/Rust do produto;
- criar banco SQLite;
- criar Host;
- criar launcher;
- alterar configurações corporativas/rede/SMB;
- testar qualquer endereço da empresa;
- instalar targets móveis;
- instalar toolchains Rust adicionais sem necessidade desta tarefa;
- modificar documentação do projeto;
- fazer commit;
- fazer push.

## 10. Critérios de aceite

- [x] repositório atualizado com segurança ou bloqueio Git reportado antes da instalação;
- [x] `winget` verificado;
- [x] Rust instalado pelo método autorizado;
- [x] `rustup` funcional;
- [x] `rustc` funcional;
- [x] `cargo` funcional;
- [x] toolchain stable MSVC confirmado;
- [x] host x64/MSVC confirmado;
- [x] nenhum scaffold Tauri criado;
- [x] nenhum código de produto criado;
- [x] alteração local anterior do diário preservada;
- [x] nenhum commit/push realizado.

## 11. Relatório final obrigatório

Responder com:

1. objetivo executado;
2. estado Git inicial e resultado do `git pull --ff-only`;
3. versão do `winget`;
4. comando/método de instalação efetivamente usado;
5. resultado da instalação;
6. `rustup --version`;
7. `rustc --version`;
8. `cargo --version`;
9. resumo relevante de `rustup show`;
10. host retornado por `rustc -Vv`;
11. confirmação de `x86_64-pc-windows-msvc`/MSVC;
12. estado Git final;
13. confirmação de ausência de `package.json`, `src-tauri` e código novo;
14. erros, prompts, reinicializações ou limitações encontradas;
15. recomendação objetiva para a próxima tarefa, sem executá-la.

## 12. Regra de parada

Parar e reportar, sem improvisar, se:

- o repositório não puder ser atualizado com segurança;
- `winget` não estiver disponível;
- a instalação solicitar origem/pacote inesperado;
- o instalador exigir alteração não prevista além do toolchain Rust;
- o toolchain final não for MSVC;
- houver erro que exija instalar componentes adicionais não previstos.

Não executar a prova Tauri nesta tarefa, mesmo que a instalação de Rust seja concluída com sucesso.