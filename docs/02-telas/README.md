# Telas e Superfícies — StepFlow Pocket

## Estado

**Bloco 8 da Fase 1 — CONCLUÍDO.**  
**Bloco 9 — Execução operacional/Atendimentos + checklist — CONCLUÍDO.**

As Telas **01–15 estão consolidadas/aprovadas**. As Telas 05, 08 e 09 foram atualizadas no Bloco 9 para incorporar contexto de execução, lifecycle, checklist/progresso, Status e capacidades operacionais.

Bloco 10 está em andamento, com Etapas 1–10 consolidadas e a validação técnica final como próxima etapa.

## Especificações atuais

- `01-login.md` — Login consolidado;
- `02-shell-sidebar.md` — Shell consolidado;
- `03-dashboard.md` — Dashboard consolidado;
- `04-lista-pesquisa-processos.md` — Lista/Pesquisa de Processos consolidada;
- `05-leitor-processo.md` — Reader consolidado + contexto operacional do Bloco 9;
- `06-editor-processo.md` — Editor consolidado;
- `07-historico-revisoes.md` — Histórico/Revisões consolidado;
- `08-lista-pesquisa-atendimentos.md` — Lista de Atendimentos consolidada + lifecycle do Bloco 9;
- `09-atendimento-execucao-equipamento.md` — workspace operacional consolidado + lifecycle/checklist do Bloco 9;
- `10-usuarios-permissoes.md` — Usuários/Permissões consolidado;
- `11-meu-perfil.md` — Meu perfil consolidado;
- `12-configuracoes-categorias.md` — Configurações + Categorias consolidado;
- `13-backup-restauracao.md` — Backup/Restauração — UX consolidado;
- `14-exportacao-impressao-ficha.md` — Exportação/Impressão + ficha — UX consolidado;
- `15-estados-transversais.md` — Estados transversais consolidado.

## Direção visual

- visual corporativo, limpo e discreto;
- sidebar esquerda persistente;
- logo pequeno no topo esquerdo;
- sem topbar global redundante;
- perfil/avatar no rodapé;
- leitura técnica como prioridade;
- Procedimentos como manual/livro;
- blocos copiáveis com ícone discreto;
- feedback curto de cópia;
- desktop Windows como alvo inicial;
- adaptação proporcional em janelas menores, sem UI mobile/hamburger inicial.

## Domínio operacional

O StepFlow distingue:

- `Processos` — documentação/modelos oficiais;
- `Atendimentos` — ocorrências reais;
- `Equipamento` — entidade opcional/reutilizável.

Consolidado:

- categorias configuráveis/múltiplas;
- vínculo do Atendimento à revisão exata do Procedimento;
- ficha com/sem Equipamento;
- tipos mínimos de computador Servidor/Desktop/Notebook;
- bateria contextual;
- identidade central da empresa;
- safety backup antes do Restore normal;
- exportação baseada na revisão selecionada;
- estados transversais comuns.

## Atualização operacional do Bloco 9

### Lifecycle

```text
Em andamento
Concluído
Cancelado
```

- primeiro save cria Atendimento;
- cancelamento exige motivo;
- concluído/cancelado são read-only;
- reabertura explícita volta a `Em andamento`.

### Reader — Tela 05

Standalone:

- checklist documental;
- sem persistência operacional.

No Atendimento:

- revisão exata vinculada;
- cabeçalho identifica `AT-...`;
- checklist persistente;
- progresso por itens marcados/total;
- 100% não conclui automaticamente.

### Lista de Atendimentos — Tela 08

- Status visível: Em andamento/Concluído/Cancelado;
- filtros: Responsável + Status + Período;
- Data/Período usam `started_at`;
- mais recente primeiro;
- busca por AT/OS/cliente/Equipamento/serial/patrimônio/MAC.

### Atendimento — Tela 09

- rascunho novo somente em memória;
- código `AT-000001` gerado no primeiro save;
- responsável + Resumo obrigatórios para concluir;
- checklist incompleto avisa, não bloqueia automaticamente;
- cancelamento por ADM/Gerência por preset;
- reabertura por ADM/Gerência por preset;
- Funcionário opera/conclui o Atendimento do qual é responsável;
- Equipamento opcional e separado do save do Atendimento;
- Funcionário pode criar/editar Equipamento;
- arquivar/reativar Equipamento por ADM/Gerência;
- conclusão congela projeção histórica relevante do Equipamento;
- ficha segue lifecycle e usa estado confirmado.

## Capacidades operacionais — resumo

Preset inicial:

- ADM/Gerência: gestão operacional ampla;
- Funcionário: criar/consultar Atendimento e operar/concluir o próprio;
- Funcionário não cancela/reabre por preset;
- gestão de categorias: ADM/Gerência;
- gerar/reimprimir ficha: ADM/Gerência/Funcionário para Atendimento acessível.

Autorização real permanece Host-side e granular.

Gerência × configuração da empresa e Gerência × Backup continuam pendentes.

## Limites ainda fora das telas

### Bloco 10

- engines PDF/DOCX/impressão;
- validação técnica final de templates, filesystem, Windows/WebView2, limites e falhas reais.

### Bloco 11

- mecanismo técnico de Backup/Restore;
- pacote/atomicidade/retenção;
- restart/reconexão/sessões;
- disaster recovery.

### Antes da implementação editorial

- regra de nova revisão de Procedimento ainda referenciando categoria arquivada.

## Regra de separação de busca

- `Processos`: código, título, termo, área, categoria e metadados documentais;
- `Atendimentos`: código, OS/ref., cliente, Equipamento, serial, patrimônio, MAC e dados operacionais.

Não misturar os dois domínios em pesquisa global sem requisito explícito.

## Regra de acompanhamento

Todo avanço consolidado de fase, bloco ou tela deve atualizar o README raiz no mesmo checkpoint documental.
