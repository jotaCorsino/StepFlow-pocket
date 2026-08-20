# Gate A — PoC Pocket do StepFlow Host em Rust

**Fase:** 1  
**Bloco:** 2 — StepFlow Host  
**Status:** PRONTO PARA EXECUÇÃO PELO PO  
**Ambiente:** sessão Windows normal `EARTH\Estudos`, não elevada  
**Local:** fora do repositório oficial

## Objetivo

Validar em uma única prova descartável que a direção Rust para o StepFlow Host pode produzir um artefato Windows compatível com o princípio Pocket.

## Escopo

A PoC deve provar:

- HTTP local funcional;
- endpoint `/health`;
- SQLite embutido via `rusqlite`/`bundled`;
- criação de arquivo SQLite em pasta de dados local à implantação;
- escrita/leitura mínima;
- shutdown controlado por endpoint de teste;
- build release x64;
- cópia isolada somente do executável;
- execução da cópia sem Node/npm/Rust/Cargo no PATH;
- nenhuma instalação no runtime;
- tamanho e SHA-256 do executável.

## Fora de escopo

- serviço Windows;
- registro/Task Scheduler/startup;
- bind em LAN;
- firewall;
- WebSocket definitivo;
- autenticação;
- concorrência definitiva;
- schema oficial;
- migrations oficiais;
- backup/restore;
- implementação dentro do repositório StepFlow.

## Critério

Se a PoC passar, Rust permanece a direção preferida do Host e o Bloco 2 avança para modelo de execução, paths, configuração, logs, diagnóstico e atualização.

Se a PoC falhar por dependência/runtime incompatível com Pocket, interromper e comparar formalmente a alternativa .NET self-contained antes de qualquer nova sequência de microprobes.
