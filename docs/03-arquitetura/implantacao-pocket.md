# Implantação Pocket — Requisitos da Máquina Central

**Status:** REQUISITO ARQUITETURAL CONSOLIDADO  
**Atualização:** 2026-08-20

## Princípio central

A máquina central já existe e pode hospedar outros serviços importantes. O StepFlow deve adaptar-se a ela com o menor impacto possível.

Implantação desejada:

```text
pacote StepFlow pronto
→ copiar/arrastar pasta
→ configuração mínima
→ iniciar sob demanda
```

## Ciclo de vida obrigatório

```text
StepFlow fechado
→ nenhum processo StepFlow ativo

Controller iniciado sob demanda
→ Host inicia
→ múltiplos Clients utilizam o sistema
→ encerramento operacional
→ Host e Controller terminam
→ nenhum processo StepFlow residual
```

Regras:

- sem instalação tradicional obrigatória na máquina central;
- sem Windows Service StepFlow persistente;
- sem auto-start, Task Scheduler, watchdog, tray agent ou daemon StepFlow;
- sem Node.js, npm, Rust, Cargo, Visual Studio/MSVC ou SQLite Server exigidos em runtime;
- sem alteração global de PATH como requisito;
- sem reboot normal para implantação/atualização;
- consumo de CPU/memória do StepFlow tende a zero quando fechado.

Qualquer exceção futura exige mudança explícita do requisito pelo PO.

## Estrutura controlada

```text
StepFlow\
├── app\
├── config\
├── data\
├── logs\
└── backups\
```

- `app/` contém artefatos substituíveis;
- `config/`, `data/`, `logs/` e `backups/` são preservados entre atualizações;
- paths reais ainda dependem do ambiente corporativo;
- preferir caminhos relativos à raiz da implantação quando apropriado.

## Host Pocket

O Host é um papel/processo da arquitetura, não um serviço permanente.

A direção vigente é `StepFlowController.exe` iniciar `StepFlowHost.exe` como processo-filho na máquina central, aguardar readiness, impedir segunda instância sobre os mesmos dados e coordenar shutdown gracioso.

Detalhes em `host-pocket.md`.

## Limitação aceita

Executar um arquivo armazenado em SMB a partir do PC do técnico executa o programa naquela estação; isso não cria automaticamente um processo na máquina central.

Como nenhum componente StepFlow fica residente, o Controller precisa ser iniciado na máquina central ou por mecanismo remoto corporativo já existente e aprovado. Não instalar silenciosamente um serviço para contornar essa limitação.

## Client

A experiência do técnico permanece:

```text
ponto de entrada interno
→ duplo clique
→ Client local atualizado
→ login
```

O launcher/cópia local não deve exigir instalação manual ou configuração complexa na estação.

## Atualização e rollback

```text
encerrar processos StepFlow
→ backup quando necessário
→ substituir/ativar artefatos de app/
→ preservar dados/configuração
→ iniciar novamente
→ validar readiness
```

Rollback de binário não pode apagar banco automaticamente. Compatibilidade de schema/migrations deve ser respeitada.

## Critérios de aceitação de tecnologias

Toda escolha deve responder:

1. exige instalação global na máquina central?
2. pode interferir em outros serviços?
3. exige reboot ou alteração permanente de sistema?
4. pode ser distribuída dentro da pasta StepFlow?
5. funciona sem Internet após implantação?
6. permite atualização/remoção simples?
7. deixa processo StepFlow ativo quando o produto está fechado?

Quanto maior a interferência, menor a aderência ao Pocket.

## Regra final

**O servidor existe antes do StepFlow e continuará existindo independentemente dele. O StepFlow deve adaptar-se ao ambiente, não remodelá-lo para recebê-lo.**
