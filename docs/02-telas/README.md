# Telas e Superfícies — StepFlow Pocket

**Status:** TELAS 01–15 CONSOLIDADAS / APROVADAS PELO PO  
**Atualização:** 2026-08-29

Este arquivo é o índice das superfícies. Estado de fase/bloco corrente pertence ao `README.md` e ao plano da Fase 1.

## Especificações

- `01-login.md` — Login;
- `02-shell-sidebar.md` — Shell/sidebar;
- `03-dashboard.md` — Dashboard;
- `04-lista-pesquisa-processos.md` — Processos;
- `05-leitor-processo.md` — Reader + contexto operacional;
- `06-editor-processo.md` — Editor;
- `07-historico-revisoes.md` — Histórico/Revisões;
- `08-lista-pesquisa-atendimentos.md` — Lista de Atendimentos;
- `09-atendimento-execucao-equipamento.md` — Atendimento/Equipamento/checklist/Ficha;
- `10-usuarios-permissoes.md` — Usuários/Permissões;
- `11-meu-perfil.md` — Meu perfil;
- `12-configuracoes-categorias.md` — Configurações/Categorias;
- `13-backup-restauracao.md` — Backup/Restauração UX;
- `14-exportacao-impressao-ficha.md` — Procedimentos + Ficha compacta;
- `15-estados-transversais.md` — estados comuns.

## Direção visual

- visual corporativo, limpo e discreto;
- sidebar esquerda persistente;
- logo pequeno no topo esquerdo;
- sem topbar global redundante;
- perfil/avatar no rodapé;
- leitura técnica como prioridade;
- Procedimentos como manual/livro;
- blocos copiáveis com ícone discreto e feedback curto;
- desktop Windows como alvo inicial;
- adaptação proporcional em janelas menores, sem UI mobile/hamburger inicial;
- baixa densidade textual permanente;
- cor nunca é o único canal de estado importante.

## Domínio operacional refletido nas telas

- `Processos` = documentação/modelos oficiais;
- `Atendimentos` = ocorrências reais;
- `Equipamento` = entidade opcional/reutilizável;
- vínculo do Atendimento preserva revisão exata do Procedimento;
- checklist persiste somente no Atendimento;
- `Observação do serviço` por Etapa é opcional e operacional;
- lifecycle: `Em andamento / Concluído / Cancelado`;
- Ficha compacta segue estado confirmado/histórico aplicável.

## Reader — Tela 05

Standalone:

- checklist documental sem persistência operacional;
- nenhuma observação de serviço persistente.

No Atendimento:

- revisão exata vinculada;
- checklist persistente;
- `Observação do serviço` opcional por Etapa;
- progresso por itens marcados/total;
- 100% não conclui automaticamente;
- stepper permanece navegação, não progresso operacional.

## Atendimento — Telas 08 e 09

- rascunho novo somente em memória;
- primeiro save cria `AT-000001`;
- responsável + `Resumo do trabalho` obrigatórios para concluir;
- checklist incompleto avisa, não bloqueia automaticamente;
- cancelamento/reabertura conforme capacidade;
- Equipamento opcional e separado do save do Atendimento;
- conclusão preserva estado histórico relevante;
- busca operacional separada da busca de Processos.

## Backup / Restauração — Tela 13

UX consolidada:

- dentro de Configurações;
- Host coordena;
- Client não seleciona SQLite/path;
- Restore exige autorização, backup elegível e confirmação reforçada;
- safety backup confirmado antes da etapa destrutiva normal;
- disaster recovery sem Host funcional permanece fluxo técnico/local.

O mecanismo técnico final pertence ao Bloco 11.

## Saída documental — Tela 14

- Procedimento: PDF, DOCX e impressão da revisão selecionada;
- Ficha: PDF canônico + preview + impressão, exatamente uma A4;
- `SHEET_OVERFLOW` bloqueia somente a saída da Ficha;
- sem truncamento, segunda página ou compactação automática;
- soft limits orientativos;
- MACs/dados excepcionais seguem projeção determinística;
- save e temporários seguem o contrato do Bloco 10;
- Word/SMB/impressoras/EDR são gates técnicos de ambiente real, não mudança de UX.

## Capacidades operacionais — resumo

- ADM/Gerência: gestão operacional ampla;
- Funcionário: criar/consultar Atendimento e operar/concluir o próprio;
- Funcionário não cancela/reabre por preset;
- gestão de categorias: ADM/Gerência;
- gerar/reimprimir Ficha: ADM/Gerência/Funcionário para Atendimento acessível;
- autorização real permanece Host-side e granular.

Pendentes de produto: Gerência × configuração da empresa e Gerência × Backup.

## Regra de separação de busca

- `Processos`: código, título, termo, área, categoria e metadados documentais;
- `Atendimentos`: código, OS/ref., cliente, Equipamento, serial, patrimônio, MAC e dados operacionais.

Não misturar os dois domínios em pesquisa global sem requisito explícito.

## Gates técnicos fora da UX

- Windows/WebView2 e fallback Pocket;
- Launcher pelo compartilhamento corporativo;
- Word corporativo;
- impressoras/drivers;
- SMB real;
- EDR/antivírus;
- limites de performance por benchmark;
- mecanismo técnico de Backup/Restore.
