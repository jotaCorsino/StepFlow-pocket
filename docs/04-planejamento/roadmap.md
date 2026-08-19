# Roadmap do StepFlow Pocket

**Status:** INICIAL

## Objetivo

Organizar a construção do StepFlow Pocket por fases dependentes e verificáveis, impedindo que o projeto avance para código de negócio antes de fechar decisões estruturais que afetam manutenção, concorrência, distribuição e UX.

---

## Fase 0 — Fundação documental e governança

**Status:** EM ANDAMENTO

### Objetivo

Transformar as decisões já discutidas em fonte de verdade versionada e estabelecer o método de trabalho PO + Assistente + Codex.

### Entregáveis

- README inicial;
- índice de documentação;
- `AGENTS.md`;
- método genérico de trabalho assistido;
- guia mestre do StepFlow;
- visão geral do produto;
- arquitetura inicial;
- roadmap;
- registro inicial de decisões;
- diário/changelog iniciais;
- templates de tela e tarefa Codex.

### Fora do escopo

- aplicação funcional;
- instalação de dependências;
- banco real;
- executáveis;
- UI implementada;
- API funcional.

### Gate de saída

A fase termina quando:

- documentação-base estiver coerente;
- decisões já aprovadas estiverem registradas;
- pendências técnicas estiverem explicitadas;
- a próxima fase tiver plano de investigação/validação claro.

---

## Fase 1 — Fechamento arquitetural e especificação

**Status:** PENDENTE

### Objetivo

Validar tecnicamente as propostas e transformar arquitetura conceitual em contratos implementáveis.

### Blocos

#### 1. Stack e compatibilidade

- validar Tauri para o Client;
- definir versões suportadas de Windows;
- validar WebView/runtime e cenário offline;
- definir formato do Host;
- definir estratégia de build/distribuição.

#### 2. Launcher e atualização

- validar duplo clique a partir do compartilhamento SMB;
- testar launcher com cópia local;
- definir manifesto de versão;
- definir atualização e rollback mínimos;
- definir comportamento quando a rede/Host estiver indisponível.

#### 3. Comunicação Client ↔ Host

- definir protocolo;
- contratos;
- versionamento de contrato;
- descoberta/endereço;
- eventos em tempo real;
- tratamento de incompatibilidade entre versões.

#### 4. Autenticação e autorização

- modelo de usuário;
- hash de senha;
- sessão;
- matriz de permissões;
- ADM inicial/bootstrap;
- regras de Gerência e Funcionário.

#### 5. Dados

- modelo conceitual;
- schema inicial;
- migrations;
- revisão/versionamento;
- auditoria;
- política de exclusão/arquivamento;
- arquivos e paths.

#### 6. Concorrência

- fila/serialização;
- optimistic concurrency;
- eventos;
- soft lock/presença;
- testes de duas ou mais estações.

#### 7. UI/UX

Documentar telas principais antes do código:

- Login;
- Shell/sidebar;
- Início/Dashboard;
- Lista/Pesquisa de processos;
- Leitura do processo em formato livro;
- Editor de processo;
- Histórico de alterações;
- Usuários;
- Meu perfil;
- Configurações da empresa;
- Backup;
- Exportação.

#### 8. Exportação e backup

- estratégia PDF;
- estratégia DOCX;
- template exportável;
- backup consistente;
- restore e validação.

### Gate de saída

- stack validada;
- arquitetura Client/Host/Data consolidada;
- estrutura oficial do repositório definida;
- contratos mínimos definidos;
- modelo de dados conceitual fechado;
- autenticação/permissões documentadas;
- telas críticas documentadas;
- estratégia de launcher validada por protótipo;
- plano oficial de implementação da Fase 2 aprovado.

---

## Fase 2 — Fundação técnica executável

**Status:** PENDENTE

### Objetivo

Criar apenas a fundação real do Client e Host, sem tentar entregar todo o produto.

### Entregáveis esperados

- estrutura física do repositório;
- build do Client;
- build do Host;
- comunicação mínima Client/Host;
- health check;
- configuração de desenvolvimento;
- banco SQLite inicial com migrations;
- logging mínimo;
- testes de fundação;
- pacote/execução local de desenvolvimento.

### Gate de saída

- Client abre;
- Host inicia;
- Client detecta Host;
- chamada simples funciona;
- banco inicializa de forma determinística;
- build limpo;
- documentação e scripts de desenvolvimento atualizados.

---

## Fase 3 — Autenticação, usuários e shell

**Status:** PENDENTE

### Objetivo

Entregar a entrada real do sistema e a estrutura base da aplicação.

### Entregáveis

- login;
- logout;
- sessão;
- ADM bootstrap;
- usuários;
- perfis/permissões;
- edição de perfil pessoal;
- avatar;
- sidebar e shell visual;
- logo/configuração básica da empresa;
- autorização real no Host.

### Gate de saída

- perfis possuem comportamento esperado;
- funcionário não consegue executar operação administrativa por API;
- Gerência respeita limites;
- ADM controla usuários;
- sessão funciona em múltiplos clientes;
- tela base preserva UX aprovada.

---

## Fase 4 — Núcleo documental de processos

**Status:** PENDENTE

### Objetivo

Entregar CRUD e persistência do modelo simplificado de documentação.

### Entregáveis

- lista/pesquisa;
- criação;
- edição;
- remoção/arquivamento conforme decisão;
- campos principais;
- etapas;
- passos;
- observações;
- checklist definido na documentação;
- blocos copiáveis;
- histórico;
- revisão/versionamento;
- permissões.

### Gate de saída

- ADM/Gerência administram processos;
- Funcionário apenas lê documentação oficial;
- histórico permanece íntegro;
- revisão concorrente não sobrescreve silenciosamente;
- CRUD passa por validações e testes previstos.

---

## Fase 5 — Experiência de execução em formato livro

**Status:** PENDENTE

### Objetivo

Consolidar o diferencial de UX do StepFlow.

### Entregáveis

- leitor em páginas/etapas;
- anterior/próxima;
- indicador de posição;
- passos claros;
- checklist de execução local/temporário na primeira versão, salvo decisão diferente;
- observações e alertas;
- blocos copiáveis com ícone;
- feedback de cópia;
- comportamento para processo longo;
- estados vazios/erro/carregamento.

### Gate de saída

- execução prática validada contra processos de exemplo;
- leitura é mais simples do que documento tradicional equivalente;
- nenhum elemento de execução altera documentação oficial sem permissão.

---

## Fase 6 — Multiusuário e atualização em tempo real

**Status:** PENDENTE

### Objetivo

Fechar comportamento simultâneo em condições reais de rede.

### Entregáveis

- eventos de alteração;
- refresh/reload controlado;
- presença/soft lock, se aprovado;
- conflito de revisão;
- fila de comandos onde necessária;
- testes com múltiplas instâncias;
- comportamento de reconexão;
- mensagens de Host indisponível.

### Gate de saída

- pelo menos duas estações podem operar simultaneamente;
- alterações não corrompem dados;
- conflito é detectado;
- lista/telas relevantes refletem atualização sem intervenção manual desnecessária.

---

## Fase 7 — Exportação, impressão e identidade

**Status:** PENDENTE

### Objetivo

Permitir saída formal das documentações.

### Entregáveis

- template exportável;
- PDF;
- DOCX;
- impressão;
- logo e dados da empresa;
- paginação/cabeçalho/rodapé conforme especificação;
- nomes de arquivos consistentes.

### Gate de saída

- documentos de exemplo exportam corretamente;
- arquivos abrem em leitores esperados;
- identidade visual permanece legível;
- exportação não depende de captura da tela de execução.

---

## Fase 8 — Backup, launcher e distribuição operacional

**Status:** PENDENTE

### Objetivo

Transformar a aplicação funcional em uma ferramenta simples de distribuir e recuperar.

### Entregáveis

- Host instalável/inicializável na máquina central;
- launcher/ponto de entrada de rede;
- atualização do Client;
- ícone customizado;
- backup;
- restauração;
- logs de diagnóstico básicos;
- documentação de implantação interna;
- teste de cenário sem Internet.

### Gate de saída

Cenário de aceite:

1. técnico abre `\\192.168.5.7\Arquivos\StepFlow\`;
2. executa o ponto de entrada com duplo clique;
3. StepFlow inicia sem configuração manual;
4. conecta ao Host;
5. realiza login;
6. utiliza o sistema;
7. atualização do Client pode ser distribuída centralmente;
8. backup/restore foram testados.

---

## Fase 9 — Hardening e release interno

**Status:** PENDENTE

### Objetivo

Revisar robustez antes de tratar a versão como release interno estável.

### Frentes

- segurança proporcional;
- validação de entradas;
- permissões;
- corrupção/falha de banco;
- recuperação de backup;
- concorrência;
- performance com base realista;
- logs;
- instalação/atualização;
- smoke tests end-to-end;
- revisão documental;
- limpeza de débitos técnicos prioritários.

### Gate de saída

Release interno versionado, reproduzível, documentado e recuperável.

---

## Regra de execução do roadmap

- Não iniciar fase apenas porque a anterior “parece suficiente”. Validar o gate.
- Uma fase pode ter tarefas paralelizáveis conceitualmente, mas a execução com Codex deve continuar em tarefas pequenas e revisáveis.
- Descobertas podem alterar o roadmap; a mudança deve ser registrada antes da implementação afetada.
- O roadmap é instrumento de dependência e direção, não promessa imutável de cronograma.
