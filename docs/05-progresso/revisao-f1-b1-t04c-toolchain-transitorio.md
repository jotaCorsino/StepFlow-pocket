# Revisão — F1-B1-T04C Toolchain Rust Transitório

**Data:** 2026-08-19  
**Status:** PARCIALMENTE CONCLUSIVA / BLOQUEIO DE DIRETÓRIO DESCARTÁVEL

## Objetivo

Revisar a tentativa de uso transitório do toolchain Rust já instalado no perfil `EARTH\Estudos` pela sessão não elevada `EARTH\CodexSandboxOffline`, sem reinstalação nem alteração permanente do Windows.

## Evidências

- sessão não elevada sob `EARTH\CodexSandboxOffline`;
- checkout local permaneceu em `main`, HEAD `97c5f4a`, com somente `docs/05-progresso/diario-de-progresso.md` modificado;
- `USERPROFILE` observado na sessão sandbox: `C:\Users\Estudos`;
- `CARGO_HOME` e `RUSTUP_HOME` já possuíam valores transitórios próprios do runtime Codex;
- toolchain `C:\Users\Estudos\.rustup\toolchains\stable-x86_64-pc-windows-msvc` localizado;
- `rustc.exe`, `cargo.exe` e `rustdoc.exe` acessíveis sem `Access denied`;
- `rustc 1.97.1` executou por caminho absoluto;
- host confirmado: `x86_64-pc-windows-msvc`;
- `cargo 1.97.1` executou por caminho absoluto;
- tentativa de criar `C:\dev\StepFlow-PoC\F1-B1-T04C\cargo-home` falhou com acesso negado;
- nenhuma variável de ambiente foi alterada, pois a regra de parada foi aplicada antes da configuração transitória;
- nenhum arquivo/pasta da PoC Tauri foi criado.

## Interpretação

A limitação observada não invalida o toolchain Rust. A sessão sandbox consegue ler e executar diretamente os binários do toolchain instalado no perfil de desenvolvimento.

O bloqueio está no caminho escolhido para dados descartáveis: a identidade sandbox não possui escrita em `C:\dev\StepFlow-PoC`.

Não há justificativa para alterar ACL de `C:\dev` somente para acomodar a prova. O Cargo permite redirecionar seu diretório de cache/home pela variável de ambiente `CARGO_HOME`, portanto a próxima validação deve utilizar um diretório temporário já gravável pela sessão sandbox.

Fontes primárias:

- https://doc.rust-lang.org/cargo/guide/cargo-home.html
- https://doc.rust-lang.org/cargo/reference/environment-variables.html

## Resultado

A F1-B1-T04C está **PARCIALMENTE CONCLUSIVA**:

- uso direto do toolchain por caminho absoluto: **VALIDADO**;
- host MSVC x64: **VALIDADO**;
- CARGO_HOME transitório em `C:\dev`: **NÃO VIÁVEL NO SANDBOX**;
- mudança permanente de PATH/ACL: **NÃO AUTORIZADA E NÃO NECESSÁRIA**.

## Próximo passo

Executar prova pequena para localizar um diretório temporário gravável pelo sandbox e validar nele `CARGO_HOME` + PATH apenas durante a sessão. A F1-B1-T04 continua bloqueada até essa pré-condição ser concluída.