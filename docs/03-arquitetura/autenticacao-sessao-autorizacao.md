# Autenticação, Sessão e Autorização — StepFlow

**Status:** NÚCLEO CONSOLIDADO PARA A FASE 1 / PARÂMETROS OPERACIONAIS PENDENTES  
**Atualização:** 2026-08-21

## Princípios consolidados

- autenticação e autorização ocorrem no Host;
- Client nunca é fonte de verdade de permissão;
- senha nunca é armazenada em texto puro;
- sessão inicial é opaca/server-side, sem JWT;
- token fica somente em memória do Client inicialmente;
- nenhuma função “lembrar-me” persistente na primeira versão;
- autorização real é por capacidade no Host;
- `ADM`, `GERENCIA` e `FUNCIONARIO` são presets de permissões.

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

Relacionamentos e histórico usam `user_id` imutável.

## Senhas

Decisão consolidada:

- usar **Argon2id** em formato PHC;
- salt aleatório;
- parâmetros codificados no próprio hash;
- aceitar frases-senha;
- nunca registrar senha em logs/auditoria.

### Parâmetros ainda não autorizados como definitivos

Os valores abaixo são referência técnica/proposta e **não devem ser implementados como política definitiva sem confirmação antes da implementação**:

- custo Argon2id aproximado de 19 MiB / 2 iterações / paralelismo 1;
- senha mínima de 10 caracteres.

A decisão final deverá equilibrar segurança, desempenho das máquinas reais e política interna aplicável.

## Login e sessão

Fluxo consolidado:

```text
Client envia login + senha
→ Host verifica usuário ativo + Argon2id
→ Host cria token opaco aleatório
→ Client mantém token em memória
→ Host valida sessão e autorização a cada operação protegida
```

Também estão consolidados:

- token com alta entropia criptográfica;
- logout revoga sessão;
- desativação/reset/mudança administrativa relevante pode revogar sessões;
- token reutilizável não deve ser persistido em texto puro.

### Parâmetros de sessão pendentes

Os valores abaixo são **PROPOSTAS, não contrato de implementação**:

- expiração por inatividade: 8 h;
- validade absoluta: 24 h;
- tamanho/entropia numérica exata do token, desde que satisfaça segurança adequada.

A duração final da sessão será confirmada antes da implementação da autenticação, considerando a rotina real da empresa.

## Perfis e permissões

Matriz consolidada onde há decisão explícita:

| Capacidade | ADM | Gerência | Funcionário |
|---|---:|---:|---:|
| Ler processos | sim | sim | sim |
| Criar/editar processos | sim | sim | não |
| Excluir/arquivar/publicar | sim | sim | não |
| Exportar/imprimir | sim | sim | sim |
| Ler/gerir usuários não-ADM | sim | sim | não |
| Criar/promover/rebaixar ADM | sim | não | não |
| Alterar configuração da empresa | sim | **PENDENTE** | não |
| Backup | sim | **PENDENTE** | não |
| Restore | sim | não | não |

Itens `PENDENTE` não podem ser interpretados pelo executor como “sim” nem “não”. Devem ser resolvidos antes da implementação correspondente.

Regras obrigatórias:

- Gerência nunca administra ADM;
- usuário não eleva a própria autoridade;
- pelo menos um ADM ativo deve existir;
- ações administrativas críticas são auditadas.

## Perfil próprio

Usuário ativo pode alterar, dentro das regras:

- avatar;
- nome de exibição;
- cargo;
- própria senha.

Não altera por conta própria `user_id`, perfil/permissões, estado ativo ou `is_primary_admin`.

## Bootstrap do primeiro ADM

O primeiro ADM é criado por fluxo local/controlado na máquina central quando o banco ainda não possui usuários. Nunca transformar “primeiro Client da rede” em ADM automaticamente.

Após bootstrap bem-sucedido, o fluxo fica desabilitado para aquele banco.

## Reset/desativação

Reset administrativo nunca revela senha antiga; define nova credencial/fluxo e revoga sessões pertinentes.

Preferir desativar contas em vez de exclusão física para preservar histórico.

## Auditoria de segurança

Registrar ações relevantes como criação/desativação de usuário, mudança de perfil, reset de senha e operações administrativas. Não registrar senha, token reutilizável ou hash de senha sem necessidade.

## Transporte

Credenciais e sessão usam o canal Client↔Host definido em `comunicacao-client-host.md`. A proteção final de transporte na LAN depende da infraestrutura corporativa real.

## Regra para implementação futura

Nenhum valor marcado como `PENDENTE`, `PROPOSTA`, “aproximado” ou equivalente neste documento pode ser convertido silenciosamente em requisito definitivo pelo Codex. A tarefa de implementação deverá carregar a decisão final correspondente.
