# Bloco 12 — Análise 4 — Parâmetros finais e pendências funcionais/técnicas

**Status:** APROVADA PELO PO EM 2026-09-01 — D12.56–D12.79  
**Bloco:** 12 — Estrutura oficial + plano da Fase 2  
**Data:** 2026-09-01

## Objetivo

Fechar os parâmetros e regras que não podem ficar à escolha do executor antes da Fase 2.

Esta análise parte de D12.1–D12.55 e dos contratos consolidados de autenticação, Tela 12, comunicação Client↔Host e Backup/Restore. A aprovação destes parâmetros não autoriza implementação antes do gate final do Bloco 12.

## Autenticação, senha e sessão

Baseline aprovado:

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
token = 32 bytes CSPRNG
idle_timeout = 30 min
absolute_timeout = 8 h
```

Regras:

- aceitar espaços/passphrases e copy/paste;
- sem regra obrigatória de maiúscula/minúscula/número/símbolo;
- sem rotação periódica obrigatória;
- nunca truncar silenciosamente;
- blocklist offline com pelo menos 10.000 valores comuns/comprometidos;
- throttling por conta: falhas 1–4 normais; 5–9 com atrasos 2/4/8/16/30 s; décima entra em cooldown de 15 min;
- não revelar existência do login;
- token opaco, somente em memória no Client; persistência server-side guarda somente digest;
- WebSocket/heartbeat/background não renovam idle;
- redução futura de parâmetros de segurança exige benchmark e decisão explícita.

## Configuração da empresa

Matriz inicial aprovada:

| Capacidade | ADM | Gerência | Funcionário |
|---|---:|---:|---:|
| Alterar configuração da empresa | sim | sim | não |

Limites:

- nome da empresa: 1–120 caracteres;
- contato: até 160;
- site: até 200;
- e-mail: até 254;
- logo: PNG/JPEG, até 2 MiB e 2048×2048;
- Host valida conteúdo, decodifica e reencoda para arquivo administrado seguro;
- SVG não entra no baseline inicial.

## Categoria arquivada em nova revisão

- categoria arquivada não aparece para nova associação;
- se já estiver na revisão-base, a nova revisão preserva a associação por default e a identifica como `Arquivada`;
- isso não bloqueia salvar/publicar;
- usuário autorizado pode removê-la;
- depois de removida, não pode ser adicionada novamente enquanto arquivada;
- para voltar a associá-la, deve ser reativada primeiro;
- nunca remover ou substituir categoria automaticamente.

## Backup / Restore — limites iniciais

```text
retention_max_confirmed_backups = 20
retention_range = 5–100

max_entries = 10_000
max_total_payload = 8 GiB
max_managed_file = 16 MiB
max_logical_path = 512 UTF-16 code units
max_path_component = 120 UTF-16 code units
max_path_depth = 8
min_free_space_reserve = 1 GiB

backup_capture_target <= 2 s
backup_capture_hard_limit = 10 s

pre_restore_no_progress_timeout = 120 s
pre_restore_total_limit = 10 min
```

- exceder 10 s de captura aborta o novo Backup e libera o barrier sem alterar estado ativo;
- `pre_restore` pode ser abortado com segurança antes do primeiro rename;
- depois de `DESTRUCTIVE_STARTED`, timeout genérico não mata a operação; journal/recovery decide o resultado;
- preflight considera temporários coexistentes e a reserva final mínima;
- backup confirmado não é apagado antecipadamente apenas para abrir espaço.

## Host, comunicação e reconexão

```text
readiness_timeout_per_launch = 30 s
restore_relaunch_attempts = 3 total
restore_relaunch_backoff = 1 s, 3 s

connect_timeout = 5 s
common_request_timeout = 30 s
websocket_reconnect = 1, 2, 4, 8, 15, 30 s
jitter = ±20%
backoff_reset_after_stable = 60 s
```

- timeout de mutação exige reconciliação antes de retry;
- reconexão WebSocket causa refetch/reconciliação;
- falha das três tentativas de relaunch encerra o ciclo automático e exige intervenção local/controlada;
- nenhum desses valores autoriza watchdog permanente.

## Rotação de logs

```text
log técnico: 20 MiB + 10 archives
admin audit: 50 MiB + 20 archives
```

- journal de Restore/Recovery não é log rotativo comum;
- nunca registrar senha/token/dump de conteúdo;
- política corporativa superior pode exigir retenção maior;
- em `uncertain/RECOVERY_REQUIRED`, proteções de D11 continuam vigentes.

## Decisões D12.56–D12.79

- **D12.56:** Argon2id usa `m=65536 KiB`, `t=3`, `p=4`, salt 16 bytes e output 32 bytes em PHC;
- **D12.57:** senha single-factor exige 15–128 caracteres Unicode após NFKC, sem composition rule/rotação periódica e sem truncamento;
- **D12.58:** Host usa blocklist offline com pelo menos 10.000 senhas comuns/comprometidas;
- **D12.59:** login usa throttling progressivo a partir da quinta falha e cooldown temporário de 15 min na décima;
- **D12.60:** token de sessão possui 32 bytes CSPRNG, é opaco, fica em memória no Client e somente digest é persistido server-side;
- **D12.61:** sessão expira após 30 min de inatividade ou 8 h absolutas; heartbeat/background não renovam idle;
- **D12.62:** GERENCIA pode alterar configuração da empresa; Funcionário não;
- **D12.63:** identidade da empresa usa limites 120/160/200/254 para nome/contato/site/e-mail;
- **D12.64:** logo aceita PNG/JPEG até 2 MiB e 2048×2048, com validação/decode/reencode Host-side; SVG fica fora;
- **D12.65:** categoria arquivada herdada é preservada com aviso, removível, mas não adicionável/re-adicionável enquanto arquivada;
- **D12.66:** retenção default = 20 backups confirmados, configurável de 5 a 100;
- **D12.67:** envelope limita 10.000 entradas, 8 GiB totais, 16 MiB por managed file e paths conforme limites aprovados;
- **D12.68:** preflight exige temporários estimados + 1 GiB de reserva final;
- **D12.69:** Backup normal tem alvo de pausa ≤2 s e hard limit 10 s;
- **D12.70:** `pre_restore` usa 120 s sem progresso e 10 min totais antes da fase destrutiva; depois dela não há kill genérico por timeout;
- **D12.71:** readiness por launch = 30 s e relaunch Restore = 3 tentativas totais com backoff 1 s/3 s;
- **D12.72:** Client usa connect timeout 5 s e request comum 30 s; mutação após timeout exige reconciliação;
- **D12.73:** WebSocket reconecta em 1/2/4/8/15/30 s com jitter ±20% e reset após 60 s estáveis;
- **D12.74:** log técnico rotaciona em 20 MiB com 10 archives;
- **D12.75:** admin audit rotaciona em 50 MiB com 20 archives, respeitando proteção de recovery/uncertain;
- **D12.76:** valores numéricos ficam centralizados no owner/config apropriado, sem magic numbers espalhados;
- **D12.77:** configuração crítica inválida nunca cai silenciosamente em endpoint, senha, path ou valor inseguro;
- **D12.78:** parâmetros de segurança são versionáveis/rehasháveis, mas redução abaixo do baseline exige decisão explícita;
- **D12.79:** estes valores fecham o baseline inicial da Fase 2; benchmark/gates corporativos ainda podem bloquear produção.

## Evidência técnica registrada

A definição foi baseada em NIST SP 800-63B-4, RFC 9106 e OWASP Password Storage/Session Management, sem declarar conformidade normativa automática do produto.

## Fora do escopo

- MFA/passkeys;
- TLS/PKI corporativa final;
- criar blocklist/configuração/código neste PR;
- autorizar scaffold antes do gate final do Bloco 12.
