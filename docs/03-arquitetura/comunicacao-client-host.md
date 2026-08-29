# Comunicação StepFlow Client ↔ Host

**Status:** CONSOLIDADO PARA A FASE 1  
**Atualização:** 2026-08-29

## Tecnologias

- HTTP + JSON para consultas/comandos;
- WebSocket autenticado para eventos;
- contratos versionados, inicialmente `/api/v1`;
- Host em Rust/Axum/Tokio.

O WebSocket sinaliza mudanças; o estado oficial continua sendo confirmado pela API e pelo banco coordenado pelo Host.

## Descoberta/configuração

O Client recebe `deployment.json` junto da distribuição local. Ele contém somente configuração não sensível, por exemplo:

```json
{
  "deployment_id": "empresa-interna",
  "host_base_url": "http://<HOST-INTERNO>:<PORTA>",
  "api_contract_major": 1
}
```

Endereço/porta reais são configuração da implantação, não hardcode do build. Credenciais/tokens não entram nesse arquivo.

## Inicialização

```text
Client inicia
→ lê deployment.json
→ consulta compatibilidade do Host
→ valida contrato/versão
→ login
→ sessão autenticada
→ HTTP/JSON + WebSocket
```

## Contrato HTTP

- JSON UTF-8 para contratos estruturados;
- artefato binário pode usar body binário quando apropriado;
- autorização sempre no Host;
- escrita só retorna sucesso após commit;
- mutações versionadas carregam revisão/base esperada;
- retries automáticos não podem duplicar comando não idempotente;
- requests relevantes podem carregar `request_id`;
- operações críticas podem usar `command_id`/idempotency key quando justificadas.

Categorias base de erro:

- `SESSION_REQUIRED` / `SESSION_EXPIRED`;
- `PERMISSION_DENIED`;
- `VALIDATION_FAILED`;
- `RESOURCE_NOT_FOUND`;
- `REVISION_CONFLICT`;
- `CLIENT_HOST_INCOMPATIBLE`;
- `SERVER_BUSY`;
- `PERSISTENCE_ERROR`;
- `INTERNAL_ERROR`.

Códigos específicos de domínio podem estender essa taxonomia.

Envelope conceitual:

```json
{
  "error": {
    "code": "REVISION_CONFLICT",
    "message": "O recurso foi alterado por outro usuário.",
    "request_id": "...",
    "details": {}
  }
}
```

Não expor stack trace, SQL, paths internos ou segredos ao Client.

## Compatibilidade Client ↔ Host

Antes do login, o Host informa versão/contrato compatível e identificador da implantação quando aplicável.

Incompatibilidade bloqueia uso normal e orienta reabertura pelo Launcher para obter versão compatível.

## Eventos WebSocket

Envelope conceitual:

```json
{
  "event_id": "...",
  "type": "process.updated",
  "resource_id": "...",
  "revision": 42,
  "occurred_at": "..."
}
```

Princípios:

- eventos somente após commit;
- evitar payload sensível/desnecessário;
- Client normalmente invalida/reconsulta recurso;
- revisão ajuda a ignorar evento antigo;
- autorização do canal segue sessão;
- evento não substitui resposta/estado oficial.

## Reconexão

Ao perder WebSocket:

1. Client tenta reconectar com backoff limitado;
2. ao reconectar, revalida/reconsulta estado relevante;
3. não presume replay completo de eventos perdidos;
4. nenhuma escrita é considerada confirmada sem resposta ou reconciliação.

## Timeouts e backoff

Valores numéricos não são congelados na Fase 1 sem medição.

A implementação deve definir por benchmark/fixtures:

- timeout de conexão;
- timeout de operação comum;
- timeout/política própria de operações longas;
- backoff de reconexão;
- limites coerentes com LAN real e experiência do usuário.

Após timeout de mutação, não repetir cegamente. Reconciliar estado primeiro.

## Operações longas

Geração documental e Backup/Restore podem usar contratos próprios de operação.

- não inventar percentual quando não houver progresso real;
- fechar um Client não implica automaticamente cancelar operação Host-side já aceita;
- operação persistente/administrativa deve permitir reconsulta do estado quando seu contrato exigir;
- `SERVER_BUSY`/backpressure é preferível a fila ilimitada.

## Sem modo offline de edição

Na primeira versão, Host indisponível significa sistema oficial indisponível para login/dados/mutações.

Não enfileirar alterações locais para sincronização posterior nem tratar cache como fonte oficial.

## Transporte

A escolha final entre HTTP controlado, HTTPS/certificado interno, reverse proxy corporativo existente ou equivalente depende da infraestrutura real.

Não instalar stack pesada apenas para o StepFlow e não hardcodar PKI/domínio antes da validação corporativa.

## Diagnóstico

- `request_id` pode correlacionar Client e logs do Host;
- mensagens ao usuário permanecem operacionais e simples;
- detalhes técnicos seguros ficam nos logs;
- segredo, senha, token reutilizável e conteúdo sensível não entram em diagnóstico por conveniência.
