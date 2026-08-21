# AGENTS.md — StepFlow Pocket

## Finalidade

Regras obrigatórias para Codex e outros agentes que atuem neste repositório.

## Fonte de verdade e fase atual

- GitHub é a fonte principal de verdade.
- Branch principal: `main`.
- Checkout local previsto: `C:\dev\StepFlow`.
- Desenvolvimento atual: computador pessoal fora da LAN corporativa.
- Fase vigente: **Fase 1 — Fechamento arquitetural e especificação**.
- Blocos 0–7 estão fechados no núcleo arquitetural; Bloco 8 (UI/UX) está em andamento.
- Novo requisito confirmado em 2026-08-21: categorização de procedimentos, registro de informações de serviço/equipamento, busca operacional e ficha compacta imprimível.
- A modelagem específica `Procedimento × Atendimento/Execução × Equipamento` está **EM PROPOSTA** e não pode ser implementada como decisão final antes da aprovação do PO.

## Precedência e autoridade da tarefa

O enunciado da tarefa define **o trabalho autorizado**, mas não revoga silenciosamente decisões vigentes.

Em caso de conflito:

1. `AGENTS.md`;
2. `docs/05-progresso/registro-de-decisoes.md`;
3. documento específico vigente;
4. `docs/01-produto/visao-geral.md`;
5. enunciado da tarefa, dentro das decisões vigentes;
6. histórico.

Se o enunciado contrariar decisão consolidada, só prosseguir quando declarar explicitamente nova decisão aprovada pelo PO e atualizar os documentos afetados. Ambiguidade nunca autoriza escolher a alternativa mais conveniente.

## Leitura do Codex por camadas

### Sempre

1. `AGENTS.md`;
2. enunciado da tarefa;
3. `docs/README.md`;
4. documentos específicos indicados.

### Quando houver impacto correspondente

- `docs/05-progresso/registro-de-decisoes.md`;
- `docs/04-planejamento/plano-oficial-fase-1.md`;
- `docs/03-arquitetura/arquitetura-vigente.md`;
- `docs/00-governanca/contexto-ambientes.md`;
- `docs/01-produto/categorizacao-atendimentos-equipamentos.md` para categorias/serviços/equipamentos/ficha compacta;
- demais documentos técnicos específicos.

`metodo-padrao-trabalho-assistido.md` e `politica-capacidade-codex.md` orientam principalmente PO/Assistente e não precisam ser relidos pelo Codex em toda tarefa.

## Papéis

- **PO:** define produto, prioridade, comportamento e aprovação visual/funcional.
- **Assistente:** analisa, arquiteta, documenta e transforma decisões aprovadas em tarefas.
- **Codex:** executa tecnicamente o escopo recebido, sem inventar produto ou ampliar tarefa.

## Pré-flight de capacidade

A seleção de modelo/raciocínio é responsabilidade do Assistente + PO antes do envio da tarefa ao Codex.

## Base Git obrigatória

Toda tarefa que permita alteração deve informar branch/base esperada e commit SHA esperado.

Antes de escrever:

```text
git rev-parse HEAD
git status --short --branch
```

Se `HEAD` divergir, não fazer `pull`, `merge`, `rebase`, `reset` ou checkout corretivo automaticamente. Parar e reportar.

## Proteção absoluta do working tree

Qualquer alteração preexistente pertence ao PO/outro fluxo.

Sem autorização explícita e específica do PO, é proibido:

- `git reset --hard`;
- `git clean`;
- `git stash`;
- descartar/restaurar alterações locais;
- sobrescrever arquivo modificado preexistente;
- trocar branch descartando trabalho;
- incluir alteração preexistente no commit da tarefa.

Se arquivo necessário já estiver modificado, parar e reportar.

## Regras operacionais

- uma tarefa por vez;
- não declarar parcial como concluído;
- não criar funcionalidade/dependência/estrutura relevante fora do escopo;
- não alterar UX/visual aprovado sem autorização;
- não transformar proposta, exemplo ou parâmetro provisório em decisão;
- manter documentação e implementação sincronizadas;
- preservar modularidade e baixo acoplamento;
- não versionar credenciais, segredos, banco real ou dados pessoais da empresa;
- exemplos de IP/hostname/share/path nunca viram configuração;
- testes de LAN corporativa fora dela são `NÃO APLICÁVEIS NESTE AMBIENTE`;
- protótipos descartáveis não viram produção silenciosamente.

## Ambiente Codex versus sessão normal do PO

Limitação do sandbox não vira requisito do produto. Codex não repara o próprio ambiente alterando ACL, Schannel, registro/PATH global, segurança ou reinstalando ferramentas válidas.

Operações que exijam credenciais, Internet confiável, elevação ou configuração global são reportadas para sessão Windows normal do PO.

## Regras Pocket obrigatórias

- implantação central por pasta pronta;
- nenhuma toolchain de desenvolvimento no servidor de produção;
- sem Windows Service persistente, auto-start, Task Scheduler, watchdog, tray agent ou daemon como padrão;
- Host/Controller sob demanda;
- Controller aberto representa ciclo central ativo; encerrado o ciclo, nenhum processo StepFlow permanece ativo;
- fechar Client não encerra Host;
- não inventar auto-shutdown por ausência de Clients/timeout;
- Client roda localmente, preparado por launcher transitório;
- launcher encerra após iniciar Client;
- dados/config/logs separados dos binários substituíveis.

## Regras técnicas consolidadas

- Client: Tauri 2 + HTML/CSS/JavaScript modular;
- Host: Rust + Tokio/Axum + `rusqlite`/SQLite bundled;
- HTTP/JSON + WebSocket;
- SQLite somente pelo Host;
- writer coordenado + fila bounded + revisão otimista;
- nenhuma sobrescrita silenciosa;
- sessão opaca e autorização Host-side;
- Argon2id, com parâmetros finais ainda pendentes;
- procedimentos usam revisões imutáveis;
- PDF, DOCX e impressão são requisitos da documentação;
- requisito novo exige categorização, registro de dados de serviço/equipamento, busca operacional, resumo do trabalho e ficha compacta imprimível.

## Regras específicas do novo requisito

Confirmado:

- o produto não é exclusivo de manutenção de PCs;
- precisa organizar procedimentos por categoria;
- manutenção de computador/notebook precisa registrar os campos solicitados pelo PO quando aplicáveis;
- busca deve aproveitar cliente/OS/identificadores úteis;
- precisa haver resumo do que foi realizado;
- precisa existir saída compacta para impressão física.

Ainda **NÃO consolidado**:

- categorias múltiplas versus únicas/hierárquicas;
- entidade reutilizável de equipamento;
- entidade formal chamada `Atendimento`/`Execução`;
- identificador interno/código StepFlow como chave operacional principal;
- múltiplos procedimentos por atendimento;
- vínculo obrigatório à revisão exata;
- `Atendimentos` como item de sidebar;
- lifecycle, checklist/progresso e matriz de permissões;
- formato/PDF/QR/barcode da ficha compacta.

Nenhum desses itens pendentes pode ser implementado por suposição.

## Tarefa Codex

Toda tarefa declara objetivo, base Git, fonte de verdade, escopo, fora do escopo, critérios de aceite, validações e documentação impactada.

O relatório final informa base/estado inicial, arquivos alterados, decisões técnicas dentro do escopo, validações, resultados, riscos/pendências e próximos passos sugeridos.

## Gate de implementação da Fase 1

Na Fase 1, trabalho estrutural significa documentação, organização documental ou PoC explicitamente descartável autorizada.

Não criar scaffold oficial, módulos runtime definitivos, árvore final ou código de negócio antes do Bloco 12/Fase 2 autorizar.

Se a fase/documentação não autorizar implementação definitiva, limitar-se ao trabalho documental/investigativo solicitado.
