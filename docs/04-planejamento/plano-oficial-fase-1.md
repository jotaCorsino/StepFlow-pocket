# Plano Oficial — Fase 1: Fechamento Arquitetural e Especificação

**Status:** EM ANDAMENTO  
**Início:** 2026-08-19  
**Atualização:** 2026-08-21

## Objetivo

Transformar requisitos e arquitetura conceitual em decisões e especificações implementáveis antes da fundação executável do StepFlow.

A Fase 1 autoriza documentação, decisões técnicas e provas descartáveis somente quando necessárias. Não autoriza antecipar funcionalidades de negócio definitivas nem criar scaffold/runtime oficial antes do Bloco 12.

## Estado dos blocos

| Bloco | Tema | Status | Fonte vigente |
|---|---|---|---|
| 0 | Bootstrap do ambiente | CONCLUÍDO | contexto/repositório validado |
| 1 | Client Windows/Tauri | CONCLUÍDO | `03-arquitetura/compatibilidade-windows-client.md` |
| 2 | Host Pocket | CONCLUÍDO | `03-arquitetura/host-pocket.md` |
| 3 | Launcher/distribuição | CONCLUÍDO | `03-arquitetura/launcher-distribuicao-client.md` |
| 4 | Comunicação Client↔Host | CONCLUÍDO | `03-arquitetura/comunicacao-client-host.md` |
| 5 | Autenticação/autorização | CONCLUÍDO NO NÚCLEO / PARÂMETROS PENDENTES | `03-arquitetura/autenticacao-sessao-autorizacao.md` |
| 6 | Dados/schema/migrations | CONCLUÍDO NO NÚCLEO; NOVA EXTENSÃO CONCEITUAL INCORPORADA | `03-arquitetura/modelo-dados-schema-fase-1.md` |
| 7 | Concorrência/eventos | CONCLUÍDO NO NÚCLEO; regras operacionais específicas dependem do Bloco 9 | `03-arquitetura/concorrencia-fila-conflitos-eventos.md` |
| 8 | UI/UX | EM ANDAMENTO | `02-telas/README.md` |
| 9 | Execução operacional, atendimentos e checklist | PENDENTE | `01-produto/categorizacao-atendimentos-equipamentos.md` |
| 10 | Exportação/impressão + ficha compacta | PENDENTE | arquitetura técnica |
| 11 | Backup/restauração | PENDENTE | política técnica/operacional |
| 12 | Estrutura oficial + Fase 2 | PENDENTE | fundação do repositório |

## Novo requisito incorporado em 2026-08-21

O produto passa a incluir:

- categorização configurável e múltipla de procedimentos;
- registro formal de atendimento/execução quando necessário;
- equipamento opcional associado ao atendimento;
- ficha de computador/notebook com dados técnicos quando aplicável;
- busca operacional por equipamento, cliente/referência, OS, serial/patrimônio/MAC e códigos internos;
- saída compacta imprimível de atendimento/equipamento.

Fonte de produto: `docs/01-produto/categorizacao-atendimentos-equipamentos.md`.

Esse requisito **não** transforma o StepFlow em CRM, help desk completo, estoque, RMM ou sistema financeiro.

## Bloco 8 — UI/UX

Documentar/aprovar antes do código as superfícies críticas:

1. Login;
2. Shell/sidebar;
3. Dashboard;
4. lista/pesquisa de procedimentos com categorias;
5. leitura em formato livro;
6. editor de procedimento/etapas + categorias;
7. histórico de alterações;
8. lista/pesquisa de atendimentos;
9. atendimento/execução e ficha de equipamento;
10. usuários/permissões;
11. perfil;
12. configurações da empresa + gestão de categorias;
13. backup/restauração;
14. exportação/impressão + ficha compacta;
15. estados transversais de erro/loading/conflito/Host indisponível.

Usar `docs/templates/template-analise-de-tela.md`. Aparência só vira contrato quando aprovada pelo PO.

### Estado atual

- `docs/02-telas/01-login.md` — contrato funcional consolidado;
- `docs/02-telas/02-shell-sidebar.md` — núcleo consolidado; **reaberto pontualmente** para posicionar a nova área `Atendimentos`;
- `docs/02-telas/03-dashboard.md` — em análise/proposta para aprovação do PO;
- requisitos de categorização/atendimento/equipamento incorporados antes de avançar às telas de Processos;
- nenhuma UI de produção foi criada.

No Bloco 8, telas operacionais podem definir UX, fluxo, campos, estados e permissões visíveis. Regras de lifecycle/checklist que dependem do Bloco 9 devem ficar marcadas como pendentes, não inventadas.

Para Backup/Restauração e Exportação/Impressão, o Bloco 8 fecha somente UX/fluxo/estados. Estratégia técnica de exportação e ficha compacta pertence ao Bloco 10; backup/restore ao Bloco 11.

## Bloco 9 — Execução operacional, atendimentos e checklist

Fechar o comportamento da ocorrência real de trabalho:

- lifecycle do atendimento (`novo/em andamento/concluído/...`) apenas na medida necessária;
- criação/edição/conclusão/reabertura;
- vínculo opcional com equipamento;
- vínculo com uma ou mais revisões de procedimentos executados;
- estado das marcações de checklist/progresso;
- histórico operacional;
- concorrência/revisão de atendimento/equipamento;
- regras quando o procedimento oficial muda durante/depois do atendimento;
- matriz de permissões operacional.

A solução deve permanecer simples e proporcional ao uso real, sem transformar atendimento em workflow burocrático.

## Bloco 10 — Exportação e impressão

PDF, DOCX e impressão continuam obrigatórios para documentação de procedimentos.

Além disso, fechar a saída compacta de atendimento/equipamento:

- geração própria, não screenshot;
- conteúdo e identidade da empresa;
- tamanho/layout físico;
- impressão direta e formatos de arquivo necessários;
- paginação/listas/blocos técnicos;
- se QR/barcode traz valor real ou permanece fora do escopo;
- critérios de validação.

## Bloco 11 — Backup e restauração

Fechar:

- backup consistente do SQLite + arquivos administrados;
- inclusão de categorias, equipamentos e atendimentos;
- formato do pacote;
- validação;
- retenção;
- coordenação durante restore;
- recuperação após falha.

## Bloco 12 — Fundação da Fase 2

Somente depois dos blocos anteriores:

- resolver parâmetros operacionais pendentes necessários à implementação;
- definir árvore oficial de Client/Host/launcher/contratos/testes/assets;
- convenções e scripts;
- arquivos ignorados/configuração de desenvolvimento;
- tarefas pequenas da fundação;
- plano oficial da Fase 2.

Até esse gate, “trabalho estrutural” não autoriza scaffold oficial nem código runtime de produção.

## Pendências do ambiente corporativo

Não bloqueiam a documentação atual, mas devem ser validadas antes da implantação:

- hostname/IP e paths reais;
- SMB/permissões/políticas;
- Windows/WebView2 reais;
- antivírus/EDR/firewall;
- transporte HTTP/HTTPS;
- mecanismo existente para iniciar o Controller central quando necessário.

## Critérios de saída da Fase 1

- [x] Client/Windows definidos;
- [x] Host Pocket definido;
- [x] launcher/update definidos arquiteturalmente;
- [x] comunicação definida;
- [x] núcleo de autenticação/autorização definido;
- [ ] parâmetros operacionais pendentes necessários à autenticação fechados antes da implementação correspondente;
- [x] modelo de dados/migrations definido no núcleo e extensão conceitual nova incorporada;
- [x] concorrência geral definida;
- [ ] telas críticas, incluindo categorias/atendimentos/equipamentos, especificadas/aprovadas;
- [ ] execução operacional e checklist decididos;
- [ ] matriz de permissões de categorias/equipamentos/atendimentos decidida;
- [ ] exportação/impressão + ficha compacta definidas;
- [ ] backup/restore definidos;
- [ ] estrutura oficial do repositório definida;
- [x] pendências não bloqueantes registradas;
- [ ] plano da Fase 2 aprovado.

## Regra de execução

Não criar scaffold, árvore runtime definitiva ou código de negócio durante a Fase 1 por conveniência. Provas técnicas novas só devem existir quando uma decisão relevante realmente depender de evidência mecânica não disponível e a tarefa as declarar explicitamente descartáveis.

Toda tarefa Codex que altere arquivos deve trazer base Git esperada (branch + SHA) e respeitar as proteções de working tree definidas em `AGENTS.md`.
