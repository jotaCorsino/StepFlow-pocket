# Bloco 12 — Análise 4 — Parâmetros finais e pendências funcionais/técnicas

**Status:** PROPOSTA PARA REVISÃO DO PO  
**Bloco:** 12 — Estrutura oficial + plano da Fase 2  
**Data:** 2026-09-01

## Objetivo

Fechar os parâmetros e regras restantes que não podem ficar à escolha do executor antes da Fase 2.

Esta análise parte de D12.1–D12.55 e dos contratos já consolidados de autenticação, Tela 12, comunicação Client↔Host e Backup/Restore.

Nenhum valor desta análise autoriza implementação antes do gate final do Bloco 12.

---

# 1. Autenticação, senha e sessão

## Argon2id

Baseline proposto para novos hashes:

```text
algorithm = Argon2id
memory = 65536 KiB (64 MiB)
time_cost = 3
parallelism = 4
salt = 16 bytes aleatórios
output = 32 bytes
encoding = PHC
```

Racional:

- preserva o algoritmo já consolidado;
- corresponde à segunda configuração recomendada pelo RFC 9106 para ambientes com menos memória que a opção de 2 GiB;
- fica acima do mínimo atual recomendado pelo OWASP para Argon2id;
- é adequado ao Host central, onde autenticação não ocorre em massa.

O hash PHC guarda versão/parâmetros. Se futuramente um hash válido estiver abaixo do baseline vigente, login bem-sucedido pode rehashar a senha com parâmetros atuais dentro de fluxo transacional apropriado.

A primeira implementação deve medir o Host alvo. Se o baseline produzir latência operacionalmente inviável, não reduzir parâmetros silenciosamente: retornar ao PO/arquitetura com benchmark e proposta explícita.

## Senhas

Baseline proposto:

- mínimo: **15 caracteres Unicode**;
- máximo: **128 caracteres Unicode**;
- normalização NFKC antes de validar comprimento, blocklist e hash;
- aceitar espaços e frases-senha;
- permitir copy/paste;
- sem regra obrigatória de maiúscula/minúscula/número/símbolo;
- sem troca periódica obrigatória;
- mudança forçada apenas por reset, comprometimento ou ação administrativa aplicável;
- nunca truncar silenciosamente.

Para limitar abuso de processamento, a entrada normalizada também fica limitada a 512 bytes UTF-8.

Como o baseline inicial é single-factor, 15 caracteres segue a exigência atual do NIST SP 800-63B-4 para senha usada como fator único.

## Blocklist offline

O Host deve rejeitar senha presente em blocklist local de valores comuns/comprometidos.

Baseline:

- pelo menos 10.000 valores comuns/comprometidos distribuídos com a aplicação;
- comparação após NFKC + case-fold apropriado à blocklist;
- incluir termos óbvios do produto/implantação quando aplicável, sem registrar a senha tentada;
- nenhuma consulta à Internet é necessária no login ou criação de senha.

A blocklist é defesa contra escolhas triviais; não substitui Argon2id nem throttling.

## Throttling de login

Baseline por conta:

```text
falhas 1–4   → resposta normal sanitizada
falha 5      → atraso 2 s
falha 6      → atraso 4 s
falha 7      → atraso 8 s
falha 8      → atraso 16 s
falha 9      → atraso 30 s
falha 10     → cooldown de 15 min
```

Após cooldown, novas falhas retomam controle equivalente. Login bem-sucedido zera o contador.

Regras:

- não revelar se login existe;
- nenhuma senha entra em log;
- não criar lockout permanente automático que exija intervenção administrativa apenas por tentativas remotas;
- Host é autoridade do contador/tempo;
- limites podem receber proteção adicional por origem se ataque real justificar, sem substituir o limite por conta.

## Token de sessão

Baseline:

- 32 bytes (256 bits) gerados por CSPRNG;
- representação externa base64url sem padding, ou codificação equivalente sem reduzir entropia;
- conteúdo totalmente opaco e sem user ID/role/timestamp embutido;
- Client mantém somente em memória;
- persistência server-side guarda somente digest SHA-256/equivalente do token, nunca o bearer reutilizável em texto puro;
- comparação em tempo constante quando aplicável;
- logout/expiração/revogação tornam o token imediatamente inválido no Host.

## Tempo de sessão

Baseline:

```text
idle_timeout = 30 min
absolute_timeout = 8 h
```

- idle é contado desde atividade autenticada significativa do usuário;
- ping/pong de WebSocket, heartbeat, polling automático e refetch de background **não** renovam idle;
- request autenticado causado por ação real do usuário renova idle;
- absolute timeout não é estendido por atividade comum;
- reautenticação bem-sucedida cria/renova a fronteira de sessão conforme implementação aprovada;
- ao expirar, Host invalida sessão e Client volta ao login;
- edição local não salva pode permanecer somente em memória para reautenticação do mesmo Client, conforme contrato vigente.

---

# 2. Configuração da empresa

## Gerência

O preset **GERENCIA recebe capacidade de alterar configuração da empresa**.

Matriz final inicial:

| Capacidade | ADM | Gerência | Funcionário |
|---|---:|---:|---:|
| Alterar configuração da empresa | sim | sim | não |

A capacidade continua granular e Host-side. Customização futura não permite autoelevação nem ultrapassar teto de delegação.

## Campos e limites

Baseline:

| Campo | Regra |
|---|---|
| Nome da empresa | obrigatório, trim, 1–120 caracteres |
| Contato | opcional, trim, até 160 caracteres |
| Site | opcional, trim, até 200 caracteres |
| E-mail | opcional, trim, até 254 caracteres |

- nenhum campo é truncado silenciosamente;
- `site` é identidade textual no baseline, não navegação automática; se superfície futura o tornar clicável, aceitar somente esquema HTTP/HTTPS validado;
- e-mail recebe validação prática de formato, sem tentar implementar toda a gramática RFC por regex gigante.

## Logo

Baseline de entrada:

- PNG ou JPEG;
- máximo 2 MiB no upload;
- máximo 2048 × 2048 pixels após decode;
- Host valida magic/content, não apenas extensão;
- Host decodifica e reencoda para arquivo administrado seguro, preferencialmente PNG, removendo metadata desnecessária;
- rejeitar imagem que não decodifica, excede limites ou contém formato não suportado;
- SVG/conteúdo ativo não entra no baseline inicial.

A ausência de logo continua válida.

---

# 3. Categoria arquivada em nova revisão

Regra editorial proposta:

1. categoria arquivada não aparece como opção para nova associação;
2. se a revisão-base do Procedimento já possui categoria que foi arquivada depois, a nova revisão **preserva essa associação por default**;
3. Editor identifica visualmente a categoria como `Arquivada` e informa que ela foi herdada;
4. isso não bloqueia salvar/publicar a nova revisão;
5. usuário autorizado pode remover a categoria arquivada;
6. depois de removida, ela não pode ser adicionada novamente enquanto permanecer arquivada;
7. para voltar a associá-la, primeiro é necessário reativá-la na Tela 12;
8. nunca remover ou substituir categoria automaticamente por outra.

Isso preserva continuidade editorial e histórico sem permitir novas associações a entidade arquivada.

---

# 4. Backup / Restore — limites numéricos iniciais

## Retenção

```text
retention_max_confirmed_backups = 20
```

- valor default central = 20;
- configuração explícita pode aceitar faixa 5–100;
- ausência/config inválida usa 20 ou falha de configuração conforme owner técnico decidir de forma segura;
- backups protegidos por Restore/migration/resultado uncertain não são candidatos enquanto a proteção existir;
- retenção continua por quantidade e sem scheduler, conforme D11.

## Envelope e parser

Baseline:

```text
max_entries = 10_000
max_total_payload = 8 GiB
max_managed_file = 16 MiB
max_logical_path = 512 UTF-16 code units
max_path_component = 120 UTF-16 code units
max_path_depth = 8
```

- `stepflow.sqlite` pode ocupar até o limite total do payload;
- `max_managed_file` aplica-se a arquivos administrados como `company/**` e `avatars/**`;
- todos os limites são verificados antes/durante coleta e extração;
- ultrapassar limite torna o pacote inelegível/backup inviável; nunca truncar arquivo para caber.

Esses valores são baseline inicial, não promessa de escala ilimitada. Necessidade real acima deles exige revisão explícita do contrato.

## Espaço livre

Preflight deve calcular bytes temporários de pior caso da operação e exigir, **depois** dessa estimativa, reserva mínima de:

```text
min_free_space_reserve = 1 GiB
```

Para Backup `Stored`, considerar pelo menos snapshot bruto + pacote final/staging. Para Restore, considerar candidato extraído + safety snapshot/pacote + demais temporários realmente coexistentes.

A operação não apaga backup confirmado antecipadamente apenas para criar espaço.

## Duração da captura

Backup normal:

```text
capture_pause_target <= 2 s
capture_pause_hard_limit = 10 s
```

Se a captura coerente não terminar em 10 s:

- abortar o novo Backup antes de confirmação;
- liberar barrier;
- manter estado ativo intacto;
- registrar diagnóstico `BACKUP_CAPTURE_TIMEOUT`/equivalente.

Se datasets representativos excederem repetidamente o alvo/hard limit, isso é gate de escala/arquitetura, não permissão para congelar mutações por minutos.

## Restore antes da fase destrutiva

Para `pre_restore`/preparação:

```text
no_progress_timeout = 120 s
pre_destructive_total_limit = 10 min
```

Se o limite for atingido antes do primeiro rename:

- abortar Restore de forma segura;
- liberar barrier quando aplicável;
- manter `data/` ativo intocado;
- preservar diagnóstico/auditoria.

Depois de `DESTRUCTIVE_STARTED`, timeout genérico **não mata** a operação. Journal/recovery decide conclusão, rollback conhecido ou `uncertain`.

---

# 5. Host, comunicação e reconexão

## Readiness do Host

```text
readiness_timeout_per_launch = 30 s
restore_relaunch_attempts = 3 total
restore_relaunch_backoff = 1 s, 3 s
```

Após três tentativas sem readiness em relaunch de Restore:

- encerrar ciclo automático;
- preservar journal/evidências;
- exigir intervenção local/controlada;
- nunca transformar em watchdog permanente.

## Client ↔ Host

Baseline inicial:

```text
connect_timeout = 5 s
common_request_timeout = 30 s
websocket_reconnect = 1, 2, 4, 8, 15, 30 s (cap)
jitter = ±20%
backoff_reset_after_stable = 60 s
```

Regras:

- timeout de mutação nunca autoriza retry cego;
- operações longas têm contrato próprio/reconsulta de estado;
- reconectar WebSocket causa refetch/reconciliação;
- background reconnect não renova sessão por inatividade;
- números são defaults versionados/configurados pelo owner, não hardcode espalhado pela UI.

---

# 6. Rotação de logs

## Log técnico

Baseline:

```text
max_file_size = 20 MiB
retained_archives = 10
```

## Auditoria administrativa

Baseline:

```text
max_file_size = 50 MiB
retained_archives = 20
```

Regras:

- rotação preserva ordem/timestamps;
- nunca incluir senha/token/dump de conteúdo;
- em `uncertain/RECOVERY_REQUIRED`, cleanup destrutivo relacionado à operação permanece suspenso conforme D11;
- política corporativa de retenção superior pode exigir preservação maior sem alterar semântica do produto;
- journal de Restore/Recovery não é tratado como log rotativo comum.

---

# Propostas P12.56–P12.79

- **P12.56:** Argon2id usa baseline `m=65536 KiB`, `t=3`, `p=4`, salt aleatório de 16 bytes e output de 32 bytes em PHC; redução futura exige benchmark + decisão explícita;
- **P12.57:** senha single-factor exige 15–128 caracteres Unicode após NFKC, aceita espaços/passphrases, sem regra de composição ou rotação periódica e sem truncamento;
- **P12.58:** Host usa blocklist offline com pelo menos 10.000 senhas comuns/comprometidas, sem depender de Internet;
- **P12.59:** login usa throttling progressivo a partir da quinta falha e cooldown temporário de 15 min na décima, sem lockout permanente automático;
- **P12.60:** token de sessão possui 32 bytes aleatórios CSPRNG, é opaco, fica só em memória no Client e somente digest é persistido server-side;
- **P12.61:** sessão expira após 30 min de inatividade ou 8 h absolutas; heartbeat/background não renovam idle;
- **P12.62:** preset GERENCIA recebe capacidade de alterar configuração da empresa; Funcionário não;
- **P12.63:** identidade da empresa usa limites 120/160/200/254 para nome/contato/site/e-mail, sem truncamento silencioso;
- **P12.64:** logo aceita PNG/JPEG até 2 MiB e 2048×2048; Host valida, decodifica e reencoda para arquivo administrado seguro; SVG não entra no baseline;
- **P12.65:** categoria arquivada já herdada é preservada em nova revisão com aviso, pode ser removida, mas não pode ser adicionada/re-adicionada enquanto arquivada;
- **P12.66:** retenção default = 20 backups confirmados, configurável explicitamente na faixa 5–100;
- **P12.67:** parser/backup limita 10.000 entradas, 8 GiB totais, 16 MiB por arquivo administrado, path lógico 512 UTF-16, componente 120 e profundidade 8;
- **P12.68:** preflight exige temporários estimados + reserva final mínima de 1 GiB;
- **P12.69:** Backup normal tem alvo de pausa ≤2 s e hard limit de 10 s; exceder aborta o novo backup sem alterar estado ativo;
- **P12.70:** preparação `pre_restore` usa 120 s sem progresso e 10 min totais antes da fase destrutiva; depois de `DESTRUCTIVE_STARTED` não há kill por timeout genérico;
- **P12.71:** readiness por launch tem timeout de 30 s; relaunch de Restore é limitado a três tentativas totais com backoff 1 s/3 s;
- **P12.72:** Client usa connect timeout 5 s e request comum 30 s; mutação após timeout exige reconciliação antes de qualquer retry;
- **P12.73:** WebSocket reconecta em 1/2/4/8/15/30 s com jitter ±20% e reseta backoff após 60 s estáveis;
- **P12.74:** log técnico rotaciona em 20 MiB com 10 arquivos arquivados;
- **P12.75:** admin audit rotaciona em 50 MiB com 20 arquivos arquivados, respeitando proteção de recovery/uncertain e política corporativa superior;
- **P12.76:** valores numéricos ficam centralizados em owner/config apropriado e não espalhados como magic numbers por Client/Host/scripts;
- **P12.77:** nenhum default desta análise permite fallback silencioso para endpoint, senha, path ou valor inseguro quando configuração crítica estiver inválida;
- **P12.78:** parâmetros de segurança são versionáveis/rehasháveis, mas mudança abaixo do baseline exige decisão explícita; ajuste para cima pode ser proposto com benchmark;
- **P12.79:** estes parâmetros fecham o baseline inicial da Fase 2; gates corporativos/benchmarks ainda podem bloquear produção sem reabrir o produto por inferência.

## Evidência técnica consultada

- NIST SP 800-63B-4 (final, julho/2025): senha single-factor mínima de 15 caracteres, sem composition rules, sem rotação periódica e com throttling;
- RFC 9106: segunda configuração recomendada Argon2id = 64 MiB, 3 passes, 4 lanes, salt de 128 bits e tag de 256 bits;
- OWASP Password Storage Cheat Sheet: Argon2id e custo mínimo moderno;
- OWASP Session Management Cheat Sheet: token CSPRNG com alta entropia e timeouts idle/absolute; recomenda ao menos 128 bits para IDs próprios;
- NIST SP 800-63B-4: session secret aleatório e timeouts server-side; os números StepFlow são mais curtos que o máximo AAL1 por decisão de risco/usabilidade interna.

## Fora do escopo desta análise

- MFA/passkeys;
- TLS/PKI corporativa final;
- criar blocklist/arquivo de configuração agora;
- implementar Argon2/login/sessão;
- criar logo/asset real;
- criar código de Backup/Restore;
- alterar os gates de ambiente real;
- autorizar scaffold antes do gate final do Bloco 12.
