# Tarefa Codex F1-B1-T01 — Inventário do Ambiente Windows e Pré-requisitos

**Fase:** 1 — Fechamento arquitetural e especificação

**Bloco:** 1 — Plataforma Windows, Client e distribuição

**Tipo:** investigação local, sem implementação de produto

**Status:** CONCLUÍDA / REVISADA

## Nota retrospectiva de contexto

Esta tarefa foi escrita quando o exemplo `\\192.168.5.7\Arquivos\StepFlow\` ainda estava sendo tratado incorretamente como se fosse um caminho confirmado.

Após esclarecimento do PO:

- o desenvolvimento atual ocorre em computador pessoal, fora da LAN da empresa;
- o caminho citado era apenas exemplo ilustrativo;
- IP, hostname, compartilhamento e pasta reais ainda não foram confirmados;
- o teste SMB realizado nesta tarefa não constitui validação do ambiente corporativo;
- seu resultado deve ser classificado como `NÃO APLICÁVEL NESTE AMBIENTE`, e não como bloqueio.

A fonte de verdade atual sobre ambientes é `docs/00-governanca/contexto-ambientes.md`.

O restante da tarefa abaixo é preservado como registro histórico do que foi executado.

## Pré-requisito obrigatório

Concluído e validado em 2026-08-19:

`docs/04-planejamento/tarefas-codex/f1-b0-t01-bootstrap-repositorio-local.md`

O ambiente `C:\dev\StepFlow` foi confirmado como clone íntegro e limpo de `jotaCorsino/StepFlow-pocket`, branch `main`, com `AGENTS.md` lido pelo Codex.

Antes de iniciar esta tarefa, executar `git pull --ff-only` para incorporar atualizações documentais remotas posteriores ao bootstrap. Se o pull falhar ou houver alterações locais inesperadas, parar e reportar; não resolver por merge ou reset automático.

## 1. Objetivo

Levantar de forma mecânica o ambiente Windows disponível em `C:\dev\StepFlow` e os pré-requisitos relevantes para uma futura prova Tauri, sem instalar dependências, sem criar scaffold e sem implementar qualquer funcionalidade do StepFlow.

A tarefa deve responder com evidências:

- qual versão/edição/build do Windows está sendo usada;
- arquitetura do sistema operacional e do processador;
- se WebView2 Runtime está instalado e qual versão é detectável;
- se Node.js/npm já existem;
- se Rust/rustup/cargo já existem;
- se Microsoft C++ Build Tools/Visual Studio Build Tools são detectáveis;
- se Git está disponível e qual configuração básica do checkout;
- tentativa histórica de acesso ao caminho SMB ilustrativo então utilizado;
- se há algum bloqueio óbvio para a próxima prova técnica local.

## 2. Contexto e fonte de verdade

Ler antes de executar:

1. `AGENTS.md`;
2. `docs/00-governanca/guia-mestre-desenvolvimento.md`;
3. `docs/04-planejamento/plano-oficial-fase-1.md`;
4. `docs/03-arquitetura/compatibilidade-windows-client.md`;
5. `docs/03-arquitetura/arquitetura-inicial.md`;
6. `docs/05-progresso/registro-de-decisoes.md`.

## 3. Estado inicial esperado

- repositório clonado/localizado em `C:\dev\StepFlow`;
- bootstrap local concluído e validado;
- branch `main` ativa;
- working tree limpo;
- checkout atualizado por `git pull --ff-only` antes da inspeção;
- nenhuma aplicação StepFlow funcional existe;
- nenhuma stack definitiva deve ser presumida como instalada;
- Tauri 2 é a direção arquitetural recomendada para prova, mas esta tarefa não cria projeto Tauri.

## 4. Escopo incluído

- atualizar o clone somente por fast-forward com `git pull --ff-only`;
- inspecionar sistema operacional;
- inspecionar arquitetura CPU/SO;
- inspecionar WebView2 por meios locais confiáveis, como registro, arquivos/runtime ou mecanismos do Windows;
- inspecionar versões existentes de Git, Node/npm, Rust/rustup/cargo;
- detectar Visual Studio/Build Tools/C++ quando possível sem instalar nada;
- realizar a tentativa histórica de leitura do caminho SMB ilustrativo então informado;
- verificar estado Git local do repositório;
- registrar resultados e limitações;
- atualizar somente documentação de progresso necessária.

## 5. Fora do escopo

É proibido nesta tarefa:

- instalar Node.js;
- instalar Rust;
- instalar Visual Studio Build Tools;
- instalar WebView2;
- executar `npm create tauri-app` ou equivalente;
- criar `package.json`;
- criar `src-tauri`;
- criar HTML/CSS/JavaScript de produto;
- criar banco SQLite;
- criar Host;
- criar launcher;
- alterar arquitetura ou roadmap por conta própria;
- fazer commit/push sem instrução explícita separada do fluxo de trabalho vigente;
- resolver divergência Git por merge, rebase, reset ou force.

## 6. Regras e restrições

- executar somente comandos de leitura/inspeção ou testes de acesso sem alteração persistente relevante;
- `git pull --ff-only` é a única atualização Git autorizada antes da inspeção;
- não alterar políticas do Windows;
- não habilitar features opcionais;
- não alterar PATH;
- não mapear unidade de rede de forma persistente;
- não gravar no compartilhamento de rede;
- se determinado mecanismo de detecção falhar, tentar alternativa de leitura segura e registrar o método;
- não mascarar ausência de requisito como sucesso;
- não considerar Windows 7/8/8.1 como alvo oficial da primeira versão;
- não concluir que a máquina representa todas as estações da empresa; ela é apenas uma amostra de desenvolvimento.

## 7. Arquivos/áreas esperadas

A tarefa não deve criar código.

Documentação que pode ser alterada:

- `docs/05-progresso/diario-de-progresso.md`;
- opcionalmente um relatório de evidências em `docs/05-progresso/` se o volume justificar.

Não alterar `compatibilidade-windows-client.md` automaticamente apenas para registrar dados de uma máquina. Primeiro reportar os resultados.

## 8. Critérios de aceite

- [x] bootstrap local anterior foi validado;
- [x] `git pull --ff-only` foi executado com sucesso antes da inspeção;
- [x] versão, edição e build do Windows foram levantados, com divergência de identificação posteriormente encaminhada para confirmação;
- [x] arquitetura x64/x86/ARM foi identificada;
- [x] presença/ausência e versão do WebView2 foram verificadas com evidência;
- [x] Git foi verificado;
- [x] Node/npm foram verificados;
- [x] Rust/rustup/cargo foram verificados;
- [x] Build Tools/C++ foram verificados na medida possível;
- [x] tentativa histórica de acesso ao caminho SMB ilustrativo foi feita sem escrita;
- [x] `git status`, branch e remote do checkout foram registrados;
- [x] nenhum pacote ou dependência foi instalado;
- [x] nenhum scaffold/código de produto foi criado;
- [x] limitações da inspeção foram explicitadas.

**Observação:** o item SMB não é critério válido de prontidão do ambiente corporativo, pois a estação estava fora da rede da empresa e o path usado era apenas exemplo.

## 9. Validações executadas

Primeiro:

```powershell
Set-Location C:\dev\StepFlow
git status --short --branch
git pull --ff-only
```

Depois foram executados equivalentes seguros para levantar Windows, arquitetura, CPU e ferramentas, incluindo:

```powershell
Get-ComputerInfo | Select-Object WindowsProductName, WindowsVersion, OsBuildNumber, OsArchitecture
[Environment]::Is64BitOperatingSystem
[Environment]::Is64BitProcess
Get-CimInstance Win32_Processor | Select-Object Name, AddressWidth, DataWidth
```

Ferramentas:

```powershell
git --version
node --version
npm --version
rustc --version
rustup --version
cargo --version
```

Também foram inspecionados Visual Studio/Build Tools, MSVC e WebView2.

A tentativa histórica de SMB utilizou o caminho ilustrativo então informado. Essa tentativa não deve ser repetida em tarefas futuras como se fosse configuração oficial.

## 10. Documentação atualizada na execução

O Codex alterou localmente:

- `docs/05-progresso/diario-de-progresso.md`.

A revisão posterior do Assistente foi registrada em:

- `docs/05-progresso/revisao-f1-b1-t01-inventario-ambiente.md`.

## 11. Resultado consolidado

- ambiente local de desenvolvimento inventariado;
- Git, Node/npm, WebView2 e toolchain Microsoft detectados;
- Rust ausente;
- identidade comercial do Windows requer confirmação específica;
- teste SMB anterior classificado posteriormente como `NÃO APLICÁVEL NESTE AMBIENTE`;
- nenhuma instalação ou implementação de produto ocorreu.

## 12. Próximo passo

A próxima investigação autorizada é a confirmação da identidade/versão real do Windows desta estação de desenvolvimento.

Validações do compartilhamento, Host real e LAN corporativa ficam para momento futuro em que exista acesso à infraestrutura da empresa e os endereços reais estejam confirmados.