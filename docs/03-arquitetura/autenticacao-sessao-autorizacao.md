# Autenticação, Sessão e Autorização — StepFlow

**Status:** CONSOLIDADO PARA A FASE 1  
**Atualização:** 2026-08-20

## Princípios

- autenticação e autorização ocorrem no Host;
- Client nunca é fonte de verdade de permissão;
- senha nunca é armazenada em texto puro;
- sessão inicial é opaca/server-side, sem JWT;
- token fica somente em memória do Client inicialmente;
- nenhuma função “lembrar-me” persistente na primeira versão.

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

Usar **Argon2id** em formato PHC, com salt aleatório e parâmetros codificados no próprio hash.

Baseline inicial de implementação:

- memória: ~19 MiB;
- iterações: 2;
- paralelismo: 1;
- mínimo inicial de senha: 10 caracteres, sujeito a ajuste antes da implementação.

Aceitar frases-senha; não exigir regras artificiais de composição como mecanismo principal. Nunca registrar senha em logs/auditoria.

## Login e sessão

```text
Client envia login + senha
→ Host verifica usuário ativo + Argon2id
→ Host cria token opaco aleatório
→ Client mantém token em memória
→ Host valida sessão e autorização a cada operação protegida
```

Direções iniciais:

- entropia do token equivalente a pelo menos 256 bits;
- persistir somente hash/representação derivada do token quando necessário;
- logout revoga sessão;
- desativação/reset/mudança administrativa relevante podem revogar sessões;
- expiração por inatividade proposta: 8 h;
- validade absoluta proposta: 24 h.

Tempos finais podem ser ajustados à rotina real da empresa.

## Perfis e permissões

`ADM`, `GERENCIA` e `FUNCIONARIO` são presets. Autorização real é por capacidade.

Matriz inicial:

| Capacidade | ADM | Gerência | Funcionário |
|---|---:|---:|---:|
| Ler processos | sim | sim | sim |
| Criar/editar processos | sim | sim | não |
| Excluir/arquivar/publicar | sim | sim | não |
| Exportar/imprimir | sim | sim | sim |
| Ler/gerir usuários não-ADM | sim | sim | não |
| Criar/promover/rebaixar ADM | sim | não | não |
| Alterar configuração da empresa | sim | não por padrão | não |
| Backup | sim | não por padrão | não |
| Restore | sim | não | não |

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
