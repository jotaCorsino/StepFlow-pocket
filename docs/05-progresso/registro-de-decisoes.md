# Registro de decisões — StepFlow

**Atualização:** 2026-09-02

## Estado atual

O projeto está em rebaseline arquitetural no PR #28.

## Decisões vigentes do rebaseline

- aplicação web estática;
- single-user;
- local-first;
- cPanel serve somente arquivos estáticos;
- IndexedDB é persistência principal;
- PWA/offline é a forma preferencial de uso sem rede;
- `StepFlow.html` autocontido pode existir como saída portátil opcional;
- Exportar/Importar dados substitui mecanismos de preservação anteriores;
- sem login, usuários, permissões, sessão ou sincronização;
- sem backend obrigatório, banco remoto, API própria ou conexão persistente;
- impressão/PDF via HTML/CSS do navegador;
- dependências mínimas e medidas;
- Web Platform nativa tem precedência sobre bibliotecas.

## Produto preservado

- Procedimentos;
- revisões e histórico;
- categorias;
- Reader livro/manual;
- Atendimentos;
- checklist operacional;
- observações por Etapa;
- Equipamentos opcionais;
- identidade da empresa;
- Responsáveis como dados locais, não contas.

## Candidatos open source

- `dexie/Dexie.js` para IndexedDB;
- `101arrowz/fflate` para compactação quando necessária;
- `evanw/esbuild` para build/dev.

Adoção final depende de necessidade concreta, licença, atividade do projeto e impacto no bundle.

## Gate

Nenhuma implementação começa antes da aprovação/consolidação do PR #28 e sincronização segura do checkout local.
