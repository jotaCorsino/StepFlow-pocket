# Tela 05 — Leitor de Processo em Formato Livro

**Status:** EM ANÁLISE / PROPOSTA PARA APROVAÇÃO DO PO  
**Bloco:** Fase 1 — Bloco 8 (UI/UX)  
**Última consolidação:** 2026-08-21

## 1. Identificação

- código/nome da tela: Tela 05 — Leitor de Processo;
- status: proposta para aprovação;
- origem: requisitos consolidados de produto + decisões das Telas 01–04;
- objetivo visual: leitura técnica guiada, limpa e semelhante a manual/livro, sem aparência de portal burocrático.

## 2. Objetivo da tela

Permitir consultar e executar mentalmente/operacionalmente um procedimento com baixo atrito, apresentando cada etapa como uma página navegável, preservando contexto, versão e hierarquia do conteúdo.

O leitor é a principal superfície de consumo dos procedimentos do StepFlow.

## 3. Atores e permissões

- ADM;
- Gerência;
- Funcionário/Técnico.

Todos podem ler os procedimentos para os quais possuem autorização.

Ações adicionais — editar, histórico, exportar/imprimir e futuramente iniciar Atendimento — só aparecem quando a sessão possuir a capacidade correspondente. O Host permanece autoridade final.

## 4. Como o usuário chega à tela

Fluxos principais:

```text
Processos
→ selecionar procedimento
→ Leitor
```

Também pode ser aberto por:

- resultado vindo do Dashboard;
- link interno/contextual para um procedimento;
- histórico, quando um usuário autorizado abrir uma revisão específica;
- futuramente, contexto de Atendimento que referencia uma revisão específica.

Ao retornar para Lista/Pesquisa, o estado anterior de busca/filtros deve ser preservado conforme Tela 04.

## 5. Layout e hierarquia visual

Direção proposta:

```text
← Processos

PR-014  Configuração de VLAN                              [⋯]
Redes  Infraestrutura      TI       Versão 2.0

Etapa 3 de 7                                      [ Sumário ▾ ]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

3. Configurar a VLAN no switch
Breve introdução da etapa...

1. Acesse o equipamento...
2. Entre no modo de configuração...

┌ Observação ────────────────────────────────────────────────┐
│ ...                                                        │
└────────────────────────────────────────────────────────────┘

┌ Comando ─────────────────────────────────────────────── [⧉]┐
│ configure terminal                                        │
└────────────────────────────────────────────────────────────┘

                         [ ← Etapa anterior ] [ Próxima etapa → ]
```

O Shell/sidebar global permanece visível. O leitor não cria uma segunda sidebar permanente.

## 6. Elementos fixos

### Cabeçalho do procedimento

Exibir de forma compacta:

- código;
- título;
- categoria(s);
- Área/Departamento;
- versão exibida;
- status editorial somente quando relevante ao perfil/contexto.

Metadados adicionais não devem ocupar permanentemente grande área da tela.

### Navegação da página

Exibir:

- nome/título da página atual;
- indicador `Etapa X de Y` quando em uma etapa;
- progresso visual linear de **navegação**, sem significar conclusão do trabalho;
- ação `Sumário` para salto entre Visão geral e etapas;
- anterior/próxima.

## 7. Visão geral do procedimento

### Proposta

Antes da Etapa 1 existe uma página lógica chamada **Visão geral**, sem ser numerada como etapa.

Ela apresenta quando houver conteúdo:

- Objetivo;
- Pré-requisitos;
- Observações gerais;
- Responsável documental;
- categorias;
- versão/informações editoriais essenciais.

O Sumário fica conceitualmente:

```text
Visão geral
Etapa 1 — ...
Etapa 2 — ...
Etapa 3 — ...
...
```

Isso evita esconder Objetivo/Pré-requisitos em modal ou repetir esses dados em todas as etapas.

## 8. Página de etapa

Cada `process_stage` é uma página do manual.

A página contém:

- número/posição;
- título;
- introdução quando existir;
- blocos ordenados;
- navegação para página anterior/próxima.

Não misturar duas etapas completas na mesma página apenas para reduzir cliques.

## 9. Blocos suportados

O leitor deve renderizar os tipos conceituais consolidados:

### `paragraph`

Texto normal com largura confortável e boa legibilidade.

### `numbered_steps`

Passos/subpassos numerados com hierarquia visual clara.

### `checklist`

Exibe a **definição documental** dos itens.

No leitor independente de Atendimento, esta especificação não atribui persistência nem significado de conclusão aos itens. Interação/persistência operacional será fechada no Bloco 9.

### `note`

Observação discreta, visualmente distinta sem dominar a página.

### `warning`

Alerta com maior destaque que `note`, mas sem estilo alarmista desnecessário.

### `command`

Comando/instrução curta em bloco monoespaçado, copiável quando aplicável.

### `code`

Trecho maior de código/configuração em bloco monoespaçado, preservando espaços/quebras relevantes.

## 10. Cópia de comandos e código

Decisão já orientada pelo produto:

- controle de cópia por ícone discreto;
- sem botão textual grande `Copiar` permanente;
- feedback curto após sucesso, por exemplo `✓ Copiado`;
- feedback não deve deslocar a página de forma perceptível;
- conteúdo copiado deve corresponder exatamente ao bloco exibido.

Quando Clipboard falhar, informar de forma breve sem expor detalhes técnicos desnecessários.

## 11. Sumário e salto entre etapas

### Proposta

Usar botão/dropdown/painel temporário `Sumário`, em vez de segunda sidebar permanente.

O Sumário:

- lista `Visão geral` + todas as etapas;
- marca a página atual;
- permite salto direto;
- fecha após seleção;
- é operável por teclado;
- não representa progresso concluído.

Para procedimentos muito longos, o Sumário deve permitir rolagem própria/controlada.

## 12. Navegação anterior/próxima

- `Visão geral` → Próxima = Etapa 1;
- Etapa 1 → Anterior = Visão geral;
- etapas intermediárias possuem Anterior + Próxima;
- última etapa não apresenta ação Próxima inválida;
- mudança de página preserva o cabeçalho/contexto do procedimento.

A rolagem da nova página volta para o início do conteúdo, salvo necessidade futura diferente.

## 13. Ações contextuais do procedimento

Menu discreto de ações conforme capacidade pode conter:

- Editar;
- Histórico;
- Exportar/Imprimir;
- outras ações documentais explicitamente aprovadas.

### `Iniciar atendimento` — proposta de ponto de entrada

A arquitetura aprovada permite que um Atendimento use um ou mais procedimentos e preserve a revisão utilizada.

Propõe-se reservar no leitor um ponto de entrada contextual `Iniciar atendimento` quando a capacidade existir.

O Bloco 8 pode aprovar **onde** essa ação aparece; lifecycle, permissões, criação efetiva e dados exigidos pertencem ao Bloco 9.

Não implementar fluxo operacional antes disso.

## 14. Revisão e versão exibida

O leitor deve saber qual revisão está mostrando.

Regras:

- Funcionário/Técnico normalmente consulta a revisão publicada/autorizada;
- Gerência/ADM podem abrir uma revisão específica quando o contexto autorizar;
- revisão histórica/draft nunca pode parecer silenciosamente a versão publicada atual;
- versão editorial exibida e revisão técnica interna continuam conceitos distintos.

## 15. Atualização em tempo real durante a leitura

### Proposta importante

Se chegar evento indicando nova revisão/publicação enquanto o usuário está lendo:

- **não substituir silenciosamente o conteúdo atual**;
- manter a revisão aberta estável;
- mostrar aviso discreto, por exemplo `Existe uma versão mais recente deste procedimento.`;
- oferecer ação `Atualizar para a versão mais recente` quando autorizado/aplicável;
- ao atualizar, reconsultar o Host e abrir a revisão nova conscientemente.

Isso evita trocar instruções no meio de uma execução.

Mudanças de metadados sem impacto no conteúdo podem ser reconciliadas sem interromper a leitura quando seguro.

## 16. Estados da interface

### Loading

- preservar Shell e estrutura do leitor;
- mostrar esqueleto do cabeçalho/conteúdo;
- não exibir dados de revisão anterior como se fossem atuais.

### Processo não encontrado

Mensagem simples com retorno para `Processos`.

### Revisão indisponível

Explicar que a revisão solicitada não está disponível/autorizada e oferecer destino seguro.

### Sem permissão

Não renderizar conteúdo protegido vindo de estado antigo/cache. Host decide autorização.

### Host indisponível

Seguir estado transversal do Shell. Não transformar cópia local antiga em fonte oficial silenciosamente.

### Processo arquivado

- Funcionário sem acesso deixa de receber o recurso;
- usuário administrativo autorizado pode receber indicação clara de `Arquivado` quando o Host permitir consulta histórica.

### Versão mais recente disponível

Usar aviso não intrusivo conforme seção 15.

## 17. Validações

Tela predominantemente de leitura.

Validar defensivamente:

- existência do `process_id`/revisão solicitada;
- ordem válida das etapas/blocos recebidos;
- tipo de bloco conhecido/suportado;
- URL/referências externas futuras somente se houver contrato específico aprovado.

Bloco desconhecido não deve derrubar a tela inteira; registrar diagnóstico seguro e apresentar fallback apropriado conforme contrato futuro.

## 18. Mensagens e feedbacks

Preferir linguagem curta e operacional:

- `Copiado`;
- `Existe uma versão mais recente deste procedimento.`;
- `Este procedimento não está mais disponível.`;
- `Não foi possível carregar o procedimento.`

Não exibir stack trace, IDs internos desnecessários, SQL, paths ou detalhes do Host ao usuário final.

## 19. Dados exibidos

Conceitualmente:

- `process_id` não precisa aparecer;
- código;
- título;
- área/departamento;
- responsável documental quando útil;
- status quando relevante;
- versão exibida;
- categorias;
- objetivo;
- observações;
- pré-requisitos;
- revisão selecionada;
- etapas ordenadas;
- blocos ordenados/tipados.

## 20. Dados enviados/alterados

No modo leitor puro:

- navegação entre páginas não altera o procedimento;
- copiar bloco não altera o Host;
- abrir Sumário não altera persistência;
- esta tela não salva estado de checklist nesta fase.

Ações que iniciem Atendimento ou alterem documentação pertencem a contratos próprios.

## 21. Regras de negócio

- conteúdo oficial vem do Host;
- cada etapa funciona como página;
- processo pode ter uma ou múltiplas categorias;
- `Visão geral` é apresentação dos metadados já existentes, não nova entidade de domínio;
- progresso `X de Y` é posição de leitura/navegação, não conclusão de serviço;
- revisão aberta não muda silenciosamente após evento de atualização;
- checklist documental é distinto de estado operacional de execução.

## 22. Regras de autorização

- Host filtra o que pode ser lido;
- Client oculta ações não autorizadas, mas isso não substitui verificação Host-side;
- acesso a revisão histórica/draft depende de capacidade;
- `Iniciar atendimento`, quando existir, depende de capacidade operacional a ser fechada no Bloco 9.

## 23. Impacto em persistência

Nenhuma nova persistência é criada por esta tela no Bloco 8.

A tela consome o modelo consolidado de procedimentos/revisões/etapas/blocos/categorias.

Estado operacional de checklist, progresso ou Atendimento não é definido aqui.

## 24. Contratos Client ↔ Host necessários

Conceitualmente:

1. obter procedimento/revisão autorizada por identidade estável;
2. obter metadados + categorias;
3. obter etapas/blocos ordenados;
4. distinguir revisão atual/publicada/histórica conforme capacidade;
5. receber capacidades da sessão;
6. reconsultar após evento de atualização;
7. no futuro, iniciar Atendimento por contrato próprio.

Nomes finais de endpoints ficam para implementação.

## 25. Eventos em tempo real relevantes

Eventos equivalentes a:

- `process.updated`;
- `process.published`;
- `process.archived`;
- alteração de categoria/metadados relevantes.

WebSocket sinaliza; o Client reconsulta o estado oficial.

## 26. Concorrência

Leitura não cria lock.

Se a revisão mudar enquanto está aberta, aplicar a regra de estabilidade da seção 15. O leitor não tenta mesclar conteúdo nem muda de revisão automaticamente.

Ações de edição permanecem regidas por revisão otimista na Tela 06/contratos correspondentes.

## 27. Acessibilidade e teclado

- foco visível;
- estrutura semântica de headings;
- `Sumário` operável por teclado;
- Previous/Next com labels acessíveis;
- botão de copiar possui nome acessível apesar de ser icon-only;
- blocos de código permitem seleção manual;
- alertas não dependem apenas de cor;
- atalhos de navegação podem usar combinação não conflitante após teste, sem serem requisito único de acesso.

## 28. Tamanhos de janela suportados

Desktop Windows é prioridade.

Em janela menor:

- cabeçalho quebra linhas sem cortar título;
- categorias podem fluir para linha seguinte;
- metadados secundários podem recolher em `Detalhes`;
- blocos de código/comando preservam conteúdo e podem rolar horizontalmente internamente;
- Anterior/Próxima continuam acessíveis;
- não criar layout mobile/hamburger sem necessidade demonstrada.

## 29. Preservação visual

Direção vigente:

- visual corporativo/clássico;
- conteúdo central com largura confortável de leitura;
- aparência de manual, sem skeuomorfismo pesado de livro/papel;
- hierarquia tipográfica clara;
- muito espaço dedicado ao conteúdo técnico;
- controles secundários discretos;
- sem cards decorativos desnecessários.

## 30. Divergências com documentação anterior

Nenhuma divergência de domínio intencional.

Esta análise torna explícitos:

- `Visão geral` como página lógica de apresentação;
- Sumário temporário em vez de segunda sidebar;
- estabilidade da revisão durante leitura;
- distinção visual entre progresso de navegação e progresso operacional.

Todos são propostas desta tela até aprovação do PO.

## 31. Propostas para aprovação do PO

1. `Visão geral` como primeira página lógica antes da Etapa 1;
2. uma etapa por página, sem juntar etapas completas;
3. `Sumário` temporário/dropdown, sem segunda sidebar fixa;
4. `Etapa X de Y` + barra simples representando posição de leitura, não conclusão;
5. Anterior/Próxima no final da página;
6. categorias como chips discretos no cabeçalho;
7. checklist no leitor puro representa somente definição documental; interação/persistência fica para Bloco 9;
8. nova revisão disponível gera aviso e **não** substitui silenciosamente a revisão aberta;
9. menu contextual para Editar/Histórico/Exportar quando autorizado;
10. reservar ponto de entrada `Iniciar atendimento` no leitor, com comportamento efetivo somente após Bloco 9;
11. conteúdo central limpo, sem segunda sidebar e sem aparência pesada de livro físico.

## 32. Pendências

- interação exata do checklist no contexto de Atendimento — Bloco 9;
- permissões de `Iniciar atendimento` — Bloco 9;
- lifecycle do Atendimento — Bloco 9;
- comportamento final de exportação/impressão — Bloco 10;
- dimensões/tipografia/paleta exatas do sistema visual;
- atalhos de teclado opcionais após teste.

## 33. Fora do escopo

- edição do conteúdo;
- persistência de checklist/progresso;
- criação/conclusão/reabertura de Atendimento;
- geração técnica de PDF/DOCX;
- ficha compacta de equipamento;
- comentários/chat;
- modo offline de edição;
- busca dentro do procedimento na primeira versão, salvo requisito posterior.

## 34. Critérios de aceite

- [x] cada etapa tratada como página;
- [x] tipos de bloco consolidados cobertos;
- [x] cópia discreta contemplada;
- [x] categorias incorporadas;
- [x] revisão/versionamento explicitados;
- [x] atualização em tempo real não sobrescreve leitura silenciosamente;
- [x] checklist separado de execução operacional;
- [x] nenhuma persistência/código de produção criado;
- [ ] estrutura `Visão geral` aprovada;
- [ ] Sumário/navegação aprovados;
- [ ] tratamento de nova revisão aprovado;
- [ ] ponto de entrada de Atendimento aprovado;
- [ ] direção visual do conteúdo aprovada.

## 35. Casos de teste/smoke futuros

1. abrir procedimento publicado pela lista;
2. abrir Visão geral e navegar para Etapa 1;
3. percorrer todas as etapas com Anterior/Próxima;
4. saltar via Sumário;
5. copiar comando e código;
6. verificar feedback de cópia;
7. renderizar note/warning/checklist/numbered steps;
8. procedimento com uma e várias categorias;
9. procedimento com etapa longa;
10. código largo usa scroll interno sem quebrar layout;
11. nova publicação durante leitura não troca conteúdo automaticamente;
12. usuário atualiza conscientemente para nova versão;
13. Funcionário não recebe ação administrativa;
14. Gerência/ADM recebem ações conforme capacidades;
15. retornar a Processos restaura busca/filtros;
16. Host indisponível mostra estado transversal correto;
17. revisão inexistente/sem permissão não vaza conteúdo;
18. teclado/foco/acessibilidade funcionam.
