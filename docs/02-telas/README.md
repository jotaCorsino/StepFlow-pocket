# Telas e Superfícies — StepFlow Pocket

## Estado

**Bloco 8 da Fase 1 — EM ANDAMENTO.**

Esta pasta contém especificações de telas em análise ou consolidadas. Cada tela relevante usa `docs/templates/template-analise-de-tela.md`.

Uma especificação só vira contrato visual/funcional quando explicitamente aprovada/consolidada.

## Especificações atuais

- `01-login.md` — **CONSOLIDADO FUNCIONALMENTE**;
- `02-shell-sidebar.md` — **CONSOLIDADO**, incluindo `Atendimentos`;
- `03-dashboard.md` — **CONSOLIDADO**;
- `04-lista-pesquisa-processos.md` — **EM ANÁLISE / PROPOSTA PARA APROVAÇÃO DO PO**.

## Domínio operacional aprovado

O StepFlow passa a distinguir:

- `Processos` — documentação/modelos oficiais;
- `Atendimentos` — ocorrências reais de execução/serviço;
- `Equipamento` — entidade opcional relacionada aos atendimentos quando aplicável.

Também estão aprovados:

- categorias configuráveis e múltiplas para procedimentos;
- identidade interna própria do equipamento, sem depender exclusivamente de MAC/serial/patrimônio;
- múltiplos procedimentos por atendimento;
- vínculo histórico à revisão realmente utilizada;
- ficha compacta imprimível de atendimento/equipamento.

Fonte: `docs/01-produto/categorizacao-atendimentos-equipamentos.md`.

## Mapa de telas atual

1. Login — consolidado;
2. Shell/sidebar — consolidado;
3. Início/Dashboard — consolidado;
4. Lista/pesquisa de processos — em análise, com categorização;
5. Leitura em formato livro — mostrar categorização de forma discreta;
6. Editor de processo — permitir categorização;
7. Histórico;
8. Lista/pesquisa de Atendimentos;
9. Atendimento/execução + ficha do equipamento;
10. Usuários/permissões;
11. Meu perfil;
12. Configurações + gestão de categorias;
13. Backup/restauração;
14. Exportação/impressão + ficha compacta;
15. estados transversais.

## Direção visual aprovada

- visual corporativo, limpo e discreto;
- sidebar esquerda persistente;
- logo pequeno no topo esquerdo;
- sem topbar global redundante;
- perfil/avatar no rodapé;
- leitura técnica como prioridade;
- procedimentos como manual/livro;
- blocos copiáveis com ícone discreto;
- feedback curto de cópia;
- Funcionário predominantemente em leitura/execução.

## Limite do Bloco 8

O Bloco 8 fecha UX, fluxo, estados, navegação, permissões visíveis e contrato visual/funcional.

Pode definir:

- como categorias aparecem em listas/leitor/editor;
- como o usuário informa dados de atendimento/equipamento;
- como pesquisa cliente/OS/identificadores na área `Atendimentos`;
- onde vê resumo do trabalho;
- onde aciona geração/impressão da ficha.

Não pode decidir sozinho:

- lifecycle/status final do atendimento;
- checklist/progresso operacional;
- matriz de permissões operacional;
- tecnologia/formato final da ficha compacta.

Esses pontos pertencem aos Blocos 9 e 10.

## Ordem de trabalho do Bloco 8

1. Login — consolidado;
2. Shell/sidebar — consolidado;
3. Dashboard — consolidado;
4. Lista/pesquisa de processos + categorização — **em análise**;
5. Leitor;
6. Editor + categorização;
7. Histórico;
8. Lista/pesquisa de Atendimentos;
9. Atendimento + equipamento;
10. Usuários/permissões;
11. Perfil;
12. Configurações + categorias;
13. Backup/restauração — UX/fluxo;
14. Exportação/impressão + ficha compacta — UX/fluxo;
15. estados transversais.

Não criar UI de produção antes da aprovação correspondente.

## Regra de separação de busca

- `Processos`: código, título, termo, área, categoria e demais metadados documentais aprovados;
- `Atendimentos`: código de atendimento, OS/referência, cliente, equipamento, serial/patrimônio/MAC e dados operacionais.

Não misturar os dois domínios em uma única pesquisa global sem requisito explícito.

## Regra de parada

Quando a tela depender de lifecycle/checklist/exportação ainda pendentes, documentar a dependência e parar no limite aprovado. Não inventar solução técnica ou regra de negócio.