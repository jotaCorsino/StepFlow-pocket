# Template de Tarefa para o Codex

## Título

Nome curto e objetivo.

## 1. Objetivo

Uma única entrega central, verificável.

## 2. Contexto e fonte de verdade

Indicar somente documentos vigentes necessários, normalmente:

- `AGENTS.md`;
- `docs/README.md`;
- `docs/03-arquitetura/arquitetura-vigente.md` quando houver impacto arquitetural;
- documento específico da arquitetura/tela;
- `docs/05-progresso/registro-de-decisoes.md` quando houver decisão relacionada.

Não mandar o Codex reconstruir contexto a partir de tarefas/revisões históricas removidas da árvore ativa.

## 3. Estado inicial esperado

Descrever pré-condições, branch/checkout quando relevante e dependências já concluídas.

## 4. Escopo incluído

- item 1;
- item 2;
- item 3.

## 5. Fora do escopo

Registrar explicitamente o que não deve ser alterado, implementado ou investigado.

## 6. Regras e restrições

Incluir apenas o que se aplica:

- comportamento funcional;
- UI/UX aprovada;
- Pocket/distribuição;
- segurança/autorização;
- concorrência;
- persistência/migrations;
- compatibilidade;
- convenções arquiteturais;
- dependências aprovadas/proibidas.

## 7. Arquivos/áreas esperadas

Indicar áreas prováveis sem exigir alteração artificial de arquivo.

## 8. Critérios de aceite

- [ ] requisitos verificáveis da tarefa;
- [ ] nenhuma mudança fora do escopo;
- [ ] documentação necessária sincronizada.

## 9. Validações obrigatórias

Conforme aplicável:

- build;
- testes automatizados;
- lint/typecheck/formatter;
- smoke test;
- teste de autorização/conflito/duas instâncias;
- inspeção de artefatos;
- `git diff --check` e estado do repositório.

Não criar provas adicionais sem necessidade real do critério de aceite.

## 10. Documentação a atualizar

Listar exatamente o documento vivo afetado. Evitar criar arquivo de revisão/gate separado quando o resultado puder ser incorporado ao documento vigente e ao changelog/decisões de forma mais simples.

O diário é histórico e só deve ser alterado quando explicitamente incluído na tarefa e sem conflito local conhecido.

## 11. Relatório final obrigatório

1. objetivo executado;
2. arquivos criados/alterados/removidos;
3. decisões técnicas dentro do escopo;
4. validações executadas;
5. resultados;
6. riscos/limitações/pendências;
7. documentação atualizada;
8. próximos passos sugeridos, sem executá-los automaticamente.

## 12. Regra de parada

Se surgir decisão de produto/UX/arquitetura necessária e não coberta pela fonte vigente, não inventar. Reportar o bloqueio e opções para decisão do PO/Assistente.
