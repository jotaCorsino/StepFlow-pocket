# Bloco 10 — Exportação / Impressão + Ficha Compacta

**Status:** EM ANDAMENTO — ETAPA 1 CONSOLIDADA / ETAPA 2 PRÓXIMA  
**Fase:** Fase 1 — Fechamento arquitetural e especificação  
**Abertura:** 2026-08-25  
**Etapa 1 consolidada:** 2026-08-25

## 1. Objetivo do bloco

Fechar, uma etapa por vez, o contrato técnico de geração documental do StepFlow, preservando o caráter Pocket e a UX já aprovada no Bloco 8.

Este documento é o mapa técnico do Bloco 10. Uma etapa futura só entra em análise quando for explicitamente aberta. Não pertence a este bloco implementar código de produção, fechar Backup/Restore técnico ou abrir o Bloco 11.

## 2. Etapas do Bloco 10

| Ordem | Etapa | Estado |
|---|---|---|
| 1 | Arquitetura de geração documental | **CONSOLIDADO / APROVADO PELO PO** |
| 2 | PDF de Procedimentos | **PRÓXIMA — AINDA NÃO EM ANÁLISE** |
| 3 | DOCX de Procedimentos | PENDENTE |
| 4 | Impressão Windows de Procedimentos | PENDENTE |
| 5 | Template físico de Procedimentos | PENDENTE |
| 6 | PDF + preview da Ficha compacta | PENDENTE |
| 7 | Template físico A4 da Ficha | PENDENTE |
| 8 | Limites textuais e densidade da Ficha | PENDENTE |
| 9 | Múltiplos MACs / Procedimentos na Ficha | PENDENTE |
| 10 | Nomes de arquivo + artefatos temporários | PENDENTE |
| 11 | QR / barcode | PENDENTE |
| 12 | Validação técnica final do Bloco 10 | PENDENTE |

Nenhuma decisão específica das Etapas 2–12 é consolidada por este checkpoint.

---

# Etapa 1 — Arquitetura de geração documental

**Status:** CONSOLIDADO / APROVADO PELO PO

## 3. Objetivo

Definir somente as fronteiras arquiteturais da geração documental:

- onde a geração acontece;
- como o Client solicita um documento;
- como o Host escolhe e congela a fonte da geração;
- como autorização e consistência são garantidas;
- como renderers futuros compartilham regras de negócio sem compartilhar HTML de tela;
- como o artefato volta ao Client;
- como concorrência e consumo de recursos são limitados;
- o que não vira persistência, fila ou serviço novo.

A Etapa 1 não escolhe biblioteca PDF, biblioteca DOCX, margens, fontes, preview, impressão Windows, limites de texto, quantidade de MACs ou QR/barcode.

## 4. Contratos herdados

Permanecem vigentes:

- PDF, DOCX e impressão são obrigatórios para Procedimentos;
- exportação usa documento próprio, nunca screenshot;
- a fonte de Procedimento é a revisão exata selecionada/autorizada;
- revisão nova não substitui silenciosamente a revisão escolhida;
- ficha pertence ao Atendimento e pode existir com ou sem Equipamento;
- ficha usa somente estado confirmado pelo Host;
- `Em andamento` pode gerar ficha de acompanhamento;
- `Concluído` reimprime estado histórico aplicável;
- `Cancelado` precisa ser identificado inequivocamente;
- ficha ocupa no máximo uma página A4;
- DOCX específico da ficha não é requisito inicial;
- autorização real permanece Host-side;
- Client nunca abre SQLite diretamente;
- evento WebSocket é sinal de mudança, não fonte oficial de estado.

## 5. Fronteira arquitetural consolidada

A geração documental pertence ao **Host**. O Client continua responsável pela experiência local do usuário.

```text
Client
  ↓ solicita geração com identidade da fonte
Host
  ↓ autentica e autoriza
  ↓ valida versão/revisão esperada
  ↓ captura snapshot consistente da fonte
  ↓ materializa DocumentModel semântico
  ↓ encerra leitura/transação SQLite
  ↓ renderiza o formato solicitado
  ↓ devolve o artefato pela conexão autenticada
Client
  ↓ recebe o artefato
  ├─→ salvar localmente
  └─→ preview/impressão quando as etapas correspondentes definirem
```

Consequências:

- Client não gera documento a partir do DOM/HTML da tela;
- Client não replica regras de negócio documentais;
- Host não recebe caminho arbitrário do computador remoto;
- Host não precisa acessar filesystem do usuário;
- não existe compartilhamento SMB obrigatório para exportar;
- geração usa somente dados que o Host pode autorizar e confirmar;
- engines futuras recebem um modelo semântico comum, não conteúdo de UI.

## 6. Requisição por identidade da fonte

O Client não envia o texto completo do documento para o Host gerar.

A solicitação contém conceitualmente:

```text
- tipo de documento;
- formato solicitado;
- identificador estável da fonte;
- revisão/versão esperada quando aplicável;
- contexto mínimo necessário à operação.
```

Para Procedimento, a revisão específica imutável identifica o snapshot documental. Para Atendimento mutável, a requisição preserva a revisão confirmada que o Client está usando.

Exemplo:

```text
Client mostra Atendimento revisão 42
→ solicita ficha para revisão esperada 42
→ Host já está na revisão 43
→ NÃO gerar silenciosamente a 43
→ informar estado obsoleto/conflito
→ Client reconsulta antes de nova tentativa
```

A forma exata de endpoint/payload fica para implementação da API; a semântica de **fonte esperada** está consolidada.

## 7. Captura consistente antes da renderização

A geração possui duas fases:

```text
FASE A — captura
Host abre leitura consistente
→ valida fonte/autorização
→ lê dados necessários
→ resolve snapshots/revisões
→ captura identidade corporativa necessária
→ materializa DocumentModel em memória
→ encerra leitura/transação

FASE B — renderização
DocumentModel já materializado
→ renderer produz artefato
→ nenhuma transação SQLite permanece aberta
```

A renderização não mantém uma transação de banco aberta por todo o tempo de geração. Isso evita segurar snapshot SQLite desnecessariamente, ampliar contenção com o writer e misturar dados de momentos diferentes.

## 8. Fonte por domínio

### Procedimento

A fonte documental é a revisão exata solicitada, já imutável. A identidade corporativa aplicável é capturada no início da geração.

### Atendimento em andamento

A fonte é um snapshot consistente do estado confirmado na revisão esperada, incluindo vínculos necessários ao documento.

### Atendimento concluído

A fonte usa o estado histórico consolidado aplicável, incluindo projeção congelada do Equipamento e revisões exatas dos Procedimentos utilizados.

### Atendimento cancelado

Usa o estado oficial aplicável e preserva identificação inequívoca do cancelamento.

## 9. DocumentModel semântico

Depois da captura, o Host cria uma representação intermediária tipada e imutável para aquela geração.

Conceitualmente:

```text
DocumentModel
├── document_kind
├── source_identity
├── source_version
├── company_identity
├── metadata
├── sections[]
│   └── semantic_blocks[]
└── generation_metadata
```

O modelo é semântico, não layout PDF nem estrutura DOCX específica. Ele pode representar título, metadados, seções, parágrafos, passos/subpassos, checklist documental, notas/alertas, comandos/código e campos estruturados da ficha.

O `DocumentModel` não contém:

- HTML arbitrário;
- JavaScript;
- comandos executáveis;
- DOM da interface;
- classes CSS da aplicação;
- token de sessão;
- caminho escolhido pelo usuário;
- estado transitório do Client.

## 10. Um modelo de conteúdo, renderers separados

```text
fonte oficial
      ↓
DocumentModel semântico
      ├─→ renderer PDF        [Etapa 2]
      └─→ renderer DOCX       [Etapa 3]
```

Regras de negócio como revisão utilizada, equipamento histórico aplicável e existência de campos são resolvidas antes dos renderers. Os renderers não reconsultam o banco nem reconstruem o domínio.

## 11. Geração é leitura derivada

Gerar documento não deve:

- entrar na fila de mutações do writer;
- criar revisão de Procedimento;
- alterar Atendimento;
- marcar checklist;
- alterar `updated_at` funcional;
- emitir evento de domínio apenas porque um arquivo foi gerado;
- criar registro permanente de exportação por padrão.

## 12. Concorrência de renderização

Renderização pode consumir CPU e memória, portanto usa limite próprio de concorrência, separado da fila de mutações.

```text
requisições de geração
        ↓
limite bounded de renderização
        ├─ vaga disponível → gerar
        └─ saturado → backpressure / SERVER_BUSY
```

Regras consolidadas:

- não bloquear o writer esperando renderer;
- não criar fila persistente de exportações;
- não gravar jobs aceitos para executar depois;
- evitar renderização pesada no executor assíncrono principal quando isso puder bloquear o Host;
- quantidade exata de renderizações simultâneas será definida por implementação/medição.

## 13. Sem job persistente na primeira versão

Não criar inicialmente:

- tabela `export_jobs`;
- scheduler documental;
- estado persistente `queued/running/completed`;
- retenção de artefatos no servidor;
- recuperação de exportação após restart;
- histórico de downloads;
- fila offline de geração.

Fluxo inicial:

```text
request autenticado
→ captura consistente
→ renderização
→ resposta com artefato
```

Arquitetura assíncrona só será proposta futuramente se testes reais demonstrarem necessidade.

## 14. Transporte do artefato

O artefato retorna pela API autenticada, como bytes/stream adequado ao formato. O Client decide o destino local conforme as etapas futuras.

O Host não recebe path do computador remoto para escrever diretamente nele.

Se um renderer precisar de arquivo temporário interno durante a requisição, esse detalhe fica em área privada controlada pelo Host, inacessível por path fornecido pelo Client, limitado à vida da operação, removido após uso/falha e fora de backup/histórico funcional.

## 15. Runtime Pocket/autocontido

A geração documental não depende operacionalmente de:

- Microsoft Office instalado;
- automação COM do Office;
- LibreOffice;
- Adobe Reader;
- Chrome/Chromium externo headless;
- `wkhtmltopdf` ou conversor executável separado;
- serviço web/cloud de conversão.

Bibliotecas compiladas com o Host podem ser usadas. Crates/versões específicas ficam para as etapas/implementação correspondentes.

## 16. Responsabilidade do Client

O Client:

- inicia a ação a partir da UX aprovada;
- mantém a sessão autenticada;
- informa fonte/revisão esperada;
- apresenta preparação/erro/cancelamento;
- recebe o artefato;
- abre fluxos locais de salvar/preview/imprimir quando definidos.

O Client não reconstitui autorização, não consulta SQLite, não monta documento final pelo HTML da tela e não escolhe silenciosamente outra revisão.

## 17. Autorização e sessão

A autorização é validada no Host no momento da solicitação.

Procedimento exige conceitualmente capacidade de ler a revisão solicitada + capacidade de exportar/imprimir. Ficha exige capacidade operacional aplicável ao Atendimento acessível.

Na arquitetura síncrona inicial, a requisição é aceita somente com sessão válida e o artefato retorna pela mesma operação autenticada. Não existe URL pública permanente de download.

## 18. Cancelamento, desconexão e falhas

Como geração não altera domínio:

- fechar/cancelar o Client pode abandonar a resposta;
- o Host pode interromper trabalho quando tecnicamente seguro;
- conexão perdida não cria mutação de negócio para reconciliar;
- nova tentativa reconsulta a fonte oficial e usa a revisão atual confirmada.

Classes mínimas de falha:

- não autenticado;
- sem permissão;
- fonte inexistente/indisponível;
- revisão esperada obsoleta;
- fonte inválida para o documento;
- renderer indisponível/falhou;
- limite de recursos excedido;
- Host ocupado/backpressure;
- conexão interrompida.

## 19. Logs, persistência e backup

Log operacional mínimo pode registrar tipo do documento, formato, identificador técnico da fonte, duração e resultado/classe de erro.

Não registrar por padrão conteúdo integral, senha/token, bytes do artefato ou paths locais escolhidos pelo usuário.

Artefatos gerados não fazem parte do estado oficial apenas por terem sido produzidos. Por padrão:

- não entram no SQLite como histórico documental;
- não entram no backup;
- não precisam sobreviver a restart do Host;
- a cópia persistente é responsabilidade do usuário quando ele escolher salvar localmente.

Auditoria funcional permanente de cada exportação não é requisito consolidado nesta etapa.

## 20. Compatibilidade de API

A geração fica sob a API versionada do Host e respeita o handshake Client↔Host existente. Nomes finais de endpoints/payloads não são fechados nesta etapa.

## 21. Decisões consolidadas da Etapa 1

1. geração documental é responsabilidade do Host;
2. Client solicita por IDs/revisão esperada, nunca enviando documento montado;
3. Host captura snapshot consistente e encerra leitura antes de renderizar;
4. Atendimento mutável usa revisão esperada para impedir geração silenciosa de estado diferente;
5. `DocumentModel` semântico é a fronteira comum entre domínio e renderers;
6. renderers não reconsultam banco nem recebem HTML da UI;
7. geração é leitura derivada e não passa pela fila de mutações;
8. renderização tem limite próprio de concorrência/backpressure, sem fila persistente;
9. fluxo inicial é request → render → resposta, sem `export_jobs` persistentes;
10. artefato retorna pela API autenticada; Host não escreve em path arbitrário do Client;
11. runtime não depende de Office/LibreOffice/Adobe/Chrome externo/cloud/conversor auxiliar obrigatório;
12. artefatos gerados não viram histórico/backup por padrão;
13. Client permanece responsável apenas pela UX e pelo destino local do artefato;
14. detalhes de PDF, DOCX, impressão, templates, limites, MACs e QR permanecem nas respectivas etapas.

## 22. Próxima etapa

**Etapa 2 — PDF de Procedimentos** é a próxima etapa do Bloco 10, mas **ainda não está em análise** neste checkpoint.

A Etapa 2 deverá ser aberta em trabalho subsequente, mantendo a regra de uma etapa por vez.