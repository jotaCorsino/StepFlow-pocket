# Autenticação, Sessão e Autorização — StepFlow Pocket

**Status:** NÚCLEO CONSOLIDADO PARA A FASE 1 / PARÂMETROS FINAIS PENDENTES  
**Atualização:** 2026-08-31

## Princípios

- autenticação e autorização ocorrem no Host;
- Client nunca é fonte de verdade de permissão;
- senha nunca é armazenada em texto puro;
- sessão inicial é opaca/server-side, sem JWT;
- token fica somente em memória do Client inicialmente;
- sem `lembrar-me` persistente na primeira versão;
- autorização real é por capacidade no Host;
- `ADM`, `GERENCIA` e `FUNCIONARIO` são presets de capacidades;
- ocultar/desabilitar ação no Client é UX, não segurança.

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

`user_id` é identidade estável usada por relacionamentos/histórico.

## Senhas

Consolidado:

- Argon2id em formato PHC;
- salt aleatório;
- parâmetros codificados no hash;
- aceitar frases-senha;
- nunca registrar senha em logs/auditoria.

Parâmetros numéricos de custo e senha mínima permanecem **PENDENTES** e serão fechados antes da implementação correspondente. Valores históricos/provisórios não são contrato.

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
- troca da própria senha mantém sessão atual e revoga demais sessões da conta;
- sessão expirada exige nova autenticação;
- edição não salva pode permanecer somente em memória/oculta durante reautenticação do mesmo Client, sem virar draft persistente.

Duração de sessão, inatividade, validade absoluta e tamanho/entropia numérica do token permanecem pendentes.

### Fronteira de sessão após Restore

Consolidado no Bloco 11:

- Restore que falha/cancela **antes** da fase destrutiva não exige revogação global apenas por ter preparado staging;
- qualquer Restore que entra na fase destrutiva invalida todas as sessões/tokens anteriores;
- a invalidação vale tanto para Restore concluído quanto para rollback após a fase destrutiva;
- conteúdo de backup restaurado nunca pode ressuscitar token reutilizável antigo;
- após o fresh Host atingir readiness, Clients precisam autenticar novamente;
- se a implementação futura persistir metadados de sessão, deve existir epoch/revogação/runtime equivalente que preserve essa regra.

Fonte: `../04-planejamento/bloco-11-analise-5-restart-sessoes-reconexao-falhas.md`.

## Capacidades documentais/administrativas

| Capacidade | ADM | Gerência | Funcionário |
|---|---:|---:|---:|
| Ler Procedimentos | sim | sim | sim |
| Criar/editar Procedimentos | sim | sim | não |
| Arquivar/publicar Procedimentos | sim | sim | não |
| Exportar/imprimir Procedimentos | sim | sim | sim |
| Ler/gerir usuários não-ADM | sim | sim | não |
| Criar/promover/rebaixar ADM | sim | não | não |
| Gerir categorias | sim | sim | não |
| Alterar configuração da empresa | sim | **PENDENTE** | não |
| Backup | sim | **PENDENTE** | não |
| Restore | sim | não | não |

`PENDENTE` não significa sim nem não. A Análise 6 do Bloco 11 propõe fechar Gerência × Backup, mas essa proposta ainda não é contrato.

## Capacidades operacionais

| Capacidade | ADM | Gerência | Funcionário |
|---|---:|---:|---:|
| Consultar Atendimentos | sim | sim | sim |
| Criar Atendimento | sim | sim | sim |
| Editar Atendimento próprio em andamento | sim | sim | sim |
| Editar qualquer Atendimento em andamento | sim | sim | não |
| Concluir Atendimento próprio | sim | sim | sim |
| Concluir qualquer Atendimento | sim | sim | não |
| Cancelar Atendimento | sim | sim | não |
| Reabrir Atendimento | sim | sim | não |
| Vincular/trocar/desvincular Equipamento editável | sim | sim | sim, quando responsável |
| Criar/editar Equipamento | sim | sim | sim |
| Arquivar/reativar Equipamento | sim | sim | não |
| Adicionar/remover Procedimento em Atendimento | sim | sim | sim, quando responsável |
| Selecionar revisão não publicada/histórica | sim | sim | não |
| Marcar/desmarcar checklist | sim | sim | sim, quando responsável |
| Gerar/reimprimir Ficha acessível | sim | sim | sim |

### Responsabilidade do Funcionário

`próprio Atendimento` significa inicialmente:

```text
service_record.responsible_user_id == session.user_id
```

- Funcionário inicia como responsável pelo Atendimento que cria;
- Funcionário padrão não reatribui para outro usuário;
- ADM/Gerência podem atribuir/reatribuir;
- usuário desativado permanece no histórico, mas não é opção normal para nova atribuição.

## Revisões de Procedimento para execução

- Funcionário seleciona normalmente somente revisão publicada que possa ler;
- ADM/Gerência usam publicada por padrão e podem selecionar explicitamente revisão histórica/atual não publicada já autorizada;
- revisão histórica/não publicada nunca é escolhida silenciosamente;
- capacidade de vincular não substitui autorização de leitura.

## Lifecycle e autorização

Estados:

```text
Em andamento
Concluído
Cancelado
```

- editar/checklist/vínculos: somente em `Em andamento`, conforme capacidade;
- concluir: capacidade + estado `Em andamento`;
- cancelar: capacidade própria + estado `Em andamento`;
- reabrir: capacidade própria + `Concluído` ou `Cancelado`;
- concluído/cancelado não recebe edição operacional direta;
- mudança posterior exige reabertura;
- Host revalida estado e revisão/base além da capacidade.

## Ficha de Atendimento

`Gerar/reimprimir Ficha de Atendimento acessível` é permitido pelos três presets, condicionado a:

- sessão poder consultar o Atendimento;
- estado confirmado do Host;
- ausência de conflito/alteração local não salva;
- lifecycle aplicável.

- `Em andamento`: gera estado confirmado atual;
- `Concluído`: reimprime estado histórico aplicável;
- `Cancelado`: identifica claramente o estado.

Tecnologia, template e política de overflow estão consolidados na Tela 14 e no Bloco 10; não são pendência deste documento.

## Delegação e capacidades personalizadas

Presets são defaults. Capacidades podem ser personalizadas dentro das regras de delegação.

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
- login quando definido como identidade somente leitura.

## Bootstrap do primeiro ADM

Primeiro ADM é criado por fluxo local/controlado na máquina central quando o banco não possui usuários.

Nunca transformar `primeiro Client da rede` em ADM automaticamente. Após bootstrap bem-sucedido, o fluxo fica desabilitado para aquele banco.

## Reset/desativação

- reset administrativo nunca revela senha antiga;
- define nova credencial/fluxo;
- revoga sessões pertinentes;
- desativação é preferida à exclusão física para preservar histórico.

## Auditoria

Registrar proporcionalmente:

- criação/desativação de usuário;
- mudança de perfil/capacidades;
- reset de senha;
- mudança relevante de responsável;
- conclusão/cancelamento/reabertura de Atendimento;
- operações administrativas críticas, incluindo Backup/Restore quando aplicável.

Nunca registrar senha, token reutilizável ou segredo.

A trilha administrativa específica que precisa atravessar Restore está em análise no Bloco 11 — Análise 6; ainda não é contrato até aprovação.

## Transporte

Credenciais e sessão usam o canal Client↔Host vigente. Proteção final de transporte na LAN depende da infraestrutura corporativa real.

## Pendências vigentes

- custo Argon2id final;
- senha mínima final;
- duração/expiração de sessão;
- entropia/tamanho numérico do token;
- Gerência × configuração da empresa;
- Gerência × Backup — proposta de fechamento na Análise 6 do Bloco 11.

Nenhum valor marcado como pendente pode ser convertido silenciosamente em requisito definitivo pelo executor.