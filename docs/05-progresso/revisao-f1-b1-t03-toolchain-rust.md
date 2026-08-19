# Revisão — F1-B1-T03 Preparação do Toolchain Rust

**Data:** 2026-08-19  
**Status:** CONCLUÍDA / APROVADA

## Objetivo

Revisar a execução da tarefa `F1-B1-T03 — Preparar Toolchain Rust para Prova Tauri` e determinar se o ambiente pessoal de desenvolvimento está preparado para a próxima prova descartável do StepFlow Client.

## Evidências recebidas

- `winget v1.29.280`;
- Rustup instalado pelo pacote oficial `Rustlang.Rustup`;
- `rustup 1.29.0 (28d1352db 2026-03-05)`;
- `rustc 1.97.1 (8bab26f4f 2026-07-14)`;
- `cargo 1.97.1 (c980f4866 2026-06-30)`;
- toolchain padrão e ativo `stable-x86_64-pc-windows-msvc`;
- host de `rustc -Vv`: `x86_64-pc-windows-msvc`;
- MSVC já estava presente antes da tarefa;
- nenhuma reinicialização foi necessária;
- nenhum `package.json`, `package-lock.json`, `src-tauri` ou código foi criado;
- alteração local autorizada em `docs/05-progresso/diario-de-progresso.md` foi preservada;
- nenhum commit ou push foi realizado pelo Codex.

## Avaliação

A instalação cumpriu o objetivo da tarefa. O toolchain Rust necessário para uma prova Tauri Windows x64/MSVC está presente e foi validado.

A primeira validação na sessão original não encontrou imediatamente os comandos pelo `PATH`, comportamento compatível com uma sessão aberta antes da atualização do ambiente. A validação posterior encontrou o toolchain existente e confirmou o host correto.

Isso não exige mudança de arquitetura nem instalação adicional.

Como proteção operacional, a próxima prova deverá começar em **nova sessão PowerShell não elevada** e verificar `rustup`, `rustc` e `cargo` antes de criar a PoC. Se os comandos não forem encontrados nessa condição normal, a prova deverá parar para diagnóstico localizado, sem executar scaffold como administrador.

## Relação com o princípio Pocket

Rust, Cargo, Node.js, npm, Visual Studio e MSVC pertencem ao **ambiente de desenvolvimento/build**.

A presença dessas ferramentas nesta estação não autoriza transformá-las em dependências do servidor corporativo nem das estações técnicas.

A implantação Pocket continua exigindo artefatos previamente construídos, baixo impacto no Windows e ausência de toolchain de desenvolvimento no servidor.

## Resultado

A F1-B1-T03 está **APROVADA**.

Não há dependência adicional conhecida para iniciar uma prova Tauri local descartável.

## Próximo passo autorizado

Executar uma prova mínima Tauri 2, fora da árvore oficial do repositório, com frontend Vanilla JavaScript e CLI local ao projeto.

A prova deverá:

- confirmar build Windows x64/MSVC;
- gerar o executável sem exigir instalador (`build --no-bundle`);
- copiar o executável resultante para uma pasta isolada;
- executar smoke test dessa cópia;
- verificar, em ambiente de processo com `PATH` sem Node/Rust, que o executável continua inicializando;
- registrar tamanho/hash e dependências/limitações observáveis;
- permanecer explicitamente descartável e fora do código oficial do StepFlow.

A prova não decide ainda o launcher, o Host ou a estratégia final de distribuição.