# StepFlow

Aplicação web estática, single-user e local-first para documentar Procedimentos técnicos e registrar Atendimentos/Execuções de forma guiada.

## Estado atual

**Rebaseline arquitetural em andamento — PR #28.**

A nova arquitetura foi motivada pelo ambiente de hospedagem gerenciado por cPanel e pelo abandono do requisito multiusuário central.

Nenhuma implementação da nova arquitetura foi iniciada.

## Arquitetura

```text
cPanel / HTTPS
→ HTML + CSS + JavaScript + assets
→ navegador / PWA
→ IndexedDB local
→ Exportar / Importar dados
```

O servidor apenas entrega arquivos estáticos. O StepFlow não exige backend, banco remoto ou processo próprio no servidor.

## Produto

```text
Procedimento
   ↓ usado em
Atendimento / Execução
   ↓ opcionalmente relacionado a
Equipamento
```

Procedimentos preservam revisões e histórico. O Reader mantém experiência de manual/livro, uma Etapa por página lógica, passos, observações, checklist e blocos de instrução/comando com ação de copiar por ícone.

Atendimentos registram o trabalho real, incluindo checklist operacional, observações por Etapa, revisão exata utilizada e Equipamento opcional.

## Single-user

Não existem no baseline:

- login;
- usuários;
- permissões;
- sessões;
- sincronização entre dispositivos;
- concorrência distribuída.

`Responsável` continua existindo como dado local de negócio.

## Dados

IndexedDB guarda o conjunto funcional no navegador. A portabilidade e preservação manual acontecem por **Exportar dados / Importar dados**.

## Offline

A forma preferencial é PWA após primeiro acesso HTTPS. Uma variante `StepFlow.html` autocontida também pode ser gerada para uso portátil por duplo clique.

## Consumo

O projeto prioriza:

- bundle pequeno;
- Web Platform nativa;
- poucas dependências;
- zero processo de servidor;
- zero banco no servidor;
- zero conexão permanente;
- fonte do sistema;
- SVG local;
- impressão/PDF pelos recursos do navegador.

## Fontes de verdade

- `AGENTS.md`;
- `docs/README.md`;
- `docs/05-progresso/registro-de-decisoes.md`;
- `docs/03-arquitetura/arquitetura-vigente.md`;
- `docs/04-planejamento/rebaseline-web-estatico-single-user.md`;
- `docs/04-planejamento/roadmap.md`.
