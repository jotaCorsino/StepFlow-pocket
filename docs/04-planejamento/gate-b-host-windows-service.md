# F1-B2 Gate B — Prova Operacional do Host como Windows Service

**Data:** 2026-08-20  
**Status:** PRONTO PARA EXECUÇÃO PELO PO  
**Ambiente:** PC pessoal de desenvolvimento, sessão `EARTH\Estudos`; operações do Service Control Manager exigem PowerShell elevado apenas no trecho administrativo.

## 1. Objetivo

Validar em uma única prova descartável que um Host Rust compatível com o Gate A pode operar como Windows Service nativo mantendo o princípio Pocket.

A prova deve validar:

1. build release fora do repositório;
2. pasta `app` separada de `data` e `logs`;
3. registro do serviço apontando para o executável da pasta Pocket;
4. start/stop/status pelo Service Control Manager;
5. `/health` e SQLite enquanto o serviço está ativo;
6. shutdown limpo ao receber stop;
7. restart;
8. remoção do serviço sem remover dados;
9. ausência de runtime/toolchain de desenvolvimento em produção.

Não há teste de reboot, LAN, firewall, autenticação ou WebSocket neste gate.

## 2. Tecnologia da prova

Usar crate `windows-service` atual compatível com o toolchain, além de Axum/Tokio e `rusqlite` bundled.

O executável será realmente service-aware; não usar NSSM, srvany ou wrapper externo.

## 3. Restrições

- não modificar `C:\dev\StepFlow`;
- não instalar runtime ou ferramenta adicional no Windows;
- não alterar PATH permanente;
- não criar regra de firewall;
- não testar bind LAN;
- não registrar serviço com nome definitivo do produto;
- não usar banco real;
- não deixar o serviço registrado ao final;
- não apagar `data` ou `logs` ao remover o serviço.

## 4. Resultado de aceite

O gate será aprovado se:

- o serviço for registrado e iniciado pelo SCM;
- `/health` responder `ok`;
- `/db-probe` criar/incrementar o banco na pasta persistente;
- `sc stop` produzir encerramento limpo e status STOPPED;
- o serviço puder iniciar novamente e o contador do banco persistir;
- `sc delete` remover somente o registro do serviço;
- binário, banco e logs permaneçam na pasta da PoC;
- nenhuma dependência global seja exigida no runtime.

## 5. Decisão após o gate

Se aprovado, consolidar Windows Service como modelo padrão do StepFlow Host, com instalação administrativa rara e operação diária automática.

Se falhar por comportamento real do serviço/SCM, revisar a decisão antes de abrir novos testes. Não iniciar sequência de microdiagnósticos.
