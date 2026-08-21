# Tela 07 — Histórico e Revisões de Processo

**Status:** EM ANÁLISE / PROPOSTA PARA APROVAÇÃO DO PO  
**Bloco:** Fase 1 — Bloco 8 (UI/UX)  
**Última consolidação:** 2026-08-21

## 1. Objetivo

Permitir consultar o histórico imutável de um procedimento, identificar com clareza a revisão atual e a revisão publicada, abrir qualquer snapshot autorizado em modo somente leitura e, quando permitido, usar uma revisão anterior como base para uma nova revisão sem reescrever o passado.

## 2. Atores e permissões

- ADM;
- Gerência;
- outros usuários apenas se receberem capacidade explícita para consultar histórico.

Funcionário/Técnico continua orientado principalmente ao consumo da revisão publicada/autorizada. O Host permanece autoridade final.

## 3. Entrada

Fluxos principais:

```text
Leitor → menu contextual → Histórico
Processos → menu do item → Histórico
Editor → ação contextual → Histórico
```

O histórico sempre pertence a um procedimento identificado de forma estável.

## 4. Estrutura proposta

```text
← Processo

PR-014  Configuração de VLAN
Histórico de revisões

Revisão   Versão   Criada em          Autor            Estado
r18       2.0      21/08/2026 08:40   Maria            Atual • Publicada
r17       1.9      20/08/2026 15:12   João             —
r16       1.9      20/08/2026 11:03   João             —
r15       1.8      18/08/2026 09:26   Maria            —

[ Abrir revisão ]   [ Criar nova revisão a partir desta* ]
```

A lista usa ordem decrescente, com revisão mais recente primeiro.

## 5. Revisão técnica × versão exibida

- `revision_no` é técnico, monotônico e identifica o snapshot histórico;
- `display_version` é editorial e pode se repetir entre revisões;
- a interface não deve tratar `Versão 2.0` como identificador único do histórico;
- quando necessário, mostrar ambas: `Revisão r18 · Versão 2.0`.

## 6. Indicadores de estado

Destacar de forma discreta:

- **Atual** — revisão apontada por `current_revision_id`;
- **Publicada** — revisão apontada por `published_revision_id`;
- ambas podem coincidir ou ser diferentes.

Uma revisão antiga que já foi publicada no passado não deve receber selo de `Publicada` apenas por histórico; o selo representa o ponteiro publicado vigente. Auditoria de eventos de publicação é conceito separado.

## 7. Abrir revisão histórica

Ao abrir uma revisão antiga:

- reutilizar o Leitor em modo somente leitura;
- manter identificação persistente de que se trata de revisão histórica;
- exibir algo equivalente a `Revisão histórica r17 · Versão 1.9`;
- não deixar a revisão histórica parecer a versão vigente;
- voltar retorna ao Histórico preservando a posição/lista quando possível.

## 8. Criar nova revisão a partir de uma antiga — proposta

Para usuário com capacidade de edição, oferecer ação explícita:

`Criar nova revisão a partir desta`

Comportamento proposto:

1. o snapshot antigo é usado como conteúdo inicial no Editor;
2. nenhuma revisão histórica é alterada;
3. nada é persistido até o usuário salvar;
4. o save usa a revisão atual vigente como base de concorrência;
5. save aceito cria uma **nova** `process_revision` imutável;
6. se a revisão atual mudar antes do save, aplicar `409 REVISION_CONFLICT` normalmente.

Não usar botão ambíguo `Restaurar` que possa sugerir reescrita/destruição do histórico.

## 9. Sem exclusão ou edição do passado

Na operação normal:

- revisão histórica não pode ser editada in-place;
- revisão histórica não pode ser excluída;
- não alterar `revision_no` existente;
- não fazer rollback destrutivo;
- não mover ponteiros silenciosamente apenas por abrir uma revisão.

## 10. Publicação

A Tela 07 não publica uma revisão histórica diretamente.

Fluxo seguro quando for necessário retomar conteúdo antigo:

```text
revisão histórica
→ Criar nova revisão a partir desta
→ revisar/salvar no Editor
→ nova revisão atual
→ Publicar revisão atual, se autorizado
```

Isso mantém a regra já aprovada de que `Salvar` e `Publicar revisão atual` são ações distintas.

## 11. Dados exibidos

Por revisão, quando disponíveis:

- `revision_no`;
- `display_version`;
- data/hora de criação;
- autor da revisão;
- indicação Atual/Publicada;
- título/código do procedimento no contexto atual;
- acesso ao snapshot completo.

Não adicionar campo obrigatório de “mensagem de commit da revisão” sem requisito específico.

## 12. Estados da interface

### Loading

Exibir estrutura da página e skeleton da lista.

### Sem histórico

Novo procedimento ainda não salvo não abre Histórico. Procedimento persistido deve possuir ao menos sua primeira revisão.

### Revisão indisponível

Informar de forma simples e manter retorno ao histórico.

### Sem permissão

Não carregar snapshots protegidos de cache. Host decide acesso.

### Host indisponível

Histórico oficial fica indisponível; não tratar cache local como fonte oficial.

### Processo arquivado

Usuário administrativo autorizado pode consultar histórico se o Host permitir; sinalizar `Arquivado` no contexto do procedimento.

## 13. Atualização em tempo real

Se uma nova revisão for criada enquanto a tela estiver aberta:

- WebSocket apenas sinaliza;
- Client reconsulta lista oficial;
- nova revisão pode aparecer no topo;
- revisão histórica atualmente aberta/selecionada não é trocada automaticamente;
- indicadores Atual/Publicada são reconciliados com o Host.

## 14. Concorrência

Consulta de histórico não cria lock.

A ação `Criar nova revisão a partir desta` entra no Editor e passa a obedecer ao mesmo `base_revision`/controle otimista da Tela 06. Histórico nunca oferece overwrite forçado.

## 15. Contratos Client ↔ Host necessários

Conceitualmente:

1. listar revisões autorizadas de um procedimento;
2. obter snapshot de uma revisão específica;
3. informar revisão atual/publicada;
4. receber capacidades da sessão;
5. reconsultar após eventos de processo;
6. iniciar Editor a partir de snapshot histórico sem persistir alteração automaticamente.

Nomes finais de endpoints pertencem à implementação.

## 16. Paginação e volume

Não carregar histórico indefinido de forma ingênua.

Primeira versão pode usar paginação simples ou carregamento incremental quando necessário. Não introduzir motor de busca ou infraestrutura adicional só para histórico.

## 17. Comparação de revisões

Comparação/diff visual lado a lado pode ser útil no futuro, mas **não é requisito obrigatório da primeira versão**.

Não criar persistência adicional de diff. Se essa capacidade for aprovada futuramente, o resultado deve ser derivado de snapshots imutáveis.

## 18. Acessibilidade e teclado

- tabela/lista navegável por teclado;
- foco visível;
- indicadores Atual/Publicada não dependem somente de cor;
- ações possuem labels claros;
- seleção de revisão não deve depender de duplo clique;
- revisão aberta em modo histórico deve ser anunciada de forma textual.

## 19. Tamanhos de janela

Desktop Windows é prioridade.

Em janela menor:

- colunas secundárias podem ser condensadas em detalhes da linha;
- Revisão, Versão e Estado continuam visíveis;
- ações permanecem acessíveis;
- não criar layout mobile específico sem necessidade demonstrada.

## 20. Fora do escopo

- editar revisão histórica;
- excluir revisão histórica;
- rollback destrutivo;
- merge automático entre revisões;
- diff visual obrigatório na primeira versão;
- workflow complexo de aprovação;
- auditoria de segurança genérica misturada à lista de revisões;
- código Tauri/Host.

## 21. Propostas para aprovação do PO

1. lista cronológica compacta, mais recente primeiro;
2. mostrar `revision_no` e `display_version` separadamente;
3. badges `Atual` e `Publicada` representam ponteiros vigentes;
4. abrir revisão histórica reutiliza o Leitor em somente leitura;
5. revisão histórica recebe identificação visual/textual persistente;
6. não permitir editar/excluir snapshots;
7. usar `Criar nova revisão a partir desta` em vez de rollback destrutivo;
8. publicação de conteúdo antigo exige primeiro criar/salvar nova revisão;
9. comparação visual de revisões fica fora da primeira versão.

## 22. Pendências

- microcopy final;
- paginação exata;
- capacidade/perfil padrão para consulta de histórico;
- eventual comparação visual futura.

## 23. Critérios de aceite da especificação

- [x] revisões permanecem imutáveis;
- [x] current/published são distinguidos;
- [x] versão editorial não substitui revision_no;
- [x] nenhuma operação destrutiva foi introduzida;
- [x] concorrência continua otimista;
- [x] nenhum código de produção foi criado;
- [ ] UX da lista aprovada pelo PO;
- [ ] ação `Criar nova revisão a partir desta` aprovada;
- [ ] escopo sem diff obrigatório aprovado.

## 24. Casos de teste futuros

1. listar procedimento com uma revisão;
2. listar muitas revisões;
3. current = published;
4. current diferente de published;
5. abrir revisão atual;
6. abrir revisão histórica;
7. negar histórico sem permissão;
8. receber nova revisão via evento/reconsulta;
9. iniciar nova revisão a partir de snapshot antigo;
10. salvar com base atual;
11. receber 409 se current mudar antes do save;
12. validar processo arquivado;
13. validar janela menor/acessibilidade.
