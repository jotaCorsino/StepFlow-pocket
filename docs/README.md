# Documentação do StepFlow Pocket

Esta pasta contém a documentação ativa do projeto. O `README.md` da raiz é o painel resumido de acompanhamento.

## Precedência

Em caso de divergência:

1. `../AGENTS.md`;
2. `05-progresso/registro-de-decisoes.md`;
3. documento específico vigente;
4. `01-produto/visao-geral.md`;
5. tarefa dentro das decisões vigentes;
6. histórico Git.

Proposta não aprovada não pode ser implementada como decisão.

## Leitura eficiente

Sempre: `AGENTS.md`, enunciado da tarefa, este índice e documentos específicos indicados.

Conforme impacto: registro de decisões, plano da Fase 1, arquitetura vigente, contexto de ambientes e documento técnico relacionado.

## Índice vigente

### Governança — `00-governanca`

- `contexto-ambientes.md` — ambientes;
- `metodo-padrao-trabalho-assistido.md` — processo PO + Assistente + Codex;
- `politica-capacidade-codex.md` — seleção de capacidade.

### Produto — `01-produto`

- `visao-geral.md` — propósito/requisitos/limites;
- `categorizacao-atendimentos-equipamentos.md` — domínio `Procedimento × Atendimento × Equipamento`.

### Telas — `02-telas`

- `README.md` — mapa do Bloco 8;
- `01-login.md` — Login;
- `02-shell-sidebar.md` — Shell;
- `03-dashboard.md` — Dashboard;
- `04-lista-pesquisa-processos.md` — Processos;
- `05-leitor-processo.md` — Reader, stepper, checklist e observação de serviço por Etapa;
- `06-editor-processo.md` — Editor;
- `07-historico-revisoes.md` — Histórico/Revisões;
- `08-lista-pesquisa-atendimentos.md` — Lista de Atendimentos;
- `09-atendimento-execucao-equipamento.md` — Atendimento/Equipamento/checklist/observações/Ficha;
- `10-usuarios-permissoes.md` — Usuários/Permissões;
- `11-meu-perfil.md` — Meu perfil;
- `12-configuracoes-categorias.md` — Configurações/Categorias;
- `13-backup-restauracao.md` — Backup/Restauração UX;
- `14-exportacao-impressao-ficha.md` — Procedimentos + Ficha compacta/PDF/preview/template A4/limites/dados excepcionais/naming/temporários;
- `15-estados-transversais.md` — Estados transversais.

### Arquitetura — `03-arquitetura`

- `arquitetura-vigente.md` — visão consolidada, incluindo Bloco 10 / Etapas 1–8;
- `modelo-dados-schema-fase-1.md` — schema conceitual, checklist, observações por Etapa e snapshots;
- `concorrencia-fila-conflitos-eventos.md` — concorrência geral;
- `implantacao-pocket.md` — implantação/ciclo central;
- `compatibilidade-windows-client.md` — Windows/WebView2;
- `host-pocket.md` — Controller/Host;
- `launcher-distribuicao-client.md` — distribuição;
- `comunicacao-client-host.md` — HTTP/JSON/WebSocket;
- `autenticacao-sessao-autorizacao.md` — autenticação/autorização.

### Planejamento — `04-planejamento`

- `roadmap.md` — fases;
- `plano-oficial-fase-1.md` — estado/gates/pendências;
- `bloco-9-atendimentos-execucao-checklist.md` — contrato operacional do Bloco 9;
- `bloco-10-exportacao-impressao-ficha.md` — mapa técnico do Bloco 10; **Etapas 1–10 consolidadas, Etapa 11 — validação técnica final próxima e ainda não aberta**;
- `tarefas-codex/README.md` — somente tarefas Codex ativas.

### Progresso — `05-progresso`

- `registro-de-decisoes.md` — decisões vigentes/pendências/gates;
- `changelog-projeto.md` — marcos;
- `diario-de-progresso.md` — histórico, não fonte superior;
- `revisao-cruzada-fase-0.md` — evidência histórica.

### Templates — `templates`

- `template-analise-de-tela.md`;
- `template-preflight-capacidade-codex.md`;
- `template-tarefa-codex.md`.

## Estado atual

**Fase 1 em andamento. Blocos 8 e 9 concluídos. Bloco 10 está em andamento com as Etapas 1–10 consolidadas. Etapa 11 — Validação técnica final é a próxima, ainda não aberta.**

Direção consolidada:

- baixa densidade textual como princípio transversal;
- Reader em formato livro/manual com `Visão geral`, uma página lógica por Etapa e stepper compacto;
- Reader operacional persiste checklist e observação de serviço opcional por Etapa;
- geração documental no Host por `DocumentModel` e snapshot consistente;
- PDF de Procedimentos via Typst e DOCX OOXML via pipeline Rust próprio;
- impressão Windows local pelo PDF oficial e WebView2;
- Procedimento físico A4 retrato multipágina;
- Ficha compacta = prestação de contas resumida ao cliente;
- PDF da Ficha + preview SVG saem do mesmo `PagedDocument`;
- Ficha válida possui exatamente uma A4;
- template físico da Ficha usa A4 retrato, margens 15 mm, cabeçalho compacto, ficha técnica sem grade, `SERVIÇO REALIZADO` e `OBSERVAÇÕES` como seções centrais;
- Noto Sans com baseline 14 / 10,5 / 10 / 9 / 8,5 pt;
- soft limits orientativos: Resumo 600, Atendimento 400, Equipamento 300 e observação por Etapa 280 caracteres;
- `SHEET_OVERFLOW` bloqueia somente a geração da Ficha e nunca destrói/trunca o dado operacional;
- correção de overflow ocorre nos campos reais, sem editor paralelo ou compactação automática;
- Procedimentos vinculados não são listados na Ficha por padrão;
- MACs usam projeção compacta: 1–2 valores, 3+ somente quantidade cadastrada;
- observações legítimas não sofrem cap/descarte automático; multiplicidade pode causar overflow real;
- nomes persistentes são previsíveis e sanitizados para Windows; Ficha usa somente o código do Atendimento por padrão;
- temporários pertencem ao Client, usam nomes opacos sem dados de negócio e só são materializados quando uma integração local exigir filesystem;
- cleanup é best-effort e não usa serviço, daemon ou tarefa agendada;
- arquivo salvo pelo usuário não entra no lifecycle de cleanup do StepFlow;
- sem assinatura, financeiro, checklist, timeline ou página 2 por padrão.

## Gate antes da Etapa 11

```text
squash merge da limpeza documental atual
→ remoção da branch remota
→ remoto somente com main
→ zero PRs abertos
```

## Pendências vigentes

- Etapa 11: validação técnica final;
- parâmetros finais de autenticação;
- Gerência × configuração da empresa;
- Gerência × Backup;
- regra editorial de categoria arquivada;
- mecanismo técnico do Bloco 11;
- validações reais do ambiente corporativo.
