# StepFlow Host — Modelo Operacional no Windows

**Data:** 2026-08-20  
**Status:** DIREÇÃO ARQUITETURAL CORRIGIDA E APROVADA PELO PO

## 1. Regra de precedência

Uma direção anterior recomendou Windows Service persistente para manter o StepFlow Host continuamente disponível. Essa direção contrariava requisito Pocket já consolidado e está **revogada**.

A prova com Windows Service realizada no PC de desenvolvimento permanece apenas como evidência técnica descartável. Ela não autoriza serviço persistente, auto-start, Task Scheduler permanente, watchdog ou daemon residente como arquitetura padrão.

A direção vigente está detalhada em `docs/03-arquitetura/host-controller-pocket.md`.

## 2. Requisito operacional obrigatório

No servidor/máquina central:

```text
pasta StepFlow copiada
        ↓
Controller iniciado sob demanda
        ↓
Host iniciado como processo-filho
        ↓
StepFlow em uso
        ↓
encerramento operacional
        ↓
shutdown gracioso do Host
        ↓
Controller encerra
        ↓
nenhum processo StepFlow permanece ativo
```

Consequências obrigatórias:

- não instalar runtimes/toolchains no servidor;
- não registrar Windows Service como requisito normal;
- não configurar serviço auto-start;
- não criar tarefa agendada permanente;
- não alterar PATH global;
- não depender de instalação tradicional;
- não deixar Host, Controller, launcher, watchdog ou agente residente quando o StepFlow não estiver em uso;
- manter consumo de CPU/memória do StepFlow praticamente zero quando fechado;
- manter binários, configuração, dados, logs e backups em árvore controlada;
- remoção próxima de encerrar processos e remover a pasta, preservando dados/backups quando aplicável.

## 3. O que permanece tecnicamente validado

O Gate A do Bloco 2 validou que o Host em Rust pode:

- executar a partir de pasta própria;
- expor HTTP;
- operar SQLite local;
- persistir dados fora da pasta de binários;
- encerrar de forma controlada;
- executar sem Node.js, npm, Rust, Cargo ou Visual Studio no runtime.

A PoC de Windows Service validou apenas a integração possível com o SCM e foi descartada como direção de produção.

## 4. Modelo operacional aprovado

A solução central será composta conceitualmente por:

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

O Controller:

- inicia sob demanda na máquina central;
- impede segunda instância incompatível;
- inicia o Host como processo-filho;
- aguarda readiness;
- coordena shutdown;
- termina depois do Host;
- não permanece residente após o ciclo operacional.

O Host continua local aos dados e atende múltiplos Clients enquanto estiver ativo.

## 5. Limitação inevitável

Sem componente residente na máquina central, um Client remoto não consegue criar por si só um novo processo naquela máquina apenas abrindo um executável via SMB.

Logo, o início inicial do Controller precisa ocorrer na máquina central ou por mecanismo remoto corporativo já existente e aprovado. Essa limitação não poderá ser contornada instalando silenciosamente serviço, tarefa ou agente StepFlow.

## 6. Critério de aceite da solução final

A solução do Host só poderá ser consolidada para implementação se demonstrar:

1. uso simultâneo por vários Clients;
2. SQLite acessado somente pelo Host local aos dados;
3. nenhuma toolchain/runtime de desenvolvimento no servidor;
4. implantação baseada em copiar pasta pronta;
5. nenhuma instalação/registro persistente desnecessário no Windows;
6. início do Host somente quando necessário;
7. apenas uma instância válida sobre os dados;
8. shutdown controlado;
9. nenhum processo StepFlow residual após encerramento;
10. dados/config/logs separados dos binários;
11. atualização/rollback por substituição controlada de artefatos.

## 7. Próximos fechamentos do Bloco 2

Sem nova PoC neste momento, ainda precisam ser especificados documentalmente:

- configuração do Controller/Host;
- paths operacionais;
- logs e diagnóstico;
- política de shutdown quando houver Clients conectados;
- comportamento em falha do Controller/Host;
- atualização/rollback dos binários.

Depois desses itens, o Bloco 2 poderá ser encerrado e o Bloco 3 seguirá com launcher/distribuição do Client.
