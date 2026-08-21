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
- `05-leitor-processo.md` — **CONSOLIDADO / APROVADO PELO PO**;
- `06-editor-processo.md` — **CONSOLIDADO / APROVADO PELO PO**;
- `07-historico-revisoes.md` — **CONSOLIDADO / APROVADO PELO PO**;
- próxima: **Tela 08 — Lista/Pesquisa de Atendimentos**.

A Tela 08 ainda não está em análise neste checkpoint.

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
5. Leitor em formato livro — consolidado;
6. Editor de Processo + categorias — consolidado;
7. Histórico/Revisões — consolidado;
8. Lista/pesquisa de Atendimentos — **próximo**;
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

## Lista/Pesquisa de Processos — consolidado

- lista/tabela compacta;
- busca por código, título ou termo;
- filtros Categoria + Área;
- Status somente quando útil;
- categorias múltiplas com semântica OU inicialmente;
- abertura padrão no Leitor;
- ações administrativas contextuais;
- `Arquivar` em vez de `Excluir`;
- retorno preserva busca/filtros.

## Leitor — consolidado

- `Visão geral` antes da Etapa 1;
- uma etapa por página;
- Sumário temporário;
- `Etapa X de Y` como posição, não conclusão;
- Anterior/Próxima;
- categorias discretas;
- checklist documental separado do operacional;
- nova revisão não substitui silenciosamente a aberta;
- ponto futuro `Iniciar atendimento`, sem antecipar Bloco 9.

Fonte: `05-leitor-processo.md`.

## Editor — consolidado

- `Informações` e `Etapas` separados;
- painel contextual `Estrutura`;
- salvamento explícito, sem autosave inicial;
- cada save aceito cria revisão imutável;
- blocos tipados, sem HTML livre;
- categorias selecionadas no Editor e gerenciadas fora dele;
- drag-and-drop apenas como complemento a ações acessíveis;
- conflito preserva alterações locais e nunca sobrescreve automaticamente;
- `Visualizar` usa última revisão salva;
- `Salvar` e `Publicar revisão atual` são ações distintas.

Fonte: `06-editor-processo.md`.

## Histórico/Revisões — consolidado

- lista cronológica compacta, mais recente primeiro;
- `revision_no` técnico separado de `display_version` editorial;
- badges `Atual` e `Publicada` representam ponteiros vigentes;
- revisão histórica abre no Leitor em somente leitura;
- revisão histórica é identificada claramente e não parece vigente;
- snapshots não podem ser editados/excluídos;
- `Criar nova revisão a partir desta` cria uma nova revisão, sem rollback destrutivo;
- conteúdo antigo só volta a ser publicado após virar nova revisão atual;
- diff visual não é obrigatório na primeira versão.

Fonte: `07-historico-revisoes.md`.

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

## Regra de acompanhamento

Todo avanço consolidado de fase, bloco ou tela deve atualizar o painel do `README.md` no mesmo checkpoint documental.

## Regra de parada

Quando uma tela depender de lifecycle/checklist/exportação ainda pendentes, documentar a dependência e parar no limite aprovado. Não inventar solução técnica ou regra de negócio.
