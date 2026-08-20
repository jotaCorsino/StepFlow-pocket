# Fechamento do Bloco 1 — Plataforma Windows, Client e Distribuição Inicial

**Data:** 2026-08-20  
**Status:** GATE DO BLOCO 1 APROVADO  
**Fase:** 1 — Fechamento arquitetural e especificação

## 1. Decisão

O StepFlow Client seguirá, na direção arquitetural da Fase 1, com:

- **Tauri 2**;
- frontend em HTML/CSS/JavaScript modular;
- primeiro alvo **Windows x64**;
- baseline recomendado **Windows 10 e Windows 11**, sujeito a inventário e validação das máquinas reais da empresa;
- **WebView2** como dependência de runtime do renderer Windows;
- toolchain Rust/Cargo/MSVC/Node/npm restrito ao ambiente de desenvolvimento/build, não ao runtime do Client.

## 2. Evidência mecânica principal

A F1-B1-T04 produziu uma PoC Tauri 2 descartável no computador pessoal de desenvolvimento.

Resultados aprovados:

- scaffold Vanilla JavaScript/npm criado fora do repositório oficial;
- `npm install` concluído;
- `npm run tauri build -- --no-bundle` concluído com exit code `0`;
- executável de release gerado;
- cópia isolada de `8.949.248 bytes` executada fora da árvore de build;
- processo permaneceu ativo após 5 segundos no smoke test normal;
- processo também permaneceu ativo após 5 segundos com PATH temporário sem Node.js, npm, Rust ou Cargo;
- nenhum instalador MSI/NSIS foi necessário para a prova;
- nenhuma toolchain de desenvolvimento precisou acompanhar o executável em runtime.

Detalhes: `docs/05-progresso/revisao-f1-b1-t04-poc-tauri-pocket.md`.

## 3. Relação com o princípio Pocket

A prova sustenta a direção de que o Client pode ser produzido em uma máquina de desenvolvimento completa e distribuído como artefato preparado, sem transformar cada estação ou o servidor em ambiente de build.

Isso é compatível com o princípio Pocket:

- build pesado no DEV;
- artefato enxuto no destino;
- sem Rust/Cargo/Node/npm/Visual Studio/MSVC no runtime do Client;
- possibilidade de cópia/atualização local pelo launcher a ser especificado no Bloco 3.

## 4. O que permanece pendente

O Gate do Bloco 1 não transforma hipóteses corporativas em fatos.

Continuam pendentes para o ambiente correto:

- inventário das versões reais de Windows nas estações;
- confirmação de arquitetura x64 das máquinas relevantes;
- validação de WebView2 em máquinas representativas;
- teste real em Windows 10 corporativo representativo;
- comportamento offline em máquina-alvo;
- política exata para ausência de WebView2;
- launcher, cópia local, atualização, rollback e SMB, tratados no Bloco 3.

Windows 7/8/8.1 continuam fora da promessa oficial inicial.

## 5. Electron

Electron permanece alternativa de contingência e não deve ser estudado novamente sem evidência real de falha do Tauri em requisito relevante do produto.

As limitações artificiais de `EARTH\CodexSandboxOffline` não constituem motivo para abandonar Tauri.

## 6. Regra operacional aprendida

Para tarefas posteriores:

- operações que dependem de instalação, credenciais, acesso confiável à Internet ou configuração global do computador devem ocorrer na sessão Windows normal do PO quando necessário;
- o Codex não deve ser usado para reparar seu sandbox até ele se comportar como o ambiente real;
- builds/testes no Codex devem ser escolhidos somente quando o sandbox tiver as capacidades necessárias;
- falhas específicas do sandbox não viram requisitos do StepFlow.

## 7. Gate

**BLOCO 1 — APROVADO PARA PROSSEGUIR.**

Não há autorização para scaffold definitivo do Client nesta etapa; a Fase 1 continua sendo de fechamento arquitetural e especificação.

O próximo bloco oficial é:

**Bloco 2 — StepFlow Host.**
