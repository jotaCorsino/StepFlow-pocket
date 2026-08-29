# Tela 15 — Estados Transversais

## 1. Identificação

- status: **CONSOLIDADO / APROVADO PELO PO**;
- natureza: contrato transversal, não página navegável;
- atualização: 2026-08-29.

## 2. Objetivo

Padronizar estados compartilhados entre as superfícies do StepFlow para que loading, falha, indisponibilidade, conflito e feedback não sejam tratados de forma inconsistente em cada tela.

## 3. Princípios gerais

1. Host continua fonte oficial de dados e autorização;
2. cache/estado antigo do Client nunca vira fonte oficial silenciosamente;
3. falhas locais permanecem locais quando possível;
4. problemas globais só ocupam Shell quando afetam a aplicação como um todo;
5. mensagens explicam o ocorrido e a ação segura quando existir;
6. não expor stack trace, SQL, path interno, token ou segredo;
7. não apagar edição local silenciosamente por erro, conflito ou reconexão;
8. não confirmar salvamento antes de commit confirmado;
9. não inventar percentual de progresso;
10. estado importante não depende apenas de cor/ícone;
11. evitar toast + banner + modal redundantes para a mesma condição;
12. estado saudável permanece discreto.

## 4. Hierarquia de apresentação

Usar a menor superfície suficiente:

```text
campo/componente
→ erro ou feedback junto ao componente

seção/recurso
→ aviso inline / estado da área

página inteira
→ estado de página

aplicação/Host/sessão
→ aviso transversal ou bloqueio global
```

## 5. Padrões transversais

- Loading/Skeleton;
- Estado vazio;
- Aviso inline;
- Erro inline;
- Banner transversal;
- Toast/feedback breve;
- Modal de confirmação;
- Estado de página;
- Estado de operação em andamento.

Nomes físicos de componentes ficam para implementação.

## 6. Loading

### Página/seção

- preservar Shell quando sessão continua válida;
- usar skeleton/placeholders na região necessária;
- não mostrar dado antigo como atual;
- evitar spinner global quando apenas uma área carrega;
- preservar busca/filtros durante reconsulta quando seguro;
- falha deve evoluir para estado real, não loading infinito.

### Ação explícita

Para `Salvar`, `Publicar`, `Criar backup`, `Exportar` etc.:

- a própria ação indica estado em andamento;
- impedir disparo duplicado acidental;
- bloquear somente o necessário;
- sucesso só após confirmação correspondente;
- operação longa usa progresso indeterminado se não houver percentual real.

## 7. Estados vazios

Distinguir:

- nenhum registro existente;
- nenhum resultado com filtros atuais.

Ação contextual de criação aparece somente quando autorizada e útil.

## 8. Host indisponível

Quando HTTP/Host oficial está indisponível:

- Shell pode permanecer para contexto seguro;
- conteúdo que depende do Host não é apresentado como atual;
- mutações são bloqueadas;
- não oferecer edição manual de IP/porta/path ao usuário final;
- ação principal é tentar novamente/reabrir pelo fluxo oficial quando aplicável.

Não criar modo offline editável por inferência.

## 9. WebSocket degradado

Se HTTP continua saudável e apenas eventos estão degradados:

- não declarar aplicação inteira offline;
- sinalizar condição de atualização em tempo real quando relevante;
- reconsulta explícita continua possível;
- após reconexão, reconciliar estado.

## 10. Reconexão

```text
conexão perdida
→ preservar contexto local seguro
→ reconectar com backoff limitado
→ revalidar sessão/compatibilidade
→ reconsultar recursos relevantes
→ reconciliar
```

Não assumir replay completo de eventos perdidos.

## 11. Sessão expirada

- exigir nova autenticação;
- conteúdo protegido não permanece exposto indefinidamente;
- edição não salva pode permanecer somente em memória/oculta no mesmo Client para reautenticação do mesmo usuário quando seguro;
- autenticação de outro usuário não herda rascunho protegido do anterior;
- nenhum draft persistente é criado por essa regra.

## 12. Sem permissão / permissão revogada

- Host é autoridade final;
- remover/ocultar ações não basta como segurança;
- se permissão for revogada com tela aberta, limpar conteúdo protegido conforme necessário e direcionar para destino autorizado;
- não manter dado sensível apenas porque já estava em memória visual.

## 13. Recurso indisponível

Se recurso deixou de existir/estar acessível entre lista e abertura:

- mostrar estado de página simples;
- oferecer retorno ao domínio correspondente;
- não apresentar snapshot antigo como registro atual oficial.

## 14. Conflito

Conflito de revisão:

- preservar edição local;
- explicar que recurso mudou;
- oferecer reconsulta/revisão consciente;
- não overwrite automático;
- não merge automático;
- granularidade deve seguir o recurso: Atendimento, Equipamento, item de checklist, observação por Etapa etc.

## 15. Evento remoto durante edição local

Evento é sinal de mudança, não ordem para substituir o formulário.

- marcar que estado remoto mudou;
- reconsultar quando seguro;
- preservar texto/formulário local em edição;
- reconciliar conscientemente quando a mesma unidade foi alterada.

## 16. `SERVER_BUSY` / backpressure

Quando Host estiver saturado:

- informar indisponibilidade temporária sem culpar usuário;
- preservar dados locais não confirmados;
- não mostrar sucesso;
- permitir retry apenas quando seguro;
- mutação potencialmente aceita/commitada exige reconciliação antes de repetir.

## 17. Resultado incerto de mutação

Se conexão cai após envio:

```text
enviou mutação
→ conexão caiu
→ resultado desconhecido
→ reconectar
→ consultar estado
→ somente então decidir próximo passo
```

Nunca repetir cegamente comando não idempotente.

## 18. Incompatibilidade Client ↔ Host

- bloquear uso antes do fluxo normal;
- explicar que versão não é compatível;
- orientar fechamento/reabertura pelo Launcher;
- não permitir “continuar mesmo assim”.

## 19. Alterações não salvas

Ao sair de formulário com mudança local relevante:

- pedir confirmação proporcional;
- não usar confirmação para navegação sem alteração;
- não salvar automaticamente apenas porque usuário tentou sair.

## 20. Feedback de cópia e ações leves

Ações como copiar comando/código usam feedback breve (`Copiado` ou equivalente), sem modal e sem ruído persistente.

## 21. Confirmações destrutivas

Usar fricção proporcional ao risco:

- arquivar/desativar: confirmação simples quando necessária;
- remover vínculo: confirmar somente se houver consequência relevante;
- Restore: confirmação reforçada conforme Tela 13.

Texto digitado é reservado a risco alto, não a ações comuns.

## 22. Operações longas

Geração documental, Backup e Restore podem durar mais que ação normal.

- mostrar qual operação está em andamento;
- não inventar porcentagem;
- distinguir `em andamento`, `concluído`, `falhou`, `cancelado` quando aplicável;
- fechar Client não significa automaticamente cancelar operação Host-side já aceita;
- após reconexão, consultar estado quando a operação possuir continuidade;
- não mostrar botão `Cancelar` após ponto não cancelável.

Detalhes específicos pertencem às Telas 13/14 e contratos técnicos correspondentes.

## 23. Mensagens e microcopy

Direção:

- frase curta;
- linguagem operacional, não técnica;
- explicar ação segura quando existir;
- evitar códigos internos como mensagem principal;
- não usar `Erro desconhecido` como única informação quando categoria funcional for conhecida.

Exemplos:

- `Não foi possível conectar ao StepFlow.`;
- `Sua sessão expirou. Entre novamente.`;
- `Este registro foi alterado por outro usuário.`;
- `Nenhum resultado encontrado com os filtros atuais.`;
- `Salve as alterações antes de gerar a ficha.`

## 24. Toast versus aviso persistente

Toast/feedback breve:

- copiado;
- salvo;
- conexão restabelecida;
- operação concluída quando não exige decisão.

Aviso persistente/inline:

- Host indisponível;
- WebSocket degradado;
- conflito;
- revisão mais nova disponível;
- formulário inválido;
- operação crítica em andamento.

## 25. Prioridade entre estados

1. segurança/autorização/incompatibilidade;
2. indisponibilidade global/estado da sessão;
3. conflito/resultado incerto;
4. validação/erro da operação atual;
5. avisos informativos;
6. sucesso.

Não sobrepor mensagens contraditórias.

## 26. Preservação de contexto

Quando seguro, preservar:

- busca;
- filtros;
- ordenação;
- página/scroll;
- Etapa/revisão do Reader;
- edição local não salva em memória.

Não preservar conteúdo protegido após perda definitiva de autorização.

## 27. Navegação segura após erro

- Processo indisponível → voltar para Processos;
- Atendimento indisponível → voltar para Atendimentos;
- sem permissão → destino autorizado anterior ou Início;
- incompatibilidade → fechar/reabrir pelo fluxo oficial;
- falha local de modal → fechar modal mantendo página subjacente quando seguro.

## 28. Acessibilidade

- mensagens assíncronas importantes anunciáveis;
- foco não é roubado por todo toast;
- modal gerencia e devolve foco;
- erro de formulário permite alcançar primeiro campo inválido;
- estados não dependem só de cor;
- loading evita animação excessiva;
- teclado permanece funcional quando ação estiver habilitada.

## 29. Janelas suportadas

- banner não cobre sidebar/ações principais;
- estados vazios permanecem compactos;
- modais/painéis podem empilhar ações;
- mensagens quebram linha sem scroll horizontal;
- sem UI mobile/hamburger inicial.

## 30. Persistência

A Tela 15 não cria persistência funcional.

Não consolidar aqui:

- fila offline persistente;
- autosave local;
- rascunho local permanente;
- cache autorizado a substituir o Host.

## 31. Contratos Client ↔ Host

Categorias comuns:

- `SESSION_REQUIRED` / `SESSION_EXPIRED`;
- `PERMISSION_DENIED`;
- `VALIDATION_FAILED`;
- `RESOURCE_NOT_FOUND`;
- `REVISION_CONFLICT`;
- `CLIENT_HOST_INCOMPATIBLE`;
- `SERVER_BUSY`;
- `PERSISTENCE_ERROR`;
- `INTERNAL_ERROR`.

Códigos específicos de domínio podem existir sem quebrar a semântica visual comum.

## 32. Pendências realmente externas a esta tela

A Tela 15 não decide:

- parâmetros numéricos finais de autenticação/sessão;
- timeouts/backoff quantitativos;
- mecanismo técnico de Backup/Restore;
- estrutura/runtime oficial da implementação;
- gates corporativos de Windows/WebView2/SMB/EDR.

Lifecycle, checklist, matriz operacional e saída documental **já possuem contratos específicos consolidados** e não são pendências desta tela.

## 33. Fora do escopo

- modo offline editável;
- fila local persistente;
- autosave de formulários;
- editor colaborativo em tempo real;
- presença/soft lock;
- merge automático;
- central de notificações;
- painel técnico de rede para usuário final;
- edição manual de IP/porta;
- telemetria externa;
- UI mobile dedicada;
- implementação funcional nesta fase.
