# Revisão — F1-B1-T04D CARGO_HOME temporário no sandbox

**Data:** 2026-08-20  
**Status:** CONCLUÍDA COM LIMITAÇÃO ESPECÍFICA DO VCS

## Objetivo

Revisar a execução da F1-B1-T04D e determinar se o ambiente transitório do sandbox está apto a usar Rust/Cargo sem alteração permanente do Windows.

## Evidências recebidas

- sessão não elevada: `earth\codexsandboxoffline`;
- branch `main`, HEAD local `97c5f4a`;
- única modificação local: `docs/05-progresso/diario-de-progresso.md`;
- `TEMP`, `TMP` e `GetTempPath()` apontam para `C:\Users\Estudos\AppData\Local\Temp`;
- diretório temporário em `...\Temp\StepFlow-PoC\F1-B1-T04D` é gravável;
- `CARGO_HOME` transitório em `...\cargo-home` foi criado e validado;
- `rustc 1.97.1`, `cargo 1.97.1` e host `x86_64-pc-windows-msvc` funcionaram via PATH transitório;
- nenhuma rede foi usada;
- nenhuma elevação foi usada;
- variáveis de ambiente foram restauradas;
- `cargo init` falhou ao tentar criar/acessar `cargo-probe\.git` com `Acesso negado`;
- `cargo check --offline` não foi executado devido à regra de parada.

## Interpretação

A tarefa demonstrou que o sandbox consegue:

1. gravar em diretório temporário padrão;
2. usar `CARGO_HOME` isolado e transitório;
3. executar o toolchain Rust instalado no perfil `Estudos`;
4. resolver `rustc` e `cargo` por nome com PATH alterado apenas na sessão.

O bloqueio restante não aponta para falha de Rust/Cargo. O `cargo init` tenta inicializar Git por padrão quando o diretório não está dentro de um VCS existente. O Cargo documenta que `--vcs none` desativa a inicialização de controle de versão.

Fontes primárias:

- https://doc.rust-lang.org/cargo/commands/cargo-init.html
- https://doc.rust-lang.org/cargo/commands/cargo-new.html
- https://doc.rust-lang.org/cargo/reference/config.html

## Resultado

A F1-B1-T04D está **CONCLUÍDA COM LIMITAÇÃO ESPECÍFICA DO VCS**.

Não há justificativa para alterar ACLs, PATH permanente ou reinstalar Rust.

## Próximo passo

Executar prova mínima adicional usando explicitamente `cargo init --vcs none`, no mesmo modelo de CARGO_HOME/PATH transitórios e diretório temporário gravável. Se `cargo init --vcs none` e `cargo check --offline` passarem, considerar encerrado o bloqueio do toolchain local para fins da PoC Tauri.
