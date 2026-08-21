# Telas e Superfícies — StepFlow Pocket

## Estado

**Bloco 8 da Fase 1 — EM ANDAMENTO.**

Esta pasta contém somente especificações de telas que estejam em análise ou consolidadas. Cada tela relevante deve usar `docs/templates/template-analise-de-tela.md`.

Uma especificação só vira contrato visual/funcional quando estiver explicitamente aprovada/consolidada.

## Especificações atuais

- `01-login.md` — **CONSOLIDADO FUNCIONALMENTE**; identidade visual detalhada será compartilhada com o sistema visual do Shell;
- `02-shell-sidebar.md` — **NÚCLEO CONSOLIDADO / REABERTO PONTUALMENTE** para posicionar a nova área de Atendimentos;
- `03-dashboard.md` — **EM ANÁLISE / PROPOSTA PARA APROVAÇÃO DO PO**.

## Novo requisito transversal incorporado

O StepFlow agora deve suportar:

- categorização configurável/múltipla de procedimentos;
- atendimentos/execuções formais quando o caso exigir;
- equipamentos opcionais associados aos atendimentos;
- ficha técnica de computador/notebook quando aplicável;
- busca operacional por equipamento/cliente/OS/serial/patrimônio/MAC;
- ficha/relatório compacto imprimível.

Fonte: `docs/01-produto/categorizacao-atendimentos-equipamentos.md`.

## Mapa de telas atualizado

1. Login — contrato funcional consolidado;
2. Shell principal/sidebar — núcleo consolidado, ajuste de Atendimentos pendente;
3. Início/Dashboard — em análise;
4. Lista e pesquisa de procedimentos — incluir categorias;
5. Leitura do procedimento em formato livro — exibir categoria(s) de forma discreta;
6. Editor de procedimento/etapas — selecionar/manter categorias aplicáveis;
7. Histórico de alterações;
8. Lista e pesquisa de atendimentos;
9. Atendimento/execução + ficha de equipamento opcional;
10. Usuários e permissões;
11. Meu perfil;
12. Configurações da empresa + gestão de categorias;
13. Backup/restauração;
14. Exportação/impressão + ficha compacta de atendimento/equipamento;
15. estados transversais: Host indisponível, loading, vazio, erro, sem permissão e conflito.

## Direção visual já aprovada

- visual corporativo, limpo e discreto;
- sidebar esquerda persistente;
- logo pequeno no topo esquerdo, proporção preservada;
- sem topbar global redundante;
- perfil/avatar no rodapé da sidebar;
- leitura técnica como prioridade;
- procedimentos apresentados como manual/livro;
- comandos/blocos copiáveis com ícone discreto;
- feedback curto de cópia;
- Funcionário em experiência predominantemente de leitura/execução.

## Separação obrigatória na UX

Não misturar:

- `Processos/Procedimentos` — documentação/modelos oficiais reutilizáveis;
- `Atendimentos` — ocorrências reais de execução/serviço;
- `Equipamentos` — ficha física opcional utilizada pelos atendimentos.

Um técnico deve conseguir consultar um procedimento sem criar atendimento. Quando houver rastreabilidade operacional, o atendimento referencia o procedimento/revisão executada.

## Limite do Bloco 8

O Bloco 8 fecha **UX, fluxo, estados, navegação, permissões visíveis e contrato visual/funcional** das telas.

Para Atendimentos/Equipamentos, o Bloco 8 pode definir:

- onde iniciar/localizar um atendimento;
- campos e agrupamento visual da ficha;
- busca e filtros;
- vínculo visível com procedimentos;
- ação de gerar/imprimir ficha;
- estados de tela.

O Bloco 8 **não decide sozinho** lifecycle, persistência de checklist ou regras operacionais ainda pertencentes ao Bloco 9.

Para Backup/Restauração e Exportação/Impressão, o Bloco 8 fecha apenas UX/fluxo/estados. O Bloco 10 define a tecnologia e o formato físico da ficha compacta; o Bloco 11 fecha backup/restore.

## Ordem de trabalho do Bloco 8

1. Login — consolidado;
2. Shell/sidebar — núcleo consolidado; ajuste Atendimentos pendente;
3. Início/Dashboard — em análise;
4. Lista/pesquisa de procedimentos + categorias;
5. Leitor em formato livro;
6. Editor + categorias;
7. Histórico;
8. Atendimentos — lista/pesquisa;
9. Atendimento/execução + equipamento;
10. Usuários e permissões;
11. Meu perfil;
12. Configurações + categorias;
13. Backup/restauração — somente UX/fluxo;
14. Exportação/impressão + ficha compacta — somente UX/fluxo;
15. estados transversais.

Não criar UI de produção antes da especificação/aprovação correspondente.

## Regra de parada

Se uma tela exigir decisão ainda marcada como pendente em outro bloco — por exemplo, lifecycle do atendimento, persistência do checklist, política técnica de backup ou estratégia de exportação — especificar apenas o que já é conhecido e registrar a pendência. Não transformar a lacuna em requisito técnico por iniciativa própria.
