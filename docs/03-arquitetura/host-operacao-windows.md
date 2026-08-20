# StepFlow Host — Modelo Operacional no Windows

**Data:** 2026-08-20  
**Status:** DIREÇÃO RECOMENDADA / SUJEITA A UMA PROVA OPERACIONAL

## 1. Contexto

O Gate A do Bloco 2 validou que o Host em Rust pode executar de forma Pocket a partir de pasta própria, expondo HTTP e usando SQLite local sem depender de Node, npm, Rust, Cargo ou Visual Studio em runtime.

Resta fechar como o Host ficará continuamente disponível na máquina central.

## 2. Alternativas avaliadas

### Processo normal iniciado manualmente

Não é suficiente como solução principal porque depende de ação humana após reinício e pode deixar o StepFlow indisponível sem diagnóstico claro.

### Inicialização por logon

Não é adequada ao Host central porque depende de sessão de usuário.

### Task Scheduler com Boot Trigger

Tecnicamente inicia no boot e não depende de logon, mas continua sendo uma tarefa agendada usada para manter um processo permanente. Exige registro administrativo e oferece um modelo de ciclo de vida menos natural para um daemon HTTP de longa duração.

### Windows Service

É o modelo nativo do Windows para processos de longa duração que precisam permanecer disponíveis independentemente de logon. O Service Control Manager oferece start/stop/status e início automático ou delayed-auto.

## 3. Direção recomendada

Adotar **um único Windows Service do StepFlow Host** como exceção mínima e explicitamente justificada ao princípio de copy-deploy.

A instalação do serviço deve ocorrer uma única vez por implantação e apontar para o executável mantido na pasta fixa do StepFlow.

O serviço não autoriza:

- instalar runtimes globais;
- alterar PATH;
- instalar banco externo;
- habilitar features amplas do Windows;
- exigir reboot;
- espalhar binários fora da pasta do produto.

## 4. Por que a exceção é aceitável

O Host precisa iniciar sem usuário logado e permanecer ativo para autenticação, API, SQLite, concorrência e eventos.

Um serviço Windows introduz somente um registro controlado no Service Control Manager e pode ser instalado/removido de forma explícita. Isso é mais previsível do que depender de login ou usar Task Scheduler para simular um daemon.

## 5. Modelo conceitual da pasta

```text
StepFlow\
├── app\
│   └── StepFlowHost.exe
├── config\
│   └── stepflow-host.toml
├── data\
│   └── stepflow.sqlite
├── logs\
├── backups\
└── tools\
```

A localização física real permanece aberta até o ambiente corporativo ser conhecido.

## 6. Regras operacionais propostas

- binário substituível separadamente de `config`, `data`, `logs` e `backups`;
- serviço aponta para caminho estável dentro da pasta StepFlow;
- bind/porta vêm de configuração local, não de hardcode de ambiente;
- logs operacionais vão para `logs`, sem depender do terminal;
- shutdown deve responder corretamente ao stop do Service Control Manager;
- atualização do binário exige parar o serviço, substituir/ativar a versão e reiniciar;
- rollback deve permitir restaurar a versão anterior sem tocar no banco, salvo migration incompatível explicitamente tratada;
- instalação/remoção do serviço será uma operação administrativa rara, não parte do uso diário.

## 7. Próxima prova necessária

Antes de consolidar definitivamente o Windows Service:

1. adaptar uma PoC descartável Rust para responder ao Service Control Manager;
2. registrar o serviço apontando para pasta descartável;
3. iniciar sem usuário interativo;
4. validar `/health` e SQLite;
5. executar `stop` e confirmar shutdown limpo;
6. reiniciar o serviço;
7. remover o registro do serviço sem deixar dependências;
8. confirmar que os dados na pasta persistente permanecem preservados.

A prova deve ser feita no PC de desenvolvimento, não no servidor corporativo, e não valida ainda políticas reais da empresa.
