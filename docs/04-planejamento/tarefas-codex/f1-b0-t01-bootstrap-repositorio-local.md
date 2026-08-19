# F1-B0-T01 — Bootstrap do Repositório Local

**Fase:** 1 — Fechamento arquitetural e especificação  
**Bloco:** 0 — Preparação do ambiente de trabalho  
**Status:** CONCLUÍDA EM 2026-08-19

## 1. Objetivo

Preparar `C:\dev\StepFlow` como cópia local íntegra do repositório oficial `jotaCorsino/StepFlow-pocket`, sem iniciar qualquer implementação do produto.

Esta tarefa existe antes de todas as demais tarefas Codex da Fase 1.

## 2. Contexto

O repositório remoto já havia sido inicializado e continha a documentação oficial do projeto. O ambiente local ainda não havia sido preparado.

Repositório remoto:

`https://github.com/jotaCorsino/StepFlow-pocket.git`

Pasta local oficial:

`C:\dev\StepFlow`

Branch principal:

`main`

## 3. Resultado executado

- Git detectado: `git version 2.55.0.windows.4`;
- `C:\dev\StepFlow` já existia e estava completamente vazia;
- clone do repositório oficial realizado com sucesso;
- branch atual: `main`;
- `origin` configurado para `https://github.com/jotaCorsino/StepFlow-pocket.git` em fetch e push;
- HEAD validado no momento do bootstrap: `39015c0 docs: add local bootstrap as first Phase 1 gate`;
- working tree validado como limpo e sincronizado com `origin/main`;
- `README.md`, `AGENTS.md` e `docs/` confirmados;
- `AGENTS.md` lido pelo Codex;
- nenhuma dependência, código, banco, pacote ou scaffold criado;
- nenhum commit ou push executado pelo Codex.

## 4. Ocorrência registrada

A primeira tentativa de clone falhou com erro HTTPS:

`schannel: AcquireCredentialsHandle failed: SEC_E_NO_CREDENTIALS`

O clone foi repetido em contexto com permissão elevada e concluído com sucesso.

A ocorrência não bloqueou o bootstrap, mas deve ser considerada em tarefas futuras que dependam de acesso HTTPS/GitHub a partir deste ambiente.

## 5. Critérios de aceite

- [x] Git disponível no ambiente;
- [x] `C:\dev\StepFlow` corresponde ao repositório oficial;
- [x] branch atual é `main`;
- [x] `origin` aponta para `https://github.com/jotaCorsino/StepFlow-pocket.git`;
- [x] `README.md` existe;
- [x] `AGENTS.md` existe;
- [x] `docs/` existe;
- [x] `git status` sem alterações locais inesperadas;
- [x] nenhuma dependência de aplicação instalada;
- [x] nenhum arquivo preexistente apagado para forçar o clone.

## 6. Gate

**APROVADO.**

O ambiente local está autorizado a seguir para `F1-B1-T01 — Inventário do Ambiente Windows e Pré-requisitos`.

## 7. Observação operacional

Autenticação de escrita no GitHub ainda não foi configurada nem validada. Ela não é requisito para as tarefas de inspeção local atuais e deverá ser tratada separadamente antes da primeira tarefa que exija `push` pelo ambiente local.