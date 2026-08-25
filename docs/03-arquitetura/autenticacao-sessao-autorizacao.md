# Autenticação, Sessão e Autorização — StepFlow

**Status:** NÚCLEO CONSOLIDADO PARA A FASE 1 / PARÂMETROS DE SEGURANÇA AINDA PENDENTES  
**Atualização:** 2026-08-25

## Princípios consolidados

- autenticação e autorização ocorrem no Host;
- Client nunca é fonte de verdade de permissão;
- senha nunca é armazenada em texto puro;
- sessão inicial é opaca/server-side, sem JWT;
- token fica somente em memória do Client inicialmente;
- nenhuma função `lembrar-me` persistente na primeira versão;
- autorização real é por capacidade no Host;
- `ADM`, `GERENCIA` e `FUNCIONARIO` são presets de capacidades;
- ocultar/desabilitar botão no Client é UX, não controle de segurança.

## Usuário

Campos conceituais principais:

```text
user_id
login
password_hash
name
job_title
avatar_file
role_preset
is_active
is_primary_admin
created_at
updated_at
```

`user_id` é imutável e usado por relacionamentos/histórico.

## Senhas

Consolidado:

- Argon2id em formato PHC;
- salt aleatório;
- parâmetros codificados no hash;
- aceitar frases-senha;
- nunca registrar senha em logs/auditoria.

### Parâmetros ainda não definitivos

Referências/propostas, não contrato de implementação:

- custo Argon2id aproximado de 19 MiB / 2 iterações / paralelismo 1;
- senha mínima de 10 caracteres.

Valores finais serão fechados antes da implementação de autenticação.

## Login e sessão

```text
Client envia login + senha
→ Host verifica usuário ativo + Argon2id
→ Host cria token opaco aleatório
→ Client mantém token em memória
→ Host valida sessão + capacidade em cada operação protegida
```

Consolidado:

- token com alta entropia criptográfica;
- logout revoga sessão;
- desativação/reset/mudança administrativa relevante pode revogar sessões;
- token reutilizável não é persistido em texto puro;
- troca da própria senha mantém a sessão atual e revoga as demais sessões da conta;
- sessão expirada exige nova autenticação;
- edição não salva pode permanecer somente em memória/oculta durante reautenticação do mesmo Client, sem virar draft persistente.

### Parâmetros de sessão pendentes

Propostas, não contrato:

- expiração por inatividade: 8 h;
- validade absoluta: 24 h;
- tamanho/entropia numérica exata do token.

## Perfis e capacidades documentais/administrativas

| Capacidade | ADM | Gerência | Funcionário |
|---|---:|---:|---:|
| Ler processos | sim | sim | sim |
| Criar/editar processos | sim | sim | não |
| Arquivar/publicar processos | sim | sim | não |
| Exportar/imprimir procedimentos | sim | sim | sim |
| Ler/gerir usuários não-ADM | sim | sim | não |
| Criar/promover/rebaixar ADM | sim | não | não |
| Gerir categorias de Procedimentos | sim | sim | não |
| Alterar configuração da empresa | sim | **PENDENTE** | não |
| Backup | sim | **PENDENTE** | não |
| Restore | sim | não | não |

`PENDENTE` não significa sim nem não.

## Capacidades operacionais — Bloco 9

Preset consolidado:

| Capacidade operacional | ADM | Gerência | Funcionário |
|---|---:|---:|---:|
| Consultar Atendimentos | sim | sim | sim |
| Criar Atendimento | sim | sim | sim |
| Editar Atendimento em andamento do qual é responsável | sim | sim | sim |
| Editar qualquer Atendimento em andamento | sim | sim | não |
| Concluir Atendimento do qual é responsável | sim | sim | sim |
| Concluir qualquer Atendimento | sim | sim | não |
| Cancelar Atendimento | sim | sim | não |
| Reabrir Atendimento | sim | sim | não |
| Vincular/trocar/desvincular Equipamento em Atendimento editável | sim | sim | sim, quando responsável |
| Criar/editar cadastro de Equipamento | sim | sim | sim |
| Arquivar/reativar Equipamento | sim | sim | não |
| Adicionar/remover Procedimento em Atendimento editável | sim | sim | sim, quando responsável |
| Selecionar revisão não publicada/histórica para execução | sim | sim | não |
| Marcar/desmarcar checklist em Atendimento editável | sim | sim | sim, quando responsável |
| Gerar/reimprimir ficha de Atendimento acessível | sim | sim | sim |

### Responsabilidade do Funcionário

Para o preset `FUNCIONARIO`, `próprio Atendimento` significa inicialmente:

```text
service_record.responsible_user_id == session.user_id
```

Regras:

- ao criar Atendimento, Funcionário começa como responsável;
- Funcionário padrão não reatribui para outro usuário;
- ADM/Gerência podem atribuir/reatribuir;
- usuário desativado permanece identificável no histórico, mas não é opção normal para nova atribuição.

## Revisões de Procedimento para execução

- Funcionário seleciona normalmente somente revisão publicada que possa ler;
- ADM/Gerência usam publicada por padrão e podem selecionar explicitamente revisão histórica/atual não publicada que já possam ler;
- revisão histórica/não publicada nunca é selecionada silenciosamente;
- autorização de leitura da revisão continua obrigatória além da capacidade operacional de vinculá-la.

## Lifecycle e autorização

Estados:

```text
Em andamento
Concluído
Cancelado
```

- editar/checklist/vínculos: somente em `Em andamento`, conforme capacidade;
- concluir: capacidade de conclusão + estado `Em andamento`;
- cancelar: capacidade própria + estado `Em andamento`;
- reabrir: capacidade própria + `Concluído` ou `Cancelado`;
- Atendimento concluído/cancelado não recebe edição operacional direta;
- mudança posterior exige reabertura.

O Host revalida estado e `row_revision`/base além da capacidade.

## Ficha de Atendimento

Preset `Gerar/reimprimir ficha de Atendimento acessível` é sim para ADM/Gerência/Funcionário.

Ainda são obrigatórios:

- sessão poder consultar aquele Atendimento;
- estado confirmado do Host;
- ausência de conflito/alteração local não salva na geração;
- regras de lifecycle.

Comportamento:

- `Em andamento`: ficha pode ser gerada para acompanhamento;
- `Concluído`: pode ser reimpressa do estado histórico aplicável;
- `Cancelado`: quando gerada/reimpressa, precisa identificar claramente o estado.

Tecnologia/template permanecem no Bloco 10.

## Delegação e capacidades personalizadas

Presets são defaults. Capacidades podem ser personalizadas dentro das regras de delegação já consolidadas.

Regras obrigatórias:

- Gerência nunca administra ADM;
- Gerência não cria/promove/rebaixa ADM;
- usuário não eleva a própria autoridade;
- pelo menos um ADM ativo deve existir;
- `is_primary_admin` não é toggle comum;
- Host impede concessão acima do teto de delegação da sessão;
- ações administrativas/operacionais relevantes são auditadas.

## Perfil próprio

Usuário ativo pode alterar:

- avatar;
- nome de exibição;
- cargo;
- própria senha.

Não altera por conta própria:

- `user_id`;
- perfil/capacidades;
- estado ativo;
- `is_primary_admin`;
- login, quando tratado como identidade de autenticação somente leitura na Tela 11.

## Bootstrap do primeiro ADM

Primeiro ADM é criado por fluxo local/controlado na máquina central quando o banco não possui usuários.

Nunca transformar `primeiro Client da rede` em ADM automaticamente.

Após bootstrap bem-sucedido, o fluxo fica desabilitado para aquele banco.

## Reset/desativação

- reset administrativo nunca revela senha antiga;
- define nova credencial/fluxo;
- revoga sessões pertinentes;
- desativação é preferida à exclusão física para preservar histórico.

## Auditoria de segurança e operação

Registrar de forma proporcional:

- criação/desativação de usuário;
- mudança de perfil/capacidades;
- reset de senha;
- mudança relevante de responsável;
- cancelamento/reabertura/conclusão de Atendimento;
- operações administrativas críticas.

Nunca registrar senha, token reutilizável ou segredo.

## Transporte

Credenciais e sessão usam o canal Client↔Host vigente. Proteção final de transporte na LAN depende da infraestrutura corporativa real.

## Pendências vigentes

Ainda não definitivos:

- custo Argon2id;
- senha mínima;
- expiração de sessão;
- entropia/tamanho numérico exato do token;
- Gerência × configuração da empresa;
- Gerência × Backup.

As capacidades operacionais do Bloco 9 e a gestão de categorias por Gerência **não estão mais pendentes**.

## Regra para implementação futura

Nenhum valor marcado como `PENDENTE`, `PROPOSTA`, `aproximado` ou equivalente pode ser convertido silenciosamente em requisito definitivo pelo executor.