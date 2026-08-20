# Gate C+D — PoC Tauri Pocket na Sessão do PO

**Fase:** 1 — Bloco 1  
**Status:** PRONTO PARA EXECUÇÃO  
**Executor:** PO em sessão Windows normal `EARTH\Estudos`

## Objetivo

Executar em uma única prova a criação da PoC Tauri 2 e a validação Pocket, evitando o `CodexSandboxOffline` nas etapas que exigem internet, npm e ambiente Windows real.

## Escopo

- criar PoC descartável fora do repositório oficial;
- Vanilla JavaScript + npm + Tauri 2;
- instalar dependências;
- compilar release com `--no-bundle`;
- registrar versões, executável, tamanho e SHA-256;
- copiar somente o executável de release para pasta isolada;
- executar a cópia isolada;
- repetir a execução com Node/npm/Rust/Cargo removidos apenas do PATH temporário;
- registrar dependências de runtime observáveis, especialmente WebView2.

## Restrições

- não criar código oficial do StepFlow;
- não copiar a PoC para `C:\dev\StepFlow`;
- não gerar MSI/NSIS;
- não alterar PATH permanentemente;
- não instalar runtime/toolchain no servidor;
- não transformar a PoC em implementação definitiva.

## Critério de sucesso

Tauri permanece candidato preferencial se o release `--no-bundle` produzir um executável que possa ser copiado isoladamente e iniciado sem Node/npm/Rust/Cargo disponíveis no PATH de runtime, sem elevação administrativa e sem instalação adicional durante o smoke test no PC de desenvolvimento.

## Fontes oficiais

- https://v2.tauri.app/start/create-project/
- https://v2.tauri.app/start/prerequisites/
- https://v2.tauri.app/distribute/
