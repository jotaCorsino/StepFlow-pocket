# Comunicação StepFlow Client ↔ Host

**Status:** CONSOLIDADO PARA A FASE 1  
**Atualização:** 2026-08-20

## Tecnologias

- HTTP + JSON para consultas/comandos;
- WebSocket autenticado para eventos;
- contratos versionados, inicialmente `/api/v1`;
- Host em Rust/Axum/Tokio.

O WebSocket sinaliza mudanças; o estado oficial continua sendo obtido/confirmado pela API e pelo banco coordenado pelo Host.

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

## Fluxo de inicialização

```text
Client inicia
→ lê deployment.json
→ consulta /api/v1/system/compatibility
→ valida Host/contrato/versão
→ login
→ sessão autenticada
→ HTTP/JSON + WebSocket
```

## Contrato HTTP

- JSON UTF-8;
- autorização sempre no Host;
- escrita só retorna sucesso após commit;
- mutações versionadas carregam revisão/base esperada;
- retries automáticos não podem duplicar comandos não idempotentes.

Categorias mínimas de erro:

- `SESSION_REQUIRED` / `SESSION_EXPIRED`;
- `PERMISSION_DENIED`;
- `VALIDATION_FAILED`;
- `RESOURCE_NOT_FOUND`;
- `REVISION_CONFLICT`;
- `CLIENT_HOST_INCOMPATIBLE`;
- `SERVER_BUSY`;
- `PERSISTENCE_ERROR`;
- `INTERNAL_ERROR`.

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

Antes do login, o Host informa versão, `api_contract_major`, versão mínima/máxima compatível do Client quando aplicável e identificador da instância.

Incompatibilidade de contrato bloqueia o uso e orienta reinício pelo launcher para obter versão compatível.

## Eventos WebSocket

Envelope mínimo:

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
- Client normalmente invalida/reconsulta o recurso;
- revisão ajuda a ignorar evento antigo;
- autorização do canal segue a sessão.

## Reconexão

Ao perder WebSocket:

1. Client tenta reconectar com backoff limitado;
2. ao reconectar, revalida/reconsulta estado relevante;
3. não presume replay completo dos eventos perdidos;
4. nenhuma escrita é considerada confirmada sem resposta/estado reconciliado.

## Timeouts e repetição

Valores iniciais aproximados podem ser ajustados por medição:

- conexão: 3–5 s;
- operações comuns: ~15 s;
- operações longas terão política própria.

Após timeout de mutação, não repetir cegamente. Reconciliar o estado; operações críticas podem usar `command_id`/idempotency key.

## Sem modo offline de edição

Na primeira versão, Host indisponível significa sistema oficial indisponível para login/dados/mutações. Não enfileirar alterações locais para sincronizar depois nem tratar cache como fonte oficial.

## Transporte

A escolha final entre HTTP controlado, HTTPS/certificado interno, reverse proxy corporativo existente ou equivalente depende da infraestrutura real. Não instalar stack pesada apenas para o StepFlow e não hardcodar PKI/domínio antes da validação corporativa.

## Diagnóstico

Requisições relevantes podem usar `request_id` para correlação com logs. Mensagens ao usuário são simples; detalhes técnicos seguros ficam no Host.
