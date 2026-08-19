# Plano Oficial — Fase 1: Fechamento Arquitetural e Especificação

**Status:** EM ANDAMENTO

**Início:** 2026-08-19

## Objetivo

Transformar a arquitetura lógica e os requisitos consolidados na Fase 0 em decisões técnicas e especificações implementáveis, reduzindo incerteza antes da criação da fundação executável do StepFlow.

A Fase 1 pode produzir pesquisas, provas técnicas descartáveis e documentação de decisão. Ela ainda não autoriza construir funcionalidades de negócio como entrega definitiva.

## Princípio de execução

Cada bloco deve seguir esta sequência:

1. identificar requisitos e restrições;
2. pesquisar alternativas e documentação primária;
3. comparar opções e riscos;
4. quando necessário, executar prova técnica pequena e descartável;
5. registrar a decisão ou a pendência restante;
6. atualizar arquitetura e documentos afetados;
7. validar o gate do bloco antes de seguir para dependências posteriores.

## Bloco 0 — Bootstrap do ambiente de trabalho

Antes de qualquer investigação executada pelo Codex, `C:\dev\StepFlow` deve corresponder a uma cópia local íntegra do repositório oficial `jotaCorsino/StepFlow-pocket`.

### Entregável

- clone local validado;
- branch `main` ativa;
- `origin` correto;
- working tree limpo;
- `README.md`, `AGENTS.md` e `docs/` disponíveis localmente.

### Tarefa oficial

`docs/04-planejamento/tarefas-codex/f1-b0-t01-bootstrap-repositorio-local.md`

### Gate

Nenhuma tarefa técnica posterior deve ser executada pelo Codex antes da conclusão desse bootstrap.

## Bloco 1 — Plataforma Windows, Client e distribuição

### Perguntas a fechar

- Tauri é adequado para o StepFlow Client?
- Qual versão/linha deve ser usada?
- Quais versões de Windows podem ser suportadas de forma realista?
- Quais requisitos de WebView/runtime existem?
- Como o cenário sem Internet influencia instalação e execução?
- O Client deve ser instalável, portátil ou iniciado por launcher com cópia local?
- Como aplicar ícone e metadados customizados?

### Entregáveis

- documento de compatibilidade Windows;
- decisão técnica do Client;
- requisitos de runtime;
- estratégia preliminar de distribuição;
- riscos e limitações conhecidos.

### Gate

Nenhum scaffold definitivo do Client será criado antes de a plataforma e a distribuição mínima estarem decididas.

## Bloco 2 — StepFlow Host

### Perguntas a fechar

- qual tecnologia deve implementar o Host;
- serviço Windows, processo em tray/background ou outro formato;
- como iniciar automaticamente na máquina central;
- como configurar porta/endereço sem exigir ação dos técnicos;
- onde ficam dados, logs e configurações;
- como o Host se atualiza sem comprometer o banco;
- como diagnosticar indisponibilidade.

### Entregáveis

- decisão tecnológica do Host;
- modelo de execução no Windows;
- paths conceituais/operacionais;
- estratégia de configuração e logs;
- política inicial de atualização do Host.

## Bloco 3 — Launcher, SMB e atualização do Client

### Requisito imutável

A experiência do técnico deve permanecer:

`ponto de entrada interno do StepFlow` → duplo clique → StepFlow.

Nenhum endereço IP, hostname ou caminho SMB específico está consolidado neste momento. Enquanto o ambiente da empresa não for confirmado, usar apenas notação conceitual, por exemplo:

`\\<HOST-OU-SERVIDOR-DA-EMPRESA>\<COMPARTILHAMENTO>\<PASTA-STEPFLOW>\`

O exemplo anteriormente usado `\\192.168.5.7\Arquivos\StepFlow\` é somente ilustrativo e não pode ser embutido em código ou tratado como requisito.

### Perguntas a fechar

- executar o Client diretamente do compartilhamento é aceitável ou deve ser evitado;
- como um launcher verifica versão;
- onde manter cópia local;
- como lidar com arquivo em uso;
- como validar integridade da versão publicada;
- como fazer rollback mínimo;
- o que ocorre quando a rede está acessível, mas o Host não;
- o que ocorre quando o compartilhamento não está acessível;
- como parametrizar o caminho real sem recompilar desnecessariamente o produto;
- como validar a solução futuramente em uma estação conectada à LAN corporativa.

### Entregáveis

- especificação do launcher;
- fluxo de atualização;
- manifesto/versionamento;
- estratégia de rollback;
- protótipo técnico pequeno, se necessário para validar comportamento SMB;
- estratégia de configuração do endereço real do ambiente sem hardcode de exemplos.

## Bloco 4 — Comunicação Client ↔ Host

### Perguntas a fechar

- protocolo de requisição/resposta;
- canal de eventos;
- endereço e descoberta do Host;
- versionamento de contratos;
- autenticação da sessão;
- timeouts e reconexão;
- incompatibilidade Client/Host;
- payloads de erro padronizados.

### Entregáveis

- contratos de comunicação em nível arquitetural;
- catálogo inicial de erros;
- política de compatibilidade;
- eventos principais;
- ADR da estratégia escolhida, quando apropriado.

## Bloco 5 — Autenticação, usuários e autorização

### Escopo

- usuário de login;
- nome de exibição;
- cargo;
- avatar;
- hash de senha;
- sessão;
- bootstrap do primeiro ADM;
- ADM, Gerência e Funcionário;
- permissões granulares;
- alteração de perfil próprio;
- reset administrativo de senha, se aprovado;
- desativação de contas.

### Entregáveis

- modelo conceitual de usuário;
- matriz de permissões;
- regras de bootstrap;
- estratégia de sessão;
- regras de alteração e desativação;
- critérios de auditoria.

## Bloco 6 — Modelo de processos, dados e histórico

### Entidades mínimas a estudar

- usuário;
- perfil/permissão;
- empresa/configuração;
- processo;
- versão/revisão de processo;
- etapa;
- passo;
- checklist documental;
- bloco de observação/informação/comando;
- histórico/auditoria;
- sessão;
- arquivos de avatar/logo;
- backup.

### Pontos a fechar

- identificadores;
- versão exibida versus revisão técnica;
- schema inicial;
- migrations;
- timestamps;
- exclusão versus arquivamento;
- snapshot/histórico;
- unicidade do código do processo;
- persistência de conteúdo estruturado;
- tamanho/limites razoáveis.

### Entregáveis

- modelo de dados conceitual;
- primeira proposta de schema;
- estratégia de migrations;
- regras de versionamento/histórico.

## Bloco 7 — Concorrência e atualização multiusuário

### Requisitos consolidados

- múltiplos Clients simultâneos;
- nenhuma abertura direta do SQLite pelos Clients;
- escrita coordenada pelo Host;
- revisão otimista;
- nenhuma sobrescrita silenciosa;
- atualização relevante sem refresh manual desnecessário.

### Pontos a fechar

- quais comandos exigem serialização explícita;
- unidade da revisão otimista;
- resposta de conflito;
- UX de recarregar/revisar;
- eventos e invalidação de cache;
- soft lock/presença, se trouxer benefício real;
- comportamento em desconexão/reconexão.

### Entregáveis

- especificação de concorrência;
- catálogo de cenários simultâneos;
- critérios de teste com duas ou mais instâncias;
- decisão sobre presença/soft lock.

## Bloco 8 — Especificação de UI/UX

Documentar como contrato antes do código as telas críticas:

1. Login;
2. Shell/sidebar;
3. Início/Dashboard;
4. Lista e pesquisa de processos;
5. Leitura do processo em formato livro;
6. Editor de processo;
7. Histórico de alterações;
8. Gestão de usuários;
9. Meu perfil;
10. Configurações da empresa;
11. Backup e restauração;
12. Exportação/impressão.

Cada tela deve seguir `docs/templates/template-analise-de-tela.md` e registrar:

- objetivo;
- estrutura visual;
- interações;
- estados;
- validações;
- permissões;
- dados;
- impactos Client/Host;
- pendências;
- critérios de aceite.

A aparência só se torna contrato visual quando aprovada pelo PO.

## Bloco 9 — Checklist durante execução

O checklist que integra a documentação é requisito consolidado. O estado das marcações durante a execução ainda precisa de decisão.

Alternativas a avaliar:

- estado apenas em memória durante a sessão atual;
- estado local por usuário/dispositivo;
- estado persistido no Host como progresso pessoal;
- criação de uma entidade formal de execução do processo.

A Fase 1 deve decidir o comportamento inicial sem criar um módulo de execução mais complexo que o necessário.

## Bloco 10 — Exportação, impressão e identidade

### Requisito consolidado

O produto deve oferecer PDF, DOCX e impressão.

### O que precisa ser validado

- onde gerar cada formato;
- biblioteca/estratégia;
- template comum de dados;
- paginação;
- logo e dados da empresa;
- fontes;
- tabelas, listas e blocos de código;
- nomeação de arquivos;
- compatibilidade com leitores esperados;
- operação completamente offline da Internet.

### Entregáveis

- arquitetura de exportação;
- modelo conceitual do documento exportável;
- decisões de bibliotecas/tecnologias;
- critérios de teste.

## Bloco 11 — Backup e restauração

### Pontos a fechar

- backup consistente do SQLite;
- inclusão de logo, avatares e demais arquivos persistentes;
- formato do pacote de backup;
- retenção manual/automática;
- validação do backup;
- restauração segura;
- bloqueio/coordenacão durante restore;
- recuperação após falha.

### Entregáveis

- política de backup/restore;
- fluxo administrativo;
- critérios de validação e recuperação.

## Bloco 12 — Estrutura oficial do repositório e plano da Fase 2

Somente após as decisões anteriores estarem suficientemente fechadas:

- definir árvore oficial de Client, Host, contratos, docs, testes, scripts e assets;
- definir convenções de nomes;
- definir arquivos de configuração e dados ignorados pelo Git;
- definir scripts de desenvolvimento;
- preparar tarefas pequenas da fundação técnica;
- criar o plano oficial da Fase 2.

## O que a Fase 1 não deve fazer

- implementar CRUD final de processos;
- entregar autenticação definitiva;
- criar UI de produção sem documentação/aprovação;
- criar banco com dados reais;
- construir launcher definitivo sem validação;
- antecipar features de fases posteriores;
- transformar protótipos de investigação em produção por conveniência.

## Critérios de saída da Fase 1

- [ ] bootstrap local validado;
- [ ] plataforma Client aprovada;
- [ ] compatibilidade Windows documentada;
- [ ] Host definido;
- [ ] launcher/update tecnicamente validado;
- [ ] comunicação Client/Host definida;
- [ ] sessão e autorização definidas;
- [ ] modelo de dados conceitual fechado;
- [ ] concorrência especificada;
- [ ] comportamento de checklist de execução decidido;
- [ ] telas críticas documentadas e direção visual aprovada no nível necessário para a Fase 2;
- [ ] exportação definida tecnicamente;
- [ ] backup/restore definido;
- [ ] estrutura oficial do repositório definida;
- [ ] pendências não bloqueantes explicitadas;
- [ ] plano oficial da Fase 2 aprovado.

## Ordem inicial autorizada

1. **Bloco 0 — Bootstrap do ambiente de trabalho**;
2. depois, **Bloco 1 — Plataforma Windows, Client e distribuição**.

Nenhum scaffold definitivo deve ser criado durante essa investigação, salvo prova técnica explicitamente descartável e identificada como tal.