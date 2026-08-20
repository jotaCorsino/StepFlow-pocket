# Compatibilidade Windows e StepFlow Client

**Status:** CONSOLIDADO PARA A FASE 1 / VALIDAÇÃO CORPORATIVA PENDENTE  
**Atualização:** 2026-08-20

## Decisão

O StepFlow Client utilizará **Tauri 2 + HTML/CSS/JavaScript modular**.

Baseline inicial:

- Windows 10 x64;
- Windows 11 x64;
- WebView2 como renderer/runtime do Client;
- Windows 7/8/8.1 fora da promessa oficial inicial;
- x86/ARM64 somente se inventário real demonstrar necessidade.

A versão mínima exata de Windows 10 será definida após inventário das estações corporativas.

## Evidência já obtida

A PoC descartável da Fase 1 confirmou no computador de desenvolvimento:

- criação de Client Tauri 2 Vanilla JavaScript/npm;
- build release `--no-bundle` bem-sucedido;
- executável isolado de aproximadamente 8,5 MiB;
- execução fora da árvore de build;
- execução sem Node.js, npm, Rust ou Cargo no `PATH` de runtime;
- nenhuma necessidade de MSI/NSIS para o smoke test.

Conclusão: toolchain pesado fica no ambiente de desenvolvimento; o runtime do Client não exige transportar Node/Rust/Visual Studio para a estação.

## WebView2

Tauri usa WebView2 no Windows. A primeira versão deve:

- detectar ausência/incompatibilidade de forma compreensível;
- não depender de Internet durante o uso normal;
- evitar instalar WebView2 repetidamente a cada abertura;
- validar no ambiente corporativo se o runtime já existe nas estações representativas.

Se alguma estação suportada não possuir WebView2, a forma administrativa/offline de preparação será decidida com base no ambiente real, sem transformar o launcher em instalador pesado.

## Distribuição vigente

O Client não será executado permanentemente sobre SMB e não terá instalador tradicional obrigatório.

A arquitetura aprovada é:

```text
ponto de entrada interno
→ launcher transitório
→ cópia validada/versionada em %LOCALAPPDATA%\StepFlow\Client
→ StepFlow.exe local
```

Detalhes em `launcher-distribuicao-client.md`.

## Ícone e metadados

Tauri suporta ícone Windows customizado. O arquivo-fonte do ícone será aprovado visualmente e mantido nos assets do projeto; não assumir automaticamente que o logo corporativo é o ícone do aplicativo.

## Alternativas

Electron permanece somente contingência. Não reabrir comparação sem evidência objetiva de que Tauri falha em requisito real do produto.

## Validações ainda dependentes da empresa

- versões/edições reais de Windows;
- arquitetura x64 das estações relevantes;
- WebView2 instalado e funcional;
- execução sem Internet;
- políticas de execução/antivírus/EDR;
- comportamento real do launcher a partir do compartilhamento corporativo.

Esses itens são validação de ambiente, não motivo para reabrir a stack enquanto não houver incompatibilidade concreta.
