# Revisão F1-B1-T04 — Prova Mínima Tauri 2 com Critério Pocket

**Data:** 2026-08-20  
**Status:** CONCLUÍDA / APROVADA FUNCIONALMENTE  
**Fase:** 1 — Fechamento arquitetural e especificação  
**Bloco:** 1 — Plataforma Windows, Client e distribuição

## 1. Objetivo

Validar mecanicamente se um Client Windows mínimo em Tauri 2 pode ser compilado no computador de desenvolvimento e executado como artefato isolado, sem exigir Node.js, npm, Rust ou Cargo no PATH de runtime.

A prova permaneceu descartável e fora do repositório oficial.

## 2. Contexto operacional final

A investigação inicial em `EARTH\CodexSandboxOffline` revelou limitações artificiais de Git, rede, escrita e ambiente. Essas limitações foram separadas dos requisitos reais do produto por meio do plano de adaptação da Fase 1.

O build final foi executado na sessão Windows normal `EARTH\Estudos`, não elevada, após instalação do workload `Desktop development with C++` no Visual Studio Community 2026.

## 3. Gate de toolchain aprovado

Antes da PoC Tauri foi concluído o Gate B, com as seguintes evidências:

- `vswhere` reconheceu a instalação `C:\Program Files\Microsoft Visual Studio\18\Community` para `Microsoft.VisualStudio.Component.VC.Tools.x86.x64`;
- `VsDevCmd.bat` existente;
- Developer Command Prompt 2026 v18.9.1 carregado;
- `cl.exe`: `VC\Tools\MSVC\14.51.36231\bin\Hostx64\x64\cl.exe`;
- `link.exe`: `VC\Tools\MSVC\14.51.36231\bin\Hostx64\x64\link.exe`;
- `rustc 1.97.1`;
- `cargo 1.97.1`;
- host `x86_64-pc-windows-msvc`;
- `cargo check --offline` aprovado;
- `cargo build --offline` aprovado;
- executável Rust mínimo executado com exit code `0`.

Conclusão: o ambiente real de build Windows está apto para Tauri/MSVC.

## 4. Criação da PoC Tauri

Local descartável:

`C:\dev\StepFlow-PoC\F1-B1-T04\stepflow-tauri-proof`

Configuração escolhida no `create-tauri-app`:

- project name: `stepflow-tauri-proof`;
- identifier: `com.stepflow.poc`;
- frontend: TypeScript / JavaScript;
- package manager: npm;
- template: Vanilla;
- flavor: JavaScript.

`create-tauri-app@4.6.2` foi obtido na criação da prova.

`npm install` concluiu com sucesso, adicionando 2 packages e reportando 0 vulnerabilities naquele momento.

A CLI local observada foi `@tauri-apps/cli@2.11.4`.

## 5. Build

O build definitivo foi executado em Developer PowerShell/ambiente de desenvolvimento do Visual Studio, com o toolchain Rust MSVC disponível.

Comando funcional:

`npm run tauri build -- --no-bundle`

Resultado:

- build exit code: `0`;
- executável de release produzido;
- caminho: `C:\dev\StepFlow-PoC\F1-B1-T04\stepflow-tauri-proof\src-tauri\target\release\stepflow-tauri-proof.exe`;
- nenhum MSI/NSIS foi necessário para a prova.

## 6. Artefato isolado

O executável foi copiado sozinho para:

`C:\dev\StepFlow-PoC\F1-B1-T04\portable-test\stepflow-tauri-proof.exe`

Tamanho registrado:

`8.949.248 bytes` (~8,53 MiB).

O comando `Get-FileHash -Algorithm SHA256` foi executado durante a prova, porém o valor do hash não ficou visível no transcript fornecido ao Assistente. Essa é uma lacuna documental menor e não altera a conclusão funcional da PoC.

## 7. Smoke test isolado

A cópia em `portable-test` foi iniciada fora da árvore de build.

Após 5 segundos:

`StillRunning = True`

Isso confirma que o processo iniciou e permaneceu ativo no intervalo da prova.

O comando de encerramento posterior encontrou o processo já encerrado entre a checagem e o `Stop-Process`. Isso é uma condição de corrida do roteiro de limpeza e não uma falha de inicialização, pois a evidência `True` já havia sido coletada.

## 8. Smoke test sem toolchain no PATH

Foi construído um PATH temporário contendo somente caminhos normais do Windows e a cópia isolada foi iniciada por `ProcessStartInfo` com esse ambiente.

Após 5 segundos:

`PortableStillRunning = True`

Portanto, o artefato compilado não depende de Node.js, npm, Rust ou Cargo no PATH para iniciar.

A tentativa posterior de `Kill()` encontrou o processo já encerrado entre a verificação e a chamada. Novamente, trata-se de corrida de encerramento após a coleta da evidência de runtime e não invalida o smoke test.

## 9. Avaliação Pocket

A prova fornece evidência positiva de que, no computador de desenvolvimento testado:

- Tauri 2 pode produzir um executável Windows x64 sem bundle/instalador;
- o executável pode ser separado da árvore de build;
- não é necessário transportar Node.js, npm, Rust, Cargo, Visual Studio ou MSVC junto ao artefato para iniciar a aplicação;
- o build pesado fica restrito ao ambiente de desenvolvimento;
- o resultado é compatível com a direção Pocket de copiar/distribuir artefatos preparados em vez de transformar o ambiente de destino em máquina de desenvolvimento.

## 10. Limites da prova

Esta aprovação NÃO significa ainda que:

- toda máquina corporativa Windows 10/11 está validada;
- WebView2 pode ser presumido em todas as estações;
- launcher/SMB/cópia local e atualização estão definidos;
- o Host está definido;
- o Client oficial pode ser scaffoldado imediatamente;
- a aplicação está validada em Windows 10 representativo da empresa.

A prova ocorreu no computador pessoal de desenvolvimento e valida apenas a aderência inicial da stack Client ao princípio Pocket.

## 11. Decisão

**F1-B1-T04: CONCLUÍDA / APROVADA FUNCIONALMENTE.**

Tauri 2 permanece a stack preferencial do StepFlow Client.

A sequência de diagnóstico do sandbox e do toolchain está encerrada. Não criar novas tarefas para reparar `CodexSandboxOffline` como se ele fosse o ambiente real de desenvolvimento.

## 12. Próximo passo recomendado

Fechar formalmente o Gate do Bloco 1 consolidando:

- Tauri 2 como direção do Client;
- x64 como primeiro alvo;
- Windows 10/11 como baseline recomendado sujeito a inventário e testes corporativos;
- WebView2 como dependência de runtime a validar nas máquinas-alvo;
- build toolchain como dependência exclusiva do ambiente de desenvolvimento;
- distribuição portátil/cópia local como direção tecnicamente viável, ainda dependente do Bloco 3 para launcher, atualização e SMB.

Após esse fechamento, iniciar o Bloco 2 — StepFlow Host.
