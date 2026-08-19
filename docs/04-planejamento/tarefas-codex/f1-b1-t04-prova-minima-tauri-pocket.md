# Tarefa Codex F1-B1-T04 — Prova Mínima Tauri 2 com Critério Pocket

**Fase:** 1 — Fechamento arquitetural e especificação  
**Bloco:** 1 — Plataforma Windows, Client e distribuição  
**Tipo:** prova técnica descartável  
**Status:** PRONTA PARA EXECUÇÃO

## 1. Objetivo

Executar uma prova mínima e descartável de Tauri 2 no computador pessoal de desenvolvimento para responder, com evidências mecânicas, se a stack consegue:

- criar um Client Windows x64 com frontend Vanilla JavaScript;
- compilar corretamente com o toolchain MSVC já preparado;
- gerar um executável de release sem exigir a criação de instalador;
- executar uma cópia isolada desse executável fora da árvore de build;
- continuar inicializando quando o processo de teste não possui Node.js, npm, Rust ou Cargo disponíveis no `PATH`.

A prova não é o início da implementação do StepFlow e não deve ser promovida a código definitivo.

## 2. Fundamentação técnica

A documentação oficial atual do Tauri 2 confirma:

- `create-tauri-app` como ferramenta oficial para criação de projetos;
- suporte a template `Vanilla` com frontend `JavaScript`;
- uso de Tauri CLI local via npm;
- `tauri build --no-bundle` para compilar o aplicativo sem gerar pacotes de instalação;
- uso de WebView2 e Microsoft C++ Build Tools no Windows.

Fontes primárias:

- https://v2.tauri.app/start/create-project/
- https://v2.tauri.app/start/prerequisites/
- https://v2.tauri.app/distribute/

## 3. Contexto e documentos obrigatórios

Antes de executar, ler:

1. `AGENTS.md`;
2. `docs/00-governanca/contexto-ambientes.md`;
3. `docs/00-governanca/politica-capacidade-codex.md`;
4. `docs/03-arquitetura/implantacao-pocket.md`;
5. `docs/03-arquitetura/compatibilidade-windows-client.md`;
6. `docs/04-planejamento/plano-oficial-fase-1.md`;
7. `docs/05-progresso/revisao-f1-b1-t03-toolchain-rust.md`.

## 4. Estado inicial esperado

- repositório oficial em `C:\dev\StepFlow`;
- branch `main`;
- existe somente a alteração local autorizada em `docs/05-progresso/diario-de-progresso.md`;
- Node.js/npm disponíveis;
- Rust/rustup/cargo instalados;
- toolchain `stable-x86_64-pc-windows-msvc`;
- WebView2 presente;
- MSVC presente;
- nenhum scaffold Tauri oficial do StepFlow existe.

## 5. Local da prova

A prova deve ficar **fora do repositório oficial** para evitar contaminação da árvore de trabalho.

Usar:

```text
C:\dev\StepFlow-PoC\F1-B1-T04\
```

Dentro dela, criar um projeto descartável com nome claro, por exemplo:

```text
stepflow-tauri-proof
```

Nenhum arquivo dessa pasta deve ser copiado para `C:\dev\StepFlow` durante a tarefa.

## 6. Gate inicial — sessão normal

A prova deve ser executada em **nova sessão PowerShell não elevada**.

Antes de criar qualquer scaffold, validar:

```powershell
node --version
npm --version
rustup --version
rustc --version
cargo --version
rustup show
rustc -Vv
```

Confirmar novamente:

```text
stable-x86_64-pc-windows-msvc
```

Se Rust/Cargo somente funcionarem em sessão elevada ou por caminho absoluto fora do `PATH` normal do usuário, **parar e reportar**. Não executar a PoC como administrador para mascarar problema de ambiente.

## 7. Estado Git antes da prova

Em `C:\dev\StepFlow`:

```powershell
Set-Location C:\dev\StepFlow
git status --short --branch
git diff --check
git pull --ff-only
```

O pull só pode prosseguir se preservar integralmente a alteração local autorizada no diário.

Se exigir stash, merge, rebase, reset ou resolução manual, parar e reportar.

Após o pull, reler os documentos obrigatórios e esta tarefa antes de criar a PoC.

## 8. Criação da PoC

Usar a ferramenta oficial:

```powershell
npm create tauri-app@latest
```

Escolher explicitamente:

- projeto: `stepflow-tauri-proof`;
- identifier descartável: `com.stepflow.poc` ou equivalente claramente não definitivo;
- frontend: `TypeScript / JavaScript`;
- package manager: `npm`;
- UI template: `Vanilla`;
- UI flavor: `JavaScript`.

Não escolher React, Vue, Angular, TypeScript ou outro framework nesta prova.

Não instalar Tauri CLI globalmente. Usar a CLI local criada/instalada no projeto npm.

Após criação:

```powershell
Set-Location C:\dev\StepFlow-PoC\F1-B1-T04\stepflow-tauri-proof
npm install
```

Registrar as versões efetivamente resolvidas de, no mínimo:

```powershell
npm ls @tauri-apps/cli @tauri-apps/api
```

Registrar também a versão Tauri/Rust observável no `Cargo.lock` ou por comando apropriado, sem editar manualmente dependências apenas para piná-las nesta PoC.

## 9. Build sem instalador

Executar:

```powershell
npm run tauri build -- --no-bundle
```

A prova deve terminar com build de release bem-sucedido.

Não gerar MSI, NSIS ou outro instalador nesta tarefa.

Localizar o executável de release produzido e registrar:

- caminho;
- nome;
- tamanho em bytes e MB;
- SHA-256.

Exemplo de comandos seguros:

```powershell
Get-Item '<CAMINHO-DO-EXE>' | Select-Object FullName, Length, LastWriteTime
Get-FileHash '<CAMINHO-DO-EXE>' -Algorithm SHA256
```

## 10. Smoke test da cópia isolada

Criar:

```text
C:\dev\StepFlow-PoC\F1-B1-T04\portable-test\
```

Copiar **somente o executável de release necessário ao teste** para essa pasta.

Não copiar `node_modules`, `.cargo`, `target`, fontes ou toolchain junto ao executável.

Executar a cópia isolada em sessão normal e verificar:

- processo inicia;
- permanece ativo por pelo menos alguns segundos;
- não encerra imediatamente com erro;
- janela Tauri é observada, se o ambiente permitir observação gráfica confiável.

Depois encerrar o processo de teste de forma limpa.

Se a UI não puder ser observada pelo Codex, não inventar resultado visual; registrar somente a evidência de processo disponível.

## 11. Smoke test sem toolchain no PATH do processo

Sem alterar o `PATH` permanente do Windows, criar um **ambiente temporário somente para o processo/sessão de teste** em que caminhos de Node/npm e Rust/Cargo não estejam disponíveis.

Antes de iniciar o executável, comprovar nessa sessão temporária que:

```powershell
Get-Command node -ErrorAction SilentlyContinue
Get-Command npm -ErrorAction SilentlyContinue
Get-Command rustc -ErrorAction SilentlyContinue
Get-Command cargo -ErrorAction SilentlyContinue
```

não resolvem as ferramentas de desenvolvimento.

Manter apenas os paths normais do Windows necessários ao sistema operacional.

Então executar **a cópia isolada já produzida**, sem recompilar.

Critério:

- o executável deve iniciar e permanecer ativo por alguns segundos mesmo sem Node/npm/Rust/Cargo no `PATH` daquele processo.

Restaurar/encerrar a sessão temporária depois do teste. Não alterar o `PATH` permanente do usuário ou do sistema.

Esse teste demonstra somente que essas toolchains não são necessárias para iniciar o artefato já compilado. Ele não prova ainda portabilidade completa para todas as máquinas corporativas.

## 12. Verificações Pocket complementares

Registrar objetivamente:

- se o artefato de release executa fora da árvore do projeto;
- se foi necessário instalar algo adicional depois do build para executar a cópia;
- quais dependências de sistema permanecem observáveis/relevantes, especialmente WebView2 e componentes nativos do Windows;
- se a execução exigiu elevação administrativa;
- se a execução escreveu arquivos fora da pasta de teste de forma perceptível;
- tamanho do executável isolado.

Não alterar sistema, registro, políticas, serviços ou firewall para investigar dependências.

## 13. Fora do escopo

É proibido nesta tarefa:

- criar o Client oficial do StepFlow dentro do repositório;
- copiar a PoC para a árvore oficial;
- implementar login, sidebar, processos ou qualquer feature de negócio;
- criar Host;
- criar SQLite;
- criar launcher;
- testar SMB ou rede corporativa;
- instalar WebView2;
- instalar novos Build Tools;
- instalar Node/Rust no servidor da empresa;
- gerar instalador MSI/NSIS;
- configurar auto-update;
- registrar serviço do Windows;
- alterar PATH permanentemente;
- executar a PoC inteira como administrador apenas para fazê-la funcionar;
- fazer commit/push da PoC;
- remover a alteração local autorizada do diário;
- iniciar tarefa seguinte.

## 14. Critérios de aceite

- [ ] comandos Node/npm/Rust funcionam em nova sessão não elevada;
- [ ] repositório oficial atualizado com segurança;
- [ ] PoC criada fora do repositório;
- [ ] template Vanilla JavaScript/npm confirmado;
- [ ] versões Tauri efetivamente instaladas registradas;
- [ ] `npm run tauri build -- --no-bundle` concluído;
- [ ] executável de release localizado;
- [ ] tamanho e SHA-256 registrados;
- [ ] executável copiado isoladamente para `portable-test`;
- [ ] cópia isolada inicia sem depender da árvore de build;
- [ ] smoke test com Node/npm/Rust/Cargo ausentes do `PATH` temporário concluído;
- [ ] nenhuma elevação foi necessária para executar o artefato;
- [ ] nenhuma dependência adicional foi instalada para executar a cópia;
- [ ] nenhum código oficial do StepFlow foi criado;
- [ ] nenhum instalador foi gerado;
- [ ] nenhum commit/push realizado.

## 15. Relatório final obrigatório

Responder com:

1. objetivo executado;
2. estado Git inicial, pull e HEAD final do repositório oficial;
3. resultado do gate em PowerShell não elevado;
4. versões Node/npm/Rust confirmadas;
5. caminho da PoC descartável;
6. escolhas feitas no `create-tauri-app`;
7. versões de `@tauri-apps/cli`, `@tauri-apps/api` e Tauri/Rust resolvidas;
8. resultado do `npm install`;
9. resultado de `npm run tauri build -- --no-bundle`;
10. caminho/nome/tamanho/SHA-256 do executável;
11. resultado do smoke test da cópia isolada;
12. resultado do smoke test com toolchains ausentes do `PATH` temporário;
13. necessidade ou não de privilégio administrativo em build e runtime;
14. dependências de runtime observadas, especialmente WebView2;
15. arquivos/pastas criados fora do repositório oficial;
16. estado Git final do repositório oficial;
17. confirmação de que a única alteração local pré-existente no repo foi preservada;
18. erros, warnings ou limitações relevantes;
19. conclusão objetiva sobre aderência inicial do Tauri ao princípio Pocket;
20. recomendação para a próxima tarefa, sem executá-la.

## 16. Regra de parada

Parar e reportar antes de ampliar escopo se:

- Rust não funcionar em sessão não elevada;
- o repositório não puder ser atualizado com segurança;
- `create-tauri-app` exigir opção inesperada que mude a stack escolhida;
- o build exigir instalar componentes adicionais não previstos;
- o build somente funcionar elevado;
- o executável isolado não iniciar;
- o executável exigir Node/Rust no `PATH` em runtime;
- a investigação exigir alteração persistente no Windows.

Não corrigir automaticamente problemas arquiteturalmente relevantes dentro desta tarefa.