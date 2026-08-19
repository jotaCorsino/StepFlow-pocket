# Tarefa Codex F1-B1-T04A — Diagnosticar Permissão de `.git/FETCH_HEAD`

**Fase:** 1 — Fechamento arquitetural e especificação  
**Bloco:** 1 — Plataforma Windows, Client e distribuição  
**Tipo:** diagnóstico operacional local, somente leitura  
**Status:** CONCLUÍDA / DIAGNÓSTICO CONFIRMADO  
**Relação:** pré-condição corretiva para `F1-B1-T04 — Prova Mínima Tauri 2 com Critério Pocket`

## 1. Objetivo

Diagnosticar, sem modificar permissões ou metadados do repositório, por que uma sessão PowerShell não elevada não consegue executar `git pull --ff-only` em `C:\dev\StepFlow`, falhando com:

```text
error: cannot open '.git/FETCH_HEAD': Permission denied
```

## 2. Contexto

O clone local foi originalmente criado durante uma execução que chegou a usar contexto elevado. Existia, portanto, hipótese de diferença de owner/ACL em `.git`, sem assumir previamente que essa seria a causa.

O Git documenta que `git fetch` grava as refs obtidas em `.git/FETCH_HEAD`. O Windows permite inspecionar owner e ACLs com `Get-Acl` e `icacls`.

Fontes de referência:

- https://git-scm.com/docs/git-fetch
- https://learn.microsoft.com/powershell/module/microsoft.powershell.security/get-acl
- https://learn.microsoft.com/windows-server/administration/windows-commands/icacls

## 3. Estado inicial conhecido

- repositório: `C:\dev\StepFlow`;
- branch `main`;
- HEAD local: `dfb9f4b`;
- remoto já avançado além desse HEAD;
- somente a alteração local autorizada em `docs/05-progresso/diario-de-progresso.md`;
- `git diff --check` sem erros;
- a tentativa anterior de `git pull --ff-only` em PowerShell não elevado falhou ao abrir `.git/FETCH_HEAD`;
- a PoC F1-B1-T04 não foi criada.

## 4. Escopo executado

Somente inspeção/leitura:

- identidade/contexto da sessão;
- estado Git local;
- caminho do Git dir;
- atributos de root, `.git` e `FETCH_HEAD`;
- owner e ACLs;
- inspeção via `icacls`;
- processos Git/SSH;
- `git fetch --dry-run origin`.

Nenhuma ACL, owner, arquivo ou configuração foi modificada.

## 5. Resultado obtido

A sessão executora era:

```text
earth\codexsandboxoffline
```

O root `C:\dev\StepFlow`, `.git` e `.git\FETCH_HEAD` pertencem a `EARTH\Estudos`.

A diferença relevante encontrada foi:

- o root do repositório possui permissões `Allow` de modificação aplicáveis e não possui `DENY` explícito correspondente;
- `.git` e `.git\FETCH_HEAD` possuem `DENY` explícito para o SID:

```text
S-1-5-21-1439205687-274586611-205630846-3092016610
```

- o `DENY` inclui direitos de escrita/remoção, entre eles `Write`, `Delete`, `ReadPermissions` e `DeleteSubdirectoriesAndFiles`;
- nenhum processo Git/SSH concorrente estava ativo.

Portanto, existe **alta confiança** de que a ACE explícita de negação é a causa imediata da falha de escrita em `.git/FETCH_HEAD` para qualquer token ao qual esse SID se aplique.

Entretanto, a identidade exata desse SID ainda deve ser formalmente resolvida antes de autorizar qualquer alteração de ACL. Não se deve presumir que seja um erro de configuração, pois a sessão executora é uma conta de sandbox do Codex e a restrição pode ser deliberada.

## 6. Segundo bloqueio independente

O comando:

```text
git fetch --dry-run origin
```

falhou com:

```text
schannel: AcquireCredentialsHandle failed: SEC_E_NO_CREDENTIALS
```

Assim, além da impossibilidade de escrever em `.git`, existe um segundo bloqueio nessa sessão relacionado ao acesso HTTPS/Schannel.

Esse erro não deve ser automaticamente interpretado como problema de credencial do repositório ou do usuário real. É necessário primeiro determinar se a sessão `codexsandboxoffline` possui restrições próprias de rede/credenciais TLS.

## 7. Decisão operacional

**Não alterar ACLs ainda.**

Antes de qualquer `icacls /remove:d`, `Set-Acl`, mudança de owner ou outra ação de segurança, deve existir uma tarefa curta de classificação que:

1. resolva formalmente o SID negado para conta/grupo, se possível;
2. determine se o SID faz parte do token da sessão `codexsandboxoffline`;
3. confirme se a negação é específica do sandbox;
4. classifique o erro Schannel como limitação do sandbox ou problema geral do Git/Windows.

A F1-B1-T04 permanece bloqueada até essa classificação e eventual correção operacional.

## 8. Restrições preservadas

Durante a F1-B1-T04A não houve:

- `git pull` adicional;
- `git fetch` sem `--dry-run`;
- alteração de ACL;
- alteração de owner;
- remoção/recriação de `FETCH_HEAD`;
- elevação para contornar o problema;
- stash, merge, rebase, reset ou checkout destrutivo;
- alteração do diário;
- criação da PoC;
- commit ou push pelo Codex.

## 9. Próximo passo

Executar `F1-B1-T04B — Classificar SID e Restrições do Sandbox Git`, ainda em modo somente leitura.

Somente depois disso deverá ser decidido se a solução adequada é corrigir ACL, mudar o contexto usado para sincronização Git, ou tratar a restrição como característica operacional do sandbox.