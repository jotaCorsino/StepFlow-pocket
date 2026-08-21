# StepFlow Host Pocket

**Status:** CONSOLIDADO PARA A FASE 1  
**Atualização:** 2026-08-21

## Tecnologia

O Host será implementado em **Rust**, usando:

- Tokio para runtime assíncrono;
- Axum para HTTP/WebSocket;
- `rusqlite` com SQLite bundled.

A prova técnica descartável confirmou build release e execução isolada sem Node.js, npm, Rust, Cargo, Visual Studio ou SQLite Server no runtime.

## Requisito Pocket

Na máquina central o StepFlow é implantado por pasta pronta. Nenhum serviço StepFlow persistente é instalado/registrado como padrão.

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

1. resolver a raiz da implantação a partir da própria localização;
2. validar config/paths/permissões;
3. detectar instância central existente;
4. impedir segundo Host sobre os mesmos dados;
5. iniciar o Host como processo-filho;
6. aguardar readiness verificável;
7. registrar falhas de startup;
8. coordenar shutdown gracioso;
9. garantir que não reste processo iniciado por ele ao final normal.

O Controller não instala serviço, altera PATH/registro, cria auto-start nem mantém watchdog residente.

## Propriedade do ciclo de vida

O ciclo de vida central pertence ao **Controller na máquina central**, não a um Client individual.

Regras consolidadas:

- fechar um Client de técnico não encerra o Host central;
- vários Clients podem entrar e sair durante o mesmo ciclo operacional;
- encerrar o Controller/ciclo central solicita shutdown gracioso do Host;
- o Controller só termina normalmente depois de confirmar a saída do Host;
- nenhum processo StepFlow deve permanecer ativo após o encerramento completo do ciclo central.

**Não está consolidado** um auto-shutdown por “último Client desconectado” ou por timeout de inatividade. Não implementar esse comportamento por suposição. Se futuramente for desejado, deverá ser decisão explícita e configurável, sem criar processo residente adicional.

Quando houver Clients conectados no momento do encerramento central, a política de confirmação/aviso da UX será definida no Bloco 8; a integridade do shutdown continua obrigatória independentemente dessa UX.

## Host

Enquanto ativo, é responsável por:

- autenticação/autorização;
- API e WebSocket;
- SQLite e migrations;
- writer/fila/transações;
- revisão e conflitos;
- auditoria;
- backup/restore;
- arquivos persistentes administrados.

## Readiness e instância única

O Controller só considera o Host pronto quando, no mínimo:

- configuração foi carregada;
- data dir está acessível;
- schema/migrations foram validados;
- SQLite abriu corretamente;
- listener está disponível.

Duas instâncias não podem coordenar o mesmo `data/`. A implementação usará exclusão mútua local por implantação, além da verificação de readiness/processo.

## Configuração

`config/stepflow-host.toml` contém parâmetros operacionais, como bind/porta e opções necessárias. Endereço corporativo real não é hardcoded e alterar configuração não deve exigir recompilar.

Segredos reutilizáveis não devem ficar nesse arquivo em texto puro.

## Logs e diagnóstico

Logs técnicos ficam em `logs/` e devem registrar timestamp, nível, componente, mensagem e contexto útil sanitizado.

Eventos mínimos:

- início do Controller/Host;
- configuração carregada ou inválida;
- data dir efetivo;
- listener/readiness;
- falha de startup;
- shutdown solicitado/concluído;
- erro fatal.

Não registrar senha, token reutilizável ou conteúdo sensível desnecessário.

## Shutdown técnico

Fluxo normal:

1. Controller inicia o encerramento central;
2. Host para de aceitar novas mutações;
3. conclui/aborta transacionalmente trabalho em andamento;
4. trata fila ainda não iniciada explicitamente;
5. encerra WebSockets;
6. fecha SQLite/conexões;
7. Host termina;
8. Controller confirma a saída e termina.

`kill` forçado não é mecanismo normal.

## Atualização e rollback

```text
encerrar Controller/Host
→ confirmar ausência de processos
→ backup quando exigido pela migration
→ substituir/ativar app/
→ preservar config/data/logs/backups
→ iniciar Controller
→ validar readiness
```

Rollback de binário é permitido somente quando o schema atual for compatível. Caso contrário, exige restauração do backup correspondente; não usar down migration destrutiva por conveniência.

## Limitação deliberada

Sem componente residente, um Client remoto não cria sozinho um processo novo na máquina central apenas executando um `.exe` armazenado em SMB. O Controller precisa ser iniciado na máquina central ou por mecanismo corporativo já existente/aprovado.

Essa limitação é consequência aceita do requisito de zero serviço/agente StepFlow residente.

## Critérios para implementação futura

- execução por pasta sem instalador/runtime global;
- sem privilégio administrativo no uso normal, salvo necessidade real do ambiente;
- instância única;
- readiness verificável;
- shutdown gracioso;
- fechar Client individual não encerra o ciclo central;
- nenhum processo residual após encerramento do Controller/ciclo central;
- dados preservados ao substituir binários;
- multiusuário durante o ciclo ativo.
