# Telas e Superfícies — StepFlow Pocket

## Estado

**Bloco 8 da Fase 1 — EM ANDAMENTO.**

Esta pasta contém somente especificações de telas que estejam em análise ou consolidadas. Cada tela relevante deve usar `docs/templates/template-analise-de-tela.md`.

Uma especificação só vira contrato visual/funcional quando estiver explicitamente aprovada/consolidada.

## Especificações atuais

- `01-login.md` — **CONSOLIDADO FUNCIONALMENTE**; identidade visual detalhada será compartilhada com o Shell;
- `02-shell-sidebar.md` — **EM ANÁLISE / PROPOSTA PARA APROVAÇÃO DO PO**.

## Mapa de telas

1. Login — contrato funcional consolidado;
2. Shell principal/sidebar — em análise;
3. Início/Dashboard;
4. Lista e pesquisa de processos;
5. Leitura do processo em formato livro;
6. Editor de processo/etapas;
7. Histórico de alterações;
8. Usuários e permissões;
9. Meu perfil;
10. Configurações da empresa;
11. Backup/restauração;
12. Exportação/impressão;
13. estados transversais: Host indisponível, loading, vazio, erro, sem permissão e conflito.

## Direção visual já aprovada

- visual corporativo, limpo e discreto;
- sidebar esquerda;
- logo pequeno no topo esquerdo, proporção preservada;
- leitura técnica como prioridade;
- processos apresentados como manual/livro;
- comandos/blocos copiáveis com ícone discreto;
- feedback curto de cópia;
- Funcionário em experiência predominantemente de leitura/execução.

## Limite do Bloco 8

O Bloco 8 fecha **UX, fluxo, estados, navegação, permissões visíveis e contrato visual/funcional** das telas.

Para Backup/Restauração e Exportação/Impressão, o Bloco 8 pode definir:

- onde o usuário acessa a função;
- quais ações e informações aparecem;
- confirmações, feedbacks e estados de erro;
- permissões percebidas na UI.

O Bloco 8 **não escolhe** bibliotecas, formato interno de pacote, mecanismo de geração de PDF/DOCX, estratégia de backup SQLite, retenção ou detalhes técnicos que pertencem aos Blocos 10 e 11. Se a especificação de tela depender desses detalhes, registrar a dependência sem inventar a solução.

## Ordem de trabalho do Bloco 8

1. Login — consolidado funcionalmente;
2. Shell/sidebar — em análise;
3. Início/Dashboard;
4. Lista/pesquisa;
5. Leitor em formato livro;
6. Editor;
7. Histórico;
8. Usuários e permissões;
9. Meu perfil;
10. Configurações da empresa;
11. Backup/restauração — somente UX/fluxo;
12. Exportação/impressão — somente UX/fluxo;
13. estados transversais.

Não criar UI de produção antes da especificação/aprovação correspondente.

## Regra de parada

Se uma tela exigir decisão ainda marcada como pendente em outro bloco — por exemplo, persistência do checklist, política técnica de backup ou estratégia de exportação — especificar apenas o que já é conhecido e registrar a pendência. Não transformar a lacuna em requisito técnico por iniciativa própria.
