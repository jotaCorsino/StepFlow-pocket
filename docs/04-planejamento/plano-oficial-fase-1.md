# Plano Oficial — Fase 1: Fechamento Arquitetural e Especificação

**Status:** EM ANDAMENTO — VALIDAÇÃO FINAL DO BLOCO 12  
**Início:** 2026-08-19  
**Atualização:** 2026-09-01

## Objetivo

Transformar requisitos e arquitetura conceitual em decisões implementáveis antes da fundação executável do StepFlow.

A Fase 1 autoriza documentação, decisões técnicas e PoCs descartáveis quando necessárias. Não autoriza scaffold/runtime oficial, migration oficial ou código de negócio antes do gate final do Bloco 12/Fase 2.

## Estado dos blocos

| Bloco | Tema | Estado | Fonte principal |
|---|---|---|---|
| 0 | Bootstrap do ambiente | CONCLUÍDO | contexto/repositório |
| 1 | Client Windows/Tauri | CONCLUÍDO | `../03-arquitetura/compatibilidade-windows-client.md` |
| 2 | Host Pocket | CONCLUÍDO | `../03-arquitetura/host-pocket.md` |
| 3 | Launcher/distribuição | CONCLUÍDO | `../03-arquitetura/launcher-distribuicao-client.md` |
| 4 | Comunicação Client↔Host | CONCLUÍDO | `../03-arquitetura/comunicacao-client-host.md` |
| 5 | Autenticação/autorização | CONSOLIDADO | `../03-arquitetura/autenticacao-sessao-autorizacao.md` |
| 6 | Dados/schema/migrations | CONSOLIDADO | `../03-arquitetura/modelo-dados-schema-fase-1.md` |
| 7 | Concorrência/eventos | CONCLUÍDO | `../03-arquitetura/concorrencia-fila-conflitos-eventos.md` |
| 8 | UI/UX | CONCLUÍDO | `../02-telas/README.md` |
| 9 | Execução operacional | CONCLUÍDO | `bloco-9-atendimentos-execucao-checklist.md` |
| 10 | Exportação/impressão/Ficha | CONCLUÍDO | `bloco-10-exportacao-impressao-ficha.md` |
| 11 | Backup/restauração | CONCLUÍDO | `bloco-11-backup-restauracao.md` |
| 12 | Estrutura oficial + Fase 2 | VALIDAÇÃO FINAL — D12.1–D12.98 APROVADAS | `bloco-12-estrutura-oficial-plano-fase-2.md` |

## Bloco 12

### Aprovado — D12.1–D12.98

- estrutura `apps/` + `crates/` e publicação `StepFlow.exe + _internal/`;
- Rust 1.98.0, Edition 2024, resolver 3, Windows x64 MSVC;
- toolchain/lockfile versionados e dependências lockfile-aware;
- Client vanilla sem Node/bundler;
- migrations Host-side imutáveis/embutidas + checksum/testes;
- parâmetros finais de autenticação/sessão, Empresa/Categorias, Backup/Restore, comunicação e logs;
- plano F2-T01…F2-T08, uma branch/PR por tarefa e pré-flight separado.

### Validação final — P12.99–P12.108 em revisão

A proposta final fecha:

- semântica de configuração inválida de retenção;
- ownership único dos parâmetros;
- materialização explícita de `deployment.json` real;
- sincronização local segura por fast-forward somente quando o working tree estiver limpo;
- classificação de gates corporativos;
- autorização explícita de F2-T01 depois do merge/remoto limpo/sync local.

Fonte: `bloco-12-analise-6-validacao-final-fase-1.md`.

## Gates corporativos reservados

Permanecem para as fases técnicas correspondentes:

- Windows/WebView2 real e fallback Pocket quando necessário;
- Launcher pelo share SMB;
- Word/impressoras;
- SMB/permissões/falhas;
- filesystem/ACL/EDR/antivírus/long paths;
- adapter Win32 e crash injection de Backup/Restore;
- transporte corporativo aplicável.

Eles não bloqueiam o encerramento documental da Fase 1, mas podem bloquear a saída da etapa/produção quando forem requisito aplicável.

## Gate atual

1. decidir P12.99–P12.108;
2. consolidar documentação final;
3. tornar PR #27 ready e revisar head/checks;
4. squash merge em `main`;
5. remover branch remota e verificar remoto limpo;
6. inspecionar/sincronizar `C:\dev\StepFlow` sem perder alterações locais;
7. PO autorizar F2-T01;
8. somente então preparar pré-flight + prompt Codex e abrir `feat/f2-01-workspace-host`.

## Regras finais

- nenhuma pendência vira decisão por inferência;
- requisito Pocket não pode ser enfraquecido para acomodar dependência;
- toda tarefa Codex informa base Git e preserva working tree preexistente;
- documento técnico estável não carrega gate histórico consumido.
