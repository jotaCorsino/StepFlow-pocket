# Roadmap — StepFlow Pocket

**Status:** FASE 1 EM ANDAMENTO  
**Atualização:** 2026-08-29

## Fase 0 — Fundação documental e governança

**CONCLUÍDA em 2026-08-19.**

Fonte de verdade, governança, visão de produto, arquitetura inicial, roadmap e templates estabelecidos.

## Fase 1 — Fechamento arquitetural e especificação

**EM ANDAMENTO.**

Já consolidados:

- Client/Tauri e Windows;
- Host Pocket sob demanda;
- launcher/update;
- HTTP/JSON + WebSocket;
- autenticação/sessão/autorização no núcleo;
- modelo de dados/migrations/histórico conceitual;
- concorrência/fila/conflitos/eventos;
- `Procedimento × Atendimento/Execução × Equipamento`;
- categorias configuráveis/múltiplas;
- UI/UX completa das Telas 01–15;
- lifecycle operacional de Atendimentos;
- checklist persistente em contexto de execução;
- matriz operacional de capacidades;
- códigos `AT-000001` / `EQP-000001`;
- gestão de categorias por ADM/Gerência;
- lifecycle/capacidade da ficha;
- Bloco 10 / Etapas 1–10 de geração, exportação, impressão, Ficha, densidade, dados excepcionais, naming e temporários.

### Bloco 8 — UI/UX

**CONCLUÍDO.**

Telas 01–15 consolidadas, incluindo estados transversais.

### Bloco 9 — Execução operacional / Atendimentos + checklist

**CONCLUÍDO.**

Consolidado:

- `Em andamento`, `Concluído`, `Cancelado`;
- primeiro save cria Atendimento;
- responsabilidade por técnico;
- conclusão/reabertura/cancelamento;
- checklist persistente apenas em Atendimento;
- progresso por checklist;
- vínculo com revisão exata;
- snapshot histórico de Equipamento na conclusão;
- matriz operacional;
- códigos legíveis;
- gestão de categorias;
- lifecycle da ficha.

### Bloco 10 — Exportação / impressão + ficha compacta

**EM ANDAMENTO — ETAPAS 1–10 CONSOLIDADAS.**

Consolidado:

- arquitetura de geração documental no Host;
- PDF de Procedimentos via Typst embutido;
- DOCX OOXML gerado diretamente em Rust;
- impressão Windows pelo PDF oficial e WebView2;
- template físico multipágina de Procedimentos;
- PDF + preview da Ficha compacta;
- template físico A4 de uma página da Ficha;
- limites textuais orientativos e `SHEET_OVERFLOW`;
- projeção de dados multiplicativos/excepcionais;
- nomes persistentes e lifecycle de artefatos temporários no Client.

**Próxima e última etapa do Bloco 10: Etapa 11 — Validação técnica final, ainda não aberta para análise.**

Depois:

1. Bloco 11 — Backup/Restore técnico;
2. Bloco 12 — estrutura oficial, parâmetros finais e plano da Fase 2.

## Fase 2 — Fundação técnica executável

**PENDENTE.**

- árvore oficial Client/launcher/Controller/Host;
- builds reproduzíveis;
- configuração de desenvolvimento;
- comunicação mínima + health/readiness;
- SQLite + migrations iniciais;
- logging mínimo;
- testes de fundação.

Gate: Client abre, Host inicia sob demanda, comunicação mínima funciona, banco inicializa deterministicamente e build limpo passa.

## Fase 3 — Autenticação, usuários e shell

**PENDENTE.**

- login/logout/sessão;
- bootstrap ADM;
- usuários/permissões;
- perfil/avatar;
- shell/sidebar;
- configuração básica da empresa;
- autorização Host-side.

## Fase 4 — Núcleo documental de Procedimentos

**PENDENTE.**

- lista/pesquisa;
- categorização;
- criação/edição/arquivamento;
- etapas/blocos;
- histórico/revisões;
- permissões;
- conflitos de revisão.

## Fase 5 — Experiência de execução e registro operacional

**PENDENTE.**

Implementará os contratos já fechados:

- Reader em páginas/etapas;
- navegação;
- passos/alertas/blocos copiáveis;
- contexto de execução ligado ao Atendimento;
- checklist persistente;
- progresso por checklist;
- Atendimentos;
- lifecycle de três estados;
- Equipamento opcional;
- busca/lista operacional;
- resumo do trabalho;
- revisão exata utilizada;
- snapshot histórico de Equipamento;
- ficha compacta conforme Blocos 8–10;
- estados transversais.

## Fase 6 — Multiusuário em ambiente real

**PENDENTE.**

- múltiplos Clients;
- conflitos/fila;
- concorrência granular de checklist;
- eventos/reconexão;
- Host indisponível;
- concorrência operacional;
- validação LAN corporativa.

## Fase 7 — Exportação e identidade

**PENDENTE.**

Implementará os contratos do Bloco 10:

- geração Host-side;
- PDF de Procedimentos;
- DOCX de Procedimentos;
- impressão de Procedimentos;
- identidade central da empresa;
- revisão selecionada preservada;
- ficha compacta com/sem Equipamento;
- impressão da ficha;
- reprodução do snapshot histórico de Atendimento/Equipamento;
- validação em leitores/impressoras.

DOCX específico da ficha não é requisito inicial.

## Fase 8 — Distribuição Pocket, backup e operação

**PENDENTE.**

- pacote central por pasta;
- Controller/Host sob demanda;
- launcher em rede + Client local versionado;
- backup/restore de banco + arquivos administrados;
- safety backup antes do Restore normal;
- disaster recovery local quando Host não inicia;
- logs;
- documentação de implantação;
- validação sem Internet e em PCs corporativos.

Não inclui serviço StepFlow persistente.

Cenário final conceitual:

```text
Controller iniciado quando StepFlow será usado
→ launcher prepara Client local
→ login/uso multiusuário
→ consulta Procedimento
→ registra/executa Atendimento quando necessário
→ checklist/progresso por revisão vinculada
→ conclusão/cancelamento/reabertura conforme capacidade
→ impressão/exportação quando necessária
→ encerramento central fecha Host/Controller
→ zero processo StepFlow residual
```

## Fase 9 — Hardening e release interno

**PENDENTE.**

- segurança/autorização;
- recuperação de falha/banco;
- backup/restore;
- concorrência/performance;
- logs;
- distribuição/update;
- smoke tests end-to-end;
- revisão documental.

## Pendências transversais antes da implementação correspondente

- parâmetros finais Argon2/senha/sessão;
- Gerência × configuração da empresa;
- Gerência × Backup;
- regra editorial de nova revisão ainda referenciando categoria arquivada;
- validações do ambiente corporativo real.

## Regra do roadmap

Fases dependem de gates, não de cronograma. Mudança de requisito atualiza documentação antes da implementação. Proposta só vira contrato após aprovação explícita.
