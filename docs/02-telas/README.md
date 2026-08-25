# Telas e Superfícies — StepFlow Pocket

## Estado

**Bloco 8 da Fase 1 — CONCLUÍDO.**

As Telas **01–15 estão consolidadas/aprovadas**. O próximo trabalho da Fase 1 é o **Bloco 9 — Execução operacional/Atendimentos + checklist**, ainda não iniciado neste checkpoint.

Cada especificação relevante usa `docs/templates/template-analise-de-tela.md`. Uma proposta só vira contrato visual/funcional após aprovação explícita do PO.

## Especificações atuais

- `01-login.md` — **CONSOLIDADO**;
- `02-shell-sidebar.md` — **CONSOLIDADO**;
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
- `14-exportacao-impressao-ficha.md` — **CONSOLIDADO / APROVADO PELO PO**;
- `15-estados-transversais.md` — **CONSOLIDADO / APROVADO PELO PO**.

## Mapa de telas

1. Login — consolidado;
2. Shell/sidebar — consolidado;
3. Início/Dashboard — consolidado;
4. Lista/pesquisa de Processos — consolidado;
5. Leitor em formato livro — consolidado;
6. Editor de Processo + categorias — consolidado;
7. Histórico/Revisões — consolidado;
8. Lista/pesquisa de Atendimentos — consolidado;
9. Atendimento/Execução + Equipamento — consolidado;
10. Usuários/Permissões — consolidado;
11. Meu perfil — consolidado;
12. Configurações + Categorias — consolidado;
13. Backup/Restauração — UX — consolidado;
14. Exportação/Impressão + ficha compacta — UX — consolidado;
15. Estados transversais — consolidado.

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
- Funcionário predominantemente em leitura/execução;
- desktop Windows como alvo inicial, com adaptação proporcional em janelas menores suportadas.

## Domínio operacional aprovado

O StepFlow distingue:

- `Processos` — documentação/modelos oficiais;
- `Atendimentos` — ocorrências reais de serviço/execução;
- `Equipamento` — entidade opcional relacionada ao Atendimento quando aplicável.

Também estão aprovados:

- categorias configuráveis e múltiplas;
- identidade interna própria do equipamento;
- múltiplos procedimentos por Atendimento;
- vínculo histórico à revisão realmente utilizada;
- ficha compacta de Atendimento com ou sem equipamento;
- tipos mínimos de computador `Servidor`, `Desktop` e `Notebook`;
- saúde da bateria contextual para Notebook;
- observações curtas e limitadas do equipamento;
- ficha com no máximo uma página A4 e cabeçalho de identidade da empresa;
- identidade da empresa centralizada em Configurações;
- categorias simples com arquivamento/reativação;
- Backup/Restauração dentro de Configurações, coordenado pelo Host;
- safety backup obrigatório antes do Restore normal destrutivo pela UI;
- exportação documental contextual usando exatamente a revisão selecionada.

## Resumo das superfícies consolidadas

### Login

Autenticação simples e centralizada no Host, sem cadastro, recuperação por e-mail, seleção de servidor ou persistência de token.

### Shell / Sidebar

Sidebar persistente com identidade da empresa, navegação por capacidades e perfil no rodapé. Fechar Client individual nunca encerra o Host central.

### Dashboard

Dashboard enxuto, com busca de Processos e atualizados recentemente, sem KPI, gráfico ou ranking.

### Lista/Pesquisa de Processos

Tabela compacta, busca documental, filtros Categoria + Área, categorias múltiplas e retorno preservando contexto.

### Leitor

`Visão geral` antes da Etapa 1, uma etapa por página, Sumário temporário, navegação anterior/próxima, blocos tipados e revisão aberta estável mesmo quando surgir versão nova.

### Editor

Informações + Etapas, salvamento explícito, revisões imutáveis, blocos tipados, categorias existentes e conflito otimista sem sobrescrita silenciosa.

### Histórico/Revisões

Histórico cronológico, revisão técnica separada da versão editorial, revisão histórica somente leitura e criação de nova revisão a partir de snapshot antigo sem rollback destrutivo.

### Lista/Pesquisa de Atendimentos

Tabela operacional compacta, busca por Atendimento/OS/cliente/equipamento/serial/patrimônio/MAC e filtros Responsável + Período. Status permanece dependente do Bloco 9.

### Atendimento/Execução + Equipamento

Workspace vertical com Atendimento, Equipamento opcional e Procedimentos utilizados. Preserva revisões efetivamente usadas e mantém edição do Equipamento separada da edição do Atendimento.

### Usuários/Permissões

Perfis como presets + capacidades granulares, desativação em vez de exclusão, Gerência limitada a não-ADM e proteção contra autoelevação.

### Meu perfil

Avatar, nome, cargo e senha próprios. Login/perfil/permissões são informativos. Troca de senha mantém a sessão atual e revoga as demais sessões da conta.

### Configurações + Categorias

Navegação local Empresa + Categorias, identidade central da empresa, categorias simples, arquivar/reativar e autorização por capacidades. Gerência × configuração da empresa permanece pendente.

### Backup/Restauração

Terceira seção de Configurações. Backup coordenado pelo Host; Restore exige backup elegível, autorização, confirmação reforçada e safety backup confirmado antes da etapa destrutiva normal.

### Exportação/Impressão + Ficha Compacta

#### Procedimentos

- `Exportar / Imprimir` permanece contextual no Leitor;
- PDF, DOCX e impressão usam documento próprio, nunca screenshot;
- a primeira versão gera o procedimento completo da revisão selecionada;
- revisão histórica/draft é identificada inequivocamente;
- uma revisão nova não substitui silenciosamente a fonte da exportação já iniciada;
- identidade da empresa vem da configuração central vigente;
- exportar/imprimir não altera, publica ou cria revisão.

#### Ficha de Atendimento

- entrada contextual pela Tela 09 em `Ficha / Imprimir`;
- usa somente estado confirmado pelo Host;
- alterações não salvas/conflitos precisam ser resolvidos antes da geração;
- pode ser gerada com ou sem equipamento vinculado;
- seção Equipamento é omitida quando não existir;
- campos vazios/não aplicáveis são omitidos;
- procedimentos usados preservam versão/revisão efetivamente utilizada;
- `Saúde da bateria` aparece somente quando aplicável/informada;
- ocupa no máximo uma página A4;
- conteúdo excepcional que não caiba bloqueia a saída em vez de gerar segunda página, reduzir tipografia excessivamente ou truncar silenciosamente;
- impressão é requisito;
- PDF específico da ficha permanece pendente do Bloco 10;
- DOCX específico da ficha não é requisito inicial;
- preview e QR/barcode permanecem para decisão do Bloco 10;
- capacidade/lifecycle para gerar/reimprimir ficha permanecem no Bloco 9.

Fonte: `14-exportacao-impressao-ficha.md` e `docs/01-produto/categorizacao-atendimentos-equipamentos.md`.

### Estados Transversais

A Tela 15 é um contrato comum, não uma nova página navegável.

Ficou consolidado:

- usar a menor superfície adequada para cada estado: campo → seção → página → Shell;
- não exibir indicador permanente de conexão saudável;
- loading não reaproveita dado antigo como se fosse atual;
- distinguir `sem registros` de `sem resultados`;
- Host indisponível aparece como estado transversal e não oferece IP/porta editáveis;
- WebSocket degradado pode ser tratado separadamente quando HTTP continuar saudável;
- reconexão exige reconsulta/reconciliação antes de assumir estado atualizado;
- mutação de resultado incerto não é repetida cegamente;
- sessão expirada exige nova autenticação;
- edição não salva pode permanecer apenas em memória e oculta durante reautenticação do mesmo Client, sem draft persistente;
- perda de permissão remove conteúdo protegido da superfície;
- conflitos preservam edição local e nunca usam overwrite/merge automático;
- eventos remotos não substituem formulário local;
- alterações não salvas recebem proteção de saída;
- incompatibilidade Client↔Host bloqueia uso normal e orienta reabrir pelo ponto de entrada oficial;
- nenhuma offline queue, autosave ou nova persistência é criada.

Fonte: `15-estados-transversais.md`, `docs/03-arquitetura/comunicacao-client-host.md` e `docs/03-arquitetura/concorrencia-fila-conflitos-eventos.md`.

## Limite do Bloco 8

O Bloco 8 fecha UX, fluxo, estados, navegação, permissões visíveis e contrato visual/funcional e está **concluído**.

Ele não decidiu sozinho:

- lifecycle/status final do Atendimento;
- checklist/progresso operacional;
- matriz operacional de permissões;
- parâmetros numéricos ainda pendentes de autenticação/sessão;
- engine/tecnologia final de PDF/DOCX/impressão;
- template físico final, margens, limites textuais e preview da ficha;
- mecanismo técnico de consistência, pacote, retenção e disaster recovery de Backup/Restore.

Esses pontos permanecem nos blocos correspondentes.

## Regra de separação de busca

- `Processos`: código, título, termo, área, categoria e metadados documentais aprovados;
- `Atendimentos`: código, OS/referência, cliente, equipamento, serial/patrimônio/MAC e dados operacionais.

Não misturar os dois domínios em pesquisa global sem requisito explícito.

## Regra de acompanhamento

Todo avanço consolidado de fase, bloco ou tela deve atualizar o painel do `README.md` no mesmo checkpoint documental.

## Próximo trabalho

O próximo bloco é o **Bloco 9 — Execução operacional/Atendimentos + checklist**, mas ele ainda não foi aberto neste checkpoint.

Quando uma regra depender de lifecycle, checklist, exportação técnica, backup técnico ou permissões ainda pendentes, documentar a dependência e parar no limite aprovado. Não inventar solução técnica ou regra de negócio.
