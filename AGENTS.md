# AGENTS.md — StepFlow

## Estado

StepFlow está em rebaseline para uma aplicação web estática, single-user e local-first.

- produção: arquivos estáticos servidos por HTTPS;
- persistência: IndexedDB local;
- offline: PWA/service worker;
- portabilidade: Exportar/Importar dados;
- sem backend obrigatório;
- sem banco remoto;
- sem login/usuários/permissões/sessões;
- sem sincronização/multiusuário.

Nenhuma implementação da nova arquitetura começa antes do fechamento do PR de rebaseline.

## Precedência

1. `AGENTS.md`;
2. `docs/05-progresso/registro-de-decisoes.md`;
3. documento específico vigente;
4. `docs/01-produto/visao-geral.md`;
5. tarefa aprovada;
6. histórico Git.

## Disciplina Git

```text
1 trabalho lógico
→ 1 branch
→ draft PR
→ revisão/aprovação
→ squash merge
→ apagar branch
→ verificar remoto limpo
```

Toda tarefa que altera arquivos informa base/branch/SHA esperado.

Antes de escrever:

```text
git rev-parse HEAD
git status --short --branch
```

Sem autorização explícita é proibido descartar trabalho local, usar `reset --hard`, `clean`, `stash` ou sobrescrever alterações preexistentes.

## Trabalho assistido

Antes de cada tarefa Codex:

1. `PRÉ-FLIGHT PARA O PO — NÃO ENVIAR AO CODEX`;
2. `PROMPT / ENUNCIADO PARA O CODEX` separado.

Codex executa o escopo aprovado e não inventa produto, dependência ou arquitetura.

## Invariantes técnicas

- HTML semântico, CSS próprio e JavaScript moderno;
- source modular por feature/componente/serviço;
- não criar HTML/JS monolítico como fonte;
- IndexedDB como datastore principal;
- Web Platform nativa antes de biblioteca;
- dependências pequenas, ativas, permissivas, pinadas e medidas no bundle;
- nenhuma CDN obrigatória em produção;
- nenhuma conexão permanente/polling sem requisito futuro explícito;
- fonte do sistema e SVG local/inline por padrão;
- service worker somente para assets/offline, não sincronização de dados;
- importação trata arquivo como não confiável;
- impressão/PDF baseline pelo navegador;
- dados funcionais não são enviados automaticamente para servidor externo.

## Produto

Preservar:

- Procedimentos versionados;
- categorias;
- Reader livro/manual com uma Etapa por página lógica;
- passos, observações, checklist e blocos de comando com copiar icon-only;
- Atendimentos/Execuções;
- Equipamentos opcionais;
- checklist operacional e observação por Etapa em Atendimento;
- histórico reproduzível;
- configuração da empresa;
- Responsáveis locais como dados, não contas.

## Regra final

Quando um contrato antigo divergir da documentação vigente, a documentação vigente prevalece. O histórico antigo permanece no Git e não deve ser reintroduzido por inferência.
