# Tela 14 — Exportação / Impressão + Ficha Compacta — UX

## 1. Identificação

- código/nome da tela: Tela 14 — Exportação / Impressão + Ficha Compacta — UX;
- status: **CONSOLIDADO / APROVADO PELO PO**;
- bloco: Fase 1 — Bloco 8 (UI/UX);
- última consolidação: 2026-08-25.

## 2. Objetivo

Definir a experiência de geração de documentos do StepFlow sem transformar telas em documentos e sem antecipar a implementação técnica do Bloco 10.

A Tela 14 consolida duas famílias distintas de saída:

1. **Procedimento** — exportar/imprimir uma revisão específica como documento próprio;
2. **Ficha compacta de Atendimento** — gerar uma saída operacional curta, com ou sem equipamento vinculado, respeitando o limite máximo de uma folha A4.

Engine de PDF/DOCX, impressão nativa, margens, tipografia, paginação, nomes finais de arquivo e demais decisões técnicas permanecem para o **Bloco 10 — Exportação/Impressão + Ficha Compacta**.

## 3. Princípios consolidados

- PDF é obrigatório para procedimentos;
- DOCX é obrigatório para procedimentos;
- impressão é obrigatória para procedimentos;
- exportação usa documento próprio, nunca screenshot da interface;
- a identidade da empresa vem da configuração central do Host;
- o conteúdo de procedimento exportado é a revisão efetivamente selecionada/autorizada;
- exportar ou imprimir não publica, não edita e não altera o procedimento;
- revisão histórica não é substituída silenciosamente pela atual;
- a ficha compacta é documento próprio e não captura da Tela 09;
- a ficha usa somente estado confirmado pelo Host;
- a ficha pode existir com ou sem equipamento vinculado;
- a ficha ocupa **no máximo uma página A4**;
- a ficha prioriza conteúdo essencial e legibilidade;
- campos vazios ou não aplicáveis são omitidos;
- `Saúde da bateria` aparece somente quando aplicável e informada;
- autorização real permanece no Host.

## 4. Dois fluxos separados

```text
Leitor de Processo
→ Exportar / Imprimir
→ PDF | DOCX | Imprimir
→ documento completo da revisão selecionada
```

```text
Atendimento
→ Ficha / Imprimir
→ ficha compacta do estado confirmado
→ impressão e demais saídas aprovadas para a ficha
```

Não criar item global `Exportações` na sidebar.

## 5. Permissões

### Procedimentos

A matriz vigente define `Exportar/imprimir` por padrão para ADM, Gerência e Funcionário.

Para gerar o documento, a sessão precisa possuir simultaneamente:

```text
capacidade de ler a revisão selecionada
+
capacidade de exportar/imprimir
```

### Ficha de Atendimento

A regra exata de quem pode gerar/reimprimir ficha operacional permanece pendente do **Bloco 9**, junto da matriz operacional de Atendimentos.

A capacidade documental genérica não concede autorização operacional por suposição.

## 6. Entrada — Procedimento

Ponto principal:

```text
Leitor
→ menu contextual
→ Exportar / Imprimir
```

Uma revisão histórica aberta pelo Histórico pode usar o mesmo fluxo a partir do Leitor daquela revisão.

## 7. Painel de exportação do Procedimento

```text
Exportar / Imprimir procedimento

PR-014 · Configuração de VLAN
Versão 2.0 · revisão r18
Publicada

O documento será gerado a partir desta revisão.

[ Exportar PDF ]  [ Exportar DOCX ]  [ Imprimir ]

[ Cancelar ]
```

Quando não for a publicação vigente, identificar claramente `Revisão histórica`, draft/não publicado ou contexto arquivado, conforme aplicável.

## 8. Escopo inicial do documento de Procedimento

A primeira versão gera o **procedimento completo da revisão selecionada**, não somente a etapa atualmente aberta.

Quando aplicável, o documento contém:

- identidade da empresa;
- código e título;
- Área/Departamento;
- responsável documental;
- categorias;
- versão editorial;
- revisão técnica quando relevante;
- estado editorial necessário para evitar ambiguidade;
- objetivo;
- pré-requisitos;
- observações gerais;
- etapas em ordem;
- parágrafos;
- passos/subpassos;
- checklist documental;
- notas/observações;
- alertas;
- comandos;
- blocos de código.

Não entram no documento elementos de UI como sidebar, botões, ícones de copiar, barra `Etapa X de Y`, painel interativo de Sumário, toasts ou estados transitórios do Client.

Sumário documental estático, se desejável, será decidido no Bloco 10.

## 9. Revisão exata como fonte

```text
revisão r17 aberta
→ solicitar exportação de r17
→ nova r18 surge/publica
→ documento continua sendo de r17
```

Nunca substituir silenciosamente por revisão atual, publicada mais recente ou conteúdo recebido por evento depois da ação.

Se a revisão deixar de estar autorizada antes da geração ser aceita, o Host rejeita a operação.

## 10. Identidade da empresa

Os documentos usam a identidade central vigente no momento da geração:

- logo, quando configurado;
- nome da empresa;
- contato;
- site;
- e-mail.

Campos opcionais vazios são omitidos. Não mostrar path técnico, placeholder quebrado ou imagem ausente.

O conteúdo documental continua preso à revisão selecionada. Versionamento histórico da identidade corporativa não é requisito inicial.

## 11. PDF, DOCX e impressão de Procedimento

`Exportar PDF` e `Exportar DOCX` preparam documentos próprios e, conforme mecanismo do Bloco 10, oferecem destino local apropriado ao usuário.

`Imprimir` prepara o mesmo conceito de documento dedicado e abre o fluxo de impressão do Windows/Client.

Cancelar um `Salvar como…` ou diálogo de impressão é cancelamento voluntário do usuário, não erro funcional.

Procedimentos podem ocupar várias páginas; o limite de uma A4 aplica-se somente à ficha compacta.

## 12. Estados de geração documental

A UX suporta pelo menos:

- preparando;
- pronto para salvar/imprimir;
- concluído;
- cancelado pelo usuário;
- falha de geração;
- Host indisponível;
- sem permissão;
- revisão indisponível.

Não inventar percentual de progresso sem progresso real fornecido pelo mecanismo.

Mensagem de falha funcional:

`Não foi possível gerar o documento. Nenhum arquivo foi confirmado.`

## 13. Exportação não altera dados

Exportar/imprimir é leitura/derivação e não deve:

- criar revisão;
- publicar;
- alterar `updated_at` funcional apenas pela exportação;
- marcar checklist como concluído;
- registrar progresso operacional;
- substituir o snapshot selecionado.

Auditoria específica de exportação só entra mediante requisito futuro explícito.

## 14. Entrada — Ficha compacta

Ponto de entrada aprovado na Tela 09:

```text
Atendimento
→ Ficha / Imprimir
```

A ação aparece quando lifecycle/capacidade do Bloco 9 permitir.

## 15. Estado confirmado antes da ficha

A ficha nunca mistura rascunho local com estado oficial.

Se houver alterações não salvas ou conflito pendente:

```text
Ficha / Imprimir
→ informar “Salve as alterações antes de gerar a ficha.”
→ resolver salvamento/conflito
→ somente depois gerar
```

Não imprimir silenciosamente valores antigos enquanto a interface mostra valores locais ainda não confirmados.

## 16. Direção funcional da ficha

```text
┌─────────────────────────────────────────────────────────────┐
│ [ LOGO ]  Nome da Empresa                                  │
│           Contato · site · e-mail                          │
├─────────────────────────────────────────────────────────────┤
│ ATENDIMENTO AT-00142                 OS/Ref.: OS-4587       │
│ Cliente: João Silva                   Data: 25/08/2026      │
│ Técnico: Maria Souza                                        │
├─────────────────────────────────────────────────────────────┤
│ EQUIPAMENTO                                                 │
│ EQP-0031 · NOTE-15 · Notebook                              │
│ CPU: i5-1135G7     RAM: 16 GB      SSD: NVMe 512 GB       │
│ Sistema: Windows 11 Pro · 24H2                             │
│ Serial: ABC123       Patrimônio: PAT-884                   │
│ MAC: A0:B1:C2:D3:E4:F5                                    │
│ Bateria: 82%                                                │
│ Observações: texto curto...                                 │
├─────────────────────────────────────────────────────────────┤
│ PROCEDIMENTOS / TRABALHO                                    │
│ PR-001 v1.3/r18 · Manutenção preventiva                    │
│ PR-022 v2.0/r7  · Substituição de SSD                      │
│ Resumo: limpeza, substituição do SSD e validação final...  │
├─────────────────────────────────────────────────────────────┤
│ Observações do atendimento: ...                             │
└─────────────────────────────────────────────────────────────┘
```

É hierarquia funcional; margens, fontes e densidade final pertencem ao Bloco 10.

## 17. Cabeçalho da ficha

Suporta:

- logo à esquerda/início quando configurado;
- nome da empresa;
- forma(s) de contato;
- site;
- e-mail.

Preservar proporção do logo. Ausência de logo ou campos institucionais não deve gerar espaços quebrados. O cabeçalho deve permanecer discreto para preservar área útil da única página.

## 18. Conteúdo da ficha

### Atendimento

Quando disponível:

- código;
- OS/referência externa;
- cliente/solicitante;
- técnico/responsável;
- data aplicável conforme lifecycle do Bloco 9;
- resumo do trabalho;
- observações curtas quando couberem no contrato final.

### Equipamento — quando houver

- código StepFlow;
- nome;
- tipo;
- processador;
- RAM;
- armazenamento;
- sistema operacional + versão;
- serial;
- patrimônio;
- um ou mais MACs conforme regra de densidade futura;
- saúde da bateria quando aplicável/informada;
- observações curtas.

### Procedimentos utilizados

- código snapshot;
- título snapshot;
- versão editorial utilizada;
- revisão técnica efetivamente utilizada.

A ficha não reproduz etapas completas dos procedimentos.

## 19. Atendimento sem equipamento — consolidado

A mesma ação pode gerar uma **ficha de Atendimento mesmo sem equipamento vinculado**.

Nesse cenário:

- a seção `Equipamento` é omitida;
- não se reserva grande área vazia;
- dados do Atendimento e procedimentos utilizados permanecem válidos.

Isso mantém a ficha útil em rede, infraestrutura, Help Desk e outras execuções sem ativo físico específico.

## 20. Regra rígida de uma A4

- no máximo uma folha A4;
- pode usar menos espaço;
- não cria segunda página como comportamento normal;
- não reduz tipografia a ponto de prejudicar legibilidade apenas para caber.

Se conteúdo excepcional não couber:

- não criar segunda página automaticamente;
- não truncar informação importante silenciosamente;
- não reduzir fonte indefinidamente;
- bloquear a saída e orientar revisão/resumo dos campos aplicáveis.

Mensagem funcional:

`A ficha possui conteúdo demais para uma página A4. Revise os campos indicados antes de imprimir.`

Regras finais de priorização, resumo, truncamento controlado e limites numéricos pertencem ao Bloco 10.

## 21. Campos vazios e bateria

Campos vazios/não aplicáveis são omitidos. Não imprimir linhas como `Serial: —`, `Patrimônio: —` ou `Bateria: —` sem necessidade.

`Saúde da bateria` aparece somente quando aplicável ao tipo de equipamento e houver valor informado.

## 22. Revisões utilizadas

Cada procedimento listado na ficha preserva a revisão efetivamente utilizada, por exemplo:

```text
PR-001 · Manutenção preventiva · v1.3 · r18
PR-022 · Substituição de SSD · v2.0 · r7
```

Uma revisão mais nova nunca substitui silenciosamente a revisão histórica do Atendimento.

## 23. Saídas da ficha

**Imprimir ficha** é requisito consolidado.

A necessidade de **PDF específico da ficha** permanece pendente do Bloco 10.

DOCX específico da ficha **não é requisito inicial** e não aparece automaticamente por herança do exportador de procedimentos.

## 24. Preview, QR e barcode

A necessidade/forma técnica de preview permanece para o Bloco 10. Se houver preview, deve usar o mesmo template/estado da saída final.

QR/barcode não aparece por padrão na primeira UX e só entra se houver benefício operacional aprovado.

## 25. Reimpressão e lifecycle

Permanece para o Bloco 9 decidir:

- quem gera/reimprime ficha;
- se a ficha pode ser gerada durante execução ou somente em estados específicos;
- comportamento após conclusão/reabertura;
- snapshot operacional exato quando necessário;
- data/status operacional usados na ficha.

Quando autorizada, a ficha sempre usa estado oficial definido pelo contrato operacional.

## 26. Estados e consistência

- Host indisponível: impedir geração sem oferecer edição de IP/porta;
- sem permissão: ocultar ação; acesso manipulado é rejeitado pelo Host;
- fonte indisponível: não gerar silenciosamente de cache antigo;
- impressão cancelada: cancelamento voluntário;
- eventos novos não trocam a fonte de uma geração já iniciada;
- ao concluir, informar qual revisão/Atendimento foi usado.

## 27. Acessibilidade e janelas menores

- ações têm nomes textuais claros;
- estados de preparação são anunciáveis;
- diálogos operam por teclado;
- revisão histórica é indicada em texto;
- ações podem empilhar em janela menor;
- dimensão física do documento independe do tamanho da janela do Client;
- sem transformação mobile/hamburger inicial.

## 28. Fora do escopo desta tela

- biblioteca/engine PDF;
- biblioteca DOCX;
- renderer de impressão;
- spooler/API nativa;
- margens/tipografia finais;
- cabeçalho/rodapé técnico final;
- numeração de páginas final;
- nomes finais de arquivo;
- assinatura digital;
- envio por e-mail;
- nuvem/compartilhamento externo;
- exportação em lote/ZIP;
- editor visual de relatórios/templates;
- armazenamento permanente de toda exportação;
- implementação funcional.

## 29. Pendências preservadas para o Bloco 9

- capacidade para gerar/reimprimir ficha;
- lifecycle de disponibilidade da ação;
- data/status operacional;
- regras após conclusão/reabertura;
- snapshot operacional exato.

## 30. Pendências preservadas para o Bloco 10

- engine/tecnologia PDF;
- engine/tecnologia DOCX;
- estratégia de impressão;
- template final dos procedimentos;
- template físico final da ficha;
- margens, tipografia e densidade;
- limites numéricos de observações/textos;
- regras de resumo/truncamento controlado;
- tratamento de muitos MACs/procedimentos na única A4;
- pré-visualização;
- necessidade ou não de PDF específico da ficha;
- naming de arquivos;
- QR/barcode somente se aprovado;
- critérios técnicos em leitores/impressoras.

## 31. Decisões consolidadas pelo PO

1. exportação/impressão permanece contextual, sem item global novo;
2. o Leitor identifica sempre a revisão utilizada;
3. a primeira versão exporta/imprime o procedimento completo da revisão selecionada;
4. PDF, DOCX e impressão usam documento próprio, nunca screenshot;
5. revisão histórica/draft autorizada é identificada inequivocamente;
6. exportar não publica nem altera dados;
7. identidade da empresa vem da configuração central vigente;
8. `Ficha / Imprimir` parte da Tela 09 e usa somente estado confirmado;
9. alterações não salvas/conflitos precisam ser resolvidos antes da ficha;
10. ficha suporta Atendimento sem equipamento;
11. ficha omite campos vazios/não aplicáveis;
12. procedimentos utilizados preservam versão/revisão efetivamente utilizada;
13. ficha nunca ultrapassa uma página A4 como comportamento normal;
14. conteúdo excessivo não cria segunda página nem é truncado silenciosamente;
15. `Saúde da bateria` aparece somente quando aplicável/informada;
16. impressão da ficha é garantida;
17. PDF específico da ficha permanece pendente do Bloco 10;
18. DOCX específico da ficha não é requisito inicial;
19. preview e QR/barcode permanecem para decisão do Bloco 10;
20. Tela 15 não foi antecipada.

## 32. Critérios de aceite — atendidos

- [x] separação entre documento de Procedimento e ficha operacional;
- [x] exportação da revisão selecionada;
- [x] procedimento completo como escopo inicial;
- [x] PDF + DOCX + impressão contextual;
- [x] documento próprio sem screenshot;
- [x] identificação de revisão histórica/draft;
- [x] identidade central nos documentos;
- [x] ficha somente de estado confirmado;
- [x] suporte de ficha sem equipamento;
- [x] omissão de campos vazios/não aplicáveis;
- [x] preservação das revisões utilizadas;
- [x] regra rígida de uma A4;
- [x] bloqueio claro se conteúdo excepcional não couber;
- [x] PDF específico da ficha continua pendente do Bloco 10;
- [x] DOCX da ficha não é requisito inicial;
- [x] preview/QR permanecem pendentes do Bloco 10;
- [x] pendências operacionais preservadas no Bloco 9;
- [x] implementação técnica preservada no Bloco 10;
- [x] Tela 15 não foi iniciada;
- [x] nenhuma implementação funcional foi criada.
