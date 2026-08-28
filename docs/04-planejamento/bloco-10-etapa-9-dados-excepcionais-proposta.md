# Bloco 10 — Etapa 9 — Múltiplos MACs / Procedimentos / dados excepcionais — Proposta para análise

**Status:** PROPOSTA / AGUARDANDO APROVAÇÃO DO PO  
**Fase:** Fase 1 — Fechamento arquitetural e especificação  
**Abertura:** 2026-08-28  
**Base consolidada:** Bloco 10 / Etapas 1–8  
**Base Git:** `main` em `6c788bd40273c6bc430487fbe54f9a1ea7c402d7`

## 1. Objetivo

Fechar o comportamento da Ficha compacta quando o Atendimento possui **quantidade de dados**, e não apenas texto individual longo, capaz de pressionar a única página A4.

Esta etapa trata principalmente:

- múltiplos identificadores de rede/MACs do Equipamento;
- múltiplos Procedimentos vinculados ao Atendimento;
- muitas observações de serviço por Etapa;
- valores estruturados excepcionalmente extensos;
- multiplicidade que não deve virar tabela, inventário ou relatório completo.

A Etapa 9 não reabre:

- a geometria A4 de uma página da Etapa 7;
- os soft limits e a política de `SHEET_OVERFLOW` da Etapa 8;
- a ordem visual principal da Ficha;
- a proibição de truncamento, IA/resumo automático, modo compacto e redução automática de fonte/margem/espaçamento.

## 2. Princípio central — Ficha resumida não é dump do domínio

A Ficha é uma projeção client-facing do Atendimento, não uma cópia de todas as estruturas persistidas.

Portanto, preservar o dado operacional **não significa imprimir todo dado existente**.

```text
Host / domínio
→ preserva o conjunto completo

Ficha
→ aplica regra de projeção explícita e previsível
→ mostra somente o que pertence à prestação de contas
```

Uma informação pode ficar fora da Ficha por regra de produto sem ser apagada, truncada ou perdida.

Isso é diferente de:

```text
conteúdo deveria aparecer
→ não coube
→ renderer corta silenciosamente
```

Esse segundo comportamento continua proibido.

## 3. Regra para dados multiplicativos

A proposta separa três situações:

```text
1. dado operacional sem valor client-facing suficiente
→ não entra na Ficha por padrão

2. dado client-facing com representação compacta determinística
→ entra de forma resumida prevista pelo contrato

3. dado client-facing que não pode ser reduzido sem perder significado
→ permanece integral
→ pode resultar em SHEET_OVERFLOW
```

Não criar escolha automática baseada apenas em “quanto espaço sobrou”.

O Typst continua autoridade para o encaixe físico final, não motor de seleção semântica.

---

# Múltiplos MACs / identificadores de rede

## 4. Fonte

O modelo conceitual já preserva múltiplos identificadores em:

```text
equipment_network_identifiers
- network_identifier_id
- equipment_id
- kind
- normalized_value
- label NULL
```

MAC não é identidade canônica do Equipamento.

## 5. Papel do MAC na Ficha

A Ficha não deve virar inventário de interfaces de rede.

Ao mesmo tempo, um ou dois MACs podem ser úteis em contextos de TI sem adicionar grande densidade.

Proposta:

```text
0 identificadores
→ não renderiza linha

1 identificador
→ renderiza valor compacto

2 identificadores
→ renderiza ambos de forma compacta

3+ identificadores
→ não despeja todos os valores
→ renderiza indicação compacta de multiplicidade
```

Exemplos conceituais:

```text
MAC Wi-Fi AA:BB:CC:DD:EE:FF
```

```text
MAC LAN AA:BB:... · Wi-Fi 11:22:...
```

Para 3+:

```text
MACs: 4 identificadores cadastrados
```

O conjunto completo continua disponível no cadastro do Equipamento.

## 6. Labels de MAC

Quando `label` existir e for curto/útil, pode contextualizar o valor:

```text
LAN · AA:BB:CC:DD:EE:FF
Wi-Fi · 11:22:33:44:55:66
```

Não inventar `principal`, `primário`, `preferido` ou hierarquia de interfaces sem dado explícito no domínio.

A proposta evita selecionar arbitrariamente “os dois primeiros” quando existem muitos identificadores.

## 7. Por que não imprimir todos os MACs

Imprimir uma lista extensa:

- transforma a Ficha em inventário técnico;
- consome espaço sem relação direta com a prestação de contas;
- torna a quantidade de interfaces um fator artificial de `SHEET_OVERFLOW`;
- cria pressão para apagar dado operacional legítimo apenas para imprimir.

A indicação `N identificadores cadastrados` é uma **regra de projeção**, não truncamento do cadastro.

---

# Múltiplos Procedimentos

## 8. Proveniência interna

Um Atendimento pode usar zero, um ou vários Procedimentos e cada vínculo preserva a revisão exata utilizada.

Esses vínculos são essenciais para histórico/auditoria interna, mas a Ficha já foi definida como prestação de contas resumida ao cliente.

## 9. Proposta para a Ficha

**Não imprimir lista de Procedimentos vinculados por padrão, independentemente da quantidade.**

```text
0 Procedimentos
→ nenhuma diferença visual

1 Procedimento
→ vínculo permanece interno

N Procedimentos
→ vínculos permanecem internos
```

Motivos:

- o cliente precisa entender o serviço realizado, não a composição interna dos guias usados pelo técnico;
- `Resumo do trabalho` e observações descrevem o resultado real;
- código/título/revisão de Procedimentos aumentariam densidade sem benefício proporcional;
- a Etapa 7 já exclui lista detalhada de Procedimentos da Ficha.

A quantidade de Procedimentos, portanto, **não deve causar overflow da Ficha**.

Se no futuro houver requisito client-facing para declarar Procedimentos utilizados, isso será nova decisão de produto e não consequência automática do vínculo interno.

---

# Muitas observações por Etapa

## 10. Diferença em relação a MACs/Procedimentos

Observações de serviço têm semântica client-facing já consolidada.

Quando preenchidas, elas podem representar fatos reais distintos:

```text
SSD — unidade anterior apresentou setores defeituosos.
Rede — cabo apresentou mau contato.
Validação — bateria requer acompanhamento.
```

Por isso, a proposta **não cria um limite fixo de quantidade de observações impressas**.

## 11. Regra proposta

```text
observações preenchidas
→ permanecem na ordem das Etapas executadas
→ nenhuma é descartada automaticamente por posição
```

Não usar:

- “mostrar apenas as três primeiras”;
- “mostrar apenas as últimas”;
- selecionar pelo tamanho;
- selecionar pelo espaço restante da página;
- eliminar textos parecidos;
- resumir várias observações automaticamente em uma só.

Se a quantidade de observações pressionar a A4:

```text
Typst
→ 2+ páginas
→ SHEET_OVERFLOW
→ diagnóstico inclui multiplicidade de observações
```

O técnico pode revisar conscientemente os textos reais quando houver redundância ou redação excessiva, mas o StepFlow não destrói conteúdo por inferência.

## 12. Atendimento que legitimamente não cabe

A proposta admite um caso importante:

> pode existir Atendimento cujo conteúdo legítimo não consiga ser transformado em uma Ficha de uma página sem revisão humana.

Nesse caso, a aplicação não promete “dar um jeito” automaticamente.

A Ficha permanece bloqueada por `SHEET_OVERFLOW` até que exista uma versão client-facing compatível com o contrato de uma página por meio de revisão consciente dos dados reais.

Isso preserva a regra de uma A4 sem introduzir segunda página ou compactação automática.

---

# Dados estruturados excepcionalmente extensos

## 13. Strings técnicas longas

Exemplos:

- nome de Equipamento muito extenso;
- modelo/CPU/SO muito extensos;
- serial ou patrimônio fora do padrão comum;
- label de interface excessivamente longa;
- referência externa longa.

Comportamento proposto:

- permitir quebra de linha natural quando semanticamente segura;
- não cortar com reticências apenas para caber;
- não reduzir fonte;
- não substituir valor por abreviação inventada;
- não converter automaticamente para código menor;
- se ainda assim exceder a A4, `SHEET_OVERFLOW`.

Hard limits técnicos desses campos continuam separados da geometria da Ficha.

## 14. Dados vazios ou não aplicáveis

Permanece a regra consolidada:

```text
campo vazio/não aplicável
→ não renderiza valor
→ não renderiza label vazio
→ não reserva espaço
```

Isso não é compactação excepcional; é comportamento normal do template.

---

# Diagnóstico de overflow multiplicativo

## 15. Contributors semânticos

A Etapa 8 já consolidou diagnóstico de `SHEET_OVERFLOW`.

A Etapa 9 propõe ampliar o diagnóstico para reconhecer quantidade, por exemplo:

```text
SHEET_OVERFLOW
contributors:
- work_summary
- stage_notes_count
- stage_note:<stage_id>
- service_general_note
- device_note
- long_structured_field:<field>
```

`process_count` não deve aparecer como contributor se Procedimentos não são renderizados na Ficha.

Para 3+ MACs, a projeção compacta por contagem evita que a lista completa seja contributor visual.

## 16. Mensagem ao usuário

A UI deve continuar operacional e curta.

Exemplo conceitual:

```text
A ficha ficou extensa demais para uma página A4.
Há muitas observações de serviço e textos longos.
Revise os campos indicados e gere novamente.
```

Não exibir detalhes internos como IDs, heurísticas de layout ou pontuação de contribuição.

---

# Densidade e determinismo

## 17. Reimpressão previsível

A mesma fonte histórica + mesma versão do template deve produzir a mesma projeção sem depender de escolha arbitrária de itens pelo Client.

Por isso a proposta evita:

- seleção manual transitória de quais observações entram;
- checkbox de “incluir na Ficha” em cada observação;
- editor paralelo de conteúdo client-facing;
- escolha automática de itens baseada no espaço restante;
- “mostrar os primeiros N” quando não existe prioridade semântica.

## 18. Sem novos campos exclusivos para impressão

A Etapa 9 não cria:

- `include_in_sheet`;
- `sheet_priority`;
- `mac_principal` apenas para impressão;
- `procedimentos_para_ficha`;
- `observacoes_compactadas`;
- variante persistida exclusiva do documento.

Se futuramente surgir necessidade real de curadoria client-facing persistente, isso exigirá decisão própria de produto e modelo de dados.

---

# Casos de referência

## 19. Notebook comum

```text
1 Atendimento
1 Equipamento
2 MACs: LAN + Wi-Fi
1 Procedimento vinculado
2 observações de Etapa
```

Ficha:

- mostra os dois MACs compactamente;
- não lista o Procedimento;
- mostra as duas observações;
- Typst valida a única página.

## 20. Servidor com muitas interfaces

```text
1 Equipamento
8 MACs
3 Procedimentos vinculados
2 observações
```

Ficha:

```text
MACs: 8 identificadores cadastrados
```

- não imprime os oito endereços;
- não lista os três Procedimentos;
- preserva ambos integralmente no domínio;
- mostra as observações normalmente.

## 21. Atendimento com muitas observações

```text
12 Etapas
9 observações preenchidas
```

- todas as observações continuam candidatas à seção `OBSERVAÇÕES`;
- nenhuma é descartada automaticamente;
- se o Typst gerar duas páginas, ocorre `SHEET_OVERFLOW`;
- diagnóstico sinaliza a quantidade de observações e os campos mais pressionantes.

## 22. Valores muito longos

```text
nome longo
serial longo
OS longo
resumo próximo/acima do soft limit
```

- quebra natural quando possível;
- sem reticências automáticas;
- sem redução de tipografia;
- overflow explícito se necessário.

---

# Fora da Etapa 9

## 23. Não decidir aqui

Permanecem para etapas posteriores:

### Etapa 10

- nomes de arquivo;
- paths locais controlados;
- lifecycle de PDF/SVG temporários;
- cleanup/cancelamento/fechamento.

### Etapa 11

- QR/barcode, somente se houver benefício operacional aprovado.

### Etapa 12

- limites técnicos finais de payload/memória/tempo;
- matriz Windows/WebView2 real;
- stress/benchmark de renderização;
- limites concretos de quantidade por API/storage;
- validação real do template com massa de dados.

---

# Pontos para aprovação do PO

## 24. Decisões centrais propostas

1. **Procedimentos vinculados permanecem fora da Ficha por padrão**, independentemente da quantidade.
2. **MACs:** 0 = omite; 1–2 = mostra valores compactos; 3+ = mostra somente `N identificadores cadastrados`, preservando todos no Equipamento.
3. Labels de interface podem contextualizar MACs, mas não se inventa interface principal.
4. **Observações por Etapa não recebem cap de quantidade nem descarte automático.**
5. Muitas observações podem legitimamente produzir `SHEET_OVERFLOW`.
6. Campos estruturados longos quebram linha quando possível, mas não são truncados/abreviados automaticamente.
7. Diagnóstico de overflow pode indicar multiplicidade (`stage_notes_count`) além de campos textuais individuais.
8. Nenhum novo campo ou editor exclusivo da Ficha é criado.
9. O template da Etapa 7 continua único; sem segunda página ou modo compacto.

## 25. Próximo passo após aprovação

Se aprovada:

```text
proposta
→ promover decisões para fontes canônicas realmente impactadas
→ remover este arquivo temporário
→ revisar diff
→ ready
→ squash merge
→ apagar branch remota
→ verificar somente main + zero PRs abertos
→ somente então abrir Etapa 10
```
