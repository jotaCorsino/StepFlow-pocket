# Tarefa Codex F1-B1-T02 — Confirmar Identidade e Versão Real do Windows

**Fase:** 1 — Fechamento arquitetural e especificação  
**Bloco:** 1 — Plataforma Windows, Client e distribuição  
**Tipo:** investigação local, somente leitura  
**Status:** CONCLUÍDA EM 2026-08-19

## Resultado consolidado

A investigação local foi concluída sem alteração do sistema e a validação externa do Assistente fechou a divergência encontrada.

Para fins do StepFlow, a estação de desenvolvimento é identificada como:

**Windows 11 Pro, versão 25H2, OS build 26200.9168, arquitetura x64.**

Evidências locais relevantes:

- `WindowsProductName`: `Windows 10 Pro` — valor literal divergente preservado como evidência;
- `EditionID`: `Professional`;
- `DisplayVersion`: `25H2`;
- `CurrentBuild`: `26200`;
- `UBR`: `9168`;
- arquitetura AMD64/x64;
- `systeminfo` e CIM retornaram acesso negado;
- `winver` não pôde ser observado de forma confiável pelo Codex.

A Microsoft documenta Windows 11 versão 25H2 como OS build `26200`, e a atualização KB5121003 de 11 de agosto de 2026 como build `26200.9168`. Assim, `ProductName = Windows 10 Pro` não é usado isoladamente para identificar o nome comercial efetivo desta instalação.

A revisão está registrada em `docs/05-progresso/revisao-f1-b1-t01-inventario-ambiente.md`.

---

## 1. Objetivo original

Resolver a inconsistência identificada no inventário anterior, que reportou simultaneamente `Windows 10 Pro`, versão `25H2` e build `26200`.

A tarefa deveria identificar, por múltiplas fontes locais e sem alterar o sistema, qual é a edição/nome comercial, versão e build efetivos do Windows instalado nesta estação, preservando separadamente os valores retornados por cada fonte.

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

A alteração local do diário não deveria ser descartada, sobrescrita ou incluída em commit.

## 4. Escopo incluído

- confirmar versão/nome/edição do Windows por múltiplas fontes locais;
- registrar exatamente o retorno de cada mecanismo, inclusive quando houver divergência;
- identificar `DisplayVersion`, build principal e UBR quando disponíveis;
- verificar a arquitetura do sistema apenas para confirmar consistência com o inventário anterior;
- concluir qual identificação é tecnicamente mais confiável para a documentação do projeto;
- reportar a divergência encontrada, se persistir.

## 5. Fora do escopo

Era proibido nesta tarefa:

- instalar ou atualizar Windows;
- alterar registro;
- alterar políticas;
- instalar Rust, Tauri ou qualquer dependência;
- diagnosticar o SMB além do que já havia sido registrado;
- criar código;
- editar `compatibilidade-windows-client.md` como decisão definitiva;
- fazer commit;
- fazer push;
- descartar a alteração local existente no diário.

## 6. Procedimento e fontes locais mínimas

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

```powershell
systeminfo
```

### 6.4. CIM do sistema operacional

```powershell
Get-CimInstance Win32_OperatingSystem |
  Select-Object Caption, Version, BuildNumber, OSArchitecture
```

### 6.5. `winver`

A informação seria registrada apenas se observável de forma confiável, sem automação invasiva.

## 7. Regra de interpretação

Não usar apenas `WindowsProductName` como fonte final quando ele conflitar com versão/build.

Comparar os resultados entre si e concluir com base no conjunto de evidências, preservando:

- o valor literal retornado por cada fonte;
- a identificação adotada na documentação do projeto;
- a explicação da fonte divergente.

## 8. Critérios de aceite — resultado

- [x] `Get-ComputerInfo` registrado;
- [x] registro `CurrentVersion` registrado;
- [x] `systeminfo` tentado e falha registrada;
- [x] CIM tentado e falha registrada;
- [x] versão/build/UBR identificados;
- [x] divergências entre fontes explicitadas;
- [x] conclusão Windows 10 vs Windows 11 validada externamente pelo Assistente;
- [x] nenhum software/configuração foi alterado;
- [x] alteração local anterior no diário foi preservada;
- [x] nenhum commit/push foi realizado pelo Codex.

## 9. Estado Git da execução

Ao final do relatório do Codex:

- branch `main`;
- HEAD `f5e3b14 docs: annotate completed inventory network test context`;
- `origin` correto;
- única alteração local: `docs/05-progresso/diario-de-progresso.md` proveniente da tarefa anterior.

## 10. Documentação

Nenhum arquivo foi alterado pelo Codex nesta tarefa.

O fechamento documental foi realizado pelo Assistente após validar o relatório e as fontes oficiais da Microsoft.

## 11. Limitações observadas

- `systeminfo`: acesso negado;
- CIM `Win32_OperatingSystem`: acesso negado;
- `winver`: janela gráfica não observável com confiança pelo Codex.

Essas limitações não impediram o fechamento porque `DisplayVersion`, `CurrentBuild`, `UBR`, arquitetura e a correspondência oficial da Microsoft foram suficientes.

## 12. Encerramento

Tarefa concluída. Nenhuma nova inspeção da identidade do Windows é necessária neste ambiente salvo se o sistema operacional for alterado futuramente.