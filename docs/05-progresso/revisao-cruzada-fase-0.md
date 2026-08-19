# Revisão Cruzada — Fase 0 do StepFlow Pocket

**Data:** 2026-08-19

**Status:** CONCLUÍDA

## Objetivo

Verificar a coerência entre governança, visão de produto, arquitetura inicial, roadmap, registro de decisões e instruções do Codex antes de encerrar a Fase 0 e autorizar a Fase 1.

## Documentos revisados

- `AGENTS.md`;
- `docs/00-governanca/guia-mestre-desenvolvimento.md`;
- `docs/00-governanca/regras-operacionais-do-projeto.md`;
- `docs/00-governanca/regras-operacionais-codex.md`;
- `docs/01-produto/visao-geral.md`;
- `docs/03-arquitetura/arquitetura-inicial.md`;
- `docs/04-planejamento/roadmap.md`;
- `docs/04-planejamento/plano-oficial-fase-0.md`;
- `docs/05-progresso/registro-de-decisoes.md`.

## Resultado geral

A base documental está coerente e suficiente para encerrar a Fase 0.

Os requisitos centrais aparecem de forma compatível entre os documentos:

- StepFlow como aplicativo interno de documentação operacional;
- experiência de início por duplo clique a partir do compartilhamento de rede;
- autenticação interna simples;
- perfis ADM, Gerência e Funcionário;
- processos com modelo de informação enxuto;
- etapas apresentadas como páginas de um manual/livro;
- JavaScript modular e rejeição de monólito HTML/JS;
- uso simultâneo por várias estações;
- separação lógica Client/Host/Data;
- SQLite acessado somente pelo Host;
- prevenção de sobrescrita silenciosa por revisão otimista;
- atualização dos clientes conectados;
- exportação PDF, DOCX e impressão como requisito de produto;
- documentação, gates e tarefas pequenas como método de execução.

## Ajustes identificados

### 1. PDF/DOCX

A exportação para PDF e DOCX é requisito do produto. A validação técnica da Fase 1 deverá escolher a estratégia e as bibliotecas adequadas; não decidir se a funcionalidade existirá.

### 2. Estado do checklist durante execução

A documentação oficial contém checklist como parte da etapa. Porém, o comportamento do estado marcado durante a execução — temporário por sessão, persistido por usuário ou registrado como execução — ainda não foi aprovado como decisão de produto.

Portanto, a Fase 1 deverá especificar esse comportamento antes da implementação correspondente.

### 3. Soft lock/presença

Indicador de que outro usuário está editando uma documentação permanece como possibilidade de UX, não requisito obrigatório. A integridade não pode depender dele.

### 4. Tecnologia

Tauri, protocolo de eventos, formato do Host, mecanismo de sessão, launcher/cópia local e bibliotecas de exportação permanecem corretamente identificados como propostas ou decisões técnicas pendentes.

## Checklist de coerência

- [x] Nenhum documento deve tratar Tauri como stack definitivamente aprovada.
- [x] Client/Host/Data está consolidado em nível lógico, mas detalhes tecnológicos permanecem abertos.
- [x] Multiusuário e concorrência aparecem como requisitos estruturais.
- [x] O cenário de duplo clique pela rede está definido como requisito de UX, não como implementação fechada.
- [x] O modelo simplificado de processos está consistente.
- [x] ADM, Gerência e Funcionário estão consistentes em nível de produto.
- [x] A metáfora de livro/páginas está preservada.
- [x] O roadmap mantém gates antes de implementação de negócio.
- [x] `AGENTS.md` está alinhado ao método PO + Assistente + Codex.
- [x] Pendências técnicas necessárias para a Fase 1 estão identificadas.

## Pendências deliberadamente transferidas para a Fase 1

- validação de Tauri e alternativas;
- faixa real de compatibilidade Windows;
- WebView/runtime e operação sem Internet;
- tecnologia e empacotamento do Host;
- launcher iniciado via SMB e eventual cópia local;
- descoberta do Host;
- protocolo Client ↔ Host e versionamento;
- canal de eventos;
- sessão e hash de senha;
- matriz final de permissões;
- modelo de dados conceitual e schema inicial;
- política de versão/revisão documental;
- comportamento do checklist durante execução;
- política de exclusão versus arquivamento;
- estratégia de PDF, DOCX e impressão;
- backup e restauração;
- especificação das telas críticas.

## Conclusão

A Fase 0 cumpriu sua finalidade: uma nova sessão do Assistente ou do Codex consegue compreender o produto, os papéis, as decisões consolidadas, as propostas ainda pendentes, a arquitetura lógica, o método de trabalho e a próxima fase sem depender do histórico da conversa.

A recomendação da revisão é **encerrar formalmente a Fase 0 e abrir a Fase 1 — Fechamento arquitetural e especificação**.