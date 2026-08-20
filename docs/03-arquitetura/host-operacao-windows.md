# StepFlow Host — Modelo Operacional no Windows

**Data:** 2026-08-20  
**Status:** REQUISITO CORRIGIDO / WINDOWS SERVICE PERSISTENTE REJEITADO COMO MODELO PADRÃO

## 1. Correção de direção

Uma direção anterior deste documento recomendou um Windows Service persistente para manter o StepFlow Host continuamente disponível. Essa direção contrariava um requisito Pocket já consolidado pelo PO e está **revogada**.

A prova com Windows Service realizada no PC de desenvolvimento permanece válida somente como evidência de viabilidade técnica do mecanismo. Ela **não autoriza** usar Windows Service persistente, serviço auto-start, Task Scheduler permanente ou processo residente como arquitetura padrão do StepFlow.

## 2. Requisito operacional obrigatório

No servidor/máquina central da empresa, o StepFlow deve buscar o modelo mais próximo possível de:

```text
pasta StepFlow copiada para o servidor
        ↓
usuário inicia o StepFlow
        ↓
processos necessários do StepFlow iniciam
        ↓
StepFlow é utilizado
        ↓
último uso/sessão do StepFlow é encerrado
        ↓
processos transitórios do StepFlow encerram
        ↓
nenhum processo StepFlow permanece consumindo recursos
```

Consequências obrigatórias:

- não instalar runtimes ou toolchains no servidor;
- não registrar Windows Service persistente como requisito normal;
- não configurar serviço auto-start;
- não criar tarefa agendada permanente para manter Host ativo;
- não alterar PATH global;
- não depender de instalação tradicional;
- não deixar Host, launcher, watchdog ou agente residente em segundo plano quando o StepFlow não estiver em uso;
- manter consumo de CPU/memória do StepFlow praticamente zero quando não estiver em uso;
- manter binários, configuração, dados, logs e backups dentro da árvore controlada do produto ou em paths explicitamente aprovados;
- remoção deve ser próxima de encerrar processos e remover a pasta, preservando dados/backups quando aplicável.

## 3. O que permanece validado

O Gate A do Bloco 2 validou que o Host em Rust pode:

- executar a partir de pasta própria;
- expor HTTP;
- operar SQLite local;
- persistir dados fora da pasta de binários;
- encerrar de forma controlada;
- executar sem Node.js, npm, Rust, Cargo ou Visual Studio no runtime.

A PoC posterior também validou que o mesmo tipo de Host pode integrar-se ao Service Control Manager. Essa capacidade fica registrada como contingência técnica, não como solução padrão.

## 4. Modelo conceitual de pasta

```text
StepFlow\
├── app\
│   ├── StepFlow.exe          # ponto de entrada/orquestrador a definir
│   └── StepFlowHost.exe      # processo Host transitório
├── config\
│   └── stepflow-host.toml
├── data\
│   └── stepflow.sqlite
├── logs\
└── backups\
```

A localização física real permanece aberta até o ambiente corporativo ser conhecido.

## 5. Questão arquitetural ainda aberta

O requisito multiusuário continua obrigatório. Portanto, a Fase 1 ainda precisa fechar **como o primeiro Client dispara o Host na máquina central e como o último uso autoriza seu encerramento**, sem instalar serviço persistente e sem deixar processo residente.

Esse problema deve ser tratado como um problema de bootstrap/orquestração do Host sob demanda, não como justificativa automática para manter um daemon permanente.

Possibilidades poderão ser estudadas, mas nenhuma está aprovada ainda:

- processo principal/orquestrador iniciado sob demanda na máquina central;
- mecanismo de execução remota já existente e aprovado no ambiente corporativo;
- desenho em que o ponto de entrada e o Host coexistam na própria máquina central quando o uso ocorrer nela;
- outra solução transitória que não exija instalação nem processo residente.

Não assumir que executar um `.exe` por SMB inicia esse processo no servidor: normalmente o processo executa na estação que o abriu. Esse detalhe deve ser resolvido explicitamente.

## 6. Critério de aceite da solução final

A solução do Host só poderá ser consolidada se demonstrar, ao mesmo tempo:

1. uso simultâneo por vários Clients;
2. SQLite acessado apenas pelo Host local aos dados;
3. nenhum runtime/toolchain de desenvolvimento instalado no servidor;
4. implantação baseada em copiar uma pasta pronta;
5. nenhuma instalação/registro persistente desnecessário no Windows;
6. início do Host somente quando necessário;
7. encerramento controlado quando o StepFlow deixa de estar em uso;
8. nenhum processo StepFlow residual após o encerramento;
9. dados/config/logs preservados independentemente do ciclo de vida do binário;
10. atualização/rollback por substituição controlada de artefatos.

## 7. Regra de precedência

Este documento corrige e substitui qualquer recomendação anterior de Windows Service persistente ou auto-start para o StepFlow Host.
