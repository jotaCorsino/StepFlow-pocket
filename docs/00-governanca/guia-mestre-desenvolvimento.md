# Guia Mestre de Desenvolvimento — StepFlow Pocket

**Status:** INICIAL / EM CONSOLIDAÇÃO

## Objetivo

Consolidar as regras de produto, arquitetura, execução e governança já definidas para o StepFlow Pocket, separando claramente decisões vigentes de itens que ainda exigem validação.

## Fonte de verdade

Ordem prática de uso:

1. decisões consolidadas mais recentes em `docs/05-progresso/registro-de-decisoes.md`;
2. documentação específica vigente de tela, arquitetura ou fase;
3. este guia mestre;
4. `docs/01-produto/visao-geral.md`;
5. materiais históricos e conversas anteriores.

Ambiguidades relevantes devem ser registradas como pendência. Codex não deve resolvê-las por preferência própria.

## Princípios do projeto

- simplicidade operacional para o usuário final;
- modularidade e manutenção simples no código;
- uma tarefa técnica por vez;
- documentação antes de decisões estruturais irreversíveis;
- GitHub como fonte principal de verdade;
- implementação e documentação sincronizadas;
- evitar monólitos acidentais;
- evitar superengenharia;
- não ignorar requisitos estruturais conhecidos em nome de uma falsa simplicidade;
- preservar UX e direção visual aprovadas;
- validar mecanicamente cada entrega proporcionalmente ao risco.

## Papéis

### PO

Define produto, prioridades, comportamento, direção visual e aprova decisões que alterem a experiência ou o escopo.

### Assistente

Analisa, consolida requisitos, propõe arquitetura, mantém documentação, identifica conflitos e transforma decisões aprovadas em tarefas executáveis.

### Codex

Executa tecnicamente tarefas pequenas e fechadas, lê a documentação vigente, evita ampliar escopo e registra evidências de conclusão.

## Contexto operacional do produto

O StepFlow Pocket será utilizado internamente por poucos usuários em computadores Windows conectados à mesma rede local.

Experiência desejada:

```text
\\192.168.5.7\Arquivos\StepFlow\
        StepFlow.exe
             ↓
        duplo clique
             ↓
           login
             ↓
         uso normal
```

A complexidade de host, persistência, atualização ou comunicação deve permanecer invisível para o técnico sempre que possível.

## Direção tecnológica corrente

A solução está sendo planejada em três blocos:

### StepFlow Client

Aplicativo desktop com interface construída em HTML, CSS e JavaScript modular, empacotada como aplicação Windows.

Direção corrente: Tauri como contêiner desktop, sujeita a validação técnica formal antes da fase de implementação.

### StepFlow Host

Processo/serviço leve executado na máquina que hospeda os dados. Responsável por:

- autenticação;
- autorização;
- usuários;
- leitura/escrita do banco;
- fila/serialização das alterações necessárias;
- controle de concorrência;
- eventos de atualização aos clientes;
- backup e operações administrativas relacionadas aos dados.

### StepFlow Data

Dados persistentes locais ao host:

- SQLite;
- logo da empresa;
- avatares;
- anexos futuros, se aprovados;
- backups;
- configurações persistentes.

## Regra de concorrência

Vários computadores devem poder abrir e utilizar o StepFlow simultaneamente.

É proibido adotar como arquitetura principal vários clientes abrindo diretamente o mesmo arquivo SQLite por caminho de rede.

O banco deve ficar local ao host responsável por acessá-lo. Clientes conversam com o host por contratos de aplicação.

Para alterações concorrentes em uma mesma documentação, a direção funcional corrente é combinar:

- serialização/fila de escrita onde fizer sentido;
- revisão/versionamento otimista;
- detecção de conflito antes de sobrescrita silenciosa;
- atualização dos demais clientes quando houver mudança relevante.

## Frontend e organização do código

O projeto não deve concentrar UI e lógica em um único HTML ou JavaScript gigante.

Diretrizes:

- usar ES Modules;
- organizar por funcionalidades/domínios;
- usar classes JavaScript quando estado e comportamento justificarem;
- preferir funções/módulos simples quando uma classe não trouxer benefício;
- separar componentes compartilhados de módulos de domínio;
- separar estilos por responsabilidade;
- manter baixo acoplamento entre tela, persistência e transporte;
- evitar framework frontend pesado sem necessidade comprovada.

## Gestão de processos — modelo de informação

Campos principais atualmente aprovados:

- Código;
- Título;
- Área / Departamento;
- Responsável;
- Status;
- Versão;
- Objetivo;
- Observações;
- Pré-requisitos;
- Etapas do processo;
- Histórico de alterações.

Campos adicionais não devem ser introduzidos apenas por convenção burocrática.

## Experiência das etapas

A etapa é o núcleo da experiência de execução.

Cada etapa deve funcionar visualmente como uma página de um livro/manual, contendo conforme necessário:

- título da etapa;
- instrução/introdução;
- passos numerados;
- observações;
- checklist;
- blocos de atenção/informação;
- blocos de texto ou linha de comando copiável;
- navegação anterior/próxima;
- indicação clara da posição atual dentro do processo.

O botão de copiar deve ser visualmente discreto e representado apenas por ícone. O feedback de cópia deve ser curto e não poluir a página.

O checklist usado durante uma execução não deve, por si só, alterar a documentação oficial.

## Direção visual

- interface corporativa, limpa e discreta;
- sidebar lateral esquerda;
- logo da empresa pequeno, clássico, no topo da sidebar, alinhado à esquerda;
- evitar aparência de portal corporativo pesado;
- priorizar legibilidade e execução prática por técnicos;
- preservar a metáfora de manual/livro na leitura do processo;
- reduzir ruído visual e botões textuais quando ícones claros forem suficientes.

A documentação detalhada de cada tela será criada em `docs/02-telas` antes de tratá-la como contrato visual definitivo.

## Configuração da empresa

O sistema deve permitir ao menos:

- identificação/nome da empresa;
- logo PNG com fundo transparente;
- orientações de dimensão e peso recomendados para o logo;
- configurações pertinentes ao cabeçalho/documentação exportada.

Detalhes finais de campos e limites serão definidos na documentação específica de configurações.

## Usuários e autenticação

Autenticação é local/interna e deliberadamente simples, sem requisitos de autenticação pública de alta complexidade.

A senha nunca deve ser armazenada em texto puro.

Dados básicos de uma conta administrativa:

- identificador/nome de usuário para login;
- nome de exibição;
- cargo;
- senha armazenada de forma segura;
- avatar;
- perfil/permissões.

Cada usuário pode editar:

- avatar;
- nome de exibição;
- cargo;
- senha.

O identificador usado para relacionamentos internos e histórico deve ser estável e não depender apenas do nome de exibição.

## Perfis e permissões

Perfis padrão:

### ADM

Controle total e autoridade para gestão de usuários e configurações.

### Gerência

Pode criar e editar documentações e possuir poderes administrativos delegados, inclusive criação de contas dentro dos limites definidos.

Não deve poder promover-se a ADM ou alterar a autoridade do ADM principal sem regra explícita futura.

### Funcionário

Foco em leitura e execução das documentações. Não edita a documentação oficial.

A implementação deve preferir perfil como conjunto padrão de permissões, preservando possibilidade de granularidade quando necessário.

## Versionamento e histórico

Documentações de processo precisam manter histórico de alterações.

Alterações relevantes devem preservar rastreabilidade de:

- versão/revisão;
- data/hora;
- usuário responsável;
- descrição da alteração quando aplicável.

O sistema não deve sobrescrever silenciosamente uma edição concorrente baseada em revisão antiga.

## Atualização entre clientes

Direção corrente: clientes conectados devem receber atualização sobre mudanças relevantes sem depender de recarregar manualmente a aplicação.

Tecnologia/protocolo específico ainda deve ser validado formalmente antes da implementação.

## Exportação

Desejado para o produto:

- PDF;
- DOCX;
- impressão.

A exportação deve partir de um modelo de documento próprio, e não simplesmente de uma captura visual da tela de execução.

A exportação deve considerar identidade da empresa e estrutura da documentação.

## Backup e recuperação

O produto deve prever mecanismo simples de backup e restauração dos dados persistentes.

A política final de backup, formato e retenção ainda será detalhada.

## Distribuição e atualização

O técnico deve poder iniciar o StepFlow a partir do compartilhamento de rede com duplo clique.

A direção proposta é usar esse ponto de entrada como launcher, permitindo que o executável real rode localmente e seja sincronizado/atualizado a partir da versão publicada na rede.

Esse mecanismo ainda exige validação técnica antes de virar decisão arquitetural definitiva.

## Regras de segurança proporcionais ao contexto

Mesmo sendo um sistema interno:

- senhas não podem ficar em texto puro;
- permissões devem ser verificadas no host, não apenas escondidas na UI;
- dados sensíveis e bancos reais não entram no Git;
- operações destrutivas relevantes devem possuir confirmação adequada;
- exclusão e alteração de documentação devem gerar rastreabilidade;
- entradas e arquivos enviados devem ter validação mínima.

## Documentação obrigatória

- toda tela relevante terá documento próprio;
- toda decisão importante será registrada;
- toda mudança estrutural relevante deverá atualizar arquitetura e/ou ADR;
- fases e status serão mantidos no roadmap;
- entregas relevantes atualizarão diário e changelog;
- tarefas Codex deverão indicar quais documentos precisam ser atualizados.

## Estado atual

Estamos na fase de documentação e consolidação inicial.

Ainda não está autorizada implementação funcional ampla.

Próximas frentes documentais:

- arquitetura detalhada Client/Host/Data;
- estrutura oficial do repositório;
- modelo de dados conceitual;
- contratos Client ↔ Host;
- fluxo de autenticação/permissões;
- documentação das telas;
- estratégia de launcher/atualização;
- exportação;
- backup;
- plano de implementação da fundação técnica.

## Pendências técnicas que precisam de validação antes do código

- versão e configuração final do Tauri;
- suporte alvo real de Windows e requisitos de WebView/runtime;
- comportamento seguro do launcher iniciado por compartilhamento SMB;
- protocolo final entre Client e Host;
- estratégia de descoberta/endereço do Host;
- mecanismo de atualização em tempo real;
- estratégia de empacotamento do Host;
- política de execução automática do Host no Windows;
- algoritmo final de hash de senha e gestão de sessão;
- biblioteca/estratégia final para PDF e DOCX;
- política de backup e restauração.

Nenhuma dessas pendências autoriza improvisação do Codex. Elas devem ser fechadas na fase arquitetural apropriada.
