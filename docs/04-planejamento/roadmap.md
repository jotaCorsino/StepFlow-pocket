# Roadmap — StepFlow

## R1 — Rebaseline documental

**EM ANDAMENTO.**

Eliminar contratos incompatíveis e fechar arquitetura web estática single-user.

## W1 — Fundação web

- source modular HTML/CSS/JS;
- build mínimo;
- testes básicos;
- dist estático.

## W2 — Shell e navegação

- sidebar;
- dashboard;
- routing local;
- componentes base;
- estados transversais.

## W3 — Persistência local

- IndexedDB;
- schema/migrations;
- camada repository/storage;
- testes transacionais.

## W4 — Procedimentos

- categorias;
- responsáveis locais;
- lista/busca;
- editor;
- revisões/histórico.

## W5 — Execução operacional

- Reader livro/manual;
- Atendimentos;
- checklist;
- observações por Etapa;
- Equipamentos.

## W6 — Exportação / Importação de dados

- formato versionado;
- validação;
- exportação integral;
- importação transacional.

## W7 — Impressão e PDF

- layouts A4;
- Procedimento;
- Ficha;
- browser print/Save as PDF.

## W8 — Offline e portátil

- manifest;
- service worker;
- PWA;
- artefato `StepFlow.html` quando viável.

## W9 — Hardening e publicação

- performance/bundle;
- segurança/CSP;
- compatibilidade de browser;
- publicação cPanel;
- documentação final.

Cada etapa usa branch/PR próprio e depende do gate da anterior.
