# Autenticação, Sessão e Autorização — StepFlow Pocket

**Status:** CONSOLIDADO PARA A FASE 1  
**Atualização:** 2026-09-01

## Princípios

- autenticação e autorização ocorrem no Host;
- Client nunca é fonte de verdade de permissão;
- senha nunca é armazenada em texto puro;
- sessão é opaca/server-side, sem JWT no baseline;
- token fica somente em memória do Client;
- sem `lembrar-me` persistente na primeira versão;
- autorização real é por capacidade no Host;
- `ADM`, `GERENCIA` e `FUNCIONARIO` são presets;
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

## Senhas — D12.56–D12.59

Baseline:

```text
Argon2id
memory = 65536 KiB
passes = 3
parallelism = 4
salt = 16 bytes aleatórios
output = 32 bytes
encoding = PHC

senha = 15–128 caracteres Unicode após NFKC
limite adicional = 512 bytes UTF-8 após normalização
```

Regras:

- aceitar espaços e frases-senha;
- permitir copy/paste;
- sem regra obrigatória de maiúscula/minúscula/número/símbolo;
- sem rotação periódica obrigatória;
- nunca truncar silenciosamente;
- nunca registrar senha em logs/auditoria;
- blocklist offline com pelo menos 10.000 senhas comuns/comprometidas;
- nenhum login/criação de senha depende de Internet;
- redução futura do custo Argon2id exige benchmark + decisão explícita;
- hash PHC preserva parâmetros para rehash futuro quando aplicável.

Throttling por conta:

```text
falhas 1–4 → resposta normal sanitizada
falha 5    → atraso 2 s
falha 6    → atraso 4 s
falha 7    → atraso 8 s
falha 8    → atraso 16 s
falha 9    → atraso 30 s
falha 10   → cooldown de 15 min
```

- login bem-sucedido zera contador;
- não revelar se login existe;
- não criar lockout permanente automático apenas por tentativas remotas.

## Login e sessão — D12.60–D12.61

```text
Client envia login + senha
→ Host verifica usuário ativo + Argon2id
→ Host cria token opaco aleatório
→ Client mantém token em memória
→ Host valida sessão + capacidade em cada operação protegida
```

Token baseline:

- 32 bytes/256 bits por CSPRNG;
- opaco, sem user ID/role/timestamp embutido;
- representação externa base64url sem padding ou equivalente;
- persistência server-side guarda apenas digest do token, nunca bearer reutilizável em texto puro;
- logout/expiração/revogação invalidam no Host.

Sessão baseline:

```text
idle_timeout = 30 min
absolute_timeout = 8 h
```

- idle renova apenas por atividade autenticada significativa do usuário;
- ping/pong WebSocket, heartbeat, polling/refetch de background não renovam idle;
- absolute timeout não é estendido por atividade comum;
- sessão expirada exige nova autenticação;
- troca da própria senha mantém sessão atual e revoga demais sessões da conta;
- edição não salva pode permanecer somente em memória/oculta durante reautenticação do mesmo Client, sem virar draft persistente.

### Fronteira de sessão após Restore

- Restore que falha/cancela antes da fase destrutiva não exige revogação global apenas por staging;
- qualquer Restore que entra na fase destrutiva invalida todas as sessões/tokens anteriores;
- vale tanto para Restore concluído quanto rollback após fase destrutiva;
- backup restaurado nunca ressuscita token reutilizável antigo;
- fresh Host exige novo login;
- implementação de sessão persistente deve preservar epoch/revogação equivalente.

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
| Alterar configuração da empresa | sim | sim | não |
| Backup | sim | sim | não |
| Restore | sim | não | não |

- Gerência pode consultar/criar Backup manual, mas nunca recebe Restore por consequência;
- D12.62 concede à Gerência alteração da identidade da empresa;
- disaster recovery local não é capability de sessão; usa acesso local/ACL/exclusividade quando o banco não consegue autenticar.

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

`próprio Atendimento` significa:

```text
service_record.responsible_user_id == session.user_id
```

- Funcionário inicia como responsável pelo Atendimento que cria;
- Funcionário padrão não reatribui para outro usuário;
- ADM/Gerência podem atribuir/reatribuir;
- usuário desativado permanece no histórico, mas não é opção normal para nova atribuição.

## Revisões de Procedimento para execução

- Funcionário seleciona normalmente somente revisão publicada que possa ler;
- ADM/Gerência usam publicada por padrão e podem selecionar explicitamente revisão histórica/atual não publicada autorizada;
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

`Gerar/reimprimir Ficha de Atendimento acessível` é permitido pelos três presets quando a sessão puder consultar o Atendimento e o estado confirmado permitir.

- `Em andamento`: gera estado confirmado atual;
- `Concluído`: reimprime estado histórico aplicável;
- `Cancelado`: identifica claramente o estado.

Tecnologia/template/overflow pertencem à Tela 14 e Bloco 10.

## Delegação e capacidades personalizadas

- Gerência nunca administra ADM;
- Gerência não cria/promove/rebaixa ADM;
- Gerência não pode conceder Restore;
- usuário não eleva a própria autoridade;
- pelo menos um ADM ativo deve existir;
- `is_primary_admin` não é toggle comum;
- Host impede concessão acima do teto de delegação da sessão;
- ações administrativas/operacionais relevantes são auditadas.

## Perfil próprio

Usuário ativo pode alterar avatar, nome de exibição, cargo e própria senha.

Não altera por conta própria `user_id`, perfil/capacidades, estado ativo, `is_primary_admin` ou login definido como identidade somente leitura.

## Bootstrap do primeiro ADM

Primeiro ADM é criado por fluxo local/controlado na máquina central quando o banco não possui usuários. Nunca transformar `primeiro Client da rede` em ADM automaticamente. Após bootstrap, o fluxo fica desabilitado para aquele banco.

## Reset/desativação

- reset administrativo nunca revela senha antiga;
- define nova credencial/fluxo e revoga sessões pertinentes;
- desativação é preferida à exclusão física para preservar histórico.

## Auditoria

Registrar proporcionalmente criação/desativação de usuário, capacidades, reset, mudança de responsável, lifecycle de Atendimento e operações administrativas críticas. Nunca registrar senha, token reutilizável ou segredo.

Backup/Restore/Recovery também emitem trilha administrativa estruturada fora de `data/` conforme D11.

## Transporte

Credenciais e sessão usam o canal Client↔Host vigente. Proteção final de transporte na LAN depende da infraestrutura corporativa real e permanece gate de ambiente, sem hardcode de PKI/domínio.

## Estado

Os parâmetros numéricos iniciais de senha/Argon2id/sessão/token e Gerência × configuração da empresa estão **consolidados em D12.56–D12.62**. Ajustes futuros exigem mudança explícita e não podem ser inferidos pelo executor.
