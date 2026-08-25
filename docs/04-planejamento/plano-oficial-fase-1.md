# Plano Oficial — Fase 1: Fechamento Arquitetural e Especificação

**Status:** EM ANDAMENTO  
**Início:** 2026-08-19  
**Atualização:** 2026-08-25

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
| 6 | Dados/schema/migrations | NÚCLEO + EXTENSÃO OPERACIONAL CONCEITUALMENTE CONSOLIDADOS | `03-arquitetura/modelo-dados-schema-fase-1.md` |
| 7 | Concorrência/eventos | CONCLUÍDO NO NÚCLEO / detalhes operacionais dependem do Bloco 9 | `03-arquitetura/concorrencia-fila-conflitos-eventos.md` |
| 8 | UI/UX | EM ANDAMENTO | `02-telas/README.md` |
| 9 | Execução operacional/Atendimentos + checklist | PENDENTE | `01-produto/categorizacao-atendimentos-equipamentos.md` |
| 10 | Exportação/impressão + ficha compacta | PENDENTE | arquitetura técnica |
| 11 | Backup/restauração | PENDENTE | política técnica/operacional |
| 12 | Estrutura oficial + Fase 2 | PENDENTE | fundação do repositório |

## Extensão de produto consolidada

Ficam incorporados à Fase 1:

- categorização configurável e múltipla de procedimentos;
- separação `Procedimento × Atendimento/Execução × Equipamento`;
- `Atendimentos` como área operacional própria;
- equipamento opcional/reutilizável com identidade interna própria;
- MAC/serial/patrimônio/cliente/OS como atributos pesquisáveis;
- múltiplos procedimentos por atendimento;
- vínculo histórico com a revisão de procedimento utilizada;
- ficha compacta imprimível para atendimento/equipamento;
- uso amplo em manutenção, TI, Service Desk, Help Desk, infraestrutura, redes e guias;
- identidade da empresa centralizada para Shell e documentos;
- gestão simples de categorias em Configurações, com arquivamento/reativação e preservação de histórico.

Fonte: `docs/01-produto/categorizacao-atendimentos-equipamentos.md` e especificações do Bloco 8.

## Bloco 8 — UI/UX

Documentar/aprovar antes do código:

1. Login;
2. Shell/sidebar;
3. Dashboard;
4. lista/pesquisa de Processos com categorização;
5. leitura em formato livro;
6. editor de Processo + categorização;
7. histórico;
8. lista/pesquisa de Atendimentos;
9. Atendimento/execução + ficha do equipamento;
10. usuários/permissões;
11. perfil;
12. configurações da empresa + gestão de categorias;
13. backup/restauração;
14. exportação/impressão + ficha compacta;
15. estados transversais.

### Estado atual

- Telas 01–12 — **consolidadas/aprovadas**;
- `12-configuracoes-categorias.md` consolida identidade da empresa e gestão simples de categorias;
- **próximo: Tela 13 — Backup/Restauração — UX**;
- Tela 14 — Exportação/Impressão + ficha — pendente;
- Tela 15 — Estados transversais — pendente;
- nenhuma UI de produção criada.

Lifecycle, checklist e permissões operacionais detalhadas permanecem para o Bloco 9. Tecnologia/formato final da ficha compacta permanece para o Bloco 10. Política técnica de Backup/Restore permanece para o Bloco 11.

## Bloco 9 — Execução operacional/Atendimentos e checklist

Fechar:

- lifecycle mínimo necessário;
- criação/edição/conclusão/reabertura;
- equipamento opcional;
- vínculo com uma ou mais revisões de procedimentos;
- checklist/progresso;
- histórico operacional;
- concorrência/revisão;
- comportamento quando procedimento oficial muda;
- matriz operacional de permissões;
- formato final dos códigos legíveis;
- capacidade/preset exato para gestão operacional de categorias quando aplicável ao contrato de autorização.

A solução deve permanecer proporcional e não virar workflow burocrático.

## Bloco 10 — Exportação e impressão

PDF, DOCX e impressão continuam obrigatórios para documentação.

Também fechar a ficha compacta de Atendimento/Equipamento:

- conteúdo final e identidade da empresa;
- tamanho/layout físico com no máximo uma página A4;
- impressão direta;
- necessidade ou não de PDF específico da ficha;
- paginação/listas/blocos;
- limites finais dos textos usados na ficha;
- QR/barcode somente se houver valor aprovado;
- critérios de validação.

A identidade da empresa consumida pelos templates vem da configuração central aprovada na Tela 12.

## Bloco 11 — Backup e restauração

Fechar backup consistente do SQLite + arquivos administrados, incluindo categorias, equipamentos, atendimentos, identidade/logo da empresa e demais arquivos persistentes aplicáveis.

A Tela 13 do Bloco 8 define apenas UX/fluxos/estados visíveis; a política técnica definitiva pertence a este bloco.

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
- [x] extensão operacional conceitual aprovada;
- [x] concorrência geral definida;
- [ ] todas as telas críticas, incluindo novos requisitos, especificadas/aprovadas;
- [x] Telas 01–12 do Bloco 8 especificadas/aprovadas;
- [x] modelagem `Procedimento × Atendimento × Equipamento` aprovada;
- [ ] execução/checklist decididos;
- [ ] matriz operacional de permissões decidida;
- [ ] exportação/impressão + ficha compacta definidas;
- [ ] backup/restore definidos;
- [ ] estrutura oficial definida;
- [x] pendências não bloqueantes registradas;
- [ ] plano da Fase 2 aprovado.

## Regra de execução

Não criar scaffold, runtime definitivo ou código de negócio durante a Fase 1.

Toda tarefa Codex que altere arquivos deve trazer base Git esperada e respeitar `AGENTS.md`.