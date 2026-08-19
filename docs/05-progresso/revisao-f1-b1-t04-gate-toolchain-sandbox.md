# Revisão — F1-B1-T04 Gate de Toolchain no Sandbox

**Data:** 2026-08-19  
**Status:** BLOQUEIO CLASSIFICADO / CORREÇÃO DE PREMISSA

## Objetivo

Revisar a parada da F1-B1-T04 no gate inicial de toolchain e determinar se o resultado indica instalação incorreta do Rust ou diferença de perfil entre o usuário real de desenvolvimento e a conta usada pelo Codex sandbox.

## Evidências recebidas

- checkout oficial em `C:\dev\StepFlow` permaneceu em `main`, HEAD `97c5f4a`;
- única alteração local continua sendo `docs/05-progresso/diario-de-progresso.md`;
- Node `v24.14.0` e npm `11.9.0` funcionam na sessão não elevada do Codex;
- `rustup`, `rustc` e `cargo` não são encontrados no `PATH` normal da sessão `EARTH\CodexSandboxOffline`;
- a PoC não foi criada;
- nenhuma elevação foi usada para contornar o gate.

A F1-B1-T03 havia confirmado que o Rust foi instalado para o ambiente de desenvolvimento do usuário `EARTH\Estudos`, com Rustup home observado em `C:\Users\Estudos\.rustup` e toolchain `stable-x86_64-pc-windows-msvc`.

## Fundamentação

A documentação oficial do rustup informa que, no Windows, os executáveis gerenciados pelo rustup ficam por padrão em `%USERPROFILE%\.cargo\bin`, e que `RUSTUP_HOME` usa por padrão `%USERPROFILE%\.rustup`.

Logo, uma sessão executada por outro usuário Windows não deve ser presumida como capaz de localizar automaticamente o toolchain instalado no perfil de `EARTH\Estudos`.

Fontes primárias:

- https://rust-lang.github.io/rustup/installation/
- https://rust-lang.github.io/rustup/environment-variables.html
- https://doc.rust-lang.org/cargo/guide/cargo-home.html

## Correção da premissa

O gate anterior exigia que o Codex sandbox encontrasse `rustup`, `rustc` e `cargo` pelo `PATH` próprio da conta sandbox.

Esse requisito é excessivo para o objetivo real do projeto.

O que precisa ser validado é se a sessão Codex consegue **consumir o toolchain já instalado no perfil de desenvolvimento**, sem:

- reinstalar Rust;
- alterar PATH permanente;
- alterar o perfil `EARTH\Estudos`;
- alterar segurança do Windows;
- elevar privilégios;
- transformar Rust em dependência de runtime do produto.

## Estratégia de validação

Antes de retomar a PoC, executar tarefa separada e curta para:

1. localizar o toolchain existente em `C:\Users\Estudos\.rustup\toolchains`;
2. confirmar acesso de leitura/execução da conta sandbox ao `rustc.exe` e `cargo.exe` do toolchain `stable-x86_64-pc-windows-msvc`;
3. criar apenas uma pasta descartável de `CARGO_HOME` fora do repositório oficial;
4. adicionar temporariamente, somente à sessão atual, o diretório `bin` do toolchain ao `PATH` do processo;
5. validar `rustc`, `cargo` e host MSVC nesse ambiente transitório;
6. restaurar/encerrar a sessão sem qualquer mudança persistente.

A prova deve preferir os binários reais do toolchain, não depender dos proxies do rustup instalados em `C:\Users\Estudos\.cargo\bin`.

## Resultado

A F1-B1-T04 permanece bloqueada apenas até a validação transitória do toolchain compartilhado entre o perfil de desenvolvimento e a sessão sandbox.

Não está autorizada nenhuma alteração permanente de PATH nem reinstalação de Rust para `CodexSandboxOffline`.

O resultado atual não invalida a F1-B1-T03 nem demonstra problema com a instalação Rust de `EARTH\Estudos`.