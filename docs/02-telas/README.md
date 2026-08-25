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
- `08-lista-pesquisa-atendimentos.md` — **CONSOLIDADO / APROVADO PELO PO**;
- `09-atendimento-execucao-equipamento.md` — **CONSOLIDADO / APROVADO PELO PO**;
- `10-usuarios-permissoes.md` — **CONSOLIDADO / APROVADO PELO PO**;
- `11-meu-perfil.md` — **CONSOLIDADO / APROVADO PELO PO**;
- `12-configuracoes-categorias.md` — **CONSOLIDADO / APROVADO PELO PO**;
- `13-backup-restauracao.md` — **CONSOLIDADO / APROVADO PELO PO**;
- próxima: **Tela 14 — Exportação / impressão + ficha compacta — UX**.

A Tela 14 ainda não está em análise neste checkpoint.

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
- ficha compacta imprimível de atendimento/equipamento;
- para computadores, tipos mínimos `Servidor`, `Desktop` e `Notebook`;
- saúde da bateria contextual para `Notebook`;
- observações curtas e limitadas do equipamento;
- ficha compacta com no máximo uma página A4 e cabeçalho de identidade da empresa;
- identidade da empresa centralizada em Configurações;
- categorias simples administradas em Configurações, com arquivamento/reativação e preservação de histórico;
- Backup/Restauração como UX administrativa Host-side dentro de Configurações;
- safety backup obrigatório antes da etapa destrutiva de Restore normal pela UI.

Fonte: `docs/01-produto/categorizacao-atendimentos-equipamentos.md`, `13-backup-restauracao.md` e arquitetura vigente.

## Mapa de telas

1. Login — consolidado;
2. Shell/sidebar — consolidado;
3. Início/Dashboard — consolidado;
4. Lista/pesquisa de Processos — consolidado;
5. Leitor em formato livro — consolidado;
6. Editor de Processo + categorias — consolidado;
7. Histórico/Revisões — consolidado;
8. Lista/pesquisa de Atendimentos — consolidado;
9. Atendimento/execução + ficha do equipamento — consolidado;
10. Usuários/permissões — consolidado;
11. Meu perfil — consolidado;
12. Configurações + gestão de categorias — consolidado;
13. Backup/restauração — consolidado;
14. Exportação/impressão + ficha compacta — **próximo**;
15. estados transversais — pendente.

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

Fonte: `04-lista-pesquisa-processos.md`.

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

## Lista/Pesquisa de Atendimentos — consolidado

- lista/tabela compacta;
- campo único de busca por código de atendimento, OS/referência, cliente/solicitante, equipamento, serial, patrimônio ou MAC;
- filtros iniciais `Responsável` + `Período`;
- equipamento aparece apenas como resumo na lista;
- linha abre a Tela 09;
- retorno preserva busca/filtros/ordenação/posição quando possível;
- não criar coluna/filtro `Status` antes do lifecycle do Bloco 9;
- mais recentes primeiro, com timestamp exato definido somente no Bloco 9.

Fonte: `08-lista-pesquisa-atendimentos.md`.

## Atendimento/Execução + Equipamento — consolidado

- uma única página vertical com seções `Atendimento`, `Equipamento` e `Procedimentos utilizados`;
- mesma Tela 09 para novo atendimento e atendimento existente;
- equipamento continua opcional;
- `Vincular equipamento` pesquisa cadastro existente antes de cadastrar novo;
- ficha técnica mostra somente campos preenchidos/aplicáveis;
- edição do Equipamento fica visualmente separada da edição do Atendimento;
- múltiplos MACs com label opcional;
- procedimentos exibem versão editorial + revisão técnica efetivamente utilizada;
- `Abrir revisão` leva ao Leitor na revisão específica;
- iniciar pelo Leitor pode pré-selecionar a revisão consultada;
- `Resumo do trabalho` separado de `Observações`;
- ponto de entrada `Ficha / Imprimir`, com fluxo final reservado à Tela 14/Bloco 10;
- não definir `Status`, conclusão, reabertura ou checklist persistente antes do Bloco 9;
- para computadores, `Tipo` suporta pelo menos `Servidor`, `Desktop` e `Notebook`;
- `Saúde da bateria` é contextual para `Notebook`;
- `Observações do equipamento` é texto curto e limitado, com valor numérico a fechar no Bloco 10;
- ficha compacta ocupa no máximo uma folha A4;
- cabeçalho da ficha suporta logo da empresa, nome, forma(s) de contato, site e e-mail;
- template deve priorizar conteúdo essencial e legibilidade, sem segunda página como comportamento normal.

Fonte: `09-atendimento-execucao-equipamento.md` e `docs/01-produto/categorizacao-atendimentos-equipamentos.md`.

## Usuários/Permissões — consolidado

- lista/tabela compacta de usuários;
- busca por nome, login ou cargo;
- filtros `Perfil` + `Estado`;
- mesmo formulário administrativo para criar/editar usuário;
- `Ativo/Inativo`, com desativação em vez de exclusão física;
- `Perfil base` como preset de capacidades;
- seção opcional `Permissões específicas` para personalização granular;
- `Restaurar padrão do perfil` para remover personalizações;
- teto de delegação validado pelo Host, impedindo Gerência de criar um ADM equivalente por capacidades;
- Gerência administra somente usuários não-ADM;
- perfil/capacidades da própria conta somente leitura nesta tela, deixando dados pessoais para `Meu perfil`;
- proteção do último ADM ativo;
- `is_primary_admin` não aparece como toggle comum;
- reset administrativo de senha sem revelar senha antiga e com revogação de sessões pertinentes;
- sem exclusão física e sem ações em massa inicialmente;
- capacidades operacionais de Atendimento permanecem pendentes do Bloco 9;
- parâmetros numéricos ainda pendentes de autenticação/sessão continuam pendentes.

Fonte: `10-usuarios-permissoes.md` e `docs/03-arquitetura/autenticacao-sessao-autorizacao.md`.

## Meu perfil — consolidado

- página simples com seções `Perfil` e `Segurança`;
- acesso pelo bloco de perfil no rodapé da sidebar, sem item global novo;
- avatar, nome de exibição e cargo editáveis pelo próprio usuário;
- login somente leitura;
- perfil base e indicação de permissões personalizadas somente informativos;
- nenhuma alteração de autoridade pela Tela 11;
- avatar com escolher/substituir/remover e preview antes do salvamento;
- placeholder/iniciais quando não houver avatar;
- `Salvar alterações` explícito para avatar/nome/cargo, sem autosave;
- alteração de senha em fluxo separado com senha atual, nova senha e confirmação;
- política numérica de senha continua vinda do contrato do Host e permanece pendente onde ainda não consolidada;
- após troca de senha, manter a sessão atual e revogar as demais sessões da conta;
- sem recuperação de senha por e-mail; esquecimento usa redefinição administrativa quando autorizada;
- concorrência com alteração administrativa não sobrescreve conteúdo local silenciosamente.

Fonte: `11-meu-perfil.md` e `docs/03-arquitetura/autenticacao-sessao-autorizacao.md`.

## Configurações + Categorias — consolidado

- uma única Tela 12 com navegação local `Empresa` + `Categorias`;
- seções aparecem conforme capacidades efetivas da sessão;
- capacidade da Gerência para alterar configuração da empresa permanece `PENDENTE`;
- identidade da empresa inicialmente limitada a logo, nome, contato, site e e-mail;
- logo com escolher/substituir/remover e preview antes do save;
- identidade confirmada pelo Host alimenta Shell e documentos/ficha sem duplicação manual;
- salvamento explícito, sem autosave;
- categorias em lista compacta com busca por nome e filtro por estado;
- categoria simples com nome + estado, sem hierarquia/cor/ícone inicialmente;
- criação/edição de categoria acontece fora do Editor de Processo;
- arquivar/reativar em vez de excluir fisicamente;
- categoria arquivada deixa de ser opção normal para novas associações e preserva histórico;
- categorias duplicadas/visualmente equivalentes após normalização devem ser impedidas;
- regra de nova revisão ainda referenciando categoria arquivada permanece pendente;
- Backup/Restore e Exportação/Impressão permanecem nas telas/blocos próprios.

Fonte: `12-configuracoes-categorias.md`, `docs/01-produto/categorizacao-atendimentos-equipamentos.md` e `docs/03-arquitetura/autenticacao-sessao-autorizacao.md`.

## Backup/Restauração — consolidado

- permanece dentro de `Configurações` como terceira seção local `Backup e restauração`, sem novo item global na sidebar;
- seção aparece somente com capacidade aplicável;
- ADM pode Backup e Restore conforme matriz vigente;
- Gerência × Backup permanece `PENDENTE`; Gerência não recebe Restore;
- lista compacta de backups conhecidos pelo Host;
- metadados iniciais: data/hora, origem, autor, tamanho e verificação;
- `Criar backup agora` não pede SQLite/path nem seleção manual de componentes;
- backup aceito é coordenado pelo Host e fechar Client não significa cancelamento silencioso;
- `Detalhes` antecede Restore;
- Restore só aparece para backup elegível e sessão autorizada;
- confirmação reforçada exige ciência explícita + texto `RESTAURAR`;
- safety backup do estado atual é obrigatório antes da etapa destrutiva de Restore normal via UI;
- sem safety backup confirmado, Restore normal não prossegue;
- sem delete, scheduler, retention, upload/download ou path editável inicialmente;
- disaster recovery quando o Host não inicia fica fora da UX normal e pertence ao Bloco 11;
- Backup permanece separado de Exportação/Impressão.

Fonte: `13-backup-restauracao.md`, `docs/03-arquitetura/autenticacao-sessao-autorizacao.md` e `docs/03-arquitetura/host-pocket.md`.

## Limite do Bloco 8

O Bloco 8 fecha UX, fluxo, estados, navegação, permissões visíveis e contrato visual/funcional.

Não decide sozinho:

- lifecycle/status final do Atendimento;
- checklist/progresso operacional;
- matriz de permissões operacional;
- parâmetros numéricos ainda pendentes de autenticação/sessão;
- tecnologia/formato final da ficha compacta;
- mecanismo técnico de consistência, pacote, retenção e disaster recovery de backup/restore.

Esses pontos pertencem aos documentos/blocos correspondentes.

## Regra de separação de busca

- `Processos`: código, título, termo, área, categoria e metadados documentais aprovados;
- `Atendimentos`: código de atendimento, OS/referência, cliente, equipamento, serial/patrimônio/MAC e dados operacionais.

Não misturar os dois domínios em pesquisa global sem requisito explícito.

## Regra de acompanhamento

Todo avanço consolidado de fase, bloco ou tela deve atualizar o painel do `README.md` no mesmo checkpoint documental.

## Regra de parada

Quando uma tela depender de lifecycle/checklist/exportação/permissões ainda pendentes, documentar a dependência e parar no limite aprovado. Não inventar solução técnica ou regra de negócio.
