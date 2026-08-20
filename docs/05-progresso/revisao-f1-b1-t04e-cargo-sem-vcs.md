# Revisão — F1-B1-T04E Cargo sem VCS

**Data:** 2026-08-20  
**Status:** CONCLUÍDA / TOOLCHAIN RUST-CARGO VALIDADO / LINKER MSVC PENDENTE

## Resultado consolidado

A F1-B1-T04E confirmou, em sessão PowerShell não elevada `EARTH\CodexSandboxOffline`:

- branch `main`, HEAD local autorizado `97c5f4a`;
- somente `docs/05-progresso/diario-de-progresso.md` modificado no repositório oficial;
- ambiente temporário gravável em `%TEMP%`;
- `CARGO_HOME` transitório funcional;
- `rustc 1.97.1` e `cargo 1.97.1`;
- host `x86_64-pc-windows-msvc`;
- `cargo init --bin --name stepflow_cargo_probe --vcs none .` concluído;
- nenhuma pasta `.git` criada;
- `cargo check --offline` concluído sem dependências externas e sem rede;
- `cargo build --offline` falhou porque `link.exe` não foi localizado;
- nenhuma elevação, instalação, ACL ou configuração permanente foi utilizada.

## Interpretação

O bloqueio de Rust/Cargo está encerrado. O sandbox consegue criar e verificar um projeto Rust mínimo sem VCS, usando o toolchain já instalado para o perfil de desenvolvimento e um `CARGO_HOME` transitório.

A falha restante ocorre na etapa de link do executável Windows e aponta para ausência do ambiente MSVC na sessão atual, não para falha de `rustc` ou Cargo.

A documentação da Microsoft estabelece que shells de desenvolvedor do Visual Studio configuram variáveis de ambiente necessárias para ferramentas de build, e que `VsDevCmd.bat` pode ser inicializado com arquitetura alvo/host apropriada. Essa configuração é transitória ao processo e não exige alteração global de PATH.

## Decisão

Antes de retomar a F1-B1-T04, executar uma prova mínima adicional para:

1. localizar a instalação Visual Studio/MSVC já existente;
2. localizar `VsDevCmd.bat` e `link.exe` sem instalar componentes;
3. inicializar um processo filho com ambiente Developer Command Prompt x64;
4. repetir `cargo build --offline` no projeto Rust mínimo;
5. confirmar que o executável é gerado sem elevação e sem alteração persistente.

Não está autorizada instalação/modificação do Visual Studio nesta etapa.
