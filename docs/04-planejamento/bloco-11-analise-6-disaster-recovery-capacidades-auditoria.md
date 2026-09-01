# Bloco 11 — Análise 6 — Disaster recovery, capacidades e auditoria

**Status:** APROVADA PELO PO  
**Bloco:** 11 — Backup / Restauração técnico  
**Data:** 2026-08-31  
**Aprovação:** 2026-09-01

## Objetivo

Fechar a fronteira técnica de recuperação quando o Host normal não consegue atingir readiness, decidir a pendência de capacidades de Backup/Restore e especificar auditoria administrativa suficiente para Backup, Restore, retenção e disaster recovery.

Esta análise parte das Análises 1–5 aprovadas. Não reabre a UX da Tela 13 e não enfraquece o contrato Pocket.

---

## 6.1 Fronteira do disaster recovery

Disaster recovery é um fluxo **excepcional, local e controlado**. Ele existe quando o Restore normal pela UI não pode ser usado com segurança porque o Host não consegue iniciar/readiness sobre o estado ativo ou porque a recuperação normal terminou em `RECOVERY_REQUIRED/uncertain`.

Exemplos elegíveis:

- `data/stepflow.sqlite` não abre ou falha validação de startup;
- estado persistente ativo está corrompido/incompleto e impede readiness;
- journal de Restore indica resultado `uncertain` que o Host normal não pode resolver automaticamente;
- estado ativo não permite criar o safety backup obrigatório do Restore normal.

Não é disaster recovery:

- preferência do operador por “pular” o safety backup;
- backup incompatível com a versão/schema atual;
- erro comum de permissão do usuário;
- Host saudável com Restore normal disponível;
- tentativa de importar pacote arbitrário pela rede.

Se o Host normal consegue atingir readiness e executar Restore seguro, deve-se usar o fluxo normal da Tela 13.

## 6.2 Superfície operacional

Baseline aprovada: **modo de recuperação explícito do StepFlowController na máquina central**, reutilizando módulos de validação/restauração do Host sem iniciar a API normal.

Conceito:

```text
operador na máquina central
→ encerra ciclo StepFlow / confirma ausência de Host
→ inicia StepFlowController em modo Recovery
→ Controller adquire exclusividade da implantação
→ Recovery não abre listener HTTP/WebSocket normal
→ enumera backups administrados
→ operador escolhe candidato
→ valida e prepara recuperação
→ confirmação local reforçada
→ troca controlada de data/
→ valida estado recuperado
→ encerra Recovery
→ inicia ciclo normal
```

Regras:

- não criar Windows Service, Task Scheduler, watchdog, tray ou daemon;
- não depender de Client remoto;
- não expor endpoint de disaster recovery na LAN;
- não iniciar Host normal em paralelo;
- reutilizar biblioteca/módulo comum de envelope, compatibilidade, migrations, journal e ativação; não duplicar algoritmo crítico em executáveis independentes;
- o detalhe de CLI/janela local fica para a fase executável, desde que continue transitório e local.

## 6.3 Exclusividade e preflight local

Antes de qualquer recovery destrutivo:

- confirmar implantação/paths esperados;
- obter o mesmo mecanismo de instância única sobre os dados;
- confirmar ausência de Host normal usando a implantação;
- validar que `backups/`, staging e local do journal pertencem à raiz administrada;
- verificar espaço/permissões necessárias;
- não seguir reparse points/symlinks/junctions;
- bloquear execução se outra operação/journal ativo puder ser reconciliado automaticamente primeiro.

Recovery não força acesso sobre uma instância ativa.

## 6.4 Autoridade quando o banco não funciona

Disaster recovery não pode depender de autenticação no `stepflow.sqlite`, porque o próprio banco pode estar indisponível.

A fronteira inicial de autoridade é:

- execução **local** na máquina central;
- acesso já autorizado à pasta/arquivos da implantação pelo Windows;
- exclusividade da instância StepFlow;
- confirmação reforçada local antes da alteração destrutiva.

O StepFlow não tenta inventar uma segunda base de usuários ou senha mestre paralela apenas para Recovery.

Quando disponível, registrar identidade do principal Windows do processo para auditoria/diagnóstico. Essa identidade não é convertida em usuário StepFlow por inferência.

Recovery não exige elevação administrativa como requisito do produto se as ACLs da implantação já concederem os direitos necessários. Falha de permissão deve ser corrigida pela administração da máquina/ACL, não por autoelevação silenciosa.

## 6.5 Origem do backup no Recovery

O Recovery normaliza a mesma fronteira física aprovada para o catálogo:

```text
backups/*.stepflow-backup
```

Baseline:

- listar somente pacotes finais administrados da implantação;
- `.staging` nunca é candidato;
- pacote precisa passar pelo mesmo parser seguro;
- `backup_id` do manifesto continua sendo identidade;
- não há upload/importação pela rede;
- não apontar para arquivo arbitrário fora do root administrado no baseline.

Se um backup externo precisar ser utilizado em desastre real, a cópia para o diretório administrado é procedimento operacional local fora do app e o pacote ainda passa pela validação integral. Isso não vira feature de upload/importação do Client.

## 6.6 Validação e compatibilidade no Recovery

Disaster recovery **não relaxa** integridade ou compatibilidade.

Antes da troca:

- validar envelope, paths, tamanhos e SHA-256 integralmente;
- extrair para staging same-volume controlado;
- executar `PRAGMA integrity_check = ok`;
- executar `PRAGMA foreign_key_check` e exigir zero violações;
- validar `format_version`;
- validar schema/migration path;
- schema mais antigo só avança com cadeia completa de migrations forward;
- schema mais novo que o runtime Recovery permanece incompatível;
- nenhuma down migration automática.

Se o runtime atual não entende o backup, Recovery não improvisa interpretação.

## 6.7 Tratamento do estado ativo danificado

No disaster recovery, safety backup válido pode ser impossível. Isso é precisamente uma das diferenças para o Restore normal.

Antes de substituir `data/`, quando o filesystem permitir:

```text
data/ atual
→ preservar como .recovery-old-<operation_id>/
→ nunca sobrescrever silently
```

Regras:

- essa preservação é **best-effort estrutural**, não é declarada backup íntegro;
- não empacotar/rotular estado corrompido como `.stepflow-backup` válido;
- falha em produzir safety backup não bloqueia disaster recovery se o estado ativo já é inelegível ao Restore normal;
- porém, se não for possível nem preservar/mover o estado antigo de forma controlada e o recovery não puder manter rollback operacional, a operação deve exigir diagnóstico adicional em vez de apagar por overwrite;
- recovery nunca copia candidato por cima de arquivos ativos um a um.

## 6.8 Ativação e journal

Recovery reutiliza a mesma estratégia de conjunto lógico e journal das Análises 4–5:

```text
candidato validado
→ journal Recovery persistido fora de data/
→ data atual → .recovery-old-<id>
→ data-next → data
→ validar data ativo
→ fresh startup normal
```

- troca same-volume;
- no-replace;
- nenhuma conclusão entre os renames;
- journal + realidade do filesystem definem recovery após crash;
- resultado ambíguo permanece `RECOVERY_REQUIRED/uncertain`;
- cleanup destrutivo fica suspenso enquanto o estado não for comprovado.

O formato físico pode reutilizar a família `backups/.operations/`, distinguindo `operation_type = recovery` de `restore`.

## 6.9 Pós-recovery

Depois de ativar e validar o candidato:

1. encerrar o modo Recovery;
2. iniciar um fresh Host normal;
3. processar qualquer journal terminal antes de readiness;
4. abrir schema/migrations de forma normal;
5. reconstruir caches/projeções;
6. não reutilizar sessão StepFlow anterior;
7. somente então disponibilizar login e uso normal.

O Recovery não mantém Host/API normal rodando em paralelo após concluir.

## 6.10 Capacidades — separação granular

O contrato de autorização deve distinguir pelo menos:

- consultar catálogo/detalhes de backups;
- criar Backup manual;
- executar Restore normal.

Restore nunca é consequência automática da permissão de Backup.

Baseline de presets aprovada:

| Ação | ADM | Gerência | Funcionário |
|---|---:|---:|---:|
| Consultar backups | sim | sim | não |
| Criar backup manual | sim | sim | não |
| Executar Restore normal | sim | não | não |

Isso fecha a pendência histórica **Gerência × Backup = sim** para consulta/criação, mantendo **Restore = ADM somente**.

Motivos:

- Backup é operação protetiva e não substitui o estado ativo;
- Gerência já possui responsabilidades administrativas/operacionais amplas;
- Restore é destrutivo e mantém fricção/autoridade superior;
- conceder Backup a Gerência não amplia sua capacidade para Restore.

## 6.11 Delegação e teto

Na primeira versão:

- capacidade de Restore permanece restrita a contas ADM;
- Gerência não pode conceder Restore a si ou a outro usuário;
- Funcionário não recebe Backup/Restore por preset;
- customização futura continua sujeita ao teto de delegação Host-side;
- ocultação Client-side é somente UX.

Disaster recovery local não é uma capacidade de sessão porque pode ocorrer sem banco/sessão. Ele é procedimento de operação central protegido pela fronteira local/ACL/exclusividade descrita acima.

## 6.12 Auditoria em duas camadas

Backup/Restore precisam de evidência administrativa que sobreviva a um Restore do próprio banco.

Contrato:

### Auditoria funcional no SQLite

Quando o banco ativo estiver saudável, registrar no mecanismo de auditoria do produto ações associadas ao usuário StepFlow, conforme modelo conceitual vigente.

### Trilha administrativa fora de `data/`

Operações críticas também emitem registros estruturados em storage de logs administrado e **não restaurável**, por exemplo:

```text
logs/admin-audit.jsonl
```

Características:

- append-only pela aplicação no fluxo normal;
- fora de `data/`, portanto não volta no tempo quando um Restore substitui o banco;
- não é parte do pacote de Backup normal;
- protegido pelas ACLs da implantação;
- não é anunciado como log criptograficamente imutável/tamper-proof;
- rotação/retention física de logs será operacional, sem apagar evidência necessária durante operação `uncertain`.

## 6.13 Eventos mínimos de auditoria administrativa

Registrar de forma proporcional pelo menos:

### Backup manual

- `operation_id`;
- ator StepFlow;
- ação solicitada;
- `backup_id` produzido quando houver;
- horário de início/fim;
- resultado real;
- código de falha/warning sanitizado quando aplicável.

### Backup de sistema

- `backup_id`;
- `origin = system`;
- motivo (`pre_restore`, `pre_migration` ou equivalente);
- operação pai;
- resultado.

### Retenção

- backup removido por `backup_id`;
- motivo `retention`;
- resultado da remoção;
- warning se o limite não puder ser cumprido.

### Restore normal

- `operation_id`;
- ator StepFlow;
- source `backup_id`;
- safety `backup_id`;
- schema de origem/final quando útil;
- entrada em fase destrutiva;
- resultado terminal `completed`, `rolled_back`, `failed_pre_destructive` ou `uncertain`;
- restart/recovery associado quando houver.

### Disaster recovery

- `operation_id`;
- principal Windows local quando disponível;
- source `backup_id`;
- razão de entrada em Recovery;
- preservação de estado antigo quando possível;
- resultado terminal;
- warnings/falhas de validação.

## 6.14 Conteúdo proibido na auditoria

Não registrar por conveniência:

- senha;
- token/sessão reutilizável;
- conteúdo integral de backup;
- conteúdo de documentos/Atendimentos;
- hashes de senha;
- segredo de configuração;
- dump SQL;
- stack trace bruto para o Client;
- path arbitrário fornecido por pacote.

Paths técnicos locais só entram em log técnico sanitizado quando necessários ao diagnóstico; a auditoria administrativa prefere IDs lógicos.

## 6.15 Falha da auditoria

A impossibilidade de gravar evidência **antes de uma fase destrutiva** é tratada como falha de preflight para Restore/Recovery normal, pois a operação crítica precisa deixar trilha mínima e também depende de storage operacional gravável para journal.

Depois que uma alteração física já ocorreu, falha ao escrever um registro de auditoria não muda a realidade do dado:

- não declarar Restore fisicamente revertido apenas porque o log falhou;
- registrar `AUDIT_WRITE_FAILED`/warning onde ainda for possível;
- preservar journal/artefatos quando necessário;
- expor condição administrativa para correção.

Para Backup já confirmado, falha posterior de auditoria não apaga nem invalida fisicamente o pacote válido; o resultado técnico do backup e o warning de auditoria são reportados separadamente.

## 6.16 Relação entre journal, logs e auditoria

São mecanismos distintos:

- **journal de Restore/Recovery**: estado operacional mínimo para permitir reconciliação determinística após crash;
- **admin audit**: evidência cronológica da ação/ator/resultado;
- **logs técnicos**: diagnóstico detalhado sanitizado.

Nenhum substitui o outro.

O journal pode ser removido após estado conhecido conforme Análise 5. A auditoria administrativa permanece conforme política de logs; logs técnicos podem ter rotação própria.

## 6.17 Exposição ao Client

A UX normal continua simples:

- Gerência/ADM autorizados veem catálogo e criação de Backup;
- somente ADM vê/aciona Restore;
- disaster recovery não aparece como botão remoto;
- detalhes de principal Windows, journal, paths, ACL e recovery interno não aparecem na Tela 13;
- resultado normal da operação continua vindo do Host por estado confirmado.

Nenhuma nova tela de produção é necessária para esta análise.

## 6.18 Decisões aprovadas — D11.83 a D11.103

- **D11.83:** disaster recovery é excepcional e somente quando Restore normal seguro não está disponível por falha/readiness/corrupção/`uncertain`;
- **D11.84:** baseline de Recovery é modo local explícito do Controller na máquina central, transitório e sem listener normal de rede;
- **D11.85:** Recovery exige exclusividade da implantação e ausência de Host normal concorrente;
- **D11.86:** Recovery não depende de login no banco; autoridade vem de acesso local controlado + ACLs da implantação + confirmação reforçada;
- **D11.87:** Recovery não exige elevação automática; permissões insuficientes falham explicitamente;
- **D11.88:** candidatos baseline vêm de `backups/*.stepflow-backup`; `.staging` e arquivo arbitrário externo não são importados pela aplicação;
- **D11.89:** Recovery usa a mesma validação integral e a mesma regra de compatibilidade/migrations forward do Restore normal, sem down migration;
- **D11.90:** ausência de safety backup válido não bloqueia disaster recovery quando o estado ativo já é inelegível ao fluxo normal;
- **D11.91:** estado ativo é preservado como `.recovery-old-<id>` quando possível, mas nunca rotulado como backup íntegro sem validação;
- **D11.92:** Recovery reutiliza staging, journal, troca same-volume/no-replace e reconciliação determinística das Análises 4–5;
- **D11.93:** após Recovery conhecido, um fresh Host normal deve atingir readiness antes do retorno ao uso;
- **D11.94:** presets iniciais: ADM e Gerência podem consultar/criar Backup; Funcionário não;
- **D11.95:** Restore normal permanece ADM-only e não é concedido por consequência de Backup;
- **D11.96:** Gerência × Backup é fechado como **SIM** para consulta/criação, mantendo Restore = **NÃO**;
- **D11.97:** disaster recovery local não é capability de sessão; é procedimento central protegido por acesso local/ACL/exclusividade;
- **D11.98:** operações críticas registram auditoria funcional quando possível e trilha administrativa estruturada fora de `data/` para sobreviver a Restore;
- **D11.99:** `logs/admin-audit.jsonl`/equivalente é append-only pela aplicação no baseline, protegido por ACL, sem alegação de tamper-proof criptográfico;
- **D11.100:** auditar Backup manual/system, retenção, Restore e disaster recovery com IDs, ator/origem, timestamps, resultado e códigos sanitizados;
- **D11.101:** auditoria não registra senhas, tokens, conteúdo integral do backup, dumps SQL ou conteúdo de negócio desnecessário;
- **D11.102:** falha de auditoria/journal antes da fase destrutiva bloqueia Restore/Recovery; falha depois de alteração física não reescreve artificialmente o resultado real e vira warning/condição administrativa;
- **D11.103:** journal operacional, admin audit e log técnico são mecanismos distintos com lifecycles diferentes.

## Pendências para a validação final

Depois desta análise resta verificar de forma cruzada:

- coerência D11.1–D11.103;
- ausência de escolha crítica não resolvida;
- impactos documentais em autenticação, Host, comunicação, Tela 13 e modelo de dados;
- parâmetros numéricos deliberadamente reservados ao Bloco 12;
- gates reais de Windows/filesystem/ACL/EDR;
- nenhuma quebra do contrato Pocket.

## Próximo passo

Seguir para **Análise 7 — validação técnica final do Bloco 11**.