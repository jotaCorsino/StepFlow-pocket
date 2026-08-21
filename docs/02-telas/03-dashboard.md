# Tela 03 — Início / Dashboard

**Status:** EM ANÁLISE / PROPOSTA PARA APROVAÇÃO DO PO  
**Bloco:** Fase 1 — Bloco 8 (UI/UX)  
**Última consolidação:** 2026-08-21

## 1. Objetivo

Ser o primeiro destino após login e oferecer um ponto de partida simples para o uso diário do StepFlow.

O Dashboard não deve se transformar em painel executivo, portal de indicadores ou central burocrática. Sua prioridade é reduzir o tempo entre abrir o aplicativo e encontrar um processo útil.

## 2. Atores

Usado por:

- ADM;
- Gerência;
- Funcionário.

O núcleo é comum a todos. Conteúdo adicional só aparece quando houver permissão e utilidade real.

## 3. Entrada

Fluxo aprovado:

```text
Login válido
→ Shell principal
→ Início/Dashboard
```

O item `Início` da sidebar retorna a esta tela.

## 4. Princípios de UX

Consolidados pelo produto:

- reduzir atrito para localizar e executar processos;
- visual corporativo, limpo e discreto;
- evitar excesso de cards, números e indicadores;
- não duplicar navegação já disponível na sidebar;
- não mostrar informações técnicas de Host/banco/porta;
- respeitar capacidades retornadas pelo Host.

## 5. Composição proposta

```text
Início
Bem-vindo, <nome>.

┌──────────────────────────────────────────────────────────────┐
│ Buscar um processo                                           │
│ [ código, título ou termo...                         ] [🔎]   │
└──────────────────────────────────────────────────────────────┘

Atualizados recentemente
---------------------------------------------------------------
[Código] Título do processo                    Área      Versão
[Código] Título do processo                    Área      Versão
[Código] Título do processo                    Área      Versão

                                         [ Ver todos os processos ]
```

A composição é deliberadamente enxuta. Não inclui gráficos, contadores decorativos, ranking de usuários ou KPIs de gestão na primeira versão.

## 6. Cabeçalho

Proposta:

- título `Início`;
- saudação curta usando nome de exibição: `Bem-vindo, <nome>.`;
- sem data/hora decorativa;
- sem banners promocionais.

A saudação não deve ocupar espaço excessivo.

## 7. Busca rápida de processos

Elemento principal proposto.

### Objetivo

Permitir que o técnico comece a localizar um processo sem precisar primeiro navegar até a tela completa de Processos.

### Comportamento

- aceitar Código, Título ou termo de busca compatível com o contrato da lista de processos;
- Enter ou ação de busca encaminha para `Processos` com o termo aplicado;
- o Dashboard não precisa reproduzir todos os filtros avançados da tela de Processos;
- busca vazia pode encaminhar para a lista completa ou manter a tela, conforme microinteração a fechar com a tela de Processos;
- não pesquisar dados que o usuário não tenha autorização para ver.

### Limite

A busca rápida é um ponto de entrada. A pesquisa/filtro completos serão definidos na Tela 04 — Lista e Pesquisa de Processos.

## 8. Processos atualizados recentemente

### Proposta

Exibir uma lista curta de processos oficiais recentemente publicados/atualizados e visíveis ao usuário.

Objetivos:

- facilitar descoberta de documentação nova ou revisada;
- fornecer conteúdo útil sem criar métricas administrativas;
- permitir abertura direta do processo.

### Campos visuais sugeridos

- Código;
- Título;
- Área/Departamento;
- Versão exibida;
- indicação discreta de atualização recente quando necessário.

A data pode existir como informação secundária se melhorar a compreensão, mas não precisa dominar a lista.

### Regras

- somente processos que o usuário pode consultar;
- não mostrar rascunhos/revisões internas a quem não possui acesso correspondente;
- clicar em item abre o leitor do processo;
- quantidade inicial deve ser pequena para não transformar o Dashboard em outra lista completa;
- ação `Ver todos os processos` navega para a Tela 04.

A quantidade exata de itens é detalhe visual a fechar posteriormente.

## 9. Ausência de seção “Favoritos” na primeira especificação

Não existe requisito consolidado de favoritos, pins ou bookmarks.

Portanto, o Dashboard não cria esse conceito por conveniência. Se houver necessidade futura, deverá ser requisito explícito com persistência/UX definidas.

## 10. Ausência de “Recentemente acessados” como requisito

Também não está consolidado um histórico pessoal de processos recentemente abertos.

Não criar armazenamento local ou server-side para isso apenas para enriquecer o Dashboard.

A primeira proposta usa **processos atualizados recentemente**, deriváveis do histórico oficial já existente, e não histórico de navegação pessoal.

## 11. Conteúdo por perfil

### Funcionário

Núcleo proposto:

- busca rápida;
- processos atualizados recentemente;
- acesso à lista completa.

Nenhum bloco gerencial adicional é necessário.

### Gerência e ADM

A mesma experiência de consulta permanece disponível.

Não adicionar por padrão:

- KPIs de quantidade de usuários;
- quantidade de alterações por pessoa;
- produtividade;
- gráficos de uso;
- filas administrativas artificiais.

Atalhos administrativos já existem na sidebar; duplicá-los no Dashboard só será feito se houver utilidade concreta aprovada.

## 12. Processos em edição/rascunho

Não incluir bloco específico de “meus rascunhos” nesta primeira proposta sem antes fechar o fluxo completo do editor/publicação.

Se a Tela 06 demonstrar necessidade real, o Dashboard poderá ser revisado sem afetar seu núcleo de busca/consulta.

## 13. Estados da interface

### Loading

- mostrar estrutura estável;
- evitar múltiplos spinners desconexos;
- busca não deve parecer funcional se o Host ainda estiver indisponível.

### Vazio — nenhum processo disponível

Exibir mensagem simples de que não há processos disponíveis para consulta.

Não sugerir criação para Funcionário.

Para perfis com permissão de criação, eventual ação contextual será decidida com a Tela de Processos/Editor, não automaticamente pelo Dashboard.

### Nenhuma atualização recente

O bloco pode mostrar mensagem curta e manter `Ver todos os processos` disponível.

### Host indisponível

Seguir o comportamento transversal aprovado no Shell:

- banner/estado claro;
- não apresentar dados antigos como se fossem atuais sem indicação;
- desabilitar ações dependentes do Host quando necessário;
- não exibir configuração técnica de rede.

### Sessão expirada

Retornar ao Login conforme Shell/autenticação.

### Erro parcial

Se a busca estiver disponível mas a lista de atualizados falhar, não é necessário derrubar todo o Dashboard. Mostrar erro contextual do bloco quando tecnicamente seguro.

## 14. Atualização em tempo real

Como o StepFlow possui canal de eventos, a lista de processos recentemente atualizados pode ser reconsultada quando eventos relevantes indicarem publicação/alteração oficial.

Não redesenhar a tela agressivamente nem reposicionar itens enquanto o usuário está interagindo sem necessidade.

O evento informa mudança; dados atuais devem ser reconsultados conforme contrato vigente.

## 15. Concorrência

O Dashboard é predominantemente leitura.

- não resolve conflitos de edição;
- não mantém cópia autoritativa de processos;
- dados exibidos refletem estado confirmado pelo Host;
- ações de edição futura seguem revisão otimista nas telas correspondentes.

## 16. Navegação

Destinos principais:

- busca → Tela 04 com termo aplicado;
- processo recente → Tela 05 — Leitor em formato livro;
- `Ver todos os processos` → Tela 04.

O Dashboard não deve criar rotas paralelas diferentes para o mesmo objeto.

## 17. Acessibilidade e teclado

- foco inicial não deve ser forçado de forma que atrapalhe leitura, mas a busca deve ser rapidamente alcançável;
- campo de busca possui label acessível;
- Enter executa a ação esperada;
- itens recentes acessíveis por teclado;
- estados vazio/erro não dependem só de cor;
- ordem de foco previsível.

Atalho global para foco na busca pode ser considerado depois, mas não é requisito inicial.

## 18. Comportamento em janelas menores

- busca permanece utilizável;
- tabela/lista recente pode reduzir colunas secundárias ou reorganizar informação sem esconder Código/Título;
- não criar layout mobile específico sem necessidade demonstrada;
- rolagem vertical é aceitável.

A estratégia exata segue a janela mínima que será aprovada no sistema visual.

## 19. Dados necessários

Para o núcleo proposto:

### Sessão/perfil

- nome de exibição;
- capacidades efetivas.

### Lista recente

- process_id;
- código;
- título;
- área/departamento;
- versão exibida;
- status/publicação aplicável;
- timestamp de atualização/publicação suficiente para ordenação;
- autorização/visibilidade aplicável.

O Dashboard não cria novo dado de domínio apenas para apresentação.

## 20. Contratos Client ↔ Host

A tela depende conceitualmente de:

1. sessão/perfil já carregados;
2. consulta de processos visíveis ordenados por atualização oficial;
3. busca/listagem de processos;
4. abertura de processo por identidade estável;
5. eventos/reconsulta quando processos relevantes mudarem.

Nomes finais de rotas pertencem ao contrato de implementação e não são inventados nesta especificação.

## 21. Segurança e autorização

- Host filtra/autoriza dados;
- Client não confia apenas em esconder elementos;
- Funcionário não recebe conteúdo administrativo indevido;
- rascunhos/revisões não publicadas respeitam autorização;
- Dashboard não expõe dados sensíveis de usuários ou auditoria.

## 22. Sistema visual

Usar o sistema visual compartilhado pelo Shell:

- área ampla e limpa;
- uma ação primária clara: encontrar processos;
- poucos blocos;
- cards somente quando ajudarem hierarquia, não como padrão obrigatório;
- tipografia e cores seguem as pendências compartilhadas do Bloco 8.

## 23. Fora do escopo

- KPIs executivos;
- analytics de produtividade/uso;
- favoritos/bookmarks;
- histórico pessoal de navegação;
- feed social/notificações genéricas;
- edição de processos no próprio Dashboard;
- detalhes completos de filtros da Tela 04;
- código de produção.

## 24. Decisões já consolidadas antes desta tela

- Dashboard é destino pós-login;
- `Início` existe na sidebar;
- técnico deve localizar processos com baixo atrito;
- aplicação não deve parecer portal burocrático;
- autorização é Host-side;
- atualizações relevantes podem chegar por eventos/reconsulta.

## 25. Propostas para aprovação do PO

1. Dashboard enxuto, sem KPIs/gráficos na primeira versão;
2. busca rápida de processos como elemento principal;
3. busca encaminha para a Tela 04 com o termo aplicado;
4. seção curta `Atualizados recentemente` com processos oficiais visíveis ao usuário;
5. clique em item recente abre diretamente o leitor;
6. sem favoritos e sem histórico pessoal de navegação inicialmente;
7. mesma base de Dashboard para todos os perfis, sem painel gerencial separado;
8. sem duplicar atalhos administrativos já disponíveis na sidebar.

## 26. Pendências

- quantidade visual de processos recentes;
- formato exato lista versus cards compactos;
- microcopy final;
- detalhes visuais compartilhados do Shell;
- eventual necessidade futura de rascunhos/atalhos após especificar Editor e Processos.

## 27. Critérios de aceite da especificação

- [x] respeita destino pós-login aprovado;
- [x] prioriza consulta de processos;
- [x] não cria conceito de favoritos/recents pessoais sem requisito;
- [x] não inventa KPIs administrativos;
- [x] autorização continua Host-side;
- [x] nenhum código de produção criado;
- [ ] composição enxuta aprovada pelo PO;
- [ ] busca rápida aprovada pelo PO;
- [ ] seção de atualizados recentemente aprovada pelo PO;
- [ ] ausência de painel gerencial separado aprovada pelo PO.

## 28. Casos de teste futuros

1. login bem-sucedido chega ao Dashboard;
2. busca por Código encaminha corretamente para Processos;
3. busca por Título/termo mantém o termo aplicado;
4. item recente abre o processo correto;
5. Funcionário não recebe rascunho administrativo indevido;
6. lista recente respeita permissões;
7. nenhum processo mostra estado vazio coerente;
8. Host indisponível mostra estado transversal correto;
9. atualização oficial relevante pode refletir após evento/reconsulta;
10. navegação por teclado funciona;
11. layout funciona na janela mínima aprovada;
12. não há dependência de favoritos/histórico pessoal para funcionamento.
