# Bloco 10 — Etapa 8 — Limites textuais e densidade da Ficha — Proposta para análise

**Status:** PROPOSTA / AGUARDANDO APROVAÇÃO DO PO  
**Fase:** Fase 1 — Fechamento arquitetural e especificação  
**Abertura:** 2026-08-28  
**Base consolidada:** Bloco 10 / Etapas 1–7  
**Base Git:** `main` em `c573942d6ca624a7f0596011a32fa1848c79a71e`

## 1. Objetivo

Fechar a política de **limites textuais, densidade e diagnóstico de overflow** da Ficha compacta de Atendimento, preservando duas regras já consolidadas:

1. a Ficha é uma prestação de contas resumida ao cliente;
2. a Ficha possui exatamente uma página A4 e nunca corta conteúdo silenciosamente.

Esta etapa não deve criar um segundo conjunto de dados apenas para impressão, nem transformar a Ficha em relatório detalhado.

Permanecem fora desta etapa:

- muitos MACs, muitos Procedimentos e demais casos multiplicativos/excepcionais — Etapa 9;
- nomes de arquivo e temporários — Etapa 10;
- QR/barcode — Etapa 11;
- limites técnicos finais de payload/memória/tempo e matriz real — Etapa 12.

## 2. Princípio central — limite da Ficha ≠ perda de dado operacional

A restrição de uma A4 pertence ao **artefato client-facing**, não deve apagar ou truncar informação real do Atendimento.

```text
Atendimento
→ preserva dados operacionais reais

Ficha
→ usa projeção resumida desses dados
→ tenta diagramar em 1 A4
→ se não couber, pede revisão consciente
```

Consequências:

- não cortar texto salvo no Atendimento para satisfazer impressão;
- não resumir automaticamente texto do técnico;
- não criar texto diferente apenas para impressão sem requisito explícito;
- não substituir observação original por versão abreviada;
- não usar IA/LLM para reescrever ou condensar conteúdo da Ficha;
- não omitir observação legítima silenciosamente.

## 3. Fonte textual da Ficha

A Ficha continua usando somente os campos já consolidados:

### Serviço

- código/data/OS/cliente/técnico conforme aplicável;
- `Resumo do trabalho`;
- observação geral do Atendimento quando preenchida e relevante.

### Equipamento

- identificação e características técnicas compactas;
- observação do Equipamento quando preenchida e relevante.

### Execução

- observações do serviço registradas nas Etapas.

Não criar na v1:

- `resumo_para_impressao` separado;
- `observacao_para_cliente` duplicada;
- editor paralelo exclusivo da Ficha.

Se futuramente houver necessidade real de redação client-facing diferente do registro técnico, isso será nova decisão de produto.

## 4. Autoridade de encaixe

**Contagem de caracteres não é autoridade para saber se cabe em uma A4.**

O resultado real do Typst permanece a validação final:

```text
DocumentModel
→ template Etapa 7
→ PagedDocument

1 página
→ válido

2+ páginas
→ SHEET_OVERFLOW
```

Motivos:

- palavras têm larguras diferentes;
- quebras de linha variam conforme conteúdo;
- presença/ausência de Equipamento muda espaço disponível;
- títulos e valores técnicos variam;
- número de observações varia;
- caracteres especiais e nomes longos alteram o layout.

Portanto, limites de caracteres abaixo são **orientação de escrita e warning antecipado**, não substituem o layout real.

## 5. Densidade alvo

A Ficha deve parecer resumida mesmo quando ainda houver espaço em branco.

Não preencher espaço disponível apenas porque existe.

Direção:

```text
cabeçalho/identificação  → compacto
Equipamento              → poucas linhas técnicas
Serviço realizado        → parágrafo curto
Observações              → somente o que foi efetivamente registrado
```

Não aumentar densidade por meio de:

- mais labels;
- metadados editoriais;
- lista de Procedimentos;
- checklist;
- histórico;
- explicações automáticas;
- campos vazios;
- mensagens como `não informado`.

## 6. Faixas recomendadas de escrita

Propõe-se orientação simples, sem bloquear o registro operacional apenas porque a Ficha pode ser impressa.

| Campo | Faixa recomendada para boa densidade da Ficha |
|---|---:|
| `Resumo do trabalho` | até **600 caracteres** |
| Observação geral do Atendimento | até **400 caracteres** |
| Observação do Equipamento | até **300 caracteres** |
| Observação do serviço por Etapa | até **280 caracteres por Etapa** |

Esses valores:

- são **soft limits**;
- não truncam conteúdo;
- não garantem sozinhos encaixe em A4;
- não impedem salvar texto maior no Atendimento por causa da Ficha;
- podem ser ajustados futuramente com evidência da validação real da Etapa 12.

A finalidade é orientar redação curta compatível com uma prestação de contas, não transformar o formulário em prova de limite de caracteres.

## 7. Como mostrar o limite sem poluir a UI

Contadores não devem aparecer permanentemente em todos os campos.

Comportamento proposto:

```text
texto confortável
→ sem contador permanente

aproxima-se da faixa recomendada
→ contador discreto aparece

ultrapassa faixa recomendada
→ aviso curto, sem bloquear save
```

Exemplos:

```text
Resumo do trabalho
[ ... ]
548 / 600 recomendado
```

```text
Observação do serviço
[ ... ]
Texto extenso pode dificultar a Ficha de uma página. 326 / 280 recomendado
```

O contador deve surgir somente próximo ao limite recomendado, por exemplo a partir de aproximadamente **80%** da faixa.

Essa regra reduz ruído visual e segue o princípio transversal de baixa densidade.

## 8. Hard limits de armazenamento

A Etapa 8 **não congela hard limits de banco/API** apenas para fazer a Ficha caber.

Hard limits existem por motivos técnicos e de segurança, mas devem ser definidos junto de:

- schema físico;
- limites de payload;
- memória;
- validação da implementação;
- testes reais.

Isso pertence ao gate técnico/implementação correspondente, especialmente Etapa 12/Bloco 12.

Não usar o limite de uma A4 como justificativa para impedir o StepFlow de preservar uma observação operacional legítima.

## 9. Normalização segura para apresentação

Antes de diagramar a Ficha, o renderer pode fazer somente normalizações que não alterem significado:

- trim de espaço no início/fim;
- normalização consistente de quebras de linha;
- colapso de linhas vazias repetidas quando não carregarem significado;
- não renderizar campo vazio;
- não renderizar label de campo vazio.

Não pode:

- remover frases;
- trocar palavras;
- resumir;
- reordenar frases dentro da observação;
- substituir o texto original por reticências.

## 10. Ordem e prioridade sem omissão automática

A prioridade define **ordem de leitura e diagnóstico**, não autorização para apagar conteúdo.

### Núcleo da prestação

Sempre que aplicável:

1. identificação mínima do Atendimento;
2. identificação do cliente/técnico;
3. identificação/características essenciais do dispositivo;
4. `SERVIÇO REALIZADO`;
5. `OBSERVAÇÕES` preenchidas.

Dentro do Equipamento, o template já consolidado mantém dados compactos e omite apenas campos inexistentes/não aplicáveis.

Dentro de Observações, nenhuma origem recebe exclusão automática por ser considerada “menos importante”.

## 11. Observações — preservação e ordem

Quando houver observações de mais de uma origem, a seção única segue ordem previsível:

```text
1. observação geral do Atendimento
2. observação relevante do Equipamento
3. observações de serviço por Etapa, na ordem das Etapas executadas
```

Somente itens não vazios aparecem.

Observação por Etapa pode receber prefixo curto quando necessário para contexto:

```text
SSD — unidade anterior apresentou setores defeituosos.
```

Evitar prefixos longos como:

```text
Etapa 03 de 07 · Procedimento PR-022 · revisão r7 · SSD
```

A Etapa 9 decidirá como lidar com quantidade excepcional de Etapas/observações, sem reabrir a ordem semântica normal.

## 12. Conteúdo repetido

O StepFlow **não deve deduplicar automaticamente por semelhança textual**.

Duas observações parecidas podem representar fatos diferentes.

Pode apenas evitar duplicação estrutural criada pelo próprio template, por exemplo:

- não repetir o mesmo código do Atendimento em dois blocos sem necessidade;
- não repetir `Concluído` em várias áreas;
- não repetir características do Equipamento dentro de Observações.

Texto digitado pelo usuário permanece sob controle do usuário.

## 13. Overflow — comportamento funcional

Quando o layout produzir mais de uma página:

```text
SHEET_OVERFLOW
→ nenhum PDF confirmado
→ nenhum preview válido da Ficha final
→ nenhuma impressão
```

A UI deve explicar o problema de forma operacional, sem erro técnico genérico.

Mensagem-base:

`A ficha ficou extensa demais para uma página A4. Revise os textos indicados e gere novamente.`

Não usar:

- `Erro no PDF`;
- `Falha desconhecida`;
- `Documento truncado` como sucesso.

## 14. Diagnóstico de overflow

O Host deve devolver diagnóstico semântico suficiente para o Client orientar o usuário.

Conceitualmente:

```text
SHEET_OVERFLOW
contributors:
- work_summary
- service_general_note
- device_note
- stage_note:<stage_id>
```

Não é necessário prometer percentual exato de contribuição visual.

O diagnóstico pode usar:

- comprimento do texto;
- quantidade de linhas/quebras;
- quantidade de observações;
- quais campos ultrapassaram faixas recomendadas;
- quais blocos variáveis estão presentes.

A finalidade é indicar **onde revisar**, não fingir uma medição exata de layout por campo.

## 15. UX de correção

Ao receber `SHEET_OVERFLOW`, o Client não deve abrir um editor separado da Ficha.

Fluxo:

```text
Ficha / Imprimir
→ SHEET_OVERFLOW
→ mensagem curta
→ ação `Revisar atendimento`
→ destacar discretamente os campos mais prováveis
```

Na Tela 09/Reader operacional:

- `Resumo do trabalho` pode ser destacado;
- observação geral do Atendimento pode ser destacada;
- observação do Equipamento pode ser indicada;
- observações de Etapa longas podem ser sinalizadas no Reader.

O usuário edita o dado real, salva no Host e gera novamente.

Isso mantém uma única fonte de verdade.

## 16. Campo longo não deve bloquear conclusão

Ultrapassar faixa recomendada da Ficha **não bloqueia salvar nem concluir Atendimento** por si só.

Motivo:

- o dado operacional pode ser legítimo;
- nem todo Atendimento será necessariamente impresso imediatamente;
- requisito de A4 não deve degradar o histórico técnico.

O bloqueio ocorre apenas na **geração da Ficha** quando o layout real exceder uma página.

## 17. Resumo do trabalho

`Resumo do trabalho` continua obrigatório para conclusão conforme Bloco 9.

Para a Ficha:

- é a síntese principal do serviço;
- deve privilegiar resultado/panorama, não sequência de passos;
- recomendação de até 600 caracteres;
- não repetir observações específicas que já estejam na seção Observações sem necessidade;
- texto maior continua persistível, mas pode gerar warning/overflow.

Exemplo adequado:

```text
Realizada limpeza preventiva, substituição do SSD e validação geral do equipamento após reinstalação do sistema.
```

Evitar transformar o resumo em diário completo de execução.

## 18. Observação geral do Atendimento

Finalidade:

- informação geral relevante para o serviço/cliente que não pertence a uma Etapa específica.

Recomendação:

- até 400 caracteres para boa densidade;
- opcional;
- não usar como substituto de várias observações de Etapa apenas para contornar a interface.

## 19. Observação do Equipamento

Na Ficha, a observação do Equipamento deve aparecer somente quando existir e for parte do snapshot aplicável.

Recomendação:

- até 300 caracteres para boa densidade client-facing;
- não repetir CPU/RAM/SSD/bateria em prosa quando esses valores já estão na ficha técnica;
- não criar resumo automático do campo original.

A política de relevância futura não autoriza omissão automática de texto salvo sem uma regra explícita de produto.

## 20. Observação do serviço por Etapa

É o principal refinamento operacional da Etapa 6 e deve continuar simples.

Recomendação:

- até 280 caracteres por Etapa;
- uma observação curta e objetiva sobre o que ocorreu naquela parte do serviço;
- sem necessidade de registrar o passo a passo já presente no Procedimento;
- continua opcional;
- texto maior permanece permitido operacionalmente, sujeito a warning e ao layout final da Ficha.

Exemplo:

```text
Unidade anterior apresentou setores defeituosos; SSD substituído e validado sem erros após o procedimento.
```

## 21. Campos técnicos compactos

CPU, RAM, armazenamento, SO, bateria, serial e patrimônio não precisam de limites “de relatório” separados nesta etapa porque já são dados estruturados e exibidos compactamente.

Regras:

- não duplicar label quando valor estiver ausente;
- não adicionar descrição automática ao redor do valor;
- strings excepcionalmente grandes ou multiplicidade de identificadores ficam para Etapa 9/12.

## 22. Estados sem aumento desnecessário de densidade

### Em andamento

- `Acompanhamento`/`Em andamento` aparece discretamente;
- não adicionar explicação longa sobre documento provisório.

### Concluído

- sem selo/faixa extensa;
- estado histórico aplicável continua sendo a fonte.

### Cancelado

- `CANCELADO` textual e inequívoco;
- não repetir motivo completo do cancelamento por padrão se ele não fizer parte da prestação client-facing consolidada.

## 23. Sem “modo compacto” alternativo

A Etapa 8 não cria dois templates como:

```text
normal
compacto
supercompacto
```

Também não cria fallback automático que reduz:

- fonte;
- margens;
- espaçamento;
- conteúdo.

Existe um único template físico consolidado na Etapa 7.

Se ele não comportar o conteúdo, o resultado é overflow e revisão consciente.

## 24. Sem abreviações automáticas perigosas

O template pode usar labels curtos já definidos, como:

- CPU;
- RAM;
- SSD/HD conforme dado;
- OS/Sistema quando aprovado no template.

Não abreviar automaticamente valores livres, nomes de cliente, observações ou resumo para ganhar espaço.

## 25. Privacidade e conteúdo client-facing

A redução de densidade também evita exposição de dados internos desnecessários.

A Ficha não acrescenta por conveniência:

- usuário interno que fez cada clique;
- timestamps de checklist;
- revision IDs;
- motivo de conflito;
- audit trail;
- paths/hostname/IP internos;
- IDs técnicos do banco;
- mensagens de erro.

Apenas dados já aprovados como parte da prestação de contas entram no artefato.

## 26. Critério de aceite da densidade

A Ficha normal deve permanecer visualmente leve quando contiver:

- identificação de serviço;
- um Equipamento com características comuns;
- um resumo de trabalho dentro da faixa recomendada;
- poucas observações curtas.

Não é objetivo usar 100% da área útil A4.

Espaço em branco é aceitável e desejável quando o serviço tiver pouca informação.

## 27. Casos a validar depois

A Etapa 12 deverá testar com conteúdo realista, incluindo:

- resumo curto/médio/longo;
- observações com acentos/Unicode;
- nomes longos;
- Equipamento com todos os campos;
- bateria presente/ausente;
- várias observações de Etapa;
- conteúdo próximo do limite de uma página;
- overflow inequívoco;
- impressão monocromática.

A validação real pode recomendar ajuste das faixas soft sem mudar a regra de não truncar/não resumir automaticamente.

## 28. Decisões propostas para aprovação

1. limite de uma A4 não limita/destrói o dado operacional salvo;
2. não criar campos paralelos exclusivos para impressão;
3. layout real Typst é a autoridade final de encaixe;
4. caracteres servem apenas como orientação antecipada;
5. faixas soft propostas: Resumo 600, Observação do Atendimento 400, Equipamento 300 e Observação de Etapa 280 caracteres;
6. contador só aparece próximo de aproximadamente 80% da faixa recomendada;
7. ultrapassar faixa soft não bloqueia save/conclusão;
8. `SHEET_OVERFLOW` bloqueia somente a geração da Ficha;
9. Host retorna diagnóstico semântico dos principais campos contribuintes;
10. Client orienta revisão dos dados reais, sem editor paralelo da Ficha;
11. nenhuma IA, truncamento, reticência, deduplicação semântica ou resumo automático;
12. normalização permitida somente quando não altera significado;
13. observações seguem ordem previsível: Atendimento → Equipamento → Etapas;
14. um único template físico; sem modo compacto/redução automática de fonte/margem/espaçamento;
15. espaço em branco é aceitável; não preencher A4 com informação desnecessária;
16. hard limits técnicos de storage/payload ficam para implementação/validação, não são derivados do tamanho da folha.

## 29. Gate

Enquanto esta proposta não for aprovada:

- somente este arquivo de proposta pode ser alterado nesta branch;
- nenhuma fonte canônica é promovida;
- Etapa 9 não é aberta.
