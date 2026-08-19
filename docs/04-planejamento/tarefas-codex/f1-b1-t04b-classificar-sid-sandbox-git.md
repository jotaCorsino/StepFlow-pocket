# Tarefa Codex F1-B1-T04B — Classificar SID e Restrições do Sandbox Git

**Fase:** 1 — Fechamento arquitetural e especificação  
**Bloco:** 1 — Plataforma Windows, Client e distribuição  
**Tipo:** diagnóstico operacional local, somente leitura  
**Status:** PRONTA PARA EXECUÇÃO  
**Relação:** continuação da `F1-B1-T04A`; pré-condição para retomar `F1-B1-T04`

## 1. Objetivo

Determinar, sem alterar segurança, credenciais, rede ou metadados Git, se os dois bloqueios observados na sessão não elevada do Codex são características do sandbox `earth\codexsandboxoffline` ou problemas gerais do checkout/Windows:

1. `DENY` explícito em `.git` e `.git\FETCH_HEAD` para o SID `S-1-5-21-1439205687-274586611-205630846-3092016610`;
2. `schannel: AcquireCredentialsHandle failed: SEC_E_NO_CREDENTIALS` ao tentar comunicação HTTPS com o GitHub via Git.

Nenhuma ACL deve ser corrigida nesta tarefa.

## 2. Contexto confirmado

A F1-B1-T04A confirmou:

- sessão não elevada: `earth\codexsandboxoffline`;
- branch `main`;
- HEAD local `dfb9f4b`;
- root do repo, `.git` e `FETCH_HEAD` com owner `EARTH\Estudos`;
- root sem `DENY` explícito equivalente;
- `.git` e `FETCH_HEAD` com `DENY` explícito para o SID acima, incluindo escrita/remoção;
- nenhum processo Git/SSH concorrente;
- `git fetch --dry-run origin` falhando com `SEC_E_NO_CREDENTIALS`;
- PoC Tauri não iniciada.

A Microsoft documenta que uma ACE de negação aplicável pode impedir direitos mesmo quando permissões de grupos permitiriam acesso. Porém, antes de remover qualquer ACE é obrigatório confirmar a qual identidade/token o SID pertence.

## 3. Escopo incluído

Somente leitura/inspeção:

- obter SID do usuário atual;
- listar SIDs dos grupos presentes no token atual;
- tentar traduzir o SID negado para nome de conta/grupo;
- confirmar se o SID negado corresponde diretamente ao usuário atual ou a algum grupo do token;
- inspecionar somente as ACEs relevantes em `.git` e `FETCH_HEAD`, incluindo `IsInherited` e `InheritanceFlags`;
- verificar configuração de SSL/credenciais do Git sem alterá-la;
- testar conectividade TCP 443 com GitHub;
- testar HTTPS por `curl.exe` sem baixar arquivo;
- testar `git ls-remote origin HEAD`, que consulta refs remotas sem atualizar refs locais ou `FETCH_HEAD`;
- classificar os bloqueios com base nas evidências.

## 4. Fora do escopo

É proibido:

- alterar ACLs;
- usar `icacls /remove`, `/grant`, `/deny`, `/reset`, `/setowner` ou equivalentes;
- usar `Set-Acl`;
- alterar owner;
- excluir/recriar `FETCH_HEAD`;
- alterar atributos;
- executar `git pull`;
- executar `git fetch` normal;
- executar `git config --global --add`, `--unset` ou qualquer escrita de configuração;
- modificar Credential Manager;
- executar `cmdkey /add` ou `/delete`;
- alterar certificados;
- instalar certificados;
- alterar proxy;
- alterar firewall;
- usar sessão elevada para contornar;
- stash, merge, rebase, reset ou checkout destrutivo;
- reclonar;
- criar PoC Tauri;
- alterar documentação/diário;
- commit/push.

## 5. Procedimento obrigatório

Executar em nova sessão PowerShell não elevada.

### 5.1. Identidade do token

```powershell
whoami
whoami /user
whoami /groups

[System.Security.Principal.WindowsIdentity]::GetCurrent() |
  Select-Object Name,User,Groups
```

Registrar explicitamente o SID do usuário atual.

### 5.2. Resolver o SID negado

```powershell
$DeniedSid = 'S-1-5-21-1439205687-274586611-205630846-3092016610'
$SidObject = New-Object System.Security.Principal.SecurityIdentifier($DeniedSid)

try {
  $SidObject.Translate([System.Security.Principal.NTAccount]).Value
} catch {
  $_.Exception.Message
}
```

Depois verificar se o SID está no token atual:

```powershell
$Identity = [System.Security.Principal.WindowsIdentity]::GetCurrent()
$CurrentUserSid = $Identity.User.Value
$CurrentGroupSids = @($Identity.Groups | ForEach-Object { $_.Value })

[pscustomobject]@{
  DeniedSid = $DeniedSid
  CurrentUserSid = $CurrentUserSid
  MatchesCurrentUser = ($DeniedSid -eq $CurrentUserSid)
  IsInCurrentGroups = ($CurrentGroupSids -contains $DeniedSid)
}
```

### 5.3. Inspecionar somente as ACEs do SID negado

```powershell
$paths = @(
  'C:\dev\StepFlow\.git',
  'C:\dev\StepFlow\.git\FETCH_HEAD'
)

foreach ($path in $paths) {
  if (Test-Path $path) {
    Write-Host "--- $path ---"
    (Get-Acl $path).Access |
      Where-Object { $_.IdentityReference.Value -eq $DeniedSid -or $_.IdentityReference.Translate([System.Security.Principal.SecurityIdentifier]).Value -eq $DeniedSid } |
      Select-Object IdentityReference,AccessControlType,FileSystemRights,IsInherited,InheritanceFlags,PropagationFlags
  }
}
```

Se a tradução de uma `IdentityReference` falhar, não interromper a tarefa; registrar e usar a saída literal disponível de `icacls`/`Get-Acl`.

### 5.4. Configuração Git relevante — somente leitura

Em `C:\dev\StepFlow`:

```powershell
git --version
git config --show-origin --get http.sslBackend
git config --show-origin --get-all http.sslCAInfo
git config --show-origin --get-all http.sslCert
git config --show-origin --get-all credential.helper
git config --show-origin --get-all http.proxy
git config --show-origin --get-all https.proxy
```

Ausência de valor é resultado válido. Não criar nem alterar configuração.

### 5.5. Testes de rede/TLS sem escrita no repo

```powershell
Test-NetConnection github.com -Port 443
```

Depois:

```powershell
curl.exe -I https://github.com
```

Registrar somente status/erro relevante. Não usar opções que gravem arquivo.

Depois:

```powershell
git ls-remote origin HEAD
```

Esse comando deve apenas consultar refs remotas; não executar `fetch` ou `pull`.

Se algum teste falhar, registrar literalmente e prosseguir com os demais testes seguros.

## 6. Critérios de aceite

- [ ] SID do usuário atual identificado;
- [ ] grupos/SIDs do token atual registrados;
- [ ] SID negado traduzido ou falha de tradução registrada;
- [ ] confirmado se SID negado corresponde ao usuário ou grupo do token;
- [ ] origem/herança da ACE negada registrada;
- [ ] configuração Git SSL/credential/proxy consultada sem escrita;
- [ ] TCP 443 testado;
- [ ] HTTPS via `curl.exe` testado;
- [ ] `git ls-remote origin HEAD` testado;
- [ ] nenhuma ACL/configuração/credencial foi alterada;
- [ ] PoC não criada;
- [ ] classificação objetiva dos dois bloqueios apresentada.

## 7. Interpretação esperada

Não presumir a conclusão.

Possíveis resultados:

- se o SID negado for o próprio `codexsandboxoffline` ou grupo específico do sandbox, classificar a ACE como provável restrição deliberada do ambiente do Codex e **não recomendar sua remoção automática**;
- se o SID não estiver presente no token, investigar apenas documentalmente a origem da ACE sem removê-la;
- se TCP 443 funcionar mas Git e/ou `curl` falharem no TLS, classificar o problema em camada TLS/sessão;
- se `curl` funcionar e Git falhar, concentrar a análise na configuração/build do Git/Schannel;
- se ambos falharem de forma semelhante somente no sandbox, considerar forte evidência de restrição operacional da sessão;
- nenhuma dessas classificações autoriza mudança nesta mesma tarefa.

## 8. Relatório final obrigatório

Responder com:

1. objetivo executado;
2. usuário e SID atual;
3. resultado relevante de `whoami /groups`;
4. tradução do SID negado;
5. confirmação se o SID negado corresponde ao usuário atual ou a grupo do token;
6. detalhes da ACE negada em `.git`;
7. detalhes da ACE negada em `FETCH_HEAD`;
8. se as ACEs são explícitas ou herdadas;
9. versão do Git;
10. `http.sslBackend` e demais configurações SSL relevantes;
11. credential helper configurado;
12. proxy configurado ou ausência;
13. resultado de `Test-NetConnection github.com -Port 443`;
14. resultado de `curl.exe -I https://github.com`;
15. resultado de `git ls-remote origin HEAD`;
16. classificação do bloqueio de ACL, com grau de confiança;
17. classificação do bloqueio Schannel/rede, com grau de confiança;
18. recomendação operacional mínima para sincronizar o repositório sem enfraquecer segurança desnecessariamente;
19. confirmação de nenhuma alteração persistente;
20. confirmação de que a F1-B1-T04 não foi executada.

## 9. Regra de parada

Se qualquer inspeção pedir elevação, alteração de segurança, credencial, certificado, proxy ou configuração persistente, não autorizar. Registrar a limitação e continuar somente com testes de leitura disponíveis.

Não executar a F1-B1-T04 nesta tarefa.