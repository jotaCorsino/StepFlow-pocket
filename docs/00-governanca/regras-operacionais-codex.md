# Regras Operacionais do Codex — StepFlow Pocket

## Objetivo

Complementar `AGENTS.md` com regras resumidas de execução técnica.

## Antes de atuar

O Codex deve:

1. ler `AGENTS.md`;
2. identificar a fase atual no roadmap;
3. ler a documentação específica indicada na tarefa;
4. confirmar mentalmente o que está dentro e fora do escopo;
5. não iniciar uma fase ou feature adicional por conta própria.

## Durante a tarefa

- Alterar apenas o necessário para o objetivo atual.
- Preservar comportamento existente não autorizado a mudar.
- Preservar UI/UX aprovada.
- Manter código modular e legível.
- Não criar dependências importantes sem autorização/documentação.
- Não transformar descoberta de débito técnico em refactor geral.
- Se encontrar conflito de produto ou arquitetura, registrar e sinalizar.
- Se o problema for bloqueante, resolver apenas o mínimo tecnicamente necessário dentro da autorização disponível ou reportar o bloqueio.

## Validação

Toda conclusão deve ser proporcionalmente comprovada por build, testes, smoke tests, inspeções ou evidências equivalentes adequadas ao tipo de mudança.

Não declarar “funciona” apenas porque o código foi escrito.

## Registro final obrigatório

Informar:

- objetivo executado;
- arquivos alterados;
- decisões técnicas tomadas;
- validações executadas e resultados;
- pendências/riscos;
- documentação atualizada;
- próximos passos sugeridos.

## Regra final

O Codex é executor técnico. Produto, prioridade, UX e mudanças estruturais fora da tarefa não devem ser decididos unilateralmente.
