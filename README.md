# StepFlow Pocket

Aplicação interna para documentar, consultar, versionar e executar procedimentos técnicos de forma guiada, com registro operacional de Atendimentos e Equipamentos quando aplicável.

## Painel de acompanhamento

**Atualização:** 2026-08-29  
**Fase atual:** Fase 1 — Fechamento arquitetural e especificação  
**Checkpoint atual:** Bloco 11 — Backup / restauração técnico em análise  
**Próximo bloco após consolidação:** Bloco 12 — Estrutura oficial + plano da Fase 2  
**Implementação funcional oficial:** ainda não iniciada

### Fase 1

| Bloco | Tema | Estado |
|---|---|---|
| 0 | Bootstrap do ambiente | ✅ Concluído |
| 1 | Client Windows / Tauri | ✅ Concluído |
| 2 | Host Pocket | ✅ Concluído |
| 3 | Launcher / distribuição | ✅ Concluído |
| 4 | Comunicação Client ↔ Host | ✅ Concluído |
| 5 | Autenticação / autorização | ✅ Núcleo concluído; parâmetros finais pendentes |
| 6 | Dados / schema / migrations | ✅ Núcleo + extensão operacional conceitual consolidados |
| 7 | Concorrência / fila / eventos | ✅ Núcleo concluído |
| 8 | UI/UX | ✅ Concluído |
| 9 | Atendimentos / execução / checklist | ✅ Concluído |
| 10 | Exportação / impressão / Ficha compacta | ✅ Concluído |
| 11 | Backup / restauração | 🟡 Em análise |
| 12 | Estrutura oficial + plano da Fase 2 | ⏳ Pendente |

## Produto

O domínio separa:

```text
Procedimento
   ↓ usado em
Atendimento / Execução
   ↓ opcionalmente relacionado a
Equipamento
```

Princípios centrais:

- Procedimentos são documentação oficial versionada;
- Atendimentos registram ocorrências reais de trabalho;
- Equipamento é opcional e reutilizável;
- Reader usa experiência de manual/livro, com uma Etapa por página lógica;
- checklist persistente existe somente no contexto de Atendimento;
- `Observação do serviço` por Etapa é opcional e operacional;
- Ficha compacta é prestação de contas resumida ao cliente;
- UI privilegia clareza e baixa densidade textual.

## Contrato Pocket

O StepFlow deve ser publicado como **pasta pronta em um servidor Windows** e usado pelas estações autorizadas sem instalação individual do aplicativo.

```text
pasta publicada no servidor
→ usuário acessa o compartilhamento
→ executa StepFlowLauncher.exe
→ Launcher prepara/valida o Client em %LOCALAPPDATA%
→ Client abre localmente
```

É obrigatório no uso normal:

- zero instalador tradicional por estação;
- zero preparação manual de dependências;
- zero elevação administrativa;
- nenhuma toolchain de desenvolvimento em produção;
- nenhuma Internet obrigatória;
- Client operacional local, não executado permanentemente pelo SMB.

O Controller/Host continua sob demanda na máquina central. WebView2 não pode enfraquecer o contrato Pocket; detalhes ficam nos documentos de implantação, Launcher e compatibilidade Windows.

## Fontes de verdade

- `AGENTS.md` — governança e regras de execução;
- `docs/README.md` — índice e ownership documental;
- `docs/05-progresso/registro-de-decisoes.md` — decisões vigentes e pendências;
- `docs/04-planejamento/plano-oficial-fase-1.md` — estado da Fase 1;
- `docs/04-planejamento/roadmap.md` — fases do projeto;
- documentos específicos de Produto, Telas e Arquitetura — contratos detalhados.

## Pendências principais da Fase 1

- parâmetros finais de autenticação/sessão;
- Gerência × configuração da empresa;
- Gerência × Backup;
- regra editorial de categoria arquivada;
- Bloco 11 — fechar o mecanismo técnico de Backup/Restore atualmente em análise;
- Bloco 12 — estrutura oficial, parâmetros finais e plano da Fase 2;
- validações corporativas de Windows/WebView2/Launcher/SMB/Word/impressoras/EDR no momento apropriado.

## Regra deste painel

O README mostra somente o estado executivo. Decisões detalhadas não devem ser duplicadas aqui; o painel aponta para a fonte específica vigente.
