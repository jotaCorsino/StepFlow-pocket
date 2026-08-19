# Tarefa Codex F1-B1-T04A — Diagnosticar Permissão de `.git/FETCH_HEAD`

**Fase:** 1 — Fechamento arquitetural e especificação  
**Bloco:** 1 — Plataforma Windows, Client e distribuição  
**Tipo:** diagnóstico operacional local, somente leitura  
**Status:** PRONTA PARA EXECUÇÃO  
**Relação:** pré-condição corretiva para `F1-B1-T04 — Prova Mínima Tauri 2 com Critério Pocket`

## 1. Objetivo

Diagnosticar, sem modificar permissões ou metadados do repositório, por que uma sessão PowerShell não elevada não consegue executar `git pull --ff-only` em `C:\dev\StepFlow`, falhando com:

```text
error: cannot open '.git/FETCH_HEAD': Permission denied
```

A F1-B1-T04 não pode ser iniciada até que a causa seja entendida e a operação normal do Git em sessão não elevada seja restaurada em tarefa posterior explicitamente autorizada.

## 2. Contexto

O clone local foi originalmente criado durante uma execução que chegou a usar contexto elevado. Existe, portanto, hipótese de diferença de owner/ACL em `.git`, mas isso não deve ser assumido como causa sem evidência.

O Git documenta que `git fetch` grava as refs obtidas em `.git/FETCH_HEAD`. O Windows permite inspecionar owner e ACLs com `Get-Acl` e `icacls`.

Fontes de referência:

- https://git-scm.com/docs/git-fetch
- https://learn.microsoft.com/powershell/module/microsoft.powershell.security/get-acl
- https://learn.microsoft.com/windows-server/administration/windows-commands/icacls

## 3. Estado inicial conhecido

- repositório: `C:\dev\StepFlow`;
- branch esperada: `main`;
- HEAD local conhecido antes do bloqueio: `dfb9f4b`;
- remoto já avançou além desse HEAD;
- existe somente a alteração local autorizada em `docs/05-progresso/diario-de-progresso.md`;
- `git diff --check` passou;
- a tentativa de `git pull --ff-only` em PowerShell não elevado falhou ao abrir `.git/FETCH_HEAD`;
- a PoC F1-B1-T04 não foi criada.

## 4. Escopo incluído

Somente inspeção/leitura:

- confirmar usuário e contexto não elevado;
- confirmar repositório, branch, HEAD e remote sem executar pull;
- localizar o diretório Git efetivo;
- verificar existência, atributos, owner e ACL de:
  - `C:\dev\StepFlow`;
  - `C:\dev\StepFlow\.git`;
  - `C:\dev\StepFlow\.git\FETCH_HEAD`, se existir;
- comparar as ACLs do root do repositório, `.git` e `FETCH_HEAD`;
- procurar entradas `DENY` explícitas ou ausência de permissão de escrita/modificação para o usuário/grupos aplicáveis;
- listar processos Git relacionados que possam estar ativos;
- executar somente um `git fetch --dry-run origin` para verificar comunicação com o remoto sem escrever `FETCH_HEAD`;
- apresentar hipótese causal baseada nas evidências.

## 5. Fora do escopo

É proibido nesta tarefa:

- executar `git pull` novamente;
- executar `git fetch` sem `--dry-run`;
- usar `icacls` com `/grant`, `/reset`, `/setowner`, `/remove`, `/inheritance` ou qualquer opção modificadora;
- usar `Set-Acl`;
- alterar owner;
- alterar atributos de arquivos;
- deletar ou recriar `FETCH_HEAD`;
- executar PowerShell elevado para contornar o problema;
- fazer stash, merge, rebase, reset ou checkout destrutivo;
- reclonar o repositório;
- editar ou criar arquivos dentro de `.git`;
- modificar o diário ou qualquer documento local;
- criar a PoC Tauri;
- fazer commit ou push.

## 6. Procedimento obrigatório

Executar em **nova sessão PowerShell não elevada**.

### 6.1. Identidade e estado do checkout

```powershell
whoami
Set-Location C:\dev\StepFlow
git status --short --branch
git diff --check
git rev-parse --show-toplevel
git rev-parse --git-dir
git branch --show-current
git log -1 --oneline
git remote -v
```

### 6.2. Existência e atributos

```powershell
Get-Item C:\dev\StepFlow | Format-List FullName,Attributes,CreationTime,LastWriteTime
Get-Item C:\dev\StepFlow\.git -Force | Format-List FullName,Attributes,CreationTime,LastWriteTime

if (Test-Path C:\dev\StepFlow\.git\FETCH_HEAD) {
  Get-Item C:\dev\StepFlow\.git\FETCH_HEAD -Force |
    Format-List FullName,Attributes,Length,CreationTime,LastWriteTime
  attrib C:\dev\StepFlow\.git\FETCH_HEAD
}

attrib C:\dev\StepFlow\.git
```

### 6.3. Owner e ACL via PowerShell

```powershell
Get-Acl C:\dev\StepFlow |
  Format-List Path,Owner,AccessToString,Sddl

Get-Acl C:\dev\StepFlow\.git |
  Format-List Path,Owner,AccessToString,Sddl

if (Test-Path C:\dev\StepFlow\.git\FETCH_HEAD) {
  Get-Acl C:\dev\StepFlow\.git\FETCH_HEAD |
    Format-List Path,Owner,AccessToString,Sddl
}
```

### 6.4. ACL via `icacls` — somente exibição

```powershell
icacls C:\dev\StepFlow
icacls C:\dev\StepFlow\.git

if (Test-Path C:\dev\StepFlow\.git\FETCH_HEAD) {
  icacls C:\dev\StepFlow\.git\FETCH_HEAD
}
```

Não usar opções modificadoras.

### 6.5. Processos potencialmente relacionados

```powershell
Get-Process git*,ssh* -ErrorAction SilentlyContinue |
  Select-Object Id,ProcessName,Path,StartTime
```

A mera existência de processo não prova lock; registrar apenas como evidência.

### 6.6. Teste de comunicação remota sem escrita de `FETCH_HEAD`

Executar:

```powershell
git fetch --dry-run origin
```

O Git documenta que, em `--dry-run`, `FETCH_HEAD` não é gravado.

Se esse comando falhar, registrar a saída exata. Não escalar nem tentar outro método de autenticação nesta tarefa.

## 7. Critérios de aceite

- [ ] usuário/contexto normal identificado;
- [ ] root do repo, `.git` e `FETCH_HEAD` inspecionados quando existente;
- [ ] owner e ACLs registrados;
- [ ] atributos registrados;
- [ ] possíveis processos Git/SSH registrados;
- [ ] `git fetch --dry-run origin` tentado;
- [ ] nenhuma ACL, owner ou arquivo foi modificado;
- [ ] alteração local do diário preservada;
- [ ] PoC não criada;
- [ ] hipótese causal apresentada com grau de confiança e evidências.

## 8. Relatório final obrigatório

Responder com:

1. objetivo executado;
2. usuário/contexto do PowerShell;
3. branch, HEAD e remote;
4. resultado de `git status` e `git diff --check`;
5. caminho retornado por `git rev-parse --git-dir`;
6. existência/atributos de `.git` e `FETCH_HEAD`;
7. owner/ACL do root do repositório;
8. owner/ACL de `.git`;
9. owner/ACL de `FETCH_HEAD`, se existir;
10. saída relevante de `icacls`;
11. processos Git/SSH encontrados;
12. resultado de `git fetch --dry-run origin`;
13. comparação objetiva entre as permissões do root, `.git` e `FETCH_HEAD`;
14. hipótese causal mais provável e grau de confiança;
15. confirmação de que nenhuma permissão/arquivo/configuração foi alterada;
16. recomendação objetiva para a tarefa corretiva seguinte, sem executá-la.

## 9. Regra de parada

Se qualquer comando de inspeção exigir elevação ou alteração persistente, não elevar e não modificar. Registrar a limitação e continuar somente com as leituras disponíveis.

Não executar a F1-B1-T04 nesta tarefa.