# Template de Tarefa para o Codex

## Título

Nome curto e objetivo da tarefa.

## 1. Objetivo

Descrever uma única entrega central verificável.

## 2. Contexto e fonte de verdade

Listar os documentos que devem ser lidos antes da execução.

Exemplo:

- `AGENTS.md`
- `docs/00-governanca/guia-mestre-desenvolvimento.md`
- documento específico da arquitetura/tela
- decisões relacionadas em `docs/05-progresso/registro-de-decisoes.md`

## 3. Estado inicial esperado

Descrever o que deve existir antes da tarefa e quais dependências já precisam estar concluídas.

## 4. Escopo incluído

- item 1;
- item 2;
- item 3.

## 5. Fora do escopo

Registrar explicitamente o que não deve ser alterado ou implementado nesta tarefa.

## 6. Regras e restrições

Incluir conforme aplicável:

- comportamento funcional obrigatório;
- restrições de UI/UX;
- compatibilidade;
- segurança;
- autorização;
- concorrência;
- persistência;
- performance;
- convenções arquiteturais;
- dependências proibidas ou já aprovadas.

## 7. Arquivos/áreas esperadas

Indicar áreas prováveis, sem obrigar mudanças artificiais em arquivos que não precisem ser tocados.

## 8. Critérios de aceite

- [ ] requisito verificável 1;
- [ ] requisito verificável 2;
- [ ] requisito verificável 3;
- [ ] nenhuma alteração fora do escopo;
- [ ] documentação obrigatória atualizada.

## 9. Validações obrigatórias

Definir comandos/evidências quando já forem conhecidos.

Exemplos:

- build;
- testes automatizados;
- lint/typecheck;
- smoke test;
- teste de duas instâncias;
- teste de autorização;
- teste de conflito de revisão;
- inspeção de arquivo gerado;
- conferência de `git diff`/estado do repositório.

## 10. Documentação a atualizar

Listar exatamente os documentos esperados, quando aplicável:

- diário de progresso;
- changelog;
- registro de decisões;
- documento de tela;
- arquitetura;
- roadmap;
- ADR.

## 11. Relatório final obrigatório

O Codex deve responder com:

1. objetivo executado;
2. arquivos criados/alterados/removidos;
3. decisões técnicas tomadas dentro do escopo;
4. validações executadas;
5. resultados;
6. riscos, limitações e pendências;
7. documentação atualizada;
8. próximos passos sugeridos, sem executá-los automaticamente.

## 12. Regra de parada

Se surgir uma decisão de produto/UX/arquitetura não coberta pela documentação e necessária para prosseguir corretamente, não inventar a regra. Registrar o bloqueio com opções e impacto para decisão do PO/assistente.
