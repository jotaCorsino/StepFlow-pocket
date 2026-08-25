# Tela 13 — Backup / Restauração — UX

## 1. Identificação

- código/nome da tela: Tela 13 — Backup / Restauração — UX;
- status: **CONSOLIDADO / APROVADO PELO PO**;
- bloco: Fase 1 — Bloco 8 (UI/UX);
- data da consolidação: 2026-08-25.

## 2. Objetivo

Definir a experiência administrativa para criar, consultar e restaurar backups do StepFlow sem expor o usuário a cópia manual do SQLite, caminhos técnicos do Host ou operações destrutivas pouco claras.

A Tela 13 fecha:

- acesso e navegação da UX de backup/restauração;
- informações apresentadas sobre backups conhecidos pelo Host;
- fluxo visual de criação de backup;
- fluxo visual, validações e confirmações de restauração;
- permissões visíveis;
- estados de carregamento, execução, sucesso e falha;
- limite entre operação normal e recuperação técnica de desastre.

A estratégia técnica de consistência, empacotamento, atomicidade, retenção, compatibilidade, storage e recuperação quando o Host não consegue iniciar permanece para o **Bloco 11 — Backup/Restauração**.

## 3. Princípios consolidados

- backup/restore é coordenado pelo Host;
- Client nunca copia `stepflow.sqlite` diretamente;
- autorização real é Host-side e baseada em capacidades;
- `Backup`: ADM = sim; Gerência = **PENDENTE**; Funcionário = não;
- `Restore`: ADM = sim; Gerência = não; Funcionário = não;
- conceder Backup nunca concede Restore por consequência;
- dados persistentes e arquivos administrados devem ser tratados como conjunto coerente;
- operação administrativa crítica deve ser auditável;
- path, IP, porta e detalhes internos não são editáveis pela UX normal;
- backup não é exportação de procedimento e não substitui PDF/DOCX/impressão;
- restauração substitui o estado ativo e exige fricção deliberada;
- nenhuma operação é declarada concluída antes de confirmação do Host.

## 4. Posição na navegação

Backup/Restauração permanece dentro de `Configurações` como terceira seção local:

```text
Configurações

[ Empresa ] [ Categorias ] [ Backup e restauração ]
```

Regras:

- não criar novo item global na sidebar apenas para Backup;
- a seção aparece somente quando a sessão possui capacidade aplicável;
- se a sessão possuir Backup mas não Empresa/Categorias, `Configurações` continua visível e abre a seção autorizada;
- Restore não recebe aba própria: é ação crítica dentro da seção de backups;
- a extensão não altera o contrato já consolidado da Tela 12 para Empresa/Categorias.

## 5. Atores e permissões visíveis

### ADM

Pode, conforme matriz vigente:

- criar backup;
- consultar backups disponíveis;
- abrir detalhes;
- iniciar restauração de backup elegível.

### Gerência

A capacidade de **Backup permanece PENDENTE**.

Portanto:

- não assumir `sim` nem `não` antes da decisão final;
- Restore permanece não autorizado para Gerência;
- futura autorização de Backup, se aprovada, não autoriza Restore.

### Funcionário

Não recebe Backup nem Restore por padrão.

Ocultar ações no Client não substitui validação Host-side. Acesso manipulado retorna permissão negada sem expor conteúdo administrativo indevido.

## 6. Estrutura visual consolidada

```text
Configurações
[ Empresa ] [ Categorias ] [ Backup e restauração ]

BACKUP
Proteja os dados e arquivos administrados pelo StepFlow.

Último backup concluído
25/08/2026 08:32 · Sistema · 128 MB

                                      [ Criar backup agora ]

BACKUPS DISPONÍVEIS

Data e hora        Origem     Criado por       Tamanho    Verificação
25/08 08:32        Sistema    StepFlow         128 MB     Íntegro        [ ⋯ ]
24/08 17:48        Manual     Maria Souza      127 MB     Íntegro        [ ⋯ ]
23/08 18:05        Manual     João Lima        126 MB     Íntegro        [ ⋯ ]

[ Atualizar lista ]
```

Direção visual:

- corporativa, compacta e administrativa;
- sem dashboard, gráficos ou KPIs decorativos;
- risco de Restore deve ser claro sem transformar a tela em painel técnico.

## 7. Conteúdo protegido — comunicação ao usuário

A UX pode informar de forma resumida que o backup protege o estado persistente do StepFlow, incluindo, conforme contrato técnico final:

- procedimentos e revisões;
- usuários/permissões;
- categorias;
- atendimentos;
- equipamentos;
- identidade/logo da empresa;
- avatares;
- demais arquivos administrados necessários à restauração coerente.

A lista técnica definitiva pertence ao Bloco 11.

Não prometer que binários, logs ou configuração operacional de rede fazem parte do pacote sem decisão técnica explícita.

## 8. Lista de backups

A lista mostra backups conhecidos e administrados pelo Host.

Metadados iniciais:

- data/hora de criação;
- origem;
- usuário responsável quando houver ator humano;
- tamanho aproximado;
- estado/verificação informado pelo Host.

Ordenação inicial: mais recente primeiro.

Não incluir inicialmente:

- exclusão de backup pela UI;
- seleção em massa;
- política de retenção configurável;
- agendamento recorrente;
- edição do nome do arquivo;
- edição do path de armazenamento;
- download/exportação genérica para o Client;
- upload/importação genérica de pacote externo.

## 9. Origem do backup

A UX pode distinguir:

- `Manual` — iniciado por usuário autorizado;
- `Sistema` — criado por fluxo técnico aprovado, por exemplo proteção antes de operação crítica ou migration quando definido tecnicamente.

`Sistema` não significa que existe scheduler periódico. Nenhum agendador recorrente é aprovado nesta tela.

## 10. Detalhes do backup

Ação `Detalhes` abre painel/modal simples:

```text
Detalhes do backup

Criado em          24/08/2026 17:48
Origem             Manual
Criado por         Maria Souza
Tamanho            127 MB
Versão StepFlow    1.x
Verificação        Íntegro
Compatibilidade    Compatível para restauração

[ Fechar ]                         [ Restaurar este backup* ]
```

`*` aparece somente quando:

- a sessão possui Restore;
- o Host validou o backup como elegível.

A versão exibida é informativa. Compatibilidade real é decisão do Host.

## 11. Estados de verificação e compatibilidade

A UX deve representar pelo menos:

- verificando;
- íntegro/elegível;
- inválido ou corrompido;
- incompatível;
- indisponível temporariamente;
- verificação falhou.

`Restaurar` permanece indisponível se o Host não considerar o backup seguro/elegível.

Checksums, manifesto, schema e algoritmo de compatibilidade pertencem ao Bloco 11.

## 12. Criar backup agora

Fluxo:

```text
Criar backup agora
→ Host valida sessão + capacidade
→ Host aceita ou rejeita a operação
→ UI mostra operação em andamento
→ Host produz backup consistente conforme Bloco 11
→ Host confirma conclusão
→ UI atualiza lista
```

Regras:

- não pedir escolha do SQLite;
- não pedir seleção manual de componentes;
- impedir disparos duplicados acidentais;
- confirmação simples pode ser usada: `Criar um novo backup do estado atual do StepFlow?`;
- feedback deve existir desde a aceitação até a conclusão.

## 13. Backup em andamento

Estado visual:

```text
Criando backup…
O StepFlow está preparando uma cópia consistente dos dados.
```

Regras:

- desabilitar novo backup enquanto houver operação incompatível em andamento;
- não inventar percentual sem progresso real fornecido pelo Host;
- usar estado indeterminado quando necessário;
- permitir refetch do estado;
- não declarar sucesso antes da confirmação do Host.

Se o Host já aceitou a operação, fechar um Client **não significa cancelamento silencioso**. O estado deve permanecer consultável posteriormente.

## 14. Uso do sistema durante backup

A Tela 13 não fixa se mutações comuns continuam disponíveis durante toda a criação.

A UX deve suportar o mecanismo que o Bloco 11 aprovar:

- backup sem bloquear uso normal; ou
- pequena janela de indisponibilidade/mutações suspensas, se tecnicamente necessária.

Se houver restrição temporária, os Clients recebem estado compreensível, não erro técnico bruto.

## 15. Backup concluído ou falhou

Sucesso:

```text
✓ Backup criado com sucesso
25/08/2026 08:32
```

O item confirmado deve aparecer na lista; não depender somente de toast efêmero.

Falhas devem suportar situações como:

- armazenamento indisponível;
- espaço insuficiente;
- leitura/escrita falhou;
- operação incompatível já em andamento;
- Host indisponível;
- autorização perdida;
- validação final falhou.

Mensagem administrativa sugerida:

`Não foi possível concluir o backup. Nenhum backup novo foi confirmado.`

Detalhes técnicos ficam nos logs quando apropriado.

## 16. Restauração — princípio de segurança

Restore substitui o estado atual do StepFlow por um estado anterior.

Não oferecer Restore:

- diretamente na linha sem abrir `Detalhes`;
- com um único clique irreversível;
- para backup inválido/incompatível;
- para sessão sem capacidade Restore;
- enquanto o Host considerar a operação insegura;
- durante operação administrativa incompatível.

## 17. Fluxo de restauração consolidado

```text
selecionar backup
→ Detalhes
→ Host valida integridade + compatibilidade + autorização
→ Restaurar este backup
→ confirmação reforçada
→ criar e confirmar safety backup do estado atual
→ Host entra na etapa destrutiva
→ Clients recebem estado de manutenção/desconexão quando necessário
→ Host conclui e valida estado restaurado
→ Client reconecta/refaz sessão conforme regra técnica
→ UI apresenta resultado confirmado
```

Restart do Host, troca de arquivos, fechamento de conexões e demais detalhes internos pertencem ao Bloco 11.

## 18. Safety backup pré-restauração — consolidado

Para o fluxo **normal pela UI**, preservar o estado imediatamente anterior ao Restore é obrigatório.

```text
Restore solicitado
→ criar backup de segurança do estado atual
→ validar/confirmar esse backup
→ somente então iniciar substituição destrutiva
```

Regra consolidada:

- se o safety backup não puder ser criado e confirmado, o Restore normal pela UI **não prossegue**.

Exceção conceitual:

- se o estado atual já estiver ilegível/corrompido e essa proteção for impossível, trata-se de **disaster recovery local/controlado**, fora da Tela 13 e definido no Bloco 11.

## 19. Confirmação reforçada de Restore

Diálogo:

```text
Restaurar backup de 24/08/2026 17:48?

O estado atual do StepFlow será substituído pelo conteúdo deste backup.
Alterações realizadas depois desse ponto poderão deixar de fazer parte do estado ativo.
Usuários conectados podem ser temporariamente desconectados.

[ ] Entendo que esta operação substitui o estado atual.

Digite RESTAURAR para confirmar:
[                         ]

[ Cancelar ]                         [ Restaurar ]
```

Regras:

- botão só habilita após ciência explícita + texto exigido;
- ação destrutiva visualmente distinta sem depender apenas de cor;
- `Esc`/Cancelar funcionam antes da etapa destrutiva;
- depois que o Host entra na fase não cancelável, não exibir falso botão de cancelamento.

## 20. Usuários conectados durante Restore

A UX informa antecipadamente que o uso pode ser interrompido temporariamente.

A Tela 13 não fixa:

- quantidade de Clients permitida;
- timeout;
- sequência técnica de desconexão;
- necessidade de restart do Controller/Host;
- invalidação/reuso exatos das sessões.

Esses pontos pertencem ao Bloco 11 e ao contrato de autenticação quando aplicável.

A interrupção deve parecer **manutenção coordenada**, nunca falha aleatória de rede.

## 21. Restauração em andamento

Estado visual:

```text
Restaurando backup…
Não feche o ciclo central do StepFlow enquanto a operação estiver em andamento.
```

O Client pode perder conexão conforme o mecanismo técnico final.

Ao reconectar, consulta o resultado conhecido pelo Host; não infere sucesso ou falha apenas pela queda da conexão.

## 22. Resultado do Restore

Sucesso confirmado:

```text
✓ Restauração concluída
Backup restaurado: 24/08/2026 17:48
```

Se a política final exigir novo login, encaminhar para Login com mensagem apropriada.

Não prometer preservação da sessão atual antes do Bloco 11 fechar essa regra.

Falhas podem ser classificadas pelo Host como:

- restauração não iniciada;
- bloqueada por validação/compatibilidade;
- falha antes da substituição;
- falha durante operação crítica;
- resultado incerto exigindo intervenção administrativa.

Em resultado incerto:

- não declarar sucesso;
- orientar a não continuar mutações;
- encaminhar para procedimento de recuperação/diagnóstico do Bloco 11.

## 23. Concorrência de operações administrativas

Backup/Restore são coordenados pelo Host.

Exemplos:

- segundo backup enquanto outro está ativo;
- Restore durante backup incompatível;
- dois ADMs tentando restaurar simultaneamente.

O Host decide aceitação/serialização. A UI apresenta `operação em andamento` em vez de criar fluxos críticos paralelos.

A política técnica de lock/fila pertence ao Bloco 11.

## 24. Eventos e atualização em tempo real

Eventos podem sinalizar:

- backup iniciado;
- backup concluído/falhou;
- verificação atualizada;
- restauração preparada/iniciada;
- manutenção temporária;
- restauração concluída/falhou.

Clients autorizados fazem refetch. Nenhum Client infere resultado definitivo apenas por evento quando reconsulta for necessária.

## 25. Auditoria

Registrar de forma proporcional:

- quem solicitou backup manual;
- identificador do backup criado;
- quem iniciou Restore;
- backup selecionado;
- resultado da operação;
- falhas administrativas relevantes.

Não registrar conteúdo completo do backup, senha, token reutilizável ou dados sensíveis desnecessários.

## 26. Backup × Exportação

```text
Backup
= recuperação do estado do sistema

Exportação / impressão
= documento para leitura, compartilhamento ou uso físico
```

A Tela 13 não oferece PDF, DOCX nem ficha A4.

## 27. Disaster recovery fora da UX normal

Quando a própria aplicação não consegue disponibilizar a Tela 13, por exemplo:

- Host não inicia;
- SQLite não abre;
- schema não valida;
- arquivos essenciais estão corrompidos;

a recuperação ocorre por fluxo **local/controlado na máquina central**, especificado no Bloco 11.

A Tela 13 não finge resolver um desastre que impede o próprio StepFlow de subir.

## 28. Caminhos e arquivos

A UX normal não oferece:

- campo para editar diretório de backup;
- campo para apontar diretamente para `stepflow.sqlite`;
- navegador de pasta do servidor;
- nome técnico do arquivo como requisito do usuário;
- edição de `stepflow-host.toml`.

Eventual exportação/importação de pacotes para armazenamento externo exige contrato futuro próprio.

## 29. Estados da interface

### Carregando

Preservar estrutura enquanto lista/metadados são consultados.

### Nenhum backup

`Nenhum backup disponível.`

Se autorizado:

`Crie o primeiro backup para proteger o estado atual do StepFlow.`

### Sem permissão

Se nenhuma capacidade aplicável existir, a seção fica oculta.

### Host indisponível

Mostrar indisponibilidade e impedir operação sem oferecer edição de IP/porta/path.

### Operação em andamento

Mostrar o tipo de operação coordenada e impedir duplicação incompatível.

### Backup inválido/incompatível

Exibir motivo em linguagem administrativa e manter Restore indisponível.

## 30. Acessibilidade e janela menor

- tabela/lista semanticamente identificada;
- estados não dependem apenas de cor;
- ações críticas têm nomes explícitos;
- confirmação de Restore acessível por teclado;
- foco entra e retorna corretamente dos diálogos;
- status de operação anunciável quando apropriado;
- erros associados à ação/backup correspondente;
- não depender de hover para informação essencial.

Em janela menor suportada:

- priorizar data/hora + origem + verificação;
- metadados secundários podem migrar para `Detalhes`;
- ações permanecem em menu contextual acessível;
- não criar experiência mobile/hamburger.

## 31. Fora do escopo

- SQLite backup API/`VACUUM INTO`/cópia coordenada;
- formato final do pacote;
- algoritmo de checksum/manifesto;
- compressão;
- criptografia do pacote;
- retenção automática;
- agendamento periódico;
- limpeza/exclusão de backups;
- destino externo/nuvem;
- upload/download genérico;
- recovery quando Host não inicia;
- política técnica de restart durante Restore;
- tempos máximos/timeouts;
- espaço mínimo livre;
- implementação funcional.

## 32. Pendências preservadas

A consolidação **não resolve**:

- se Gerência recebe capacidade de Backup;
- formato/estrutura do pacote;
- lista técnica exata de arquivos incluídos;
- política de retenção;
- necessidade de criptografia;
- comportamento técnico durante mutações concorrentes;
- restart/reconexão/sessões exatos após Restore;
- estratégia de disaster recovery;
- limites de tamanho/espaço/tempo;
- política de backups automáticos além de safety backups exigidos por fluxos técnicos aprovados.

## 33. Decisões consolidadas

1. Backup/Restauração permanece dentro de `Configurações` como terceira seção local autorizada;
2. não existe novo item global na sidebar;
3. `Criar backup agora` é a ação principal de criação;
4. Gerência × Backup permanece `PENDENTE`; Restore de Gerência permanece não autorizado;
5. lista compacta mostra data/hora, origem, autor, tamanho e verificação;
6. `Detalhes` antecede qualquer Restore;
7. não há delete, scheduler, retention, upload/download ou path editável inicialmente;
8. backup aceito pelo Host não é cancelado silenciosamente ao fechar Client;
9. Restore só aparece para capacidade correspondente e backup elegível;
10. Restore exige confirmação reforçada com ciência explícita + texto `RESTAURAR`;
11. safety backup do estado atual é obrigatório antes da etapa destrutiva do Restore normal via UI;
12. sem safety backup confirmado, o Restore normal não prossegue;
13. Clients podem ser temporariamente interrompidos durante Restore, com mecanismo exato reservado ao Bloco 11;
14. após queda/reconexão, o Client consulta o resultado do Host;
15. disaster recovery sem Host funcional é local/controlado e pertence ao Bloco 11;
16. Backup permanece separado de Exportação/Impressão.

## 34. Critérios de aceite do checkpoint

- [x] PO aprovou posição em Configurações;
- [x] PO aprovou lista compacta de backups;
- [x] PO aprovou metadados iniciais;
- [x] PO aprovou criação manual simples;
- [x] PO aprovou ausência de scheduler/retention/delete/import/export inicialmente;
- [x] PO aprovou Restore somente após Detalhes + validação Host;
- [x] PO aprovou confirmação reforçada de Restore;
- [x] PO aprovou safety backup pré-restauração no fluxo normal;
- [x] permissão de Gerência para Backup permanece pendente;
- [x] Restore de Gerência permanece não autorizado;
- [x] mecanismo técnico permanece para Bloco 11;
- [x] recovery sem Host permanece para Bloco 11;
- [x] Tela 14 não foi antecipada;
- [x] nenhuma implementação funcional foi criada.
