# StepFlow Host Pocket

**Status:** CONSOLIDADO PARA A FASE 1  
**Atualização:** 2026-08-31

## Tecnologia

O Host será implementado em **Rust**, usando:

- Tokio para runtime assíncrono;
- Axum para HTTP/WebSocket;
- `rusqlite` com SQLite bundled.

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
│   └── stepflow.sqlite
├── logs\
└── backups\
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
10. coordenar relaunch **bounded** quando o contrato de Restore exigir fresh Host para recovery/finalização.

O Controller não instala serviço, altera PATH/registro, cria auto-start nem mantém watchdog residente.

Relaunch de Restore é uma transição limitada da operação administrativa; não autoriza restart automático ilimitado para crashes normais.

## Propriedade do ciclo de vida

O ciclo central pertence ao **Controller na máquina central**, não a um Client individual.

- fechar um Client não encerra Host;
- vários Clients podem entrar e sair durante o mesmo ciclo;
- encerramento central solicita shutdown gracioso;
- Controller só termina normalmente após confirmar saída do Host;
- nenhum processo StepFlow permanece após encerramento completo.

Não está consolidado auto-shutdown por “último Client” ou inatividade. Não implementar por suposição.

Se houver Clients conectados no encerramento central, a UX deve apresentar manutenção/encerramento coordenado conforme os estados transversais vigentes; isso não altera a obrigação de integridade do shutdown.

## Responsabilidades do Host

Enquanto ativo:

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

O Host só fica pronto quando, no mínimo:

- configuração foi carregada;
- exclusividade da implantação foi obtida;
- recovery pendente de Restore foi reconciliado para estado conhecido;
- data dir está acessível;
- schema/migrations foram validados;
- SQLite abriu corretamente;
- listener está disponível.

Ordem consolidada quando existir `restore-active.json`/artefato relevante:

```text
resolver deployment + exclusividade
→ reconciliar Restore pendente
→ somente após estado conhecido seguir migrations/readiness normais
```

Enquanto `uncertain/RECOVERY_REQUIRED`:

- não anunciar readiness normal;
- não aceitar mutações de negócio;
- não iniciar nova operação destrutiva normal;
- não executar retenção/cleanup destrutivo;
- preservar source/safety backup, journal, old e staging relevantes.

Duas instâncias não podem coordenar o mesmo `data/`. Usar exclusão mútua local por implantação, além de verificação de readiness/processo.

## Configuração

`config/stepflow-host.toml` contém parâmetros operacionais. Endereço corporativo real não é hardcoded e alterar configuração não deve exigir recompilar.

Segredos reutilizáveis não devem ficar nesse arquivo em texto puro.

## Logs e diagnóstico

Logs técnicos em `logs/` com timestamp, nível, componente, mensagem e contexto sanitizado.

Registrar pelo menos:

- início do Controller/Host;
- configuração carregada ou inválida;
- data dir efetivo;
- listener/readiness;
- falha de startup;
- shutdown solicitado/concluído;
- erro fatal;
- operações administrativas críticas quando aplicável;
- entrada/resultado de reconciliação de Restore quando aplicável.

Nunca registrar senha, token reutilizável ou conteúdo sensível desnecessário.

A trilha administrativa específica que precisa atravessar Restore está em análise na Análise 6 do Bloco 11 e ainda não é contrato.

## Shutdown técnico

Fluxo normal:

1. Controller inicia encerramento central;
2. Host para de aceitar novas mutações;
3. conclui/aborta transacionalmente trabalho em andamento;
4. trata fila ainda não iniciada explicitamente;
5. coordena operações longas/administrativas conforme seus contratos;
6. encerra WebSockets;
7. fecha SQLite/conexões;
8. Host termina;
9. Controller confirma saída e termina.

`kill` forçado não é mecanismo normal.

### Restart controlado após Restore

Se Restore entrou na fase destrutiva e terminou com candidato aplicado ou rollback conhecido:

```text
estado físico escolhido validado
→ persistir RESTART_REQUIRED
→ encerrar listeners/WebSockets
→ fechar SQLite/handles
→ Host sai com motivo controlado
→ Controller relança um Host fresco de forma bounded
→ fresh Host reconcilia journal
→ somente então readiness normal
```

Uma tentativa de recovery que também falhe deve falhar fechado e exigir intervenção local/controlada; não virar watchdog geral.

## Backup / Restore

Backup/Restore pertence ao Host e está sendo fechado tecnicamente no Bloco 11.

Invariantes já aprovadas:

- Client não copia SQLite diretamente;
- backup representa conjunto coerente de banco + arquivos administrados;
- Restore normal revalida pacote e exige safety backup confirmado antes da etapa destrutiva;
- ativação usa troca lógica do `data/`, com staging/old controlados, não overwrite arquivo a arquivo;
- compatibilidade permite somente schema igual ou migration forward completa conhecida; sem down migration automática;
- Restore usa journal fora de `data/` e fresh Host para reconciliar/finalizar após fase destrutiva;
- qualquer Restore destrutivo invalida sessões anteriores;
- `uncertain` bloqueia readiness normal até recuperação controlada;
- disaster recovery quando Host não inicia é fluxo local/controlado; detalhes finais estão na Análise 6 em revisão;
- operações administrativas críticas não executam concorrentemente sem coordenação explícita.

Fontes:

- `../04-planejamento/bloco-11-backup-restauracao.md`;
- `../04-planejamento/bloco-11-analise-5-restart-sessoes-reconexao-falhas.md`.

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

Executar um `.exe` armazenado em SMB a partir da estação executa-o na estação; não cria processo remoto na máquina central.

Sem componente residente, o Controller precisa ser iniciado na máquina central ou por mecanismo corporativo já existente/aprovado. Não instalar silenciosamente serviço para contornar essa limitação.

## Critérios para implementação futura

- execução por pasta sem instalador/runtime global;
- sem privilégio administrativo no uso normal;
- instância única;
- readiness verificável e recovery anterior à readiness quando necessário;
- shutdown gracioso;
- fresh Host/relaunch de Restore bounded e não-watchdog;
- fechar Client individual não encerra ciclo central;
- nenhum processo residual após encerramento central;
- dados preservados ao substituir binários;
- multiusuário durante ciclo ativo;
- integração de Backup/Restore conforme contrato do Bloco 11.