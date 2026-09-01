# Comunicação StepFlow Client ↔ Host

**Status:** CONSOLIDADO PARA A FASE 1  
**Atualização:** 2026-09-01

## Tecnologias

- HTTP + JSON para consultas/comandos;
- WebSocket autenticado para eventos;
- contratos versionados, inicialmente `/api/v1`;
- Host em Rust/Axum/Tokio.

O WebSocket sinaliza mudanças; o estado oficial continua confirmado pela API e pelo banco coordenado pelo Host.

## Descoberta/configuração

O Client recebe `deployment.json` junto da distribuição local. Ele contém somente configuração não sensível, por exemplo:

```json
{
  "deployment_id": "<DEPLOYMENT-ID>",
  "host_base_url": "http://<HOST-INTERNO>:<PORTA>",
  "api_contract_major": 1
}
```

Endpoint real é configuração da implantação, não hardcode do build. Credenciais/tokens não entram nesse arquivo.

Template/fixture versionado não deve virar configuração de produção silenciosamente. A materialização do `deployment.json` real de implantação está fechada na validação final do Bloco 12.

## Inicialização

```text
Client inicia
→ lê deployment.json
→ consulta compatibilidade do Host
→ valida contrato/versão
→ login quando a fase de autenticação estiver implementada
→ sessão autenticada
→ HTTP/JSON + WebSocket
```

Na fundação da Fase 2, o Client usa somente a consulta HTTP mínima de compatibilidade; WebSocket operacional autenticado entra junto da sessão/autorização apropriada.

## Contrato HTTP

- JSON UTF-8 para contratos estruturados;
- artefato binário pode usar body binário quando apropriado;
- autorização sempre no Host;
- escrita só retorna sucesso após commit;
- mutações versionadas carregam revisão/base esperada;
- retries automáticos não duplicam comando não idempotente;
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

Não expor stack trace, SQL, paths internos ou segredos ao Client.

## Compatibilidade Client ↔ Host

Antes do login, o Host informa versão/contrato compatível e identificador da implantação quando aplicável.

Incompatibilidade bloqueia uso normal e orienta reabertura pelo Launcher para obter versão compatível.

## Eventos WebSocket

Princípios:

- eventos somente após commit;
- payload mínimo e não sensível;
- Client normalmente invalida/reconsulta recurso;
- revisão ajuda a ignorar evento antigo;
- autorização do canal segue sessão;
- evento não substitui resposta/estado oficial.

## Reconexão

Ao perder WebSocket:

1. Client tenta reconectar com backoff bounded;
2. ao reconectar, revalida/reconsulta estado relevante;
3. não presume replay completo de eventos perdidos;
4. nenhuma escrita é considerada confirmada sem resposta ou reconciliação.

Defaults D12.72–D12.73:

```text
connect_timeout = 5 s
common_request_timeout = 30 s
websocket_reconnect = 1, 2, 4, 8, 15, 30 s (cap)
jitter = ±20%
backoff_reset_after_stable = 60 s
```

Esses valores pertencem ao serviço de comunicação do Client e não ficam espalhados pela UI.

Após timeout de mutação, não repetir cegamente. Reconciliar estado primeiro.

### Manutenção/Restore

- aviso/evento de manutenção antes da troca física é best-effort;
- queda da conexão durante Restore não significa sucesso nem falha;
- Restore aplicado ou rollback depois da fase destrutiva passa por fresh Host;
- enquanto recovery estiver pendente, readiness normal fica indisponível;
- depois do fresh Host, token pré-Restore é rejeitado;
- após novo login, Client refaz consultas;
- resultado do Restore é reconsultado, nunca inferido da desconexão.

## Operações longas

Geração documental e Backup/Restore usam contratos próprios:

- não inventar percentual sem progresso real;
- fechar Client não cancela automaticamente operação Host-side já aceita;
- operação persistente/administrativa permite reconsulta quando seu contrato exigir;
- `SERVER_BUSY`/backpressure é preferível a fila ilimitada;
- `uncertain/RECOVERY_REQUIRED` não vira sucesso por timeout.

## Sem modo offline de edição

Host indisponível significa sistema oficial indisponível para login/dados/mutações. Não enfileirar alterações locais para sincronização posterior nem tratar cache como fonte oficial.

## Transporte

A escolha entre HTTP controlado, HTTPS/certificado interno, reverse proxy corporativo existente ou equivalente depende da infraestrutura real. Não instalar stack pesada apenas para o StepFlow nem hardcodar PKI/domínio antes da validação corporativa.

## Diagnóstico

- `request_id` pode correlacionar Client e logs do Host;
- mensagens ao usuário permanecem operacionais e simples;
- detalhes técnicos seguros ficam nos logs;
- segredo, senha, token reutilizável e conteúdo sensível não entram em diagnóstico por conveniência.
