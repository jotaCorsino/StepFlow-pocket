# Bloco 11 — Backup / Restauração técnico

**Status:** PROPOSTA / EM ANÁLISE — NÃO CONSOLIDADO  
**Fase:** 1 — Fechamento arquitetural e especificação  
**Abertura:** 2026-08-29

## Objetivo

Fechar o contrato técnico de Backup/Restore antes da estrutura oficial e da Fase 2, sem criar implementação funcional nesta etapa.

## Contratos já consolidados

A UX permanece definida em `../02-telas/13-backup-restauracao.md`.

Este bloco não reabre, sem bloqueador técnico concreto:

- Backup/Restore coordenado pelo Host;
- Client sem acesso direto ao SQLite;
- autorização Host-side;
- Restore normal somente com backup elegível e confirmação reforçada;
- safety backup obrigatório antes da etapa destrutiva do Restore normal;
- falha do safety backup bloqueia o Restore normal;
- recuperação sem Host funcional pertence a disaster recovery local/controlado;
- sucesso somente após confirmação do Host;
- Backup separado de exportação documental;
- ausência de scheduler periódico por inferência;
- Restore de Gerência não autorizado; Gerência × Backup permanece pendente;
- contrato Pocket como gate superior.

## Tópicos que o Bloco 11 deve fechar

1. conjunto exato de dados e arquivos que formam o estado recuperável;
2. snapshot consistente de SQLite + arquivos administrados;
3. formato e identidade do backup;
4. manifesto, verificação e compatibilidade entre versões/schema;
5. escrita completa, promoção e tratamento de backup parcial;
6. catálogo e retenção inicial;
7. coordenação com mutações e operações administrativas;
8. Restore normal e safety backup;
9. restart, reconexão e sessões após Restore;
10. falhas parciais e resultado incerto;
11. disaster recovery local quando o Host não inicia;
12. capacidades e auditoria;
13. validação técnica final.

## Ordem de análise proposta

1. estado recuperável + envelope do backup;
2. consistência + escrita/promoção/verificação;
3. catálogo + retenção + coordenação;
4. Restore + safety backup + compatibilidade;
5. restart/sessões/reconexão + falhas;
6. disaster recovery + capacidades/auditoria;
7. validação técnica final.

A ordem organiza o trabalho; não aprova antecipadamente nenhuma alternativa técnica.

## Critérios de fechamento

O bloco só pode ser considerado concluído quando as decisões acima permitirem implementação futura sem escolhas críticas deixadas ao executor e quando:

- a UX da Tela 13 continuar coerente;
- o modelo de dados/migrations souber quais impactos precisará absorver na fase executável;
- o contrato Pocket permanecer intacto;
- nenhum backup parcial puder ser tratado como válido;
- Restore tiver estados de falha e recuperação definidos;
- disaster recovery possuir fronteira clara em relação ao Restore normal;
- decisões aprovadas forem sincronizadas nas fontes específicas.

## Fora do escopo desta abertura

- implementação funcional;
- migrations oficiais;
- scheduler periódico;
- serviço persistente de backup;
- backup em nuvem;
- integração com destino externo específico;
- nova UX;
- números finais de performance sem evidência.

## Próxima análise

Começar pela fronteira fundamental: **o que exatamente constitui o estado recuperável do StepFlow e qual forma de pacote permite restaurá-lo de modo coerente**.
