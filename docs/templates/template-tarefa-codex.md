# Template de Tarefa para o Codex

## Título

Nome curto e objetivo.

## 1. Objetivo

Uma única entrega central, verificável.

## 2. Base Git obrigatória

Toda tarefa que permita alteração deve informar explicitamente:

```text
branch/base esperada: <branch>
commit esperado: <SHA completo ou abreviado não ambíguo>
```

Antes de escrever, o Codex deve executar:

```text
git rev-parse HEAD
git status --short --branch
```

Se `HEAD` não corresponder ao commit esperado, parar e reportar. Não fazer `pull`, `merge`, `rebase`, `reset` ou checkout corretivo por iniciativa própria.

Qualquer modificação preexistente no working tree pertence ao PO/outro fluxo. Não resetar, limpar, stashar, restaurar, sobrescrever nem incluir essas alterações no commit da tarefa. Se a tarefa precisar tocar um arquivo já modificado, parar e reportar o conflito.

## 3. Contexto e fonte de verdade

Indicar somente documentos vigentes necessários, normalmente:

- `AGENTS.md`;
- `docs/README.md`;
- documento específico da arquitetura/tela/fase;
- `docs/05-progresso/registro-de-decisoes.md` quando houver decisão relacionada;
- `docs/03-arquitetura/arquitetura-vigente.md` quando houver impacto arquitetural;
- `docs/00-governanca/contexto-ambientes.md` quando houver ambiente/rede/instalação/toolchain.

Não mandar o Codex reconstruir contexto a partir de tarefas/revisões históricas removidas da árvore ativa.

O prompt define o escopo autorizado, mas não revoga silenciosamente decisões vigentes. Se houver conflito com `AGENTS.md`, decisão consolidada ou documento específico vigente, parar, salvo quando o próprio enunciado declarar nova decisão aprovada pelo PO e exigir a atualização documental correspondente.

## 4. Estado inicial esperado

Descrever pré-condições, dependências concluídas e qualquer alteração local conhecida que deva permanecer intocada.

## 5. Escopo incluído

- item 1;
- item 2;
- item 3.

## 6. Fora do escopo

Registrar explicitamente o que não deve ser alterado, implementado ou investigado.

## 7. Regras e restrições

Incluir apenas o que se aplica:

- comportamento funcional;
- UI/UX aprovada;
- Pocket/distribuição;
- segurança/autorização;
- concorrência;
- persistência/migrations;
- compatibilidade;
- convenções arquiteturais;
- dependências aprovadas/proibidas;
- ambiente em que a validação realmente é aplicável.

Não converter parâmetro marcado como `PROPOSTA`, `PENDENTE`, “sujeito a ajuste” ou equivalente em comportamento definitivo sem autorização expressa.

## 8. Arquivos/áreas esperadas

Indicar áreas prováveis sem exigir alteração artificial de arquivo.

## 9. Critérios de aceite

- [ ] base Git inicial corresponde à tarefa;
- [ ] alterações preexistentes foram preservadas;
- [ ] requisitos verificáveis da tarefa foram atendidos;
- [ ] nenhuma mudança fora do escopo;
- [ ] documentação necessária sincronizada.

## 10. Validações obrigatórias

Conforme aplicável:

- build;
- testes automatizados;
- lint/typecheck/formatter;
- smoke test;
- teste de autorização/conflito/duas instâncias;
- inspeção de artefatos;
- `git diff --check`;
- `git status --short --branch`;
- `git diff --name-only` para confirmar o conjunto de arquivos alterados.

Não criar provas adicionais sem necessidade real do critério de aceite.

Se uma validação depender de capacidade indisponível no sandbox Codex, não modificar ACL, segurança, registro, PATH global ou ferramentas do sistema para “consertar” o sandbox. Reportar a limitação e, quando necessário, indicar execução controlada na sessão normal do PO.

## 11. Documentação a atualizar

Listar exatamente o documento vivo afetado. Evitar criar arquivo de revisão/gate separado quando o resultado puder ser incorporado ao documento vigente e ao changelog/decisões de forma mais simples.

O diário é histórico e só deve ser alterado quando explicitamente incluído na tarefa e sem conflito local conhecido.

## 12. Relatório final obrigatório

1. objetivo executado;
2. base Git e estado inicial observados;
3. alterações preexistentes preservadas;
4. arquivos criados/alterados/removidos pela tarefa;
5. decisões técnicas tomadas dentro do escopo;
6. validações executadas;
7. resultados;
8. riscos/limitações/pendências;
9. documentação atualizada;
10. próximos passos sugeridos, sem executá-los automaticamente.

## 13. Regras de parada

Parar e reportar quando ocorrer qualquer uma destas situações:

- `HEAD` diferente da base esperada;
- arquivo necessário já modificado antes da tarefa;
- conflito entre prompt e decisão/documento vigente;
- decisão de produto/UX/arquitetura necessária e não coberta;
- necessidade de tratar valor provisório como definitivo;
- operação essencial exigir credenciais/elevação/configuração global fora do ambiente autorizado;
- conclusão correta exigir ampliar materialmente o escopo.

Não inventar solução para atravessar essas barreiras.
