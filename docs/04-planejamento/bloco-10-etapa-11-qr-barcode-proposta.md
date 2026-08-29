# Bloco 10 — Etapa 11 — QR / barcode — Proposta para análise

**Status:** PROPOSTA / AGUARDANDO APROVAÇÃO DO PO  
**Fase:** Fase 1 — Fechamento arquitetural e especificação  
**Abertura:** 2026-08-29  
**Base consolidada:** Bloco 10 / Etapas 1–10  
**Base Git:** `main` em `a6b0a8520c73a6b693cc929b1ebbc540f450ad66`

## 1. Objetivo

Decidir se QR code ou barcode agrega benefício operacional real aos documentos físicos do StepFlow na versão inicial.

A Etapa 11 não parte do pressuposto de que o recurso precisa existir. O contrato já consolidado estabelece que QR/barcode **não é requisito por padrão** e só deve entrar se houver fluxo de leitura concreto, destino estável e ganho claro para o usuário.

## 2. Contratos herdados

Permanecem vigentes:

- Procedimento físico é documento técnico A4 multipágina;
- Ficha é prestação de contas resumida ao cliente e deve caber em exatamente uma A4;
- baixa densidade visual é princípio transversal;
- não reservar espaço em documento para elementos sem utilidade comprovada;
- códigos humanos já existem (`PR-...`, `AT-...`, `EQP-...`);
- revisão exata do Procedimento é preservada;
- Ficha não deve virar relatório técnico, etiqueta patrimonial ou formulário carregado;
- não criar integração externa/cloud obrigatória;
- autenticação/autorização continuam Host-side;
- não expor segredo, sessão, token ou dado operacional desnecessário em artefato físico.

## 3. Pergunta operacional

Para justificar QR/barcode, a leitura precisa responder:

```text
alguém aponta um leitor/câmera
→ obtém uma ação útil e previsível
→ com menos esforço/erro do que usar o código textual já impresso
```

Hoje não existe contrato aprovado para:

- aplicativo móvel do StepFlow;
- portal web público/interno aberto por câmera;
- protocolo/deep link `stepflow://...` registrado no Windows;
- leitor de barcode integrado ao Client;
- fluxo de etiquetas de Equipamento;
- estação de recepção/triagem por scanner;
- URL estável que permaneça válida entre ambientes/hosts;
- autenticação automática segura após leitura física.

Sem um desses fluxos, QR/barcode seria essencialmente decorativo ou duplicaria um código já legível.

## 4. Direção proposta

### 4.1 Baseline da v1

**Não adicionar QR code nem barcode aos Procedimentos ou à Ficha na versão inicial.**

Consequências:

- template físico de Procedimento permanece sem área reservada para QR/barcode;
- template A4 da Ficha permanece sem QR/barcode;
- não alterar margens, footer, cabeçalho ou densidade para acomodar marcador;
- não adicionar biblioteca/dependência de geração de QR/barcode;
- não criar endpoint/deep link apenas para justificar o elemento gráfico;
- Etapa 12 não precisa validar scanner/decodificação na baseline da v1.

Essa decisão não proíbe evolução futura; apenas evita implementar infraestrutura sem caso de uso aprovado.

## 5. Por que o código textual já é suficiente hoje

Procedimentos, Atendimentos e Equipamentos já possuem referências humanas compactas.

Exemplos:

```text
PR-014
AT-000142
EQP-0031
```

Esses códigos:

- funcionam em papel sem tecnologia adicional;
- podem ser digitados/pesquisados;
- são copiáveis de PDF/DOCX;
- não dependem de câmera, scanner ou software associado;
- continuam compreensíveis mesmo se a infraestrutura mudar.

Um QR que apenas codifique `AT-000142` ou `PR-014` não acrescenta benefício suficiente para justificar espaço e complexidade na baseline.

## 6. QR contendo URL

Não propomos codificar URL na v1.

Motivos:

- hostname/IP/path reais ainda pertencem à validação do ambiente corporativo;
- implantação Pocket pode variar entre ambientes;
- URL impressa fica permanente enquanto endpoints podem mudar;
- câmera de celular pode não alcançar a LAN corporativa;
- abrir URL não resolve autenticação/autorização;
- expor hostname, IP, porta ou rota interna em papel não deve ser feito sem necessidade operacional;
- criar portal HTTP específico apenas para o QR ampliaria o produto sem requisito.

É proibido usar como payload por inferência:

```text
http://<host>:<porta>/...
https://<host>/...
file://...
\\servidor\share\...
```

A Etapa 11 não congela URL interna ou topologia de implantação.

## 7. QR contendo deep link

Também não propomos criar esquema `stepflow://...` nesta fase.

Um deep link útil exigiria fechar, no mínimo:

- registro do protocolo no Windows;
- parsing e validação do payload;
- comportamento quando Client não está instalado/aberto;
- descoberta/conexão ao Host correto;
- login/sessão/autorização após abertura;
- comportamento para revisão inexistente ou não autorizada;
- compatibilidade entre versões do Client;
- segurança contra payload manipulado;
- política de links antigos impressos.

Isso é um novo contrato de navegação/integração, não um detalhe de impressão. Não deve ser criado somente para preencher um QR.

## 8. Barcode linear

Barcode linear também não entra na baseline.

Hoje não existe fluxo aprovado com leitor dedicado em:

- recepção;
- bancada técnica;
- estoque;
- patrimônio;
- triagem;
- pesquisa de Atendimento/Equipamento.

Além disso, barcode faria mais sentido em **etiquetas físicas de ativos ou fluxo de scanner**, que é um problema diferente da exportação documental do Bloco 10.

Se esse caso surgir no futuro, deve ser especificado no contexto operacional correto e não introduzido silenciosamente no template da Ficha.

## 9. Não usar QR como armazenamento de dados

Mesmo em evolução futura, QR/barcode não deve funcionar como mini-banco de dados offline.

Não codificar por padrão:

- nome de cliente/solicitante;
- resumo ou observações do serviço;
- serial, patrimônio ou MAC;
- conteúdo de Procedimento;
- checklist;
- credenciais;
- token de sessão;
- API key;
- segredo assinado de longa duração;
- dados pessoais desnecessários;
- configuração de rede;
- path SMB;
- payload JSON completo do domínio.

O marcador, se algum dia aprovado, deve carregar **referência mínima**, não conteúdo operacional sensível.

## 10. Não usar QR para contornar autenticação

Leitura física nunca equivale a autorização.

```text
scan
→ identifica uma referência
→ Client/Host autentica e autoriza normalmente
→ somente então abre o recurso permitido
```

Não criar QR que conceda acesso permanente ou contenha credencial reutilizável para evitar login/permissão.

## 11. Revisões de Procedimento

Se no futuro existir navegação por QR para Procedimentos, uma decisão explícita será necessária entre:

```text
referência ao Procedimento atual
ou
referência à revisão exata impressa
```

Para documento físico versionado, a direção mais coerente seria preservar a **revisão exata impressa**, evitando que um papel de r18 leve silenciosamente a r25.

Mas essa semântica não precisa ser implementada agora porque o fluxo de scan ainda não existe.

## 12. Ficha de Atendimento

A Ficha possui restrição mais forte: exatamente uma A4 e baixa densidade.

Na baseline proposta:

- nenhum QR/barcode;
- nenhum espaço reservado vazio;
- nenhum código gráfico duplicando `AT-...`;
- nenhuma redução de área de `SERVIÇO REALIZADO`/`OBSERVAÇÕES` para acomodar marcador;
- nenhum QR para cliente acessar portal inexistente.

O código textual do Atendimento permanece suficiente para referência.

## 13. Procedimento físico

No Procedimento:

- código, título, versão/revisão e paginação já identificam o documento;
- QR não é usado como enfeite no cabeçalho/rodapé;
- não repetir um mesmo QR em todas as páginas;
- não reservar área para eventual recurso futuro.

Se um fluxo real for aprovado mais tarde, o posicionamento físico poderá ser decidido então sem comprometer o template atual.

## 14. Critério para reabrir a decisão no futuro

QR/barcode só deve ser reconsiderado se existir um caso operacional explícito, por exemplo:

1. técnico escaneia um Procedimento impresso e abre exatamente a revisão correspondente no Client;
2. recepção escaneia uma Ficha e localiza o Atendimento sem digitação;
3. etiqueta física de Equipamento é aprovada e scanners passam a fazer parte do fluxo;
4. cliente possui um portal real e aprovado para consultar determinado recurso.

A proposta futura precisa definir:

- quem escaneia;
- com qual dispositivo/software;
- qual payload mínimo;
- qual destino/ação;
- comportamento offline/inacessível;
- autenticação/autorização;
- longevidade do identificador;
- benefício sobre digitação/pesquisa textual;
- impacto físico no documento;
- segurança/privacidade.

Sem essas respostas, o recurso continua fora da baseline.

## 15. Se um marcador for aprovado futuramente

Diretrizes já recomendadas para não repetir a discussão:

- payload mínimo e opaco sempre que possível;
- nenhuma credencial embutida;
- referência estável, não hostname/IP hard-coded;
- autorização normal após leitura;
- documento continua compreensível sem scanner;
- referência textual humana permanece ao lado/na folha;
- falha de leitura não impede uso do documento;
- não depender de cor;
- tamanho/quiet zone/contraste seguem o padrão técnico escolhido;
- renderer gera vetor quando suportado para manter qualidade física;
- a decisão deve preservar o limite de uma A4 da Ficha sem compactação automática.

Esses pontos são guardrails, não autorização de implementação.

## 16. Impacto arquitetural da decisão proposta

Com QR/barcode fora da v1:

- `DocumentModel` não precisa de bloco específico de barcode;
- nenhuma alteração de schema;
- nenhuma nova entidade/tabela;
- nenhuma dependência de geração/decodificação;
- nenhuma rota/deep link nova;
- nenhuma permissão nova;
- nenhuma mudança na impressão Windows;
- nenhuma mudança em temporários;
- nenhuma alteração nos filenames;
- nenhum impacto em Backup/Restore.

A Etapa 11 pode ser encerrada como uma decisão explícita de **não adicionar complexidade sem benefício operacional**.

## 17. Relação com a Etapa 12

Se esta proposta for aprovada, a Etapa 12 deve validar o Bloco 10 **sem QR/barcode na baseline**.

Não é necessário acrescentar matriz de:

- câmera;
- scanner;
- simbologia QR/Code 128/etc.;
- quiet zone;
- nível de correção de erro;
- deep link;
- portal de destino.

A Etapa 12 permanece focada nos contratos realmente aprovados: PDF/DOCX, WebView2/Windows, impressão, Ficha A4, limites de recursos, filesystem/temporários, erros e ambiente corporativo.

## 18. Decisões propostas para aprovação

1. QR code e barcode **não entram na baseline da v1** de Procedimentos ou Ficha;
2. não reservar espaço físico nos templates para marcador futuro;
3. códigos textuais `PR-...`, `AT-...` e `EQP-...` continuam suficientes para referência atual;
4. não criar URL, portal, endpoint ou deep link apenas para justificar QR;
5. não codificar hostname/IP/porta/path SMB em documento;
6. não usar QR/barcode para transportar dados operacionais completos ou informação pessoal desnecessária;
7. nunca embutir credencial, token ou mecanismo que contorne autenticação/autorização;
8. barcode de ativo/scanner, se surgir, pertence a um fluxo operacional explicitamente aprovado e não ao template documental por inferência;
9. QR/barcode pode ser reconsiderado futuramente somente com caso de scan concreto, destino estável e benefício demonstrável;
10. se reaberto para Procedimento físico, deve ser decidida explicitamente a semântica de revisão exata versus revisão atual;
11. documento deve continuar totalmente utilizável sem scanner;
12. com a baseline sem QR/barcode, a Etapa 12 não precisa validar infraestrutura de scan.

## 19. Fora do escopo desta proposta

- implementação de QR/barcode;
- escolha de biblioteca ou simbologia;
- deep linking Windows;
- portal web/mobile;
- etiqueta física de Equipamento;
- compra/configuração de scanners;
- código de produção;
- sincronização do checkout local;
- Backup/Restore técnico do Bloco 11.