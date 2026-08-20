# Concorrência, Fila de Escrita, Conflitos e Eventos — StepFlow

**Data:** 2026-08-20  
**Status:** DIREÇÃO ARQUITETURAL CONSOLIDADA PARA A FASE 1

## 1. Objetivo

Garantir uso simultâneo por múltiplos Clients sem corrupção, sobrescrita silenciosa ou estados divergentes, mantendo SQLite local ao Host.

## 2. Modelo geral

```text
Clients simultâneos
      ↓ HTTP
StepFlow Host
      ↓
validação/autorização
      ↓
coordenador de escrita
      ↓ fila bounded
writer único
      ↓ transação
SQLite local em WAL
      ↓ commit
resposta + evento
```

A fila e a revisão otimista têm funções diferentes e ambas são necessárias.

## 3. SQLite

Direção inicial:

```text
PRAGMA foreign_keys = ON
PRAGMA journal_mode = WAL
busy_timeout configurado
```

Motivos para WAL:

- leitores podem continuar enquanto existe escritor;
- escrita continua centralizada no Host;
- banco permanece no mesmo computador do Host;
- nenhum Client acessa WAL/SQLite por rede.

WAL não muda a regra de implantação: banco, `-wal` e `-shm` permanecem no filesystem local da máquina central.

## 4. Um único writer lógico

O Host terá um coordenador explícito para comandos que alteram estado persistente.

Direção recomendada:

- uma fila assíncrona bounded recebe comandos mutáveis;
- um worker/writer processa comandos em ordem;
- o writer mantém conexão de escrita controlada ao SQLite;
- transações são curtas;
- não realizar chamadas de rede ou espera do usuário dentro da transação;
- leituras podem usar conexões separadas controladas pelo Host.

Não depender de múltiplos writers competindo e `SQLITE_BUSY` como mecanismo normal de coordenação.

## 5. Fila bounded e backpressure

A fila não pode crescer sem limite.

Se o Host estiver temporariamente saturado:

- aplicar backpressure;
- rejeitar/adiar novas mutações com erro operacional do tipo `SERVER_BUSY`/503 quando necessário;
- nunca descartar comando aceito silenciosamente;
- nunca responder sucesso antes do commit.

Tamanho exato da fila será medido na implementação; não precisa ser grande para o uso interno esperado.

## 6. Estrutura conceitual de comando

Cada mutação chega ao coordenador com contexto suficiente, por exemplo:

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

`command_id` poderá atuar como chave de idempotência em operações que justificarem isso.

## 7. Ordem de processamento

Fluxo de uma escrita:

```text
request HTTP
   ↓
validar sessão/formato
   ↓
enfileirar
   ↓
writer recebe
   ↓
revalidar invariantes relevantes
   ↓
BEGIN transaction
   ↓
ler revisão/estado atual
   ↓
comparar base_revision
   ↓
aplicar mudança ou rejeitar
   ↓
COMMIT
   ↓
responder ao request
   ↓
publicar evento pós-commit
```

Evento nunca deve anunciar mudança que não foi commitada.

## 8. Revisão otimista de processos

Para processo:

```text
Client A lê revision 41
Client B lê revision 41

A envia base_revision=41
→ writer confirma atual=41
→ cria revision 42
→ commit

B envia base_revision=41
→ writer encontra atual=42
→ NÃO salva
→ retorna conflito
```

Fila não transforma o comando de B em válido. Ela apenas garante ordem previsível.

## 9. Resposta de conflito

Usar HTTP `409 Conflict` ou semântica equivalente no contrato.

Envelope conceitual:

```json
{
  "error": {
    "code": "PROCESS_REVISION_CONFLICT",
    "message": "O processo foi alterado por outro usuário.",
    "details": {
      "submitted_revision": 41,
      "current_revision": 42
    }
  }
}
```

O Client oferece recarregar/revisar. Não sobrescreve automaticamente a revisão mais nova.

## 10. Outras entidades mutáveis

Entidades relevantes além de processo devem possuir controle equivalente quando edição concorrente puder causar perda silenciosa.

Direção:

- `processes`: `revision_no` obrigatório;
- `users`: `row_revision`/equivalente;
- `company_settings`: `row_revision`/equivalente;
- configurações de baixo risco somente do Host podem usar transação/invariante sem expor revisão ao usuário quando não houver edição concorrente real.

## 11. Criações simultâneas

Constraints do banco continuam sendo última linha de defesa.

Exemplo: dois usuários tentam criar o mesmo `process.code`.

Mesmo que os comandos sejam enfileirados:

- primeiro válido pode commit;
- segundo viola unicidade e recebe erro de validação/conflito de negócio;
- nunca gerar dois processos com código duplicado.

## 12. Arquivamento versus edição

Arquivar/excluir logicamente um processo também exige `base_revision`.

Se outro usuário alterou o processo antes do arquivamento:

- operação antiga é rejeitada;
- usuário precisa revisar o estado atual;
- não arquivar silenciosamente uma revisão que o solicitante não viu.

## 13. Leituras concorrentes

Leituras não passam pela fila de escrita salvo necessidade específica.

Com WAL:

- consultas usam snapshots consistentes;
- leitores não precisam aguardar toda escrita normal;
- após receber evento de atualização, Client refaz consulta para obter estado commitado mais recente.

Consultas longas devem ser evitadas para não prejudicar checkpoints/WAL.

## 14. `busy_timeout`

Configurar timeout curto/moderado como defesa contra contenção transitória de SQLite.

Ele não é mecanismo principal de concorrência.

Se `SQLITE_BUSY` ocorrer de forma recorrente dentro de um único Host coordenado, tratar como sinal de diagnóstico/bug de coordenação em vez de simplesmente aumentar timeout indefinidamente.

## 15. Publicação de eventos

Após commit bem-sucedido:

```text
commit SQLite
   ↓
construir evento
   ↓
broadcast para Clients autorizados/conectados
```

Se um Client não receber o evento:

- o commit continua válido;
- WebSocket reconecta;
- Client refaz consultas relevantes;
- banco permanece fonte de verdade.

Não reverter transação porque um socket falhou.

## 16. Ordem e revisão de eventos

Eventos relacionados a recurso versionado carregam `revision`.

Client deve:

- ignorar evento claramente mais antigo que seu estado conhecido;
- atualizar/refazer consulta ao receber revisão mais nova;
- não assumir que `event_id` substitui revisão de domínio.

A ordem global entre eventos de entidades diferentes não precisa representar transação distribuída.

## 17. Presença / soft lock

**Não implementar soft lock/presença na primeira fundação.**

Motivo:

- revisão otimista já protege integridade;
- presença adiciona heartbeat/TTL/UX sem necessidade comprovada;
- usuário pode editar sem ser bloqueado por sessão abandonada.

Se futuramente houver benefício, presença será apenas informativa e nunca substituirá revisão no Host.

## 18. Hard lock

Não usar lock pessimista de longa duração para edição de processo na primeira versão.

Evitar cenário em que queda de Client deixe documentação bloqueada.

## 19. Desconexão durante escrita

Se o Client perder a conexão após enviar comando, é possível que o Host tenha commitado mesmo sem a resposta chegar.

Consequência:

- Client não repete cegamente mutação não idempotente;
- reconecta e consulta estado atual;
- operações críticas podem usar `command_id`/idempotency key;
- Host pode reconhecer comando já processado quando a operação justificar persistir idempotência.

## 20. Queda do Host

Transações SQLite fornecem atomicidade do commit.

Ao reiniciar:

- Host abre banco local;
- verifica schema/migrations;
- WAL é tratado pelo SQLite;
- não existe fila persistente de comandos “aceitos mas ainda não commitados” na primeira versão;
- request sem resposta confirmada deve ser reconciliado pelo Client.

## 21. Proteção contra dois Hosts sobre o mesmo banco

O Controller/Host deve impedir duas instâncias StepFlow Host coordenando o mesmo diretório de dados.

Usar exclusão mútua local por implantação, por exemplo mutex/lock de processo apropriado ao Windows.

Se outra instância válida já possui o banco:

- segunda instância não inicia operação normal;
- não tenta competir pelo SQLite;
- informa claramente que o Host já está ativo.

O mecanismo não é serviço e desaparece quando o processo termina.

## 22. Shutdown coordenado

Ao encerrar Host:

1. parar de aceitar novas mutações;
2. aguardar comando atualmente em commit dentro de limite seguro;
3. rejeitar/encerrar fila ainda não iniciada de forma explícita;
4. fechar WebSockets;
5. fechar conexões SQLite;
6. checkpoint quando apropriado;
7. encerrar processo.

Nenhuma mutação pode ficar com resposta de sucesso sem commit correspondente.

## 23. Cenários obrigatórios de teste futuro

Na Fase de implementação, validar mecanicamente pelo menos:

1. dois Clients editam mesma revisão;
2. dois Clients editam processos diferentes;
3. criação simultânea com mesmo código;
4. editar enquanto outro arquiva;
5. atualização de usuário concorrente;
6. evento chega após commit;
7. WebSocket cai e Client reconcilia;
8. timeout após comando potencialmente commitado;
9. tentativa de segundo Host no mesmo data dir;
10. shutdown durante fila/escrita;
11. múltiplas leituras enquanto há escrita.

## 24. Gate do Bloco 7

Bloco 7 arquiteturalmente fechado com:

1. SQLite local em WAL;
2. um único writer lógico no Host;
3. fila bounded e backpressure;
4. revisão otimista obrigatória para processos;
5. conflitos retornados sem sobrescrita automática;
6. constraints SQLite como última defesa;
7. eventos somente pós-commit;
8. reconciliação após perda de eventos/conexão;
9. sem soft/hard lock inicial de edição;
10. proteção contra dois Hosts no mesmo banco.

Próximo bloco: **Bloco 8 — especificação das telas críticas e contrato de UI/UX**.
