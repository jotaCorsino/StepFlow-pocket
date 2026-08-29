# StepFlow Host Pocket

**Status:** CONSOLIDADO PARA A FASE 1  
**Atualização:** 2026-08-29

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
9. garantir ausência de processo iniciado por ele após encerramento normal.

O Controller não instala serviço, altera PATH/registro, cria auto-start nem mantém watchdog residente.

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
- data dir está acessível;
- schema/migrations foram validados;
- SQLite abriu corretamente;
- listener está disponível.

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
- operações administrativas críticas quando aplicável.

Nunca registrar senha, token reutilizável ou conteúdo sensível desnecessário.

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

## Backup / Restore

Backup/Restore pertence ao Host e será fechado tecnicamente no Bloco 11.

Invariantes já aprovadas:

- Client não copia SQLite diretamente;
- backup precisa representar conjunto coerente de banco + arquivos administrados;
- Restore normal exige safety backup confirmado antes da etapa destrutiva;
- disaster recovery quando Host não inicia é fluxo local/controlado;
- operações administrativas críticas não podem ser executadas concorrentemente sem coordenação explícita.

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
- readiness verificável;
- shutdown gracioso;
- fechar Client individual não encerra ciclo central;
- nenhum processo residual após encerramento central;
- dados preservados ao substituir binários;
- multiusuário durante ciclo ativo;
- integração de Backup/Restore conforme contrato do Bloco 11.
