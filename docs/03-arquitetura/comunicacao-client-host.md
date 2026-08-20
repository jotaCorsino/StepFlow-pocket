# Comunicação StepFlow Client ↔ Host

**Data:** 2026-08-20  
**Status:** DIREÇÃO ARQUITETURAL CONSOLIDADA PARA A FASE 1

## 1. Objetivo

Definir como o Client encontra o Host, troca comandos/consultas, recebe atualizações em tempo real e detecta incompatibilidades, sem acesso direto ao SQLite e sem configuração manual pelo técnico.

## 2. Tecnologias consolidadas

A comunicação inicial será composta por:

- **HTTP + JSON** para consultas, comandos e respostas;
- **WebSocket** para eventos em tempo real;
- Host em **Rust + Axum/Tokio**;
- contratos versionados sob prefixo de API, inicialmente conceitualmente `/api/v1`.

O WebSocket não substitui a API HTTP. Ele serve prioritariamente para sinalizar mudanças e manter o Client atualizado.

## 3. Descoberta do Host

Não usar descoberta automática por broadcast, mDNS ou varredura de rede na primeira versão.

A implantação terá um arquivo não sensível de configuração publicado junto ao launcher, conceitualmente:

```text
client\
├── StepFlowLauncher.exe
├── manifest.json
└── deployment.json
```

Exemplo conceitual, sem endereço real:

```json
{
  "deployment_id": "empresa-interna",
  "host_base_url": "http://<HOST-INTERNO>:<PORTA>",
  "api_contract_major": 1
}
```

Regras:

- endereço/porta reais são configuração da implantação, não hardcode do build;
- launcher copia/atualiza a configuração junto da cópia local do Client;
- Client lê a configuração local validada;
- nenhuma credencial, senha ou token fica em `deployment.json`;
- alterar hostname/porta não deve exigir recompilar o Client.

## 4. Fluxo inicial do Client

```text
Client inicia
   ↓
carrega deployment.json
   ↓
GET /api/v1/system/compatibility
   ↓
Host acessível e compatível?
   ├── não → estado operacional de indisponibilidade/incompatibilidade
   └── sim
        ↓
login
        ↓
sessão autenticada
        ↓
HTTP/JSON + WebSocket de eventos
```

Autenticação e formato da sessão serão fechados no Bloco 5.

## 5. Contrato HTTP

Diretrizes:

- JSON UTF-8 para payloads comuns;
- endpoints agrupados por recurso/domínio;
- operações de leitura usam métodos HTTP adequados;
- comandos de alteração passam exclusivamente pelo Host;
- respostas de escrita retornam a revisão/estado confirmado após commit;
- nenhuma resposta positiva de escrita é emitida antes do commit persistente;
- nenhuma lógica de autorização depende somente do Client.

Estrutura conceitual:

```text
/api/v1/system/compatibility
/api/v1/auth/...
/api/v1/processes/...
/api/v1/users/...
/api/v1/company/...
/api/v1/backup/...
/api/v1/events
```

Os endpoints de domínio exatos serão definidos nos blocos correspondentes, não antecipados aqui.

## 6. Erros padronizados

Toda falha de API relevante deve usar envelope consistente, por exemplo:

```json
{
  "error": {
    "code": "PROCESS_REVISION_CONFLICT",
    "message": "O processo foi alterado por outro usuário.",
    "request_id": "...",
    "details": {}
  }
}
```

Campos conceituais:

- `code`: código estável para tratamento pelo Client;
- `message`: mensagem segura e operacional;
- `request_id`: correlação com logs quando aplicável;
- `details`: dados estruturados estritamente necessários, por exemplo erros de campo ou revisão atual.

Não expor stack traces, paths internos, SQL ou segredos ao Client.

## 7. Categorias mínimas de erro

O Client deve conseguir distinguir ao menos:

- `HOST_UNAVAILABLE` — erro local de conectividade;
- `SESSION_REQUIRED`;
- `SESSION_EXPIRED`;
- `PERMISSION_DENIED`;
- `VALIDATION_FAILED`;
- `RESOURCE_NOT_FOUND`;
- `REVISION_CONFLICT`;
- `CLIENT_HOST_INCOMPATIBLE`;
- `PERSISTENCE_ERROR`;
- `INTERNAL_ERROR`.

Códigos específicos de domínio poderão especializar essas categorias.

## 8. Compatibilidade Client ↔ Host

Antes do login, o Client consulta um endpoint público mínimo de compatibilidade.

Resposta conceitual:

```json
{
  "host_version": "x.y.z",
  "api_contract_major": 1,
  "minimum_client_version": "x.y.z",
  "maximum_client_version": null,
  "instance_id": "..."
}
```

Regras:

- incompatibilidade de `api_contract_major` bloqueia uso;
- Host pode declarar versão mínima de Client;
- Client antigo incompatível deve orientar o usuário a reiniciar pelo launcher para obter a publicação atual;
- sessões já abertas em versão anterior só continuam enquanto forem declaradas compatíveis pelo Host;
- atualização não deve depender de comparação lexicográfica simples de strings de versão.

## 9. WebSocket de eventos

Após autenticação, o Client abre um canal de eventos no Host.

Eventos iniciais conceituais:

```text
process.created
process.updated
process.deleted
process.version.created
user.updated
company.settings.updated
```

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

- evento é notificação, não necessariamente snapshot completo;
- evitar dados sensíveis desnecessários no canal;
- Client normalmente invalida/refaz a consulta autorizada do recurso;
- `revision` ajuda a descartar notificações obsoletas;
- autorização do canal segue a sessão do usuário.

## 10. Perda e reconexão do canal de eventos

Na primeira versão não é obrigatório manter um journal completo de eventos para replay.

Ao perder WebSocket:

1. Client continua distinguindo Host acessível de canal de eventos indisponível;
2. tenta reconectar com backoff limitado;
3. ao reconectar, invalida/refaz as consultas relevantes em vez de presumir que recebeu todos os eventos perdidos;
4. nenhuma escrita é perdida silenciosamente porque comandos continuam sendo confirmados pelo HTTP/Host.

Backoff inicial recomendado para implementação: aproximadamente `1s → 2s → 5s → 10s`, limitado a intervalo máximo razoável enquanto a aplicação estiver aberta.

## 11. Timeouts iniciais

Direção inicial, ajustável por medição futura:

- conexão com Host: curta, aproximadamente 3–5 segundos;
- operações comuns de API: aproximadamente 15 segundos;
- operações naturalmente longas, como exportação/backup, terão política própria nos blocos correspondentes;
- timeout não significa sucesso/falha desconhecida de escrita: se o Client não receber confirmação, deve reconciliar estado antes de repetir comando não idempotente.

Valores finais poderão ser parametrizados, mas não devem resultar em espera indefinida da UI.

## 12. Idempotência e repetição

O Client não deve repetir automaticamente comandos de escrita não idempotentes após timeout apenas porque a resposta foi perdida.

Para comandos críticos poderá ser usado identificador de operação/idempotency key quando o domínio justificar.

A regra fundamental é evitar criação/alteração duplicada por retry cego.

## 13. Concorrência

Os contratos de escrita que alteram recursos versionados deverão carregar a revisão/base esperada.

Exemplo conceitual:

```json
{
  "base_revision": 41,
  "changes": {}
}
```

Se o estado atual já estiver em revisão 42, o Host rejeita a alteração com conflito estruturado.

A especificação completa pertence ao Bloco 7.

## 14. Operação sem Host

A primeira versão não terá modo offline de edição.

Se o Host estiver desligado/indisponível:

- Client pode abrir a interface e apresentar diagnóstico;
- login e dados oficiais não são considerados operacionais;
- não enfileirar alterações oficiais localmente para sincronização posterior;
- não usar cache local como fonte oficial de verdade.

Isso reduz risco de divergência e mantém o Host como fonte coordenadora.

## 15. Segurança de transporte

O produto é interno, mas credenciais e sessões não devem ser tratadas como dados públicos.

A forma final de proteção do transporte na LAN (HTTP local controlado, HTTPS com certificado interno, reverse proxy existente ou outra política corporativa) precisa ser fechada considerando a infraestrutura real da empresa.

Nenhuma solução deve exigir instalar uma stack pesada apenas para o StepFlow.

Até conhecer a LAN/políticas reais, não hardcodar certificado, domínio ou suposição de PKI.

## 16. Diagnóstico

Cada requisição relevante pode receber `request_id` gerado/propagado pelo Host para correlação com logs.

O Client deve apresentar mensagens simples ao usuário, enquanto logs técnicos do Host preservam contexto suficiente para diagnóstico sem gravar senhas/tokens.

## 17. Gate do Bloco 4

O Bloco 4 está arquiteturalmente fechado com estas decisões:

1. HTTP/JSON para API;
2. WebSocket para eventos;
3. endpoint do Host via `deployment.json`;
4. prefixo/versionamento explícito de contrato;
5. handshake de compatibilidade antes do login;
6. envelope padronizado de erros;
7. reconexão WebSocket com revalidação de estado;
8. nenhuma edição offline;
9. controle de revisão integrado aos futuros comandos de escrita;
10. proteção de transporte mantida como decisão dependente da infraestrutura corporativa real.

Próximo bloco: **Bloco 5 — Autenticação, sessão e autorização**.
