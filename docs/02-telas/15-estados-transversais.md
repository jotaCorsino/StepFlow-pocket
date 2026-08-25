# Tela 15 — Estados Transversais

## 1. Identificação

- código/nome da superfície: Tela 15 — Estados Transversais;
- status: **CONSOLIDADO / APROVADO PELO PO**;
- bloco: Fase 1 — Bloco 8 (UI/UX);
- origem: contratos consolidados das Telas 01–14 + comunicação Client↔Host + concorrência/eventos;
- última consolidação: 2026-08-25.

## 2. Objetivo

Definir um contrato visual e comportamental comum para estados que aparecem em várias superfícies do StepFlow, evitando que cada tela trate carregamento, falha, indisponibilidade, conflito ou feedback de maneira diferente.

Esta especificação não cria uma nova página navegável. Ela padroniza componentes e regras transversais usados pelas Telas 01–14 e pelas futuras superfícies de implementação.

## 3. Princípios gerais

1. o Host continua fonte oficial de dados e autorização;
2. cache/estado antigo do Client nunca vira fonte oficial silenciosamente;
3. falhas locais devem permanecer locais sempre que possível;
4. problemas globais só ocupam o Shell quando afetam a aplicação como um todo;
5. mensagens devem explicar o que aconteceu e, quando possível, qual ação segura o usuário pode tomar;
6. não expor stack trace, SQL, path interno, comando, token ou segredo;
7. não apagar edição local silenciosamente por causa de erro, conflito ou reconexão;
8. não confirmar salvamento antes de o Host confirmar o commit;
9. não inventar percentual de progresso quando não existir progresso real;
10. estados não dependem apenas de cor ou ícone;
11. uma mesma condição não deve produzir simultaneamente toast, banner e modal redundantes sem necessidade;
12. fluxo normal não exibe indicador permanente de “tudo conectado”. Estados saudáveis permanecem discretos.

## 4. Hierarquia de apresentação

A UX usa a menor superfície suficiente para comunicar o estado.

```text
campo/componente
→ erro ou feedback junto do próprio componente

seção/recurso
→ aviso inline ou estado da área

página inteira
→ estado de página

aplicação/Host/sessão
→ aviso transversal no Shell ou bloqueio global quando necessário
```

Exemplos:

- senha inválida → junto ao campo/formulário;
- lista sem resultado → corpo da lista;
- processo não encontrado → conteúdo principal do Leitor;
- conflito no Editor → banner/área do Editor;
- Host indisponível → estado transversal do Shell;
- Client incompatível com Host → bloqueio antes do uso normal.

## 5. Componentes transversais conceituais

A primeira versão pode reutilizar os seguintes padrões visuais:

- `Loading/Skeleton`;
- `Estado vazio`;
- `Aviso inline`;
- `Erro inline`;
- `Banner transversal`;
- `Toast/feedback breve`;
- `Modal de confirmação`;
- `Estado de página`;
- `Estado de operação em andamento`.

Os nomes de componentes no código não são definidos por esta especificação.

## 6. Loading inicial de página

Ao abrir uma superfície que depende do Host:

- preservar Shell/sidebar quando a sessão já estiver válida;
- mostrar skeleton/placeholders simples na região que está carregando;
- não mostrar dados antigos como se fossem atuais;
- evitar spinner central gigante quando apenas uma lista/seção está carregando;
- não remover filtros/busca já escolhidos durante uma reconsulta;
- não deslocar excessivamente o layout quando o conteúdo chega.

Se o carregamento falhar, o estado deve evoluir para a condição real — Host indisponível, sem permissão, item indisponível etc. — e não permanecer indefinidamente em loading.

## 7. Loading de ação local

Para ações como `Salvar`, `Publicar`, `Criar backup`, `Exportar` ou outra operação explícita:

- a própria ação indica que está em andamento;
- impedir clique duplicado acidental da mesma operação;
- não bloquear o restante da aplicação além do necessário;
- ação só mostra sucesso após confirmação correspondente;
- operações longas usam estado indeterminado se não houver progresso real disponível.

Exemplo:

```text
[ Salvando… ]
```

não:

```text
Salvando… 67%
```

quando o sistema não possui percentual real.

## 8. Estado vazio — nenhum registro existente

Distinguir ausência real de registros de ausência causada por filtros.

Exemplo em Atendimentos:

`Nenhum atendimento disponível.`

Quando o usuário possuir capacidade de criação, a própria área pode oferecer ação contextual como `Novo atendimento`.

Não mostrar CTA que o usuário não pode executar.

## 9. Estado vazio — nenhum resultado para busca/filtros

Quando existem registros, mas os critérios atuais não retornam itens:

`Nenhum resultado encontrado com os filtros atuais.`

Ações apropriadas:

- `Limpar filtros`;
- ajustar busca/filtros existentes.

Não confundir com banco vazio ou falta de permissão.

## 10. Sucesso e feedback breve

Ações simples e concluídas podem usar feedback curto, preferencialmente próximo da ação ou em toast discreto.

Exemplos já compatíveis com o produto:

- `✓ Copiado`;
- `Alterações salvas.`;
- `Categoria arquivada.`;
- `Conexão restabelecida.` quando uma interrupção perceptível terminou.

Regras:

- não depender apenas de verde;
- não exigir que o usuário feche modal para feedback trivial;
- feedback crítico ou que exige decisão não deve desaparecer sozinho;
- toast não deve ser a única fonte de uma informação que o usuário precisa consultar depois.

## 11. Validação de formulário

Erros de validação devem, quando possível:

1. preservar os valores informados;
2. indicar o campo relacionado;
3. explicar o requisito em linguagem simples;
4. posicionar foco/erro de forma acessível;
5. não limpar o formulário.

Quando a validação Host-side envolver o formulário como um todo, pode haver resumo curto no topo + marcações nos campos aplicáveis.

Exemplo:

`Já existe uma categoria com este nome.`

Não mostrar código SQL ou constraint interna.

## 12. Host indisponível

Quando HTTP/Host deixar de estar acessível, o estado é transversal.

Direção visual:

```text
┌─────────────────────────────────────────────────────────────┐
│ Não foi possível conectar ao StepFlow. Tentando reconectar. │
└─────────────────────────────────────────────────────────────┘
```

Regras:

- mostrar o aviso no Shell sem criar uma topbar global permanente quando tudo está saudável;
- impedir novas ações que dependam do Host;
- não oferecer edição de IP, porta ou path técnico;
- não tratar cache como estado oficial;
- preservar edição local em memória enquanto o Client continuar aberto, quando for seguro fazê-lo;
- não enfileirar alterações locais para sincronização posterior;
- tentar reconexão conforme política técnica consolidada;
- após reconectar, revalidar/reconsultar o estado relevante antes de assumir que a tela continua atual.

## 13. WebSocket indisponível com HTTP ainda funcional

Falha do canal de eventos não significa necessariamente que o Host inteiro está indisponível.

Direção consolidada:

`Atualizações em tempo real temporariamente indisponíveis. Reconectando…`

Enquanto HTTP continuar saudável:

- consultas e comandos podem continuar conforme seus contratos normais;
- revisão otimista continua protegendo mutações;
- Client tenta restabelecer WebSocket;
- ao reconectar, refaz consultas relevantes;
- não presume replay completo de eventos perdidos.

O aviso deve ser discreto e desaparecer após reconciliação bem-sucedida.

## 14. Reconexão concluída

Ao recuperar conectividade:

```text
reconectar
→ revalidar sessão/compatibilidade quando aplicável
→ refazer consultas relevantes
→ reconciliar operações incertas
→ liberar ações apropriadas
```

Pode existir feedback breve `Conexão restabelecida.`.

Não exibir um indicador verde permanente depois disso.

## 15. Desconexão durante mutação / resultado incerto

Se o Client enviou uma alteração e perdeu comunicação antes de receber resposta, não pode afirmar imediatamente nem “salvo” nem “não salvo”.

Estado consolidado:

`Não foi possível confirmar o resultado da alteração. Verificando o estado atual…`

Fluxo:

```text
conexão perdida após envio
→ não repetir cegamente a mutação
→ reconectar
→ consultar/reconciliar estado
→ informar resultado conhecido
```

Se não for possível determinar automaticamente o resultado com segurança, a UI deve exigir recarga/revisão consciente em vez de duplicar a operação.

## 16. Sessão expirada

Quando o Host retornar sessão ausente/expirada:

- conteúdo protegido deixa de ser tratável como estado autorizado;
- solicitar nova autenticação;
- mensagem simples: `Sua sessão expirou. Entre novamente.`;
- não mostrar dados protegidos atrás do estado de login como se a sessão continuasse válida.

### Edição não salva durante expiração

Na primeira versão:

- enquanto o mesmo processo Client permanecer aberto, alterações locais não salvas podem permanecer **somente em memória e ocultas durante a reautenticação**;
- depois que o mesmo usuário autenticar novamente e continuar autorizado, o Client reconsulta o recurso e reaplica o fluxo normal de revisão/conflito antes de permitir novo salvamento;
- se outro usuário entrar, a conta perder permissão, o recurso não estiver mais disponível ou o Client for fechado, esse conteúdo local não vira draft persistente e não é recuperado automaticamente.

Isso não cria autosave nem armazenamento local de rascunho.

## 17. Sem permissão

Quando a sessão não possui capacidade:

- ações não aplicáveis ficam ocultas/desabilitadas conforme o contexto aprovado;
- acesso direto/manipulado é rejeitado pelo Host;
- estado de página simples: `Você não tem permissão para acessar este conteúdo.`;
- não exibir conteúdo protegido que tenha ficado em memória de uma autorização anterior.

Se a permissão for removida enquanto a tela está aberta:

```text
evento/reconsulta detecta perda de acesso
→ limpar conteúdo protegido da superfície
→ informar perda de permissão
→ retornar a destino seguro quando necessário
```

## 18. Recurso não encontrado ou indisponível

Exemplos:

- item removido entre lista e abertura;
- revisão não mais acessível;
- recurso arquivado e não autorizado para aquele perfil;
- link antigo/inválido.

Mensagem funcional:

`Este item não está mais disponível.`

Quando apropriado:

- voltar para a lista de origem;
- atualizar a lista preservando busca/filtros possíveis.

A UI não tenta recriar recurso inexistente nem usar cache antigo como substituto.

## 19. Conflito de alteração

Para recursos protegidos por revisão otimista:

```text
Você está editando uma versão que não é mais a atual.
As suas alterações não foram sobrescritas.
```

Princípios:

- preservar edição local;
- nunca aplicar last-write-wins silencioso;
- não substituir automaticamente a tela pelo conteúdo remoto;
- permitir ação consciente de recarregar/revisar;
- `Descartar minhas alterações` exige intenção clara quando houver perda de conteúdo local;
- merge automático não faz parte da primeira versão.

A terminologia específica pode variar (`Processo`, `Equipamento`, `Atendimento`, `Usuário`, `Configuração`), mas o padrão deve ser reconhecível em todas as superfícies.

## 20. Evento remoto enquanto existe edição local

Quando WebSocket sinalizar alteração de recurso que o usuário está editando:

- não substituir formulário local;
- mostrar aviso discreto de que existe versão mais recente;
- salvamento continua sujeito à base/revisão esperada;
- se a base estiver obsoleta, Host rejeita e entra no fluxo de conflito.

Exemplo:

`Este registro foi alterado por outro usuário. Revise as mudanças antes de salvar.`

## 21. Atualização remota em superfícies somente leitura

Em listas ou páginas que não possuem edição local:

- evento pós-commit sinaliza reconsulta;
- preservar busca/filtros/ordenação/posição quando possível;
- evitar deslocar abruptamente o usuário durante leitura;
- Leitor de Processo mantém a revisão aberta estável e apenas informa existência de revisão mais nova;
- histórico/revisão específica nunca é trocado silenciosamente.

## 22. Host ocupado / backpressure

Quando o Host rejeitar temporariamente uma mutação por saturação (`SERVER_BUSY` ou equivalente):

`O StepFlow está ocupado no momento. Tente novamente em instantes.`

Regras:

- não fingir sucesso;
- não repetir automaticamente mutação não idempotente de forma cega;
- manter dados locais do formulário;
- permitir tentativa consciente quando seguro;
- leituras podem adotar retry limitado conforme contrato técnico futuro.

## 23. Falha de persistência / erro interno

Para falha que o usuário não consegue corrigir alterando um campo:

`Não foi possível concluir a operação.`

Quando houver uma ação segura:

- `Tentar novamente`;
- `Recarregar`;
- retornar a uma tela segura.

Detalhes técnicos ficam nos logs. `request_id` pode ser usado internamente para correlação; a forma exata de exposição como referência de suporte fica para implementação/diagnóstico e não autoriza mostrar informações sensíveis.

## 24. Client/Host incompatíveis

Incompatibilidade de contrato bloqueia uso normal antes do login.

Direção consolidada:

```text
Esta versão do StepFlow precisa ser atualizada.
Feche o aplicativo e abra-o novamente pelo ponto de entrada normal do StepFlow.

[ Fechar ]
```

Não oferecer:

- edição manual de versão;
- download arbitrário pela Internet;
- troca de URL/porta;
- ignorar incompatibilidade e continuar.

O launcher continua responsável pelo fluxo de distribuição/atualização definido na arquitetura.

## 25. Alterações não salvas e saída

Superfícies com edição explícita devem detectar navegação/fechamento que descartaria mudanças locais.

Exemplo:

`Existem alterações não salvas. Deseja sair sem salvar?`

Ações:

- `Continuar editando` — padrão seguro;
- `Sair sem salvar` — destrutiva em relação ao rascunho local.

Regras:

- não perguntar quando nada mudou;
- não perguntar no Leitor puro apenas por navegar entre etapas;
- não criar autosave por causa dessa confirmação;
- fechar o Client com edição local não salva segue o mesmo princípio de proteção contra descarte acidental.

## 26. Confirmações destrutivas

Ações com efeito relevante devem usar confirmação proporcional ao risco.

Exemplos:

- arquivar/desativar: confirmação simples quando necessária;
- remover vínculo: confirmar apenas se houver consequência relevante;
- Restore: confirmação reforçada específica já consolidada na Tela 13.

Não exigir texto digitado para toda ação comum; a confirmação reforçada fica reservada a riscos realmente altos.

## 27. Operações longas

Backup, Restore e geração documental podem possuir duração maior que uma ação normal.

Regras transversais:

- mostrar qual operação está em andamento;
- não inventar porcentagem;
- distinguir `em andamento`, `concluído`, `falhou`, `cancelado` quando o contrato permitir;
- fechar Client não significa automaticamente cancelar operação Host-side já aceita;
- após retornar/reconectar, consultar estado quando a operação possuir continuidade Host-side;
- não permitir botão `Cancelar` depois do ponto em que a operação já não puder ser cancelada com segurança.

Detalhes específicos permanecem nas Telas 13/14 e Blocos 10/11.

## 28. Mensagens de erro e microcopy

Direção textual:

- frase curta;
- linguagem operacional, não técnica;
- explicar ação segura quando existir;
- evitar culpa ao usuário;
- evitar códigos internos como mensagem principal;
- não usar “Erro desconhecido” como única informação quando uma categoria funcional é conhecida.

Exemplos adequados:

- `Não foi possível conectar ao StepFlow.`;
- `Sua sessão expirou. Entre novamente.`;
- `Este registro foi alterado por outro usuário.`;
- `Nenhum resultado encontrado com os filtros atuais.`;
- `Salve as alterações antes de gerar a ficha.`.

## 29. Toasts e avisos persistentes

### Toast/feedback breve

Usar para resultado que não exige decisão imediata:

- copiado;
- salvo;
- conexão restabelecida;
- operação concluída quando o usuário pode continuar normalmente.

### Aviso persistente/inline

Usar quando a condição continua exigindo atenção:

- Host indisponível;
- WebSocket degradado;
- conflito;
- versão mais recente disponível;
- formulário inválido;
- operação crítica em andamento.

## 30. Prioridade entre estados

Quando múltiplas condições ocorrerem, a UI não deve sobrepor mensagens contraditórias.

Direção:

1. segurança/autorização e incompatibilidade;
2. indisponibilidade global/estado da sessão;
3. conflito ou resultado incerto de mutação;
4. validação/erro da operação atual;
5. avisos informativos;
6. feedback de sucesso.

Exemplo: se a sessão expirou enquanto havia aviso de revisão mais nova, o estado de sessão tem prioridade visual.

## 31. Preservação de contexto

Quando uma reconsulta/erro não invalida o contexto, preservar:

- busca;
- filtros;
- ordenação;
- página/scroll quando aplicável;
- etapa/revisão do Leitor;
- edição local não salva enquanto ainda for seguro mantê-la em memória.

Não preservar conteúdo protegido após perda definitiva de autorização.

## 32. Navegação segura após erro

Estados de página devem oferecer destino coerente:

- Processo indisponível → `Voltar para Processos`;
- Atendimento indisponível → `Voltar para Atendimentos`;
- sem permissão → retorno ao último destino autorizado ou Início;
- incompatibilidade → fechar/reabrir pelo fluxo oficial;
- falha local de modal → fechar modal mantendo página subjacente quando seguro.

## 33. Acessibilidade e teclado

- mensagens assíncronas importantes devem ser anunciáveis por tecnologia assistiva;
- foco não é roubado por todo toast;
- modal move foco para seu conteúdo e devolve ao controle de origem ao fechar;
- erro de formulário permite alcançar o primeiro campo inválido;
- estados não dependem só de cor;
- botões desabilitados precisam continuar compreensíveis pelo contexto;
- loading não deve produzir animação excessiva;
- navegação por teclado permanece disponível sempre que a ação estiver habilitada.

## 34. Tamanhos de janela suportados

- banner transversal não deve cobrir sidebar ou ações principais;
- estados vazios continuam legíveis sem ocupar toda a altura desnecessariamente;
- modais/painéis podem empilhar ações em janelas menores;
- mensagens longas quebram linha sem criar scroll horizontal;
- não criar transformação mobile/hamburger nesta primeira versão.

## 35. Autorização

Toda representação visual de capacidade é conveniência de UX.

O Host sempre revalida autorização para:

- leitura protegida;
- mutação;
- exportação;
- administração;
- Backup/Restore;
- ações operacionais futuras.

Ocultar botão não é controle de segurança suficiente.

## 36. Persistência

A Tela 15 não cria nova persistência funcional.

Estados visuais são transitórios no Client, salvo quando representam estado oficial já existente no Host.

Não consolidar nesta tela:

- fila offline persistente;
- autosave local;
- rascunho local permanente;
- cache autorizado a substituir o Host.

## 37. Contratos Client ↔ Host necessários

A UX depende das categorias funcionais já previstas no contrato:

- `SESSION_REQUIRED` / `SESSION_EXPIRED`;
- `PERMISSION_DENIED`;
- `VALIDATION_FAILED`;
- `RESOURCE_NOT_FOUND`;
- `REVISION_CONFLICT`;
- `CLIENT_HOST_INCOMPATIBLE`;
- `SERVER_BUSY`;
- `PERSISTENCE_ERROR`;
- `INTERNAL_ERROR`.

Códigos mais específicos por domínio podem existir sem quebrar a semântica visual comum.

## 38. Eventos em tempo real

- evento é sinal de mudança, não novo estado oficial completo;
- Client reconsulta;
- evento nunca anuncia mudança não commitada;
- evento mais antigo que estado conhecido pode ser ignorado;
- perda de eventos durante desconexão é tratada por reconsulta após reconexão;
- evento remoto não sobrescreve formulário local silenciosamente.

## 39. Concorrência

- writer/fila coordenam ordem de escrita;
- revisão otimista protege contra sobrescrita;
- UI sempre trata `409`/conflito como decisão consciente;
- fila não autoriza salvar edição obsoleta;
- timeout/desconexão após envio exige reconciliação, não retry cego.

## 40. Divergências com telas anteriores

Esta especificação não revoga estados específicos já aprovados.

Ela os organiza sob um padrão comum. Quando uma tela específica tiver regra mais forte — por exemplo Restore da Tela 13, revisão histórica do Leitor ou ficha de uma A4 da Tela 14 — a regra específica prevalece naquele contexto.

## 41. Decisões consolidadas

1. Tela 15 é contrato transversal, não página navegável;
2. usar a menor superfície adequada: campo → seção → página → Shell;
3. não exibir indicador permanente de conexão saudável;
4. loading não mostra dado antigo como atual;
5. distinguir `sem registros` de `sem resultados`;
6. Host indisponível gera aviso transversal e bloqueia ações dependentes do Host;
7. WebSocket degradado pode ser tratado separadamente quando HTTP continuar saudável;
8. reconexão sempre reconsulta/reconcilia antes de assumir estado atual;
9. mutação com resultado incerto nunca é repetida cegamente;
10. sessão expirada exige nova autenticação e protege conteúdo;
11. edição não salva pode permanecer somente em memória/oculta durante reautenticação do mesmo Client, sem virar draft persistente;
12. perda de permissão limpa conteúdo protegido da superfície;
13. conflito preserva edição local e nunca faz overwrite/merge automático;
14. eventos remotos não substituem formulário local;
15. listas/revisões somente leitura reconsultam sem deslocamento desnecessário;
16. `SERVER_BUSY` preserva dados locais e não confirma sucesso;
17. incompatibilidade Client↔Host bloqueia uso e orienta reabrir pelo ponto de entrada oficial;
18. saída com alterações não salvas pede confirmação proporcional;
19. feedback breve usa toast/inline; condição persistente usa aviso persistente;
20. nenhuma nova persistência/offline queue/autosave é criada pela Tela 15.

## 42. Pendências preservadas

A Tela 15 não resolve:

- lifecycle/status de Atendimento;
- checklist/progresso operacional;
- matriz operacional de permissões;
- parâmetros numéricos finais de autenticação/sessão;
- detalhes técnicos de retry/backoff/timeouts além do já consolidado;
- engine de exportação/impressão;
- mecanismo técnico de Backup/Restore;
- árvore/runtime oficial da implementação.

## 43. Fora do escopo

- modo offline editável;
- fila local persistente;
- autosave de formulários;
- editor colaborativo em tempo real;
- presença/soft lock;
- merge automático de conflitos;
- central de notificações;
- histórico de toasts;
- painel técnico de rede/servidor para usuário final;
- edição manual de IP/porta;
- telemetria externa;
- UI mobile dedicada;
- implementação funcional.

## 44. Critérios de aceite

- [x] PO aprova hierarquia campo/seção/página/Shell;
- [x] PO aprova ausência de indicador permanente de conexão saudável;
- [x] PO aprova padrão de loading e estados vazios;
- [x] PO aprova tratamento de Host indisponível e WebSocket degradado;
- [x] PO aprova reconciliação após desconexão e mutação incerta;
- [x] PO aprova fluxo de sessão expirada;
- [x] PO aprova preservação apenas em memória de edição local durante reautenticação do mesmo Client;
- [x] PO aprova limpeza de conteúdo após perda de permissão;
- [x] PO aprova padrão único de conflitos sem overwrite/merge automático;
- [x] PO aprova proteção de alterações não salvas;
- [x] PO aprova tratamento de incompatibilidade Client↔Host;
- [x] PO aprova distinção entre feedback breve e aviso persistente;
- [x] PO confirma que não há offline queue/autosave/draft persistente;
- [x] nenhuma regra do Bloco 9/10/11 foi antecipada;
- [x] nenhuma implementação funcional foi criada.

## 45. Casos de teste/smoke futuros sugeridos

1. abrir lista com loading e resultado vazio;
2. aplicar filtro que retorna zero resultados;
3. perder Host durante leitura;
4. perder apenas WebSocket e manter HTTP funcional;
5. reconectar e reconsultar estado;
6. perder conexão após enviar uma mutação;
7. sessão expirar durante leitura;
8. sessão expirar com formulário não salvo e reautenticar o mesmo usuário;
9. outro usuário autenticar após expiração com rascunho em memória;
10. permissão ser revogada com tela aberta;
11. recurso desaparecer entre lista e abertura;
12. dois usuários editarem o mesmo recurso e gerar conflito;
13. evento remoto chegar enquanto existe edição local;
14. Host retornar `SERVER_BUSY`;
15. Client/Host incompatíveis antes do login;
16. tentar sair com alterações não salvas;
17. operação longa perder a conexão do Client;
18. erro interno não expor stack/SQL/path/segredo;
19. navegação por teclado em modal/erro;
20. janela menor manter mensagens e ações utilizáveis.