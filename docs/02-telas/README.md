# Telas e Superfícies — StepFlow Pocket

## Estado

**Bloco 8 da Fase 1 — EM ANDAMENTO.**

Esta pasta contém especificações de telas em análise ou consolidadas. Cada tela relevante usa `docs/templates/template-analise-de-tela.md`.

Uma especificação só vira contrato visual/funcional quando explicitamente aprovada/consolidada.

## Especificações atuais

- `01-login.md` — **CONSOLIDADO FUNCIONALMENTE**;
- `02-shell-sidebar.md` — **CONSOLIDADO**, incluindo `Atendimentos`;
- `03-dashboard.md` — **CONSOLIDADO**;
- `04-lista-pesquisa-processos.md` — **CONSOLIDADO / APROVADO PELO PO**;
- próxima: **Tela 05 — Leitor em formato livro**.

## Domínio operacional aprovado

O StepFlow distingue:

- `Processos` — documentação/modelos oficiais;
- `Atendimentos` — ocorrências reais de execução/serviço;
- `Equipamento` — entidade opcional relacionada aos atendimentos quando aplicável.

Também estão aprovados:

- categorias configuráveis e múltiplas;
- identidade interna própria do equipamento;
- múltiplos procedimentos por atendimento;
- vínculo histórico à revisão realmente utilizada;
- ficha compacta imprimível de atendimento/equipamento.

Fonte: `docs/01-produto/categorizacao-atendimentos-equipamentos.md`.

## Mapa de telas

1. Login — consolidado;
2. Shell/sidebar — consolidado;
3. Início/Dashboard — consolidado;
4. Lista/pesquisa de Processos — consolidado;
5. Leitor em formato livro — **próximo**;
6. Editor de Processo + categorias;
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

## Lista/Pesquisa de Processos — decisões consolidadas

- lista/tabela compacta;
- busca por código, título ou termo;
- filtros principais por Categoria e Área;
- Status somente quando útil ao perfil;
- categorias múltiplas com semântica OU inicialmente;
- categorias como labels/chips discretos;
- abertura padrão no leitor;
- ações administrativas contextuais;
- `Arquivar` em vez de `Excluir` na operação normal;
- retorno do leitor preserva pesquisa/filtros;
- busca documental separada da busca operacional de Atendimentos.

## Limite do Bloco 8

O Bloco 8 fecha UX, fluxo, estados, navegação, permissões visíveis e contrato visual/funcional.

Não decide sozinho:

- lifecycle/status final do Atendimento;
- checklist/progresso operacional;
- matriz de permissões operacional;
- tecnologia/formato final da ficha compacta.

Esses pontos pertencem aos Blocos 9 e 10.

## Regra de separação de busca

- `Processos`: código, título, termo, área, categoria e metadados documentais aprovados;
- `Atendimentos`: código de atendimento, OS/referência, cliente, equipamento, serial/patrimônio/MAC e dados operacionais.

Não misturar os dois domínios em pesquisa global sem requisito explícito.

## Regra de parada

Quando uma tela depender de lifecycle/checklist/exportação ainda pendentes, documentar a dependência e parar no limite aprovado. Não inventar solução técnica ou regra de negócio.
