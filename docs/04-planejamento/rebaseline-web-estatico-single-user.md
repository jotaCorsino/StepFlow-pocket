# Rebaseline — StepFlow web estático single-user

**Status:** EM ANÁLISE / PR #28  
**Data:** 2026-09-02

## Motivo

O ambiente de produção é cPanel gerenciado e não aceita executáveis próprios do StepFlow. O produto também deixa de perseguir multiusuário central.

## Nova arquitetura

```text
cPanel / HTTPS
→ arquivos estáticos
→ Browser / PWA
→ IndexedDB local
→ Exportar / Importar dados
```

Sem backend obrigatório, banco remoto, autenticação, sessões, sincronização, WebSocket ou processos de servidor.

## Distribuição

Principal: site estático por HTTPS com PWA/offline.

Opcional: `StepFlow.html` autocontido para duplo clique. A portabilidade confiável dos dados depende de Exportar/Importar, não de garantias de storage em `file://`.

## Persistência

IndexedDB é o datastore principal. `dexie/Dexie.js` é candidata para encapsular schema/migrations/transações se o custo no bundle for aceitável.

## Dados

Permanecem Procedimentos, Categorias, Responsáveis locais, Atendimentos, Equipamentos, configuração da empresa, revisões e histórico.

## Exportação / Importação

Substitui qualquer mecanismo anterior de preservação. O arquivo deve ser versionado, validável e transacional na importação. Compactação com `101arrowz/fflate` é opcional e somente entra se justificar o custo.

## Documentos

Impressão/PDF passam a usar HTML/CSS de impressão + `window.print()`. DOCX automático fica fora do baseline inicial.

## Recursos

Prioridade absoluta para baixo consumo: Web Platform nativa, sem framework pesado, sem CDN obrigatória, fonte do sistema, SVG local e zero timers/conexões permanentes.

## Sequência proposta

```text
R1 rebaseline documental
→ W1 workspace web + build estático
→ W2 shell/UI + routing
→ W3 IndexedDB + migrations
→ W4 Procedimentos/Categorias/Responsáveis
→ W5 Reader/Atendimentos/Equipamentos
→ W6 Exportar/Importar dados
→ W7 impressão/PDF
→ W8 PWA/offline + HTML portátil
→ W9 hardening/performance + publicação cPanel
```

Nenhuma implementação começa antes da aprovação e consolidação deste rebaseline.
