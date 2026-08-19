# Método Padrão de Trabalho Assistido — PO + Assistente + Codex

**Natureza:** documento genérico e reutilizável.

**Objetivo:** definir um processo padronizado para criação e evolução de software com três papéis complementares: responsável pelo produto (PO), assistente de análise/arquitetura/documentação e Codex como executor técnico.

Este documento foi escrito para poder ser copiado para outros projetos com poucas adaptações. Ele não depende do StepFlow.

---

## 1. Princípio central

O desenvolvimento não começa pelo código. O fluxo padrão é:

**entender → documentar → decidir → planejar → executar uma tarefa pequena → validar → registrar → avançar.**

O objetivo é reduzir improviso, deriva de escopo, decisões implícitas e retrabalho causado por implementações tecnicamente corretas, porém desalinhadas com o produto.

---

## 2. Papéis e responsabilidades

### 2.1. PO / responsável pelo produto

É a autoridade final sobre o produto.

Responsabilidades:

- definir objetivo, escopo e prioridade;
- explicar o problema real a ser resolvido;
- aprovar ou rejeitar comportamentos de produto;
- aprovar direção visual, layout e UX quando aplicável;
- decidir trade-offs de negócio;
- resolver pendências que alterem produto, fluxo ou prioridade;
- determinar quando uma proposta está madura para virar decisão consolidada;
- validar entregas do ponto de vista do uso real.

O PO não precisa transformar suas ideias em especificações técnicas. Essa tradução faz parte do papel do assistente.

### 2.2. Assistente

Atua como analista, arquiteto, documentador e coordenador da execução.

Responsabilidades:

- ouvir e decompor a intenção do PO;
- recuperar contexto vigente antes de propor mudanças;
- identificar requisitos funcionais e não funcionais;
- apontar inconsistências, riscos e impactos técnicos;
- separar decisão consolidada, hipótese, proposta e pendência;
- produzir e manter documentação viva;
- desenhar arquitetura proporcional ao problema;
- organizar roadmap, fases, dependências e gates;
- transformar decisões aprovadas em tarefas objetivas para o Codex;
- revisar evidências de execução;
- impedir que conveniência técnica substitua decisão de produto;
- manter continuidade entre conversas por meio do repositório, e não apenas pela memória do chat.

O assistente pode recomendar, comparar e explicar alternativas. Não deve transformar silenciosamente uma preferência própria em decisão oficial do produto.

### 2.3. Codex

É o executor técnico do projeto.

Responsabilidades:

- ler a documentação vigente antes da execução;
- trabalhar somente dentro do escopo da tarefa atual;
- implementar de forma modular, legível e compatível com a arquitetura aprovada;
- preservar comportamentos e visuais que não estejam autorizados a mudar;
- validar o resultado mecanicamente sempre que possível;
- atualizar a documentação exigida pela tarefa;
- relatar com precisão o que mudou e o que foi validado;
- sinalizar conflitos ou bloqueios em vez de inventar solução de produto;
- deixar o repositório em estado coerente e revisável.

O Codex não é o PO e não deve criar requisito, regra de negócio, tela ou redesign fora do escopo por iniciativa própria.

---

## 3. Fonte de verdade

O GitHub deve ser a fonte principal de verdade do projeto.

Conversas, rascunhos e memórias ajudam na descoberta, mas decisões importantes precisam chegar ao repositório.

Uma organização recomendada é:

```text
docs/
├── 00-governanca/
├── 01-produto/
├── 02-telas/
├── 03-arquitetura/
├── 04-planejamento/
├── 05-progresso/
└── templates/
```

### 3.1. Governança

Contém:

- regras operacionais;
- papéis;
- precedência de documentos;
- guia mestre;
- método de trabalho;
- políticas específicas do projeto.

### 3.2. Produto

Contém:

- visão;
- problema;
- atores;
- casos de uso;
- requisitos funcionais;
- requisitos não funcionais;
- limites do escopo;
- linguagem e conceitos do domínio.

### 3.3. Telas e fluxos

Uma tela ou superfície relevante deve ter documento próprio quando o projeto tiver UI significativa.

A documentação de tela deve cobrir:

- objetivo;
- layout e hierarquia;
- componentes;
- interações;
- estados;
- validações;
- mensagens de erro;
- dados exibidos;
- dados necessários;
- permissões;
- regras de negócio;
- impacto técnico;
- pendências;
- critérios de aceite.

### 3.4. Arquitetura

Contém:

- visão arquitetural;
- estrutura do repositório;
- componentes;
- responsabilidades;
- persistência;
- modelo de dados;
- contratos/API;
- concorrência;
- segurança;
- integrações;
- ADRs para decisões estruturais relevantes.

### 3.5. Planejamento

Contém:

- roadmap;
- fases;
- dependências;
- plano de implementação;
- gates de entrada e saída;
- blocos de trabalho.

### 3.6. Progresso

Manter três registros com funções distintas:

**Registro de decisões** — explica o que passou a valer e por quê quando necessário.

**Diário de progresso** — registra o que foi feito, validado, descoberto e o que ficou pendente.

**Changelog do projeto** — resume alterações relevantes de forma mais objetiva e cronológica.

---

## 4. Estados de uma definição

Nem toda ideia discutida deve virar regra imediatamente.

Usar explicitamente estados como:

- **PROPOSTA** — alternativa em avaliação;
- **PENDENTE** — exige decisão antes de determinada implementação;
- **CONSOLIDADA** — decisão vigente e autorizada;
- **SUPERADA** — decisão antiga substituída por outra;
- **HISTÓRICA** — preservada apenas para rastreabilidade.

Essa separação evita que o executor trate uma conversa exploratória como requisito obrigatório.

---

## 5. Precedência em caso de conflito

Cada projeto deve declarar sua própria ordem de precedência.

Um padrão recomendado:

1. decisão consolidada mais recente;
2. especificação específica vigente da funcionalidade/tela/arquitetura;
3. guia mestre vigente;
4. visão de produto;
5. documentação histórica;
6. conversa antiga não consolidada.

Se a precedência não resolver o conflito, a questão deve voltar ao PO/assistente como pendência.

Nunca usar ambiguidade como autorização para escolher a alternativa mais conveniente ao código.

---

## 6. Ciclo completo de trabalho

### Etapa 1 — Receber a demanda

O PO apresenta uma ideia, problema, tela, correção ou objetivo.

Nesse momento não se presume solução técnica final.

### Etapa 2 — Recuperar contexto

Antes de decidir, o assistente deve verificar:

- documentação vigente;
- decisões anteriores relacionadas;
- arquitetura atual;
- dependências;
- código existente, quando já houver implementação;
- divergências entre o pedido atual e o estado documentado.

### Etapa 3 — Analisar impacto

Decompor a demanda em:

- comportamento esperado;
- atores afetados;
- dados envolvidos;
- permissões;
- UI/UX;
- persistência;
- concorrência;
- segurança;
- integração;
- compatibilidade;
- riscos;
- impactos em funcionalidades existentes.

### Etapa 4 — Consolidar documentação

Registrar o que já está decidido e listar separadamente o que permanece pendente.

Não esconder incerteza atrás de linguagem definitiva.

### Etapa 5 — Definir arquitetura proporcional

Escolher a solução mais simples que atenda aos requisitos reais sem criar dívida estrutural óbvia.

Princípios:

- modularidade;
- responsabilidades claras;
- baixo acoplamento;
- coesão;
- evitar monólitos acidentais;
- evitar abstrações prematuras;
- evitar superengenharia;
- não adiar requisitos estruturais já conhecidos.

### Etapa 6 — Planejar por fases e dependências

Não planejar apenas uma lista solta de features.

Cada fase deve declarar:

- objetivo;
- dependências;
- entregáveis;
- o que está fora do escopo;
- critérios de entrada;
- critérios de saída.

Uma fase nova não deve começar apenas porque “há tempo”; ela deve começar quando as dependências reais estiverem fechadas.

### Etapa 7 — Abrir uma tarefa pequena para o Codex

A unidade preferida de execução é uma tarefa pequena, fechada e verificável.

Evitar prompts como:

> implemente todo o sistema

Preferir blocos como:

> criar a fundação do módulo de autenticação sem UI final, com contratos X, persistência Y, testes Z e sem alterar outros módulos.

### Etapa 8 — Executar

O Codex lê a documentação indicada e modifica somente o necessário.

Se descobrir conflito de produto, deve reportá-lo. Não deve expandir a tarefa para “resolver tudo”.

### Etapa 9 — Validar mecanicamente

Sempre que aplicável executar:

- build;
- testes automatizados;
- lint/formatter;
- typecheck;
- smoke test;
- teste de rota/endpoint;
- teste de persistência/migração;
- comparação visual quando houver handoff;
- validação de permissões;
- validação de concorrência;
- inspeção de arquivos gerados;
- conferência de estado Git.

Uma afirmação de sucesso deve ser acompanhada por evidência compatível com o tipo de mudança.

### Etapa 10 — Revisar contra os critérios de aceite

Não basta o projeto compilar.

Confirmar item a item:

- o comportamento pedido foi entregue?
- algo fora do escopo mudou?
- os estados de erro foram considerados?
- permissões continuam corretas?
- visual aprovado foi preservado?
- dados existentes permanecem compatíveis?
- documentação necessária foi atualizada?

### Etapa 11 — Registrar

Atualizar conforme o impacto:

- diário;
- changelog;
- registro de decisões;
- ADR;
- documento da tela;
- arquitetura;
- roadmap/status da fase.

### Etapa 12 — Versionar e avançar

O commit deve representar uma unidade coerente.

Só então abrir a próxima tarefa.

---

## 7. Regra de uma tarefa por vez

Uma tarefa deve ter um objetivo central verificável.

Isso não significa alterar apenas um arquivo. Significa que todos os arquivos alterados precisam servir ao mesmo objetivo.

Uma boa tarefa:

- é pequena o bastante para revisão;
- possui critérios claros;
- pode ser validada;
- deixa o projeto em estado consistente;
- não depende de promessas de “terminar depois” para ser considerada concluída.

---

## 8. Estrutura obrigatória de uma tarefa para o Codex

Usar, preferencialmente:

```text
TÍTULO

OBJETIVO

CONTEXTO / FONTE DE VERDADE

ESCOPO INCLUÍDO

FORA DO ESCOPO

REGRAS E RESTRIÇÕES

ARQUIVOS / ÁREAS ESPERADAS

CRITÉRIOS DE ACEITE

VALIDAÇÕES OBRIGATÓRIAS

DOCUMENTAÇÃO A ATUALIZAR

FORMATO DO RELATÓRIO FINAL
```

### Por que incluir “fora do escopo”

Porque agentes de implementação tendem a encontrar oportunidades adjacentes. Declarar explicitamente o que não deve mudar reduz deriva e refactors oportunistas.

---

## 9. Checklist antes de liberar implementação

Antes de enviar uma tarefa ao Codex, verificar:

- [ ] O objetivo está claro?
- [ ] Existe fonte de verdade identificada?
- [ ] As decisões necessárias estão consolidadas?
- [ ] As pendências bloqueantes foram resolvidas?
- [ ] O ator e as permissões estão claros?
- [ ] Os dados necessários estão identificados?
- [ ] A persistência/contratos relevantes estão definidos?
- [ ] Existe impacto visual? Se sim, há referência/aprovação?
- [ ] Estados de erro, vazio, loading e conflito foram considerados quando aplicável?
- [ ] Concorrência e idempotência foram consideradas quando aplicável?
- [ ] Segurança/autorização foram consideradas quando aplicável?
- [ ] O escopo está pequeno e revisável?
- [ ] Há critérios de aceite objetivos?
- [ ] Há forma de validar mecanicamente a entrega?
- [ ] Está claro quais documentos deverão ser atualizados?

Se uma pendência não bloqueia a tarefa, registrá-la explicitamente. Se bloqueia, não escondê-la no prompt.

---

## 10. Relatório obrigatório do Codex

Ao concluir, o executor deve devolver:

```text
1. Objetivo executado
2. Arquivos criados/alterados/removidos
3. Decisões técnicas tomadas dentro do escopo
4. Validações executadas
5. Resultados
6. Riscos/limitações/pendências
7. Documentação atualizada
8. Próximos passos sugeridos
```

“Próximos passos” são recomendações, não autorização automática para continuar implementando.

---

## 11. Preservação visual e handoff

Quando existe UI aprovada, ela deve ser tratada como contrato visual até que o PO autorize mudança.

Isso pode incluir:

- layout;
- hierarquia;
- tipografia;
- espaçamento;
- cores;
- componentes;
- densidade;
- posição dos controles;
- navegação;
- comportamento aparente.

O executor pode implementar dados, estados e comportamento por trás da UI, mas não deve “melhorar” o design por gosto pessoal.

Se não existe UI aprovada ainda, o processo deve primeiro definir e documentar a direção visual antes de tratá-la como contrato.

---

## 12. Decisões arquiteturais

Mudanças estruturais importantes devem ser registradas antes ou junto da implementação.

Exemplos:

- troca de banco;
- troca de framework;
- separação de serviço;
- novo protocolo de comunicação;
- nova estratégia de autenticação;
- mudança de modelo de concorrência;
- alteração de estrutura do repositório;
- inclusão de dependência crítica.

Para decisões de alto impacto, usar ADR contendo:

- contexto;
- problema;
- alternativas consideradas;
- decisão;
- consequências;
- status.

---

## 13. Versionamento e commits

Princípios:

- commits coerentes com a tarefa;
- mensagens descritivas;
- evitar misturar refactor não solicitado com feature;
- não versionar segredos ou dados locais;
- manter branch/PR conforme a política do projeto;
- confirmar que o estado remoto corresponde ao que foi validado quando a entrega depender disso.

Quando a tarefa alterar documentação e código do mesmo comportamento, ambos devem preferencialmente viajar juntos.

---

## 14. Gates de fase

Uma fase deve ter critérios objetivos para ser considerada encerrada.

Exemplo:

```text
FASE DOCUMENTAL
Entrada:
- visão inicial do produto disponível

Saída:
- requisitos consolidados
- arquitetura inicial aprovada
- estrutura do repositório definida
- roadmap definido
- pendências críticas explicitadas
```

Sem gate, fases viram apenas rótulos cronológicos.

---

## 15. Tratamento de mudanças no meio do projeto

Mudança de decisão é normal.

O procedimento correto é:

1. registrar a nova decisão;
2. identificar documentos impactados;
3. marcar/substituir a regra antiga sem apagar indevidamente o histórico;
4. avaliar impacto no código já existente;
5. abrir tarefa específica de migração/ajuste;
6. validar regressões.

Nunca alterar silenciosamente a documentação antiga para esconder que houve mudança de direção.

---

## 16. Como lidar com descoberta de débito técnico

Durante uma tarefa, o Codex pode encontrar problemas adjacentes.

Classificar:

- **bloqueante** — impede a entrega correta; deve ser comunicado e tratado dentro do mínimo necessário;
- **relacionado e pequeno** — pode ser incluído se for indispensável ao objetivo e não ampliar produto;
- **não bloqueante** — registrar como débito/pendência para tarefa futura.

Não transformar toda feature em uma reforma geral do projeto.

---

## 17. Critério de “pronto”

Uma tarefa só está pronta quando:

- escopo prometido está completo;
- critérios de aceite foram satisfeitos;
- validações obrigatórias passaram ou falhas estão explicitamente reportadas;
- não há alteração colateral desconhecida;
- documentação necessária foi atualizada;
- repositório está em estado revisável;
- o relatório final corresponde ao que realmente foi executado.

“Compila” não é sinônimo de “pronto”.

---

## 18. Antipadrões proibidos

Evitar:

- começar pelo código antes de entender o problema;
- arquivo monolítico por conveniência;
- microarquitetura com dezenas de abstrações sem necessidade;
- implementar várias fases de uma vez;
- alterar UI aprovada por preferência pessoal;
- inventar requisito para preencher lacuna;
- declarar tarefa concluída com partes essenciais pendentes;
- fazer refactor amplo dentro de correção pequena sem autorização;
- adicionar dependência sem justificativa;
- manter decisão importante apenas no chat;
- deixar documentação deliberadamente desatualizada;
- esconder conflito em vez de registrá-lo;
- confundir recomendação do Codex com decisão do PO.

---

## 19. Ritmo recomendado de colaboração

O ciclo humano ideal é curto:

1. PO apresenta próximo problema/decisão;
2. assistente analisa e documenta;
3. PO valida pontos de produto necessários;
4. assistente fecha tarefa de execução;
5. Codex executa;
6. resultado e evidências são revisados;
7. documentação é sincronizada;
8. próxima tarefa é aberta.

Isso permite que o projeto avance continuamente sem depender de um planejamento gigantesco e imutável.

---

## 20. Adaptação para um novo projeto

Ao reutilizar este documento:

1. copiar o arquivo para `docs/00-governanca/`;
2. criar um `AGENTS.md` específico para o projeto;
3. definir a estrutura documental;
4. declarar a fonte de verdade e precedência;
5. definir tecnologias somente quando justificadas;
6. registrar decisões iniciais;
7. criar roadmap com uma fase documental/arquitetural;
8. só então liberar tarefas de implementação.

O método deve permanecer estável; detalhes de produto, stack, segurança, deploy e UI pertencem à documentação específica de cada projeto.

---

## 21. Regra final

**O repositório deve permitir que uma nova sessão do assistente ou do Codex descubra o que o projeto é, o que já foi decidido, em que ponto está, qual tarefa está autorizada e como validar a próxima entrega sem depender de memória informal.**

Esse é o principal critério de qualidade deste modelo de trabalho.
