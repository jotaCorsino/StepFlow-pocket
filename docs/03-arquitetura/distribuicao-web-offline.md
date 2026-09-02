# Distribuição web e offline

## Hospedada

O build gera `dist/` puramente estático para upload no cPanel.

```text
index.html
manifest.webmanifest
sw.js
assets/*
icons/*
```

HTTPS é obrigatório para a experiência PWA/service worker.

## PWA

Após primeiro acesso, assets essenciais são cacheados. O usuário pode instalar/adicionar o StepFlow para abrir como aplicação e continuar sem rede.

A atualização do código é controlada pelo service worker. Dados não são sincronizados porque pertencem ao IndexedDB local.

## HTML portátil

O build pode gerar `StepFlow.html` autocontido para duplo clique.

Objetivos:

- abrir sem servidor;
- permitir navegação e operações do produto;
- carregar CSS/JS/assets embutidos;
- usar Exportar/Importar dados como mecanismo confiável de portabilidade.

Não prometer preservação baseada exclusivamente em storage de `file://`, pois origem/comportamento local varia entre browsers.

## Sem runtimes no servidor

Nenhum runtime ou executável próprio é publicado. Ferramentas de build existem apenas em desenvolvimento/CI.
