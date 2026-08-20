# F1-B2 Gate B — Prova Operacional do Host como Windows Service

**Data:** 2026-08-20  
**Status:** CONCLUÍDA TECNICAMENTE / ARQUITETURA REJEITADA COMO PADRÃO  
**Ambiente:** PC pessoal de desenvolvimento, sessão `EARTH\Estudos`; operações do Service Control Manager foram executadas em PowerShell elevado.

## 1. Resultado técnico

A prova descartável foi concluída com sucesso:

- build release concluído;
- pasta `app` separada de `data` e `logs`;
- serviço descartável registrado pelo Service Control Manager;
- start/stop/status funcionaram;
- `/health` respondeu;
- SQLite persistiu dados;
- restart preservou o banco;
- remoção do registro do serviço preservou binário, dados e logs;
- runtime não exigiu Node.js, npm, Rust, Cargo ou Visual Studio.

## 2. Reclassificação arquitetural

Apesar do sucesso técnico, o Windows Service persistente **não atende ao requisito Pocket consolidado pelo PO** como modelo padrão.

O requisito correto é:

- copiar/publicar a pasta StepFlow no servidor;
- não instalar runtimes ou componentes globais;
- não registrar serviço persistente como requisito normal;
- não deixar Host/agente/watchdog em segundo plano quando o StepFlow não estiver em uso;
- iniciar os processos necessários quando o StepFlow for utilizado;
- encerrar todos os processos transitórios do StepFlow quando o uso terminar;
- manter consumo praticamente zero quando o produto estiver fechado.

Portanto, o sucesso desta PoC prova somente que a tecnologia Rust pode integrar-se ao SCM se algum cenário futuro justificar essa contingência. **Não consolida Windows Service para produção.**

## 3. Motivo da correção

Uma recomendação anterior interpretou disponibilidade contínua do Host como requisito superior ao ciclo de vida Pocket. Essa interpretação estava incorreta.

O requisito de baixo impacto e ausência de processos residuais já existia como intenção do produto e deve prevalecer.

## 4. Próxima questão válida do Bloco 2

Fechar o mecanismo de **Host central sob demanda**:

```text
primeiro uso do StepFlow
        ↓
iniciar Host na máquina central
        ↓
múltiplos Clients utilizam o mesmo Host
        ↓
último uso termina
        ↓
Host encerra com segurança
        ↓
nenhum processo StepFlow residual
```

O ponto crítico é descobrir como disparar o Host na máquina central sem instalar serviço persistente. Executar um `.exe` localizado em um compartilhamento SMB normalmente executa o processo na estação cliente, portanto esse bootstrap precisa de solução explícita.

## 5. Regra de eficiência

Não abrir nova sequência de microprovas. A próxima etapa deve primeiro analisar e escolher uma estratégia operacional que satisfaça simultaneamente Pocket e multiusuário; somente depois executar uma prova pequena e conclusiva, se necessária.
