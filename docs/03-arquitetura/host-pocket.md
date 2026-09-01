# StepFlow Host Pocket

**Status:** CONSOLIDADO PARA A FASE 1  
**Atualização:** 2026-09-01

## Tecnologia

O Host será implementado em **Rust**, usando Tokio, Axum e `rusqlite` com SQLite bundled.

A prova técnica descartável da Fase 1 confirmou build release e execução isolada sem toolchain no runtime.

## Requisito Pocket

Na máquina central, o StepFlow é implantado por pasta pronta. Nenhum serviço StepFlow persistente é instalado como baseline.

```text
StepFlow fechado
→ nenhum processo StepFlow

Controller iniciado sob demanda
→ Host iniciado como processo-filho
→ Clients utilizam o Host
→ encerramento do ciclo central
→ shutdown controlado
→ Host termina
→ Controller termina
→ nenhum processo residual
```

## Estrutura conceitual

```text
StepFlow\
├── app\
│   ├── StepFlowController.exe
│   └── StepFlowHost.exe
├── config\
│   └── stepflow-host.toml
├── data\
│   ├── stepflow.sqlite
│   ├── company\
│   └── avatars\
├── logs\
│   └── admin-audit.jsonl      # equivalente permitido
└── backups\
    └── .operations\          # journal operacional, não catálogo
```

Binários de `app/` são substituíveis sem sobrescrever `config/`, `data/`, `logs/` ou `backups/`.

## Controller

Responsabilidades:

1. resolver a raiz da implantação;
2. validar config/paths/permissões;
3. detectar instância central existente;
4. impedir segundo Host sobre os mesmos dados;
5. iniciar o Host como processo-filho;
6. aguardar readiness verificável;
7. registrar falhas de startup;
8. coordenar shutdown gracioso;
9. garantir ausência de processo iniciado por ele após encerramento normal;
10. coordenar relaunch **bounded** quando Restore exigir fresh Host;
11. oferecer modo Recovery local/transitório quando o Host normal não consegue readiness e disaster recovery for aplicável.

O Controller não instala serviço, altera PATH/registro, cria auto-start nem mantém watchdog residente.

### Relaunch de Restore

É uma transição limitada da operação administrativa. Não autoriza restart automático ilimitado para crashes normais.

### Modo Recovery

- somente na máquina central;
- sem listener HTTP/WebSocket normal;
- exclusividade da implantação e ausência de Host normal concorrente;
- acesso local/ACLs em vez de depender de login no banco indisponível;
- sem autoelevação silenciosa;
- reutiliza módulos comuns de validação, compatibilidade, staging, journal e ativação.

## Propriedade do ciclo de vida

O ciclo central pertence ao Controller na máquina central, não a um Client individual.

- fechar Client não encerra Host;
- vários Clients podem entrar/sair durante o mesmo ciclo;
- encerramento central solicita shutdown gracioso;
- Controller só termina normalmente após confirmar saída do Host;
- nenhum processo StepFlow permanece após encerramento completo.

Não está consolidado auto-shutdown por “último Client” ou inatividade.

## Responsabilidades do Host

- autenticação/autorização;
- API HTTP/JSON + WebSocket;
- SQLite e migrations;
- writer/fila/transações;
- revisão e conflitos;
- Procedimentos, Atendimentos e Equipamentos;
- checklist e observações de serviço;
- auditoria;
- Backup/Restore;
- geração documental.

## Readiness e instância única

O Host só fica ready quando, no mínimo:

- configuração carregada;
- exclusividade da implantação obtida;
- recovery pendente reconciliado para estado conhecido;
- `data/` acessível;
- schema/migrations válidos;
- SQLite aberto;
- listener disponível.

Ordem quando houver journal/artefato de Restore:

```text
resolver deployment + exclusividade
→ reconciliar Restore/Recovery pendente
→ somente após estado conhecido seguir migrations/readiness normais
```

Enquanto `uncertain/RECOVERY_REQUIRED`:

- sem readiness normal;
- sem mutações de negócio;
- sem nova operação destrutiva normal;
- sem retenção/cleanup destrutivo;
- preservar source/safety backup, journal, old e staging relevantes.

Duas instâncias não podem coordenar o mesmo `data/`.

## Configuração

`config/stepflow-host.toml` contém parâmetros operacionais. Endereço corporativo real não é hardcoded e alterar configuração não deve exigir recompilar.

Segredos reutilizáveis não ficam nesse arquivo em texto puro.

## Logs e diagnóstico

Logs técnicos em `logs/` com timestamp, nível, componente, mensagem e contexto sanitizado.

Registrar pelo menos:

- início do Controller/Host;
- configuração carregada/inválida;
- data dir efetivo;
- listener/readiness;
- falha de startup;
- shutdown solicitado/concluído;
- erro fatal;
- operações administrativas críticas;
- entrada/resultado de reconciliação Restore/Recovery.

Nunca registrar senha, token reutilizável ou conteúdo sensível desnecessário.

### Auditoria administrativa

Backup/Restore/retention/Recovery também emitem trilha estruturada fora de `data/`, por exemplo `logs/admin-audit.jsonl`.

- append-only pela aplicação no fluxo normal;
- protegida pelas ACLs;
- não restaurada junto com `data/`;
- não é anunciada como tamper-proof criptográfica;
- usa IDs lógicos, ator/origem, timestamps, resultado e códigos sanitizados;
- sem senha, token, dump SQL ou conteúdo integral do backup.

Journal operacional, admin audit e log técnico têm funções/lifecycles distintos.

## Shutdown técnico

1. Controller inicia encerramento;
2. Host para novas mutações;
3. conclui/aborta transacionalmente trabalho em andamento;
4. trata fila ainda não iniciada;
5. coordena operações longas/administrativas;
6. encerra WebSockets;
7. fecha SQLite/handles;
8. Host termina;
9. Controller confirma saída e termina.

`kill` forçado não é mecanismo normal.

## Restart controlado após Restore

Se Restore entrou na fase destrutiva e terminou com candidato aplicado ou rollback conhecido:

```text
estado físico escolhido validado
→ persistir RESTART_REQUIRED
→ encerrar listeners/WebSockets
→ fechar SQLite/handles
→ Host sai com motivo controlado
→ Controller relança Host fresco de forma bounded
→ fresh Host reconcilia journal
→ somente então readiness normal
```

Falha da tentativa de recovery fecha o ciclo automático e exige intervenção local/controlada; não vira watchdog geral.

## Backup / Restore — D11.1–D11.116

Backup/Restore pertence ao Host e está tecnicamente consolidado no Bloco 11.

### Invariantes

- Client nunca copia SQLite diretamente;
- backup representa SQLite + arquivos administrados como conjunto coerente;
- pacote final = `.stepflow-backup`, ZIP `Stored`, manifesto versionado e SHA-256;
- SQLite usa Online Backup API;
- Backup normal usa barrier curto de mutações para captura;
- promoção é same-volume/no-replace;
- Restore revalida pacote, provenance, paths, hashes e banco;
- schema antigo somente com migrations forward completas no staging;
- sem down migration automática;
- safety backup confirmado é obrigatório no Restore normal;
- ativação troca logicamente `data/`, preservando `old`;
- Restore usa journal fora de `data/` e fresh Host após fase destrutiva;
- Restore destrutivo invalida sessões anteriores;
- `uncertain` bloqueia readiness até recuperação controlada;
- disaster recovery é local/transitório pelo Controller;
- operações administrativas críticas não executam concorrentemente sem coordenação explícita.

### Safety barrier final

Para `pre_restore`:

```text
suspender/drain mutações
→ capturar safety backup
→ manter barrier
→ finalizar/verificar/promover safety backup
→ revalidar data-next
→ DESTRUCTIVE_STARTED
→ primeiro rename
```

Depois da captura:

- nenhuma mutação em SQLite/company/avatars;
- journal/admin-audit externos continuam permitidos;
- falha/cancelamento antes do primeiro rename libera o barrier sem alterar `data/`.

### Revalidação do candidato

Antes de `DESTRUCTIVE_STARTED`, confirmar digest, schema e root/volume de `data-next/`. Mudança aborta antes do primeiro rename e exige nova validação.

### Paths e provenance

- canonicalização Windows estrita;
- rejeitar drive/UNC/device/ADS/reserved names/trailing dot-space/case collision/reparse/non-regular/escape do root;
- criação de Backup aplica a mesma disciplina aos arquivos administrados;
- `manifest.json` inclui `source_deployment_id`;
- Restore/Recovery baseline bloqueiam deployment diferente com `source_mismatch`.

### Limites e confiança

- parser/extração bounded por entradas/tamanho/path/bytes e preflight de espaço;
- números finais ficam no Bloco 12;
- baseline sem criptografia application-level;
- baseline sem assinatura criptográfica application-level;
- SHA-256 é integridade, não autenticidade;
- trust boundary = root administrado + ACLs + infraestrutura de volume + deployment ID + auditoria.

### Limite operacional

Backup local não promete proteção contra perda física total, ransomware com acesso ao mesmo storage ou site loss. Offsite/cópia corporativa de `backups/` é responsabilidade operacional externa.

### Gates antes de produção

- adapter Win32 de rename/promoção/journal;
- filesystem real;
- ACLs;
- EDR/antivírus;
- long paths;
- espaço/performance;
- crash/restart injection.

Fonte principal: `../04-planejamento/bloco-11-backup-restauracao.md`.

## Atualização e rollback

```text
encerrar Controller/Host
→ confirmar ausência de processos
→ backup quando exigido
→ substituir/ativar app/
→ preservar config/data/logs/backups
→ iniciar Controller
→ validar readiness
```

Rollback de binário só é permitido quando schema atual for compatível; caso contrário, exige Restore correspondente. Não usar down migration destrutiva por conveniência.

## Limitação deliberada

Executar `.exe` armazenado em SMB a partir da estação executa-o na estação; não cria processo remoto na máquina central.

Sem componente residente, o Controller precisa ser iniciado na máquina central ou por mecanismo corporativo já existente/aprovado. Não instalar serviço para contornar essa limitação.

## Critérios para implementação futura

- execução por pasta sem instalador/runtime global;
- sem privilégio administrativo no uso normal;
- instância única;
- readiness verificável e recovery anterior à readiness quando necessário;
- shutdown gracioso;
- fresh Host/relaunch de Restore bounded e não-watchdog;
- Recovery local/transitório sem listener normal;
- fechar Client individual não encerra ciclo central;
- nenhum processo residual após encerramento central;
- dados preservados ao substituir binários;
- multiusuário durante ciclo ativo;
- implementação conforme D11.1–D11.116 e gates corporativos correspondentes.
