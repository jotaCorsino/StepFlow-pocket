# Concorrência, Fila de Escrita, Conflitos e Eventos — StepFlow Pocket

**Status:** DIREÇÃO ARQUITETURAL CONSOLIDADA PARA A FASE 1  
**Atualização:** 2026-08-29

## Objetivo

Garantir uso simultâneo por múltiplos Clients sem corrupção, sobrescrita silenciosa ou estados divergentes, mantendo SQLite local ao Host.

## Modelo geral

```text
Clients simultâneos
      ↓ HTTP
StepFlow Host
      ↓
validação/autorização
      ↓
coordenador de escrita
      ↓ fila bounded
writer lógico
      ↓ transação
SQLite local em WAL
      ↓ commit
resposta + evento pós-commit
```

Fila e revisão otimista têm funções diferentes e ambas são necessárias.

## SQLite

Direção inicial:

```text
PRAGMA foreign_keys = ON
PRAGMA journal_mode = WAL
busy_timeout configurado por evidência
```

- banco, `-wal` e `-shm` permanecem no filesystem local da máquina central;
- nenhum Client acessa SQLite/WAL por rede;
- WAL permite leituras concorrentes ao writer coordenado.

## Writer lógico e fila bounded

O Host possui coordenador explícito para comandos que alteram estado persistente.

- fila assíncrona bounded;
- processamento ordenado por writer lógico;
- conexão de escrita controlada;
- transações curtas;
- sem chamada de rede ou espera do usuário dentro da transação;
- leituras usam conexões controladas separadas quando apropriado.

Se houver saturação:

- aplicar backpressure;
- rejeitar/admitir de forma controlada com `SERVER_BUSY`/503 ou equivalente;
- nunca descartar comando aceito silenciosamente;
- nunca responder sucesso antes do commit.

Tamanho da fila, limites e timeouts serão definidos por benchmark, não por números arbitrários.

## Estrutura conceitual de comando

```text
command_id
request_id
actor_user_id
operation
entity_id
base_revision
payload validado
received_at
```

`command_id` pode ser chave de idempotência quando a operação justificar.

## Fluxo de escrita

```text
request HTTP
→ validar sessão/formato
→ admitir/enfileirar
→ writer recebe
→ revalidar invariantes
→ BEGIN
→ ler estado/revisão atual
→ comparar base_revision
→ aplicar ou rejeitar
→ COMMIT
→ responder
→ publicar evento
```

Evento nunca anuncia mudança não commitada.

## Revisão otimista

Exemplo:

```text
Client A lê revisão 41
Client B lê revisão 41

A envia base=41
→ commit revisão 42

B envia base=41
→ atual já é 42
→ rejeitar conflito
```

A fila não transforma o comando de B em válido.

Conflito retorna `409 Conflict` ou semântica equivalente e o Client oferece reconsulta/revisão consciente. Sem overwrite ou merge automático.

## Granularidade por domínio

Controle concorrente deve ser proporcional ao recurso:

- Procedimento/revisão: revisão otimista do recurso;
- usuários/configuração: `row_revision`/equivalente quando necessário;
- Atendimento: revisão própria;
- Equipamento: revisão própria;
- checklist: por item/equivalente;
- observação de serviço: por Etapa/equivalente.

Alterações independentes não devem gerar conflito global apenas por conveniência de implementação.

## Constraints como última defesa

Unicidade, foreign keys e demais constraints continuam protegendo invariantes no banco.

Exemplo: duas criações simultâneas com mesmo código não podem produzir duas entidades válidas; uma deve falhar de modo determinístico.

## Leituras concorrentes

Leituras não passam pela fila de escrita salvo necessidade específica.

- consultas usam snapshots consistentes;
- eventos sinalizam mudança e Client reconsulta;
- consultas longas devem ser evitadas quando prejudicarem WAL/checkpoints;
- geração documental captura snapshot consistente e renderiza depois, fora da transação de leitura.

## `busy_timeout`

É defesa contra contenção transitória, não mecanismo principal de concorrência.

Valor numérico será definido por medição. `SQLITE_BUSY` recorrente dentro de um único Host coordenado deve ser tratado como sinal de diagnóstico/coordenação, não resolvido indefinidamente aumentando timeout.

## Eventos

Após commit:

```text
commit SQLite
→ construir evento
→ broadcast para Clients autorizados
```

Se Client perder evento:

- commit continua válido;
- WebSocket reconecta;
- Client refaz consultas relevantes;
- banco permanece fonte oficial.

Eventos de recurso versionado carregam revisão quando útil. Ordem global entre entidades diferentes não precisa simular transação distribuída.

## Presença e locks

Não implementar presença/soft lock na primeira fundação.

- revisão otimista já protege integridade;
- presença futura, se existir, será apenas informativa;
- não usar hard lock pessimista de longa duração para edição comum.

## Desconexão durante mutação

Se conexão cair após envio, o Host pode ter commitado sem a resposta chegar.

Consequência:

- Client não repete cegamente mutação não idempotente;
- reconecta e consulta estado;
- operação crítica pode usar `command_id` quando justificado.

## Queda do Host

- transações SQLite protegem atomicidade do commit;
- ao reiniciar, Host valida schema/migrations e abre banco local;
- não há fila persistente de comandos “aceitos mas não commitados” inicialmente;
- request sem confirmação precisa ser reconciliado pelo Client.

## Proteção contra dois Hosts

Controller/Host deve impedir duas instâncias coordenando o mesmo diretório de dados.

Usar exclusão mútua local por implantação. Segunda instância válida não compete pelo SQLite e informa que o Host já está ativo.

## Operações administrativas e longas

Backup/Restore e outras operações críticas devem possuir coordenação explícita com mutações normais.

O Bloco 11 definirá:

- serialização/lock administrativo necessário;
- janela de manutenção ou convivência com mutações durante backup;
- comportamento durante Restore;
- reconexão e resultado após operação crítica.

Não criar paralelismo destrutivo por inferência.

## Shutdown coordenado

Ao encerrar Host:

1. parar de aceitar novas mutações;
2. concluir/abortar transacionalmente trabalho em andamento;
3. tratar fila não iniciada explicitamente;
4. coordenar operação administrativa ativa conforme contrato;
5. encerrar WebSockets;
6. fechar SQLite/conexões;
7. checkpoint quando apropriado;
8. encerrar processo.

Nenhuma mutação pode receber sucesso sem commit correspondente.

## Cenários de teste futuros

Validar mecanicamente, entre outros:

- dois Clients editando mesmo recurso;
- Clients editando recursos independentes;
- criação simultânea com unicidade;
- editar versus arquivar;
- checklist/observações concorrentes;
- evento somente após commit;
- perda/reconexão de WebSocket;
- timeout após mutação potencialmente commitada;
- tentativa de segundo Host;
- shutdown com fila/escrita;
- leituras durante escrita;
- saturação e `SERVER_BUSY`;
- coordenação de Backup/Restore após contrato do Bloco 11.

## Gate arquitetural preservado

A direção consolidada exige:

1. SQLite local em WAL;
2. writer lógico coordenado;
3. fila bounded + backpressure;
4. revisão otimista;
5. conflitos sem sobrescrita automática;
6. constraints como última defesa;
7. eventos somente pós-commit;
8. reconciliação após perda de conexão/eventos;
9. sem soft/hard lock de edição comum inicialmente;
10. proteção contra dois Hosts sobre os mesmos dados.
