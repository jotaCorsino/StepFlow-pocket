# Categorização, Atendimentos e Equipamentos — StepFlow

**Status:** REQUISITO DE PRODUTO CONSOLIDADO / DETALHES DE UX E EXECUÇÃO PENDENTES  
**Atualização:** 2026-08-21

## 1. Objetivo

Expandir o StepFlow para documentar e executar procedimentos de diferentes áreas técnicas, sem restringir o produto à manutenção de computadores.

O produto deve suportar, entre outros usos:

- manutenção de computadores e notebooks;
- TI geral;
- Service Desk;
- Help Desk;
- infraestrutura e servidores;
- redes de computadores;
- procedimentos internos;
- passo a passos/guias técnicos.

A expansão introduz três conceitos distintos que não devem ser misturados:

1. **Procedimento** — modelo reutilizável/documentação oficial;
2. **Equipamento** — ativo físico opcional associado a um atendimento;
3. **Atendimento/Execução** — ocorrência concreta em que um ou mais procedimentos são realizados.

## 2. Categorização dos procedimentos

Procedimentos devem poder ser organizados por categorias configuráveis pela empresa.

### Requisitos consolidados

- categorias não são hardcoded no código;
- usuários autorizados podem criar/manter categorias conforme o uso real;
- um procedimento pode pertencer a uma ou mais categorias;
- categorias são pesquisáveis/filtráveis na lista de procedimentos;
- categorias podem ser exibidas de forma discreta no leitor e em listas;
- arquivar uma categoria não destrói histórico nem remove silenciosamente a classificação de revisões antigas;
- exemplos como `Manutenção`, `TI`, `Service Desk`, `Help Desk`, `Infraestrutura`, `Redes` e `Guias` são exemplos de uso, não conjunto fixo.

### Limite inicial

Não exigir hierarquia complexa, árvore ilimitada, tags paralelas ou taxonomia corporativa avançada na primeira versão. Se categorias simples e múltiplas atenderem ao uso real, essa é a opção preferida.

## 3. Separação entre modelo e execução

Um procedimento descreve **como fazer**.

Um atendimento/execução registra **o que foi feito em uma ocorrência real**.

Exemplo:

```text
Procedimento oficial
"Manutenção preventiva de notebook"
        ↓ usado em
Atendimento ATD-...
        ↓ relacionado a
Equipamento EQP-...
        ↓ registra
procedimentos realizados + resumo + observações
```

Alterar o procedimento oficial depois não deve reescrever historicamente um atendimento já concluído. O atendimento deve referenciar a revisão do procedimento efetivamente utilizada.

## 4. Equipamento

Equipamento é opcional. Procedimentos de rede, servidor, Help Desk ou guia geral podem ser executados sem cadastro de um computador físico quando isso não fizer sentido.

Para manutenção de computadores/notebooks, a ficha deve suportar inicialmente:

- código interno estável do equipamento;
- nome do equipamento;
- tipo do equipamento, quando aplicável;
- nome/referência de cliente, solicitante ou responsável;
- processador;
- memória RAM;
- armazenamento;
- sistema operacional;
- versão do sistema operacional;
- número de série, quando disponível;
- patrimônio/asset tag, quando disponível;
- um ou mais endereços MAC, quando úteis;
- saúde da bateria, quando aplicável;
- observações.

Campos que não se aplicam a um equipamento específico permanecem vazios/ocultos; não devem virar burocracia obrigatória.

## 5. Identidade e busca do equipamento

A identidade canônica do equipamento deve ser gerada pelo StepFlow e não depender exclusivamente de um atributo físico mutável.

Consolidado:

- `equipment_id` interno estável;
- código legível gerado pelo sistema para identificação operacional;
- número de série, patrimônio e MAC são atributos pesquisáveis, não a chave canônica obrigatória;
- o sistema deve permitir localizar equipamento/atendimento por informações úteis ao operador.

Busca deve considerar, conforme dados disponíveis:

- código interno do equipamento;
- nome do equipamento;
- cliente/solicitante/responsável;
- ordem de serviço/referência externa;
- número de série;
- patrimônio;
- MAC normalizado;
- código do atendimento.

O formato exato dos códigos legíveis (`EQP-...`, `ATD-...` ou equivalente) será definido antes da implementação; não deve ser inventado pelo executor.

## 6. Atendimento / execução

O atendimento é o registro operacional concreto.

Requisitos iniciais:

- identificador interno estável;
- código legível do atendimento;
- referência/ordem de serviço externa opcional;
- cliente/solicitante/responsável em texto quando aplicável;
- equipamento associado opcional;
- técnico/responsável pelo atendimento;
- data/hora de início e conclusão quando aplicável;
- observações;
- resumo do que foi realizado;
- lista dos procedimentos/revisões efetivamente utilizados;
- histórico/auditoria compatível com as regras do StepFlow.

Um atendimento pode usar mais de um procedimento quando o serviço real exigir.

## 7. Procedimentos realizados

Para cada procedimento utilizado em um atendimento, o sistema deve preservar referência suficiente para saber:

- qual `process_id` foi usado;
- qual revisão publicada/permitida foi executada;
- título/código exibidos no momento quando necessário para relatório histórico;
- ordem ou agrupamento na execução, se relevante.

A decisão detalhada sobre marcações de checklist, progresso por etapa e estado de execução será fechada no Bloco 9.

## 8. Ficha compacta / etiqueta imprimível

Atendimentos com equipamento devem poder gerar uma saída compacta destinada a acompanhamento físico do equipamento.

Objetivo: produzir uma ficha/relatório resumido que possa ser impresso e anexado ao computador.

Conteúdo mínimo esperado, quando disponível:

- identidade da empresa/StepFlow;
- código do atendimento;
- ordem de serviço/referência externa;
- cliente/solicitante;
- código/nome do equipamento;
- processador;
- RAM;
- armazenamento;
- sistema/versão;
- serial/patrimônio/MAC conforme aplicável;
- saúde da bateria para notebook, quando informada;
- resumo dos procedimentos realizados;
- observações relevantes;
- técnico/responsável;
- data do atendimento/conclusão.

A saída deve ser pensada como documento próprio, não captura de tela.

Tamanho físico, paginação, uso ou não de QR/barcode, mecanismo de geração e formatos finais serão tratados no Bloco 10. Nenhum deles é requisito implícito neste momento.

## 9. Generalidade do produto

A ficha de computador é uma especialização útil, não o núcleo obrigatório de todo procedimento.

O StepFlow deve continuar permitindo:

```text
procedimento sem atendimento formal
procedimento + atendimento sem equipamento
procedimento + atendimento + equipamento
atendimento com múltiplos procedimentos
```

Isso preserva os cenários de Service Desk, servidores, redes, guias e manutenção física sem criar estruturas artificiais onde não são necessárias.

## 10. Permissões

Consolidado em princípio:

- leitura de procedimentos continua conforme capacidades existentes;
- criação/alteração da documentação oficial continua separada da execução cotidiana;
- Funcionário/Técnico pode executar procedimentos sem receber permissão para editar o documento oficial;
- autorização real permanece no Host.

A matriz exata para criar/editar/encerrar atendimentos, manter equipamentos e administrar categorias será fechada antes da implementação correspondente.

## 11. Concorrência e histórico

- Equipamento e atendimento são dados oficiais do Host;
- Clients não editam SQLite diretamente;
- alterações concorrentes relevantes devem usar revisão/controle otimista equivalente ao restante do produto;
- atendimento concluído não deve perder o vínculo com a revisão de procedimento utilizada;
- impressão/relatório deve refletir estado confirmado pelo Host.

## 12. Impactos nos blocos da Fase 1

### Bloco 8 — UI/UX

Deve incorporar:

- categorias na lista/editor/leitor de procedimentos;
- superfícies de atendimento/execução;
- ficha de equipamento;
- busca por atendimento/equipamento;
- ação de gerar/imprimir ficha compacta.

Detalhes dependentes do estado de execução/checklist permanecem condicionados ao Bloco 9.

### Bloco 9 — Execução operacional e checklist

O antigo escopo restrito a “checklist durante execução” passa a incluir:

- entidade formal de atendimento/execução;
- vínculo com equipamento opcional;
- vínculo com revisão de procedimento executada;
- progresso/checklist durante execução;
- regras de início/conclusão/reabertura, se necessárias;
- concorrência e histórico operacional.

### Bloco 10 — Exportação/impressão

Além do documento completo do procedimento, deve fechar a estratégia da ficha/etiqueta compacta de atendimento/equipamento.

## 13. Fora do escopo por enquanto

Não transformar automaticamente esta necessidade em:

- CRM completo de clientes;
- gestão financeira/faturamento;
- estoque de peças;
- inventário corporativo completo;
- RMM/monitoramento de máquinas;
- descoberta automática de hardware pela rede;
- sistema de chamados completo com SLA;
- QR/barcode obrigatório;
- taxonomia hierárquica complexa.

Esses itens só entram mediante requisito futuro explícito.

## 14. Decisões pendentes antes da implementação

- formato dos códigos legíveis de equipamento e atendimento;
- matriz de permissões operacional;
- lifecycle exato do atendimento;
- persistência e comportamento dos checklists;
- edição da ficha de equipamento após atendimento concluído;
- tamanho/layout físico da ficha imprimível;
- se a ficha compacta também exige PDF além de impressão direta;
- se categorias simples múltiplas são suficientes ou haverá necessidade real de hierarquia.
