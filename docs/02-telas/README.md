# Telas e Superfícies — StepFlow Pocket

## Estado

**Bloco 8 da Fase 1 — EM ANDAMENTO.**

Esta pasta contém somente especificações de telas que estejam em análise ou consolidadas. Cada tela relevante deve usar `docs/templates/template-analise-de-tela.md`.

Uma especificação só vira contrato visual/funcional quando estiver explicitamente aprovada/consolidada.

## Especificações atuais

- `01-login.md` — **CONSOLIDADO FUNCIONALMENTE**;
- `02-shell-sidebar.md` — **NÚCLEO CONSOLIDADO / REABERTO PONTUALMENTE** para avaliar a nova área operacional;
- `03-dashboard.md` — **EM ANÁLISE / PROPOSTA PARA APROVAÇÃO DO PO**.

## Novo requisito confirmado

O StepFlow deve passar a suportar:

- categorização de procedimentos;
- registro de informações de serviço/equipamento em cenários aplicáveis;
- ficha técnica de computador/notebook com os dados solicitados pelo PO;
- busca por cliente/OS/identificadores úteis;
- resumo do trabalho/procedimentos realizados;
- ficha compacta imprimível.

Fonte: `docs/01-produto/categorizacao-atendimentos-equipamentos.md`.

## Modelagem/UX ainda em proposta

Ainda não é contrato:

- separar Procedimento × Atendimento/Execução × Equipamento;
- `Atendimentos` como nome/item próprio da sidebar;
- categorias múltiplas;
- equipamento reutilizável;
- múltiplos procedimentos por atendimento;
- vínculo à revisão exata utilizada.

O Bloco 8 pode avançar nos requisitos confirmados, mas não deve congelar essas escolhas sem aprovação do PO.

## Mapa de telas atual

1. Login — consolidado;
2. Shell/sidebar — núcleo consolidado, extensão operacional pendente;
3. Início/Dashboard — em análise;
4. Lista/pesquisa de procedimentos — incluir categorização;
5. Leitura em formato livro — mostrar categorização de forma discreta;
6. Editor de procedimento — permitir categorização conforme modelo aprovado;
7. Histórico;
8. lista/pesquisa de registros operacionais de serviço — **estrutura/nome pendentes**;
9. registro do serviço + ficha do equipamento — **estrutura pendente**;
10. Usuários/permissões;
11. Meu perfil;
12. Configurações + gestão de categorias;
13. Backup/restauração;
14. Exportação/impressão + ficha compacta;
15. estados transversais.

## Direção visual já aprovada

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

O Bloco 8 fecha UX, fluxo, estados, navegação, permissões visíveis e contrato visual/funcional apenas onde as decisões necessárias estiverem aprovadas.

Para os novos requisitos, pode definir:

- como categorias aparecem em listas/leitor/editor;
- como o usuário informa os dados de computador/equipamento;
- como busca cliente/OS/identificadores;
- onde vê o resumo do trabalho;
- onde aciona geração/impressão da ficha.

Não pode decidir sozinho:

- entidades finais e cardinalidades;
- lifecycle do registro operacional;
- checklist/progresso;
- matriz de permissões;
- tecnologia/formato da ficha compacta.

## Ordem de trabalho do Bloco 8

1. Login — consolidado;
2. Shell/sidebar — núcleo consolidado, extensão operacional em aprovação;
3. Dashboard — em análise;
4. Lista/pesquisa de procedimentos + categorização;
5. Leitor;
6. Editor + categorização;
7. Histórico;
8. superfícies do registro de serviço/equipamento após aprovação da modelagem;
9. Usuários/permissões;
10. Perfil;
11. Configurações + categorias;
12. Backup/restauração — UX/fluxo;
13. Exportação/impressão + ficha compacta — UX/fluxo;
14. estados transversais.

Não criar UI de produção antes da aprovação correspondente.

## Regra de parada

Quando a tela depender de modelagem/lifecycle/checklist/exportação ainda pendentes, documentar a dependência e parar no limite aprovado. Não inventar solução técnica ou regra de negócio.
