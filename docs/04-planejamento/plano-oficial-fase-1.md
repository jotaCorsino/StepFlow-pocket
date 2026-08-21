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
| 6 | Dados/schema/migrations | NÚCLEO CONCLUÍDO / EXTENSÃO OPERACIONAL PROPOSTA | `03-arquitetura/modelo-dados-schema-fase-1.md` |
| 7 | Concorrência/eventos | CONCLUÍDO NO NÚCLEO / detalhes operacionais dependem do Bloco 9 | `03-arquitetura/concorrencia-fila-conflitos-eventos.md` |
| 8 | UI/UX | EM ANDAMENTO | `02-telas/README.md` |
| 9 | Execução operacional/registro de serviço + checklist | PENDENTE | `01-produto/categorizacao-atendimentos-equipamentos.md` |
| 10 | Exportação/impressão + ficha compacta | PENDENTE | arquitetura técnica |
| 11 | Backup/restauração | PENDENTE | política técnica/operacional |
| 12 | Estrutura oficial + Fase 2 | PENDENTE | fundação do repositório |

## Novo requisito confirmado em 2026-08-21

O PO confirmou que o produto deve incluir:

- categorização de procedimentos;
- registro de dados específicos de computadores/notebooks quando aplicável;
- campos como nome, processador, RAM, armazenamento, SO/versão, MAC/identificador útil, saúde da bateria e observações;
- associação a cliente e/ou ordem de serviço/referência útil para busca;
- busca operacional por informações disponíveis;
- resumo dos procedimentos/trabalho realizados;
- ficha compacta gerável/imprimível para anexação física ao equipamento;
- uso amplo também em TI, Service Desk, Help Desk, infraestrutura, redes e guias.

Fonte: `docs/01-produto/categorizacao-atendimentos-equipamentos.md`.

## Modelagem recomendada — ainda não aprovada

A direção proposta é separar:

- Procedimento — modelo oficial;
- Atendimento/Execução — ocorrência real;
- Equipamento — ativo físico opcional.

Também estão em proposta:

- categorias configuráveis e potencialmente múltiplas;
- identificador interno/código StepFlow para equipamento;
- múltiplos procedimentos por atendimento;
- vínculo histórico à revisão efetivamente utilizada;
- item `Atendimentos` na sidebar.

Esses detalhes não viram contrato até aprovação do PO.

## Bloco 8 — UI/UX

Documentar/aprovar antes do código:

1. Login;
2. Shell/sidebar;
3. Dashboard;
4. lista/pesquisa de procedimentos com categorização;
5. leitura em formato livro;
6. editor de procedimento + categorização;
7. histórico;
8. lista/pesquisa do registro operacional de serviço;
9. registro do serviço + ficha do equipamento quando aplicável;
10. usuários/permissões;
11. perfil;
12. configurações da empresa + gestão de categorias;
13. backup/restauração;
14. exportação/impressão + ficha compacta;
15. estados transversais.

A nomenclatura/estrutura exatas das telas 8–9 dependem da aprovação da modelagem proposta.

### Estado atual

- `01-login.md` — consolidado;
- `02-shell-sidebar.md` — núcleo consolidado, reaberto apenas para a proposta de nova área operacional;
- `03-dashboard.md` — em análise/proposta;
- novo requisito incorporado antes de avançar às telas de Processos;
- nenhuma UI de produção criada.

Bloco 8 pode fechar UX/campos/fluxos já confirmados. Lifecycle, checklist e regras dependentes da modelagem devem aguardar Bloco 9.

## Bloco 9 — Execução operacional/registro de serviço e checklist

Primeiro fechar a modelagem da ocorrência real de trabalho. Se `Atendimento/Execução × Equipamento` for aprovada, detalhar:

- lifecycle mínimo necessário;
- criação/edição/conclusão/reabertura;
- equipamento opcional;
- vínculo com procedimento/revisão;
- possibilidade de um ou vários procedimentos;
- checklist/progresso;
- histórico operacional;
- concorrência/revisão;
- comportamento quando procedimento oficial muda;
- matriz operacional de permissões.

A solução deve permanecer proporcional, sem virar workflow burocrático.

## Bloco 10 — Exportação e impressão

PDF, DOCX e impressão continuam obrigatórios para documentação.

Novo requisito confirmado: ficha compacta de serviço/equipamento. Fechar:

- conteúdo final e identidade da empresa;
- tamanho/layout físico;
- impressão direta;
- necessidade ou não de PDF da ficha;
- paginação/listas/blocos;
- QR/barcode somente se houver valor aprovado;
- critérios de validação.

## Bloco 11 — Backup e restauração

Fechar backup consistente do SQLite + arquivos administrados. Depois da modelagem operacional aprovada, incluir os novos dados correspondentes.

## Bloco 12 — Fundação da Fase 2

Somente depois dos blocos anteriores:

- resolver parâmetros operacionais pendentes;
- definir árvore oficial de Client/Host/launcher/contratos/testes/assets;
- convenções/scripts;
- configuração de desenvolvimento;
- tarefas pequenas da fundação;
- plano oficial da Fase 2.

## Pendências do ambiente corporativo

- hostname/IP e paths reais;
- SMB/permissões/políticas;
- Windows/WebView2 reais;
- antivírus/EDR/firewall;
- HTTP/HTTPS;
- mecanismo real de start do Controller.

## Critérios de saída da Fase 1

- [x] Client/Windows definidos;
- [x] Host Pocket definido;
- [x] launcher/update definidos;
- [x] comunicação definida;
- [x] núcleo de autenticação/autorização definido;
- [ ] parâmetros finais de autenticação necessários à implementação;
- [x] modelo de dados original definido;
- [ ] extensão operacional do schema aprovada após decisão da modelagem nova;
- [x] concorrência geral definida;
- [ ] telas críticas, incluindo novos requisitos, especificadas/aprovadas;
- [ ] modelagem do registro operacional/equipamento aprovada;
- [ ] execução/checklist decididos;
- [ ] matriz operacional de permissões decidida;
- [ ] exportação/impressão + ficha compacta definidas;
- [ ] backup/restore definidos;
- [ ] estrutura oficial definida;
- [x] pendências não bloqueantes registradas;
- [ ] plano da Fase 2 aprovado.

## Regra de execução

Não criar scaffold, runtime definitivo ou código de negócio durante a Fase 1. Propostas do novo requisito não podem ser implementadas antes da aprovação explícita correspondente.

Toda tarefa Codex que altere arquivos deve trazer base Git esperada e respeitar `AGENTS.md`.
