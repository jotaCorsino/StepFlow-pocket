# Revisão — F1-B1-T04B Sandbox Git

**Data:** 2026-08-19  
**Status:** CONCLUÍDA / FLUXO OPERACIONAL ADAPTADO

## Objetivo

Revisar os resultados das tarefas F1-B1-T04A e F1-B1-T04B, classificar os bloqueios observados na sessão `EARTH\CodexSandboxOffline` e definir o fluxo mínimo seguro para retomar a prova Tauri F1-B1-T04.

## Evidências consolidadas

- a sessão Codex não elevada usa `EARTH\CodexSandboxOffline`;
- SID do usuário atual: `S-1-5-21-4274444639-1290711936-3977769617-1008`;
- o token inclui `EARTH\CodexSandboxUsers`, SID `S-1-5-21-4274444639-1290711936-3977769617-1007`;
- `.git` contém ACEs `DENY` explícitas para o SID desconhecido `S-1-5-21-1439205687-274586611-205630846-3092016610`;
- `FETCH_HEAD` herda a negação de `.git`;
- o SID negado não corresponde ao usuário atual nem a grupos presentes no token observado;
- o SID negado não pôde ser traduzido para conta Windows conhecida;
- não havia processo Git/SSH concorrente;
- `git pull --ff-only` havia falhado ao abrir `.git/FETCH_HEAD` com `Permission denied`;
- TCP para `github.com:443` funciona;
- `curl.exe -I https://github.com` e `git ls-remote origin HEAD` falham com `schannel: AcquireCredentialsHandle failed: SEC_E_NO_CREDENTIALS`;
- Git usa backend SSL `schannel`, credential helper `manager` e não possui proxy configurado;
- nenhuma ACL, owner, configuração, credencial ou arquivo foi alterado durante os diagnósticos;
- a F1-B1-T04 não foi iniciada.

## Classificação

### ACL

Existe uma negação real em `.git`, mas o SID negado não pertence ao token observado na sessão F1-B1-T04B. Portanto, os dados disponíveis não justificam remover a ACE automaticamente.

A correção de segurança do Windows não é necessária para prosseguir com o objetivo do projeto e poderia enfraquecer uma proteção específica do ambiente de execução.

### Git/HTTPS no sandbox

A conectividade TCP está disponível, mas a camada Schannel falha antes de uma consulta HTTPS útil. Como a falha ocorre tanto no `curl.exe` quanto no Git dentro da sessão `CodexSandboxOffline`, a sincronização Git via rede não deve ser considerada uma capacidade confiável dessa sessão.

Essa conclusão é operacional e limitada ao contexto observado; não é tratada como defeito do GitHub, do repositório ou da arquitetura do StepFlow.

## Decisão operacional

A partir desta revisão, enquanto o Codex estiver executando sob `EARTH\CodexSandboxOffline` com essas restrições:

1. o Codex não será responsável por executar `git pull` ou `git fetch` para sincronizar `C:\dev\StepFlow`;
2. o PO fará a sincronização do checkout em sua sessão Windows normal antes de tarefas que dependam do HEAD remoto atualizado;
3. o Codex verificará apenas localmente branch, HEAD, working tree e `git diff --check`;
4. o Codex não deve alterar ACLs, credenciais, certificados ou configuração Schannel para contornar o sandbox;
5. se o HEAD informado pelo PO não estiver presente localmente ou o working tree apresentar alterações não autorizadas, a tarefa deve parar.

## Preservação do diário local

Existe uma alteração local autorizada em:

`docs/05-progresso/diario-de-progresso.md`

A sincronização manual deve preservá-la. O comando preferido continua sendo `git pull --ff-only`, executado pelo PO em sua sessão Windows normal, somente se o Git puder fazer fast-forward sem exigir stash, merge, rebase, reset ou resolução manual.

## Resultado

As F1-B1-T04A e F1-B1-T04B estão concluídas.

Não está autorizada nenhuma remoção de ACL ou correção Schannel no sandbox.

A F1-B1-T04 pode ser retomada depois que o PO sincronizar manualmente o checkout e informar o HEAD resultante. A tarefa foi atualizada para remover a dependência de `git pull` executado pelo Codex.