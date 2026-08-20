# StepFlow Host Controller — Ciclo de Vida Pocket

**Data:** 2026-08-20  
**Status:** DIREÇÃO ARQUITETURAL APROVADA PELO PO / IMPLEMENTAÇÃO AINDA NÃO AUTORIZADA

## 1. Objetivo

Definir o ciclo de vida do StepFlow na máquina central respeitando simultaneamente:

- copy-deploy por pasta;
- nenhuma instalação de runtime/toolchain no servidor;
- nenhum Windows Service persistente;
- nenhum processo StepFlow residente quando o produto não estiver em uso;
- SQLite acessado somente pelo Host local aos dados;
- suporte a múltiplos Clients simultâneos enquanto o Host estiver ativo.

## 2. Modelo aprovado

O StepFlow terá um ponto de entrada/orquestrador portátil na máquina central, conceitualmente chamado neste documento de **StepFlow Host Controller**.

Ele não é serviço do Windows, não é daemon, não é tarefa agendada e não permanece ativo depois do encerramento operacional do StepFlow.

Fluxo conceitual:

```text
pasta StepFlow na máquina central
        ↓
StepFlow Controller é iniciado sob demanda
        ↓
valida configuração e paths persistentes
        ↓
impede segunda instância central incompatível
        ↓
inicia StepFlowHost.exe como processo-filho
        ↓
aguarda Host ficar pronto
        ↓
Host disponibiliza API/eventos e abre SQLite local
        ↓
Clients utilizam o sistema
        ↓
encerramento do ciclo operacional
        ↓
Controller solicita shutdown gracioso do Host
        ↓
Host fecha conexões/SQLite e termina
        ↓
Controller confirma término e encerra
        ↓
nenhum processo StepFlow permanece ativo
```

## 3. Estrutura conceitual

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

Os nomes finais dos executáveis e o path real serão definidos posteriormente. Essa estrutura não autoriza scaffold definitivo na Fase 1.

## 4. Responsabilidades do Controller

O Controller deverá, no mínimo:

1. localizar a raiz da implantação a partir da própria pasta;
2. validar existência/acesso aos diretórios necessários;
3. carregar a configuração operacional do Host;
4. detectar se já existe instância central válida;
5. impedir duas instâncias do Host usando os mesmos dados;
6. iniciar o Host como processo-filho quando necessário;
7. aguardar readiness do Host antes de declarar o StepFlow disponível;
8. registrar erro operacional de inicialização de forma legível;
9. coordenar shutdown gracioso;
10. garantir que, encerrado o ciclo, não reste processo StepFlow iniciado por ele.

O Controller não deve:

- instalar serviço;
- alterar registro global;
- alterar PATH global;
- instalar runtime;
- instalar SQLite Server;
- configurar auto-start do Windows;
- manter watchdog residente após o encerramento.

## 5. Responsabilidades do Host

Enquanto ativo, o Host continua responsável por:

- autenticação e autorização;
- API e contratos;
- canal de eventos;
- acesso exclusivo/coordenado ao SQLite;
- transações;
- revisão/conflitos;
- serialização das escritas quando necessária;
- auditoria;
- backup/restauração;
- arquivos persistentes administrados pelo sistema.

O Host deverá responder a um mecanismo controlado de shutdown solicitado pelo Controller e finalizar apenas depois de encerrar trabalho em andamento de maneira segura.

## 6. Segunda instância

Duas instâncias do Host não poderão operar simultaneamente sobre o mesmo conjunto de dados.

Direção:

- Controller verifica se a instância central já existe/está saudável;
- se já existir, não inicia outro Host;
- se houver estado ambíguo, não assume que é seguro iniciar uma segunda instância;
- a proteção final deve combinar verificação de processo/readiness com um mecanismo local de exclusividade apropriado, a definir tecnicamente.

## 7. Encerramento e múltiplos Clients

O encerramento não poderá corromper sessões ativas de forma silenciosa.

A política exata ainda deve ser especificada, mas a arquitetura deve distinguir:

- encerramento solicitado pelo operador central;
- Clients ainda conectados;
- operações/escritas em andamento;
- shutdown normal;
- falha/encerramento inesperado do Controller;
- falha/encerramento inesperado do Host.

O requisito do PO permanece: terminado o uso do StepFlow, os processos transitórios devem encerrar e o consumo de recursos do StepFlow deve tender a zero.

## 8. Limitação inevitável do modelo zero-residente

Sem serviço, tarefa agendada, agente ou outro processo residente na máquina central, um Client remoto não consegue iniciar por conta própria um novo processo na máquina central apenas executando um arquivo localizado por SMB.

Portanto:

- o Controller precisa ser iniciado na máquina central por algum operador/mecanismo já existente no ambiente;
- se o ambiente corporativo possuir mecanismo remoto oficial já disponível, ele poderá ser avaliado posteriormente sem instalar componente StepFlow residente;
- na ausência desse mecanismo, a disponibilidade inicial do Host depende de iniciar o Controller na máquina central.

Essa limitação não deve ser escondida nem contornada instalando silenciosamente um serviço.

## 9. Implantação e atualização

A implantação desejada permanece:

```text
pacote StepFlow pronto
        ↓
copiar pasta para a máquina central
        ↓
ajustar configuração mínima
        ↓
iniciar Controller
```

Atualização:

1. encerrar Controller/Host;
2. confirmar ausência dos processos;
3. substituir/ativar binários de `app`;
4. preservar `config`, `data`, `logs` e `backups`;
5. iniciar novamente;
6. permitir rollback dos binários sem apagar dados.

Migrations e compatibilidade de schema serão tratadas no Bloco 6.

## 10. Critérios de aceite para implementação futura

A implementação dessa arquitetura só poderá ser considerada correta se provar:

- execução por pasta sem instalação tradicional;
- nenhuma dependência global adicional no servidor;
- início e shutdown sem privilégio administrativo, salvo necessidade real do ambiente a ser validada;
- apenas uma instância válida sobre a base;
- Host realmente filho/orquestrado pelo Controller;
- readiness antes de liberar uso;
- shutdown gracioso;
- nenhum processo StepFlow residual após término;
- `data` preservado independentemente de substituição de `app`;
- funcionamento multiusuário durante o ciclo ativo.

## 11. Fora do escopo desta decisão

Ainda não estão definidos aqui:

- protocolo Client ↔ Host;
- mecanismo exato de eventos;
- mecanismo exato de exclusividade de instância;
- regra final para encerrar quando Clients ainda estiverem conectados;
- porta/endereço reais;
- mecanismo corporativo real para iniciar o Controller;
- launcher do Client;
- paths reais do servidor.

Esses pontos serão fechados nos blocos correspondentes da Fase 1.
