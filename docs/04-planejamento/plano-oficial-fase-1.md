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
| 6 | Dados/schema/migrations | CONCLUÍDO | `03-arquitetura/modelo-dados-schema-fase-1.md` |
| 7 | Concorrência/eventos | CONCLUÍDO | `03-arquitetura/concorrencia-fila-conflitos-eventos.md` |
| 8 | UI/UX | PRÓXIMO | `02-telas/README.md` |
| 9 | Checklist durante execução | PENDENTE | decisão de produto |
| 10 | Exportação/impressão | PENDENTE | arquitetura técnica |
| 11 | Backup/restauração | PENDENTE | política técnica/operacional |
| 12 | Estrutura oficial + Fase 2 | PENDENTE | fundação do repositório |

## Bloco 8 — UI/UX

Documentar/aprovar antes do código as superfícies críticas:

1. Login;
2. Shell/sidebar;
3. Dashboard;
4. lista/pesquisa de processos;
5. leitura em formato livro;
6. editor de processo/etapas;
7. histórico;
8. usuários/permissões;
9. perfil;
10. configurações da empresa;
11. backup/restauração;
12. exportação/impressão;
13. estados transversais de erro/loading/conflito/Host indisponível.

Usar `docs/templates/template-analise-de-tela.md`. Aparência só vira contrato quando aprovada pelo PO.

No Bloco 8, telas de backup/exportação fecham apenas **UX, fluxo, estados e permissões visíveis**. Estratégia técnica de exportação pertence ao Bloco 10 e estratégia técnica de backup/restore ao Bloco 11. A tela deve registrar dependências sem inventar solução técnica.

O encerramento do Host também deve respeitar `host-pocket.md`: Client individual não derruba o Host; o ciclo central pertence ao Controller. A UX de confirmação/aviso quando houver Clients conectados pode ser especificada no Bloco 8.

## Bloco 9 — Checklist durante execução

O checklist documental já existe no modelo. Falta decidir se a marcação feita pelo técnico será:

- somente em memória da sessão;
- local por usuário/dispositivo;
- persistida como progresso pessoal;
- parte de uma entidade formal de execução.

Escolher a alternativa mais simples que atenda ao uso real, sem criar workflow burocrático sem requisito.

## Bloco 10 — Exportação e impressão

PDF, DOCX e impressão são obrigatórios. Fechar:

- onde gerar cada formato;
- bibliotecas/estratégia offline;
- modelo comum exportável;
- identidade da empresa;
- paginação, listas, tabelas e blocos técnicos;
- critérios de validação.

## Bloco 11 — Backup e restauração

Fechar:

- backup consistente do SQLite + arquivos administrados;
- formato do pacote;
- validação;
- retenção;
- coordenação durante restore;
- recuperação após falha.

## Bloco 12 — Fundação da Fase 2

Somente depois dos blocos anteriores:

- resolver parâmetros operacionais ainda pendentes necessários à implementação, como valores finais de sessão/senha/permissões marcadas como pendentes;
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
- [x] modelo de dados/migrations definidos;
- [x] concorrência definida;
- [ ] telas críticas especificadas/aprovadas;
- [ ] checklist de execução decidido;
- [ ] exportação/impressão definidas;
- [ ] backup/restore definidos;
- [ ] estrutura oficial do repositório definida;
- [x] pendências não bloqueantes registradas;
- [ ] plano da Fase 2 aprovado.

## Regra de execução

Não criar scaffold, árvore runtime definitiva ou código de negócio durante a Fase 1 por conveniência. Provas técnicas novas só devem existir quando uma decisão relevante realmente depender de evidência mecânica não disponível e a tarefa as declarar explicitamente descartáveis.

Toda tarefa Codex que altere arquivos deve trazer base Git esperada (branch + SHA) e respeitar as proteções de working tree definidas em `AGENTS.md`.
