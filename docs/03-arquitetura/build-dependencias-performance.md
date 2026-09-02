# Build, dependências e performance

## Runtime

HTML/CSS/JavaScript moderno sem framework SPA obrigatório.

## Build

Um bundler pequeno como `evanw/esbuild` pode ser usado apenas em desenvolvimento para:

- resolver módulos;
- minificar;
- tree-shake;
- gerar hashes;
- produzir variante hospedada e HTML autocontido.

Nenhum Node/runtime de build vai para produção.

## Dependências candidatas

- `dexie/Dexie.js` — IndexedDB, schema, migrations e transações locais;
- `101arrowz/fflate` — compactação/arquivo único quando a medição justificar;
- `SortableJS/Sortable` — reordenação robusta de Etapas/passos por mouse/touch, somente se a implementação nativa não atender bem;
- `evanw/esbuild` — build/dev apenas.

A política é **não adotar todas automaticamente**. Cada dependência precisa de necessidade concreta, versão pinada, licença permissiva compatível, projeto ativo e impacto medido no bundle.

Funcionalidades simples permanecem nativas: routing, dialogs, clipboard, download via Blob, impressão, UUID via `crypto.randomUUID()`, busca básica e ícones SVG não exigem biblioteca por padrão.

## Orçamento inicial

Não fixar números artificiais antes do primeiro build. O gate é comparativo: qualquer dependência nova deve provar valor e não pode dominar o tamanho/custo de execução.

## Práticas

- fonte do sistema;
- SVG inline/local;
- sem icon font;
- sem CDN obrigatória;
- imagens otimizadas;
- event listeners e observers somente quando úteis;
- sem timers periódicos desnecessários;
- paginação/virtualização apenas para listas grandes.
