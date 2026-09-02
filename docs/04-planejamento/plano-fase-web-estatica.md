# Plano — Fase web estática

## Objetivo

Entregar o StepFlow como aplicação estática single-user, local-first, offline-capable e de baixo consumo.

## Gates gerais

- nenhuma arquitetura de servidor próprio;
- nenhum requisito multiusuário;
- nenhuma biblioteca sem justificativa/tamanho medido;
- nenhuma perda silenciosa de dados;
- dados exportáveis/importáveis;
- source modular;
- produção somente estática.

## Gate atual

1. concluir PR #28;
2. validar que a documentação ativa não contém contratos obsoletos;
3. squash merge;
4. apagar branch;
5. verificar remoto limpo;
6. sincronizar checkout local com segurança;
7. preparar pré-flight W1.

## Critério de conclusão do rebaseline

- README/AGENTS/índice coerentes;
- telas sem login/usuários/permissões;
- arquitetura sem processos/servidor/banco remoto;
- persistência local definida;
- Exportar/Importar dados definido;
- roadmap novo;
- nenhum documento ativo dependente da arquitetura substituída.
