# Tarefa Codex F1-B1-T02 — Confirmar Identidade e Versão Real do Windows

**Fase:** 1 — Fechamento arquitetural e especificação  
**Bloco:** 1 — Plataforma Windows, Client e distribuição  
**Tipo:** investigação local, somente leitura  
**Status:** PRONTA PARA EXECUÇÃO

## 1. Objetivo

Resolver a inconsistência identificada no inventário anterior, que reportou simultaneamente `Windows 10 Pro`, versão `25H2` e build `26200`.

A tarefa deve identificar, por múltiplas fontes locais e sem alterar o sistema, qual é a edição/nome comercial, versão e build efetivos do Windows instalado nesta estação, preservando separadamente os valores retornados por cada fonte.

## 2. Contexto e fonte de verdade

Ler antes de executar:

1. `AGENTS.md`;
2. `docs/00-governanca/politica-capacidade-codex.md` apenas como política do projeto, sem alterar modelo por conta própria;
3. `docs/04-planejamento/plano-oficial-fase-1.md`;
4. `docs/03-arquitetura/compatibilidade-windows-client.md`;
5. `docs/05-progresso/revisao-f1-b1-t01-inventario-ambiente.md`;
6. `docs/04-planejamento/tarefas-codex/f1-b1-t01-inventario-ambiente-windows.md`.

## 3. Estado inicial esperado

- `C:\dev\StepFlow` é o clone oficial do repositório;
- branch `main`;
- existe uma alteração local autorizada em `docs/05-progresso/diario-de-progresso.md` proveniente da tarefa anterior;
- não há código de produto, scaffold Tauri ou banco;
- nenhuma instalação deve ser realizada.

A alteração local do diário não deve ser descartada, sobrescrita ou incluída em commit.

## 4. Escopo incluído

- confirmar versão/nome/edição do Windows por múltiplas fontes locais;
- registrar exatamente o retorno de cada mecanismo, inclusive quando houver divergência;
- identificar `DisplayVersion`, build principal e UBR quando disponíveis;
- verificar a arquitetura do sistema apenas para confirmar consistência com o inventário anterior;
- concluir qual identificação é tecnicamente mais confiável para a documentação do projeto;
- reportar a divergência encontrada, se persistir.

## 5. Fora do escopo

É proibido nesta tarefa:

- instalar ou atualizar Windows;
- alterar registro;
- alterar políticas;
- instalar Rust, Tauri ou qualquer dependência;
- diagnosticar o SMB além do que já foi registrado;
- criar código;
- editar `compatibilidade-windows-client.md` como decisão definitiva;
- fazer commit;
- fazer push;
- descartar a alteração local existente no diário.

## 6. Procedimento e fontes locais mínimas

Executar em PowerShell, adaptando apenas quando necessário.

### 6.1. `Get-ComputerInfo`

```powershell
Get-ComputerInfo | Select-Object WindowsProductName, WindowsEditionId, WindowsVersion, OsName, OsVersion, OsBuildNumber, OsArchitecture
```

### 6.2. Registro CurrentVersion

```powershell
Get-ItemProperty 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion' |
  Select-Object ProductName, EditionID, DisplayVersion, CurrentBuild, CurrentBuildNumber, UBR, BuildLabEx, InstallationType
```

### 6.3. `systeminfo`

Executar e capturar ao menos as linhas equivalentes a:

```powershell
systeminfo
```

Registrar `OS Name`, `OS Version` e `System Type`.

### 6.4. CIM do sistema operacional

```powershell
Get-CimInstance Win32_OperatingSystem |
  Select-Object Caption, Version, BuildNumber, OSArchitecture
```

Se CIM for negado, registrar a falha; não tentar alterar permissões.

### 6.5. `winver`

Se for possível obter informação de `winver` de forma segura sem automação invasiva, registrar versão/build exibidos. Se exigir interação gráfica que o Codex não consiga observar com confiança, declarar como não verificado em vez de inventar resultado.

## 7. Regra de interpretação

Não usar apenas `WindowsProductName` como fonte final quando ele conflitar com versão/build.

Comparar os resultados entre si e concluir com base no conjunto de evidências, deixando claro:

- o valor literal retornado por cada fonte;
- qual nome/versão deve ser adotado na documentação do projeto;
- por que a fonte divergente não deve ser usada isoladamente.

Não consultar Internet nesta tarefa. A validação externa já será feita pelo Assistente.

## 8. Critérios de aceite

- [ ] `Get-ComputerInfo` registrado;
- [ ] registro `CurrentVersion` registrado;
- [ ] `systeminfo` registrado;
- [ ] CIM tentado e resultado/falha registrado;
- [ ] versão/build/UBR identificados quando disponíveis;
- [ ] divergências entre fontes explicitadas;
- [ ] conclusão objetiva sobre Windows 10 vs Windows 11 apresentada;
- [ ] nenhum software/configuração foi alterado;
- [ ] alteração local anterior no diário foi preservada;
- [ ] nenhum commit/push realizado.

## 9. Estado Git

Antes e depois da inspeção executar:

```powershell
Set-Location C:\dev\StepFlow
git status --short --branch
git diff --check
```

Não executar `git pull` se o working tree local impedir o fast-forward com segurança. Caso o remoto tenha avançado e seja necessário atualizar a tarefa, preservar a alteração local e reportar antes de qualquer ação destrutiva.

## 10. Documentação a atualizar

Nenhum arquivo precisa ser alterado nesta tarefa.

O resultado deve ser entregue no relatório para revisão do Assistente. A documentação oficial será atualizada após validação.

## 11. Relatório final obrigatório

Responder com:

1. objetivo executado;
2. saída relevante de `Get-ComputerInfo`;
3. valores relevantes de `CurrentVersion` no registro;
4. saída relevante de `systeminfo`;
5. saída/falha do CIM;
6. informação de `winver`, se verificável;
7. conclusão: nome comercial, edição, versão, build e arquitetura;
8. explicação da divergência anterior;
9. estado Git final;
10. bloqueios ou limitações;
11. confirmação de que nenhuma alteração no sistema/repositório foi feita.

## 12. Regra de parada

Se qualquer verificação exigir elevação, alteração de configuração ou escrita no sistema para funcionar, não escalar privilégios automaticamente. Registrar a limitação e seguir com as demais fontes de leitura disponíveis.