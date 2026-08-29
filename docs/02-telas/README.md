# Telas e Superfícies — StepFlow Pocket

## Estado

**Bloco 8 da Fase 1 — CONCLUÍDO.**  
**Bloco 9 — Execução operacional/Atendimentos + checklist — CONCLUÍDO.**  
**Bloco 10 — Exportação/impressão + Ficha compacta — ETAPAS 1–11 CONSOLIDADAS / gate remoto pendente.**

As Telas **01–15 estão consolidadas/aprovadas**. As Telas 05, 08 e 09 incorporam contexto de execução, lifecycle, checklist/progresso, Status e capacidades operacionais. A Tela 14 incorpora os contratos documentais consolidados do Bloco 10.

## Especificações atuais

- `01-login.md` — Login consolidado;
- `02-shell-sidebar.md` — Shell consolidado;
- `03-dashboard.md` — Dashboard consolidado;
- `04-lista-pesquisa-processos.md` — Lista/Pesquisa de Processos consolidada;
- `05-leitor-processo.md` — Reader consolidado + contexto operacional;
- `06-editor-processo.md` — Editor consolidado;
- `07-historico-revisoes.md` — Histórico/Revisões consolidado;
- `08-lista-pesquisa-atendimentos.md` — Lista de Atendimentos consolidada;
- `09-atendimento-execucao-equipamento.md` — workspace operacional consolidado;
- `10-usuarios-permissoes.md` — Usuários/Permissões consolidado;
- `11-meu-perfil.md` — Meu perfil consolidado;
- `12-configuracoes-categorias.md` — Configurações + Categorias consolidado;
- `13-backup-restauracao.md` — Backup/Restauração — UX consolidado;
- `14-exportacao-impressao-ficha.md` — Exportação/Impressão + Ficha — UX consolidado;
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
- adaptação proporcional em janelas menores, sem UI mobile/hamburger inicial;
- baixa densidade textual permanente;
- cor nunca é o único canal de estado importante.

## Domínio operacional

O StepFlow distingue:

- `Processos` — documentação/modelos oficiais;
- `Atendimentos` — ocorrências reais;
- `Equipamento` — entidade opcional/reutilizável.

Consolidado:

- categorias configuráveis/múltiplas;
- vínculo do Atendimento à revisão exata do Procedimento;
- Ficha com/sem Equipamento;
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
- Concluído/Cancelado são read-only;
- reabertura explícita volta a `Em andamento`.

### Reader — Tela 05

Standalone:

- checklist documental;
- sem persistência operacional.

No Atendimento:

- revisão exata vinculada;
- cabeçalho identifica `AT-...`;
- checklist persistente;
- `Observação do serviço` opcional por Etapa;
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
- cancelamento/reabertura por ADM/Gerência por preset;
- Funcionário opera/conclui o Atendimento do qual é responsável;
- Equipamento opcional e separado do save do Atendimento;
- Funcionário pode criar/editar Equipamento;
- arquivar/reativar Equipamento por ADM/Gerência;
- conclusão congela projeção histórica relevante;
- Ficha segue lifecycle e usa estado confirmado/histórico aplicável.

## Tela 14 — saída documental consolidada

- Procedimento: PDF, DOCX e impressão contextual da revisão selecionada;
- Ficha: PDF canônico + preview + impressão, exatamente uma A4;
- `SHEET_OVERFLOW` bloqueia somente a geração da Ficha;
- sem truncamento, segunda página ou compactação automática;
- soft limits orientativos para textos;
- MACs e dados excepcionais usam projeção determinística;
- save e temporários seguem o contrato do Bloco 10;
- Word/SMB/impressoras/EDR são gates técnicos do ambiente real, não mudança de UX;
- contrato Pocket permanece externo à tela: usuário inicia pelo Launcher no compartilhamento e o Client roda localmente sem instalação manual.

## Capacidades operacionais — resumo

Preset inicial:

- ADM/Gerência: gestão operacional ampla;
- Funcionário: criar/consultar Atendimento e operar/concluir o próprio;
- Funcionário não cancela/reabre por preset;
- gestão de categorias: ADM/Gerência;
- gerar/reimprimir Ficha: ADM/Gerência/Funcionário para Atendimento acessível.

Autorização real permanece Host-side e granular.

Gerência × configuração da empresa e Gerência × Backup continuam pendentes.

## Limites ainda fora das telas

### Ambiente técnico

- validação real de Windows/WebView2 e fallback Pocket;
- Word corporativo;
- impressoras/drivers;
- SMB real;
- EDR/antivírus;
- parâmetros de performance por benchmark.

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
