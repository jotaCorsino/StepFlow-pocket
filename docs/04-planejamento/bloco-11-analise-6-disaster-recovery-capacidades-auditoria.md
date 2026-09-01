# Bloco 11 — Análise 6 — Disaster recovery, capacidades e auditoria

**Status:** APROVADA PELO PO / CONSOLIDADA  
**Bloco:** 11 — Backup / Restauração técnico  
**Aprovação:** 2026-09-01

## Objetivo

Fechar a recuperação quando o Host normal não atinge readiness, as capacidades finais de Backup/Restore e a auditoria administrativa necessária para Backup, retenção, Restore e disaster recovery.

## Fronteira do disaster recovery

Disaster recovery é **excepcional, local e controlado**.

Elegível quando:

- SQLite ativo não abre/valida;
- estado persistente está corrompido/incompleto e impede readiness;
- Restore normal terminou em `RECOVERY_REQUIRED/uncertain`;
- estado ativo não permite criar o safety backup obrigatório do Restore normal.

Não é disaster recovery:

- preferência por pular safety backup;
- pacote incompatível;
- erro comum de permissão da sessão;
- Host saudável com Restore normal disponível;
- importação remota arbitrária.

Se o Host normal consegue readiness e Restore seguro, usa-se a Tela 13.

## Modo Recovery local

Baseline:

```text
operador na máquina central
→ encerrar/confirmar ausência de Host normal
→ iniciar Controller em modo Recovery
→ adquirir exclusividade da implantação
→ sem listener HTTP/WebSocket normal
→ enumerar backups administrados
→ escolher candidato
→ validar/preparar
→ confirmação local reforçada
→ preservar estado antigo quando possível
→ troca controlada de data/
→ validar
→ encerrar Recovery
→ iniciar fresh Host normal
→ readiness/login
```

Regras:

- sem Windows Service, Task Scheduler, watchdog, tray ou daemon;
- sem Client remoto;
- sem endpoint de disaster recovery na LAN;
- sem Host normal paralelo;
- reutiliza módulos comuns de envelope, compatibilidade, migrations, journal e ativação;
- detalhe futuro de CLI/janela local pode variar sem quebrar essa fronteira.

## Exclusividade, ACL e autoridade

Antes de recovery destrutivo:

- validar implantação/roots;
- adquirir o mesmo mecanismo de instância única;
- confirmar ausência de Host normal;
- verificar `backups/`, staging, journal e permissões;
- não seguir reparse points;
- reconciliar automaticamente journal conhecido antes de oferecer intervenção manual, quando possível.

Recovery não depende de login no `stepflow.sqlite`, porque o banco pode estar indisponível.

Autoridade baseline:

- execução local na máquina central;
- acesso autorizado pelo Windows/ACL à implantação;
- exclusividade StepFlow;
- confirmação reforçada local.

Não existe senha mestre paralela. Principal Windows pode ser registrado para auditoria/diagnóstico, sem virar usuário StepFlow por inferência.

Sem autoelevação: permissão insuficiente falha explicitamente.

## Origem do candidato

Baseline:

```text
backups/*.stepflow-backup
```

- somente pacotes finais administrados;
- `.staging` não é candidato;
- parser seguro e validação integral;
- `backup_id` do manifesto permanece identidade;
- sem upload/importação pela rede;
- sem path arbitrário fora do root administrado.

Backup externo em desastre real pode ser copiado operacionalmente para o root administrado fora do app; depois ainda passa por todas as validações, incluindo `source_deployment_id`.

## Validação/compatibilidade

Recovery não relaxa regras do Restore normal:

- envelope/paths/tamanhos/SHA-256;
- canonicalização Windows estrita;
- `source_deployment_id`;
- limites estruturais/preflight de espaço;
- `integrity_check = ok`;
- `foreign_key_check` vazio;
- `format_version` suportado;
- schema/migration path;
- schema antigo somente com migrations forward completas;
- schema novo = incompatível;
- sem down migration automática.

## Estado ativo danificado

Safety backup válido pode ser impossível. Nessa situação, antes da troca e quando o filesystem permitir:

```text
data/ atual
→ .recovery-old-<operation_id>/
```

Regras:

- preservação estrutural best-effort, não declaração de backup íntegro;
- não empacotar estado corrompido como `.stepflow-backup` válido;
- ausência de safety backup não bloqueia Recovery quando o estado já é inelegível ao Restore normal;
- se nem a preservação controlada nem a troca segura forem possíveis, exigir diagnóstico em vez de overwrite.

## Ativação e pós-Recovery

Recovery reutiliza staging/journal/troca same-volume/no-replace das Análises 4–5.

Depois de estado conhecido:

1. encerrar modo Recovery;
2. iniciar fresh Host normal;
3. processar journal terminal antes de readiness;
4. abrir schema/migrations normalmente;
5. reconstruir caches/projeções;
6. não reutilizar sessão anterior;
7. somente então disponibilizar login/uso.

## Capacidades

| Ação | ADM | Gerência | Funcionário |
|---|---:|---:|---:|
| Consultar backups | sim | sim | não |
| Criar backup manual | sim | sim | não |
| Executar Restore normal | sim | não | não |

Regras:

- Backup de Gerência não concede Restore;
- Restore permanece restrito a ADM;
- Gerência não concede Restore a si/outros;
- disaster recovery local não é capability de sessão.

## Auditoria em duas camadas

### Funcional no SQLite

Quando o banco estiver saudável, registrar ação associada ao usuário StepFlow conforme o mecanismo de auditoria do produto.

### Administrativa fora de `data/`

Baseline conceitual:

```text
logs/admin-audit.jsonl
```

- append-only pela aplicação no fluxo normal;
- fora de `data/`, portanto não volta no tempo em Restore;
- fora do pacote normal;
- protegido por ACL;
- não é anunciado como tamper-proof criptográfico;
- rotação física fica para política operacional/Bloco 12 sem destruir evidência necessária em `uncertain`.

## Eventos mínimos de auditoria

### Backup manual

- operation ID;
- ator StepFlow;
- ação;
- backup ID quando houver;
- início/fim;
- resultado;
- código sanitizado.

### Backup de sistema

- backup ID;
- `origin=system`;
- motivo (`pre_restore`, `pre_migration` etc.);
- operação pai;
- resultado.

### Retenção

- backup ID removido;
- motivo `retention`;
- resultado/warning.

### Restore

- operation ID;
- ator;
- source/safety backup IDs;
- schema útil;
- entrada em fase destrutiva;
- resultado terminal;
- restart/recovery associado.

### Disaster recovery

- operation ID;
- principal Windows quando disponível;
- source backup ID;
- razão de Recovery;
- preservação do estado antigo;
- resultado/warnings.

## Conteúdo proibido

Não registrar:

- senha;
- token/sessão reutilizável;
- conteúdo integral do backup;
- conteúdo documental/Atendimento desnecessário;
- password hash;
- segredo de configuração;
- dump SQL;
- path arbitrário do pacote.

## Falha de auditoria

Antes da fase destrutiva, impossibilidade de gravar journal/evidência mínima bloqueia Restore/Recovery.

Depois que alteração física já ocorreu, falha de auditoria:

- não reescreve artificialmente o resultado físico;
- gera warning/`AUDIT_WRITE_FAILED` quando possível;
- preserva artefatos necessários;
- expõe condição administrativa.

Backup já confirmado não é apagado/inválido apenas por falha posterior da auditoria.

## Journal × admin audit × log técnico

- **journal:** estado operacional para recovery determinístico;
- **admin audit:** evidência cronológica de ação/ator/resultado;
- **log técnico:** diagnóstico sanitizado.

Nenhum substitui o outro.

## Decisões aprovadas — D11.83 a D11.103

- **D11.83:** disaster recovery só quando Restore normal seguro não está disponível;
- **D11.84:** Recovery baseline é modo local/transitório do Controller sem listener normal de rede;
- **D11.85:** Recovery exige exclusividade e ausência de Host normal concorrente;
- **D11.86:** Recovery não depende de login no banco; autoridade vem de acesso local/ACL/exclusividade/confirmação;
- **D11.87:** Recovery não autoeleva; permissão insuficiente falha explicitamente;
- **D11.88:** candidatos baseline vêm de `backups/*.stepflow-backup`; staging/arquivo arbitrário externo não são importados pela aplicação;
- **D11.89:** Recovery usa mesma validação/compatibilidade/migrations forward do Restore normal;
- **D11.90:** ausência de safety backup não bloqueia disaster recovery real quando estado ativo já é inelegível ao fluxo normal;
- **D11.91:** estado ativo é preservado como `.recovery-old-<id>` quando possível, sem ser rotulado como backup íntegro;
- **D11.92:** Recovery reutiliza staging, journal, troca same-volume/no-replace e reconciliação determinística;
- **D11.93:** após Recovery conhecido, fresh Host normal deve atingir readiness antes do uso;
- **D11.94:** ADM/Gerência podem consultar/criar Backup; Funcionário não;
- **D11.95:** Restore normal permanece ADM-only e não decorre de Backup;
- **D11.96:** Gerência × Backup = SIM para consulta/criação, Restore = NÃO;
- **D11.97:** disaster recovery local não é capability de sessão;
- **D11.98:** operações críticas registram auditoria funcional quando possível + trilha administrativa fora de `data/`;
- **D11.99:** `logs/admin-audit.jsonl`/equivalente é append-only pela aplicação, protegido por ACL, sem alegação de tamper-proof;
- **D11.100:** auditar Backup manual/system, retenção, Restore e disaster recovery com IDs/ator/origem/timestamps/resultado/códigos sanitizados;
- **D11.101:** auditoria não registra senhas, tokens, conteúdo integral, dumps SQL ou conteúdo de negócio desnecessário;
- **D11.102:** falha de auditoria/journal antes da fase destrutiva bloqueia Restore/Recovery; falha posterior vira warning sem falsificar resultado físico;
- **D11.103:** journal, admin audit e log técnico são mecanismos distintos com lifecycles diferentes.

## Relação com validação final

D11.104–D11.116 complementam esta análise com safety barrier contínuo, canonicalização Windows, `source_deployment_id`, limites estruturais, baseline de criptografia/assinatura e gates finais. Não permanece pendência arquitetural desta análise.
