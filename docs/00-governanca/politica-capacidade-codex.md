# Política Padrão de Capacidade do Codex

**Natureza:** documento genérico e reutilizável.

## Objetivo

Definir como escolher, antes de cada tarefa destinada ao Codex, o modelo e o nível de raciocínio adequados ao risco e à complexidade do trabalho, evitando consumo desnecessário de capacidade em tarefas simples e evitando subdimensionamento em tarefas críticas.

Esta política é aplicada pelo Assistente antes de escrever ou entregar o prompt da tarefa ao Codex.

## Regra absoluta de apresentação

A recomendação de capacidade é uma instrução para o PO/usuário, não para o Codex.

Portanto, toda nova tarefa deve ser apresentada em duas partes claramente separadas:

1. **PRÉ-FLIGHT PARA O PO — NÃO ENVIAR AO CODEX**
2. **PROMPT / ENUNCIADO PARA O CODEX**

O bloco de pré-flight nunca deve ser incorporado silenciosamente ao enunciado técnico da tarefa.

O PO usa o pré-flight para selecionar manualmente o modelo e o nível de raciocínio no Codex antes de enviar o prompt.

## Critérios de avaliação

Antes de recomendar capacidade, avaliar no mínimo:

- complexidade lógica;
- tamanho do escopo;
- quantidade de arquivos/domínios afetados;
- clareza da especificação;
- necessidade de exploração do repositório;
- risco de perda de dados ou regressão;
- impacto arquitetural;
- concorrência e estado compartilhado;
- segurança e autorização;
- migrations e compatibilidade retroativa;
- dificuldade de debugging;
- necessidade de julgamento visual/produto;
- custo de uma decisão errada;
- reversibilidade da tarefa.

## Perfis recomendados

A nomenclatura exata disponível na interface pode mudar. Quando o seletor oferecer a família GPT-5.6, usar esta lógica como padrão.

### GPT-5.6 Luna — Baixo

Usar para tarefas estritamente mecânicas, pequenas e de baixo risco, por exemplo:

- listar arquivos;
- localizar símbolos;
- conferir estado Git;
- executar comandos de leitura já especificados;
- alterações documentais triviais;
- formatação simples;
- verificações determinísticas com pouca interpretação.

Não usar quando a tarefa exigir decisão técnica relevante.

### GPT-5.6 Luna — Médio

Usar para investigação limitada ou manutenção simples que exija alguma interpretação, mas possua escopo fechado e baixo risco, por exemplo:

- inventário de ambiente;
- inspeção de pré-requisitos;
- pequenas correções documentais com várias evidências;
- pequenos scripts descartáveis bem especificados;
- diagnóstico inicial de erro localizado;
- execução de checklist técnico com relatório estruturado.

Este é o perfil econômico padrão para tarefas pequenas que não sejam puramente mecânicas.

### GPT-5.6 Terra — Médio

Usar como perfil padrão de implementação comum:

- feature pequena ou média bem especificada;
- correção localizada envolvendo alguns arquivos;
- criação de módulo convencional;
- testes e integração de baixo/médio risco;
- refactor delimitado;
- implementação de contratos já decididos.

### GPT-5.6 Terra — Alto

Usar quando a implementação comum passa a exigir integração mais ampla ou raciocínio técnico significativo:

- vários módulos relacionados;
- refactor estrutural moderado;
- bugs com causa não óbvia;
- integração entre Client e Host já arquitetada;
- mudanças com compatibilidade relevante;
- testes de concorrência já especificados.

### GPT-5.6 Sol — Alto

Reservar para trabalho de alto impacto ou julgamento difícil:

- arquitetura;
- concorrência complexa;
- segurança/autorização sensível;
- migrations de alto risco;
- corrupção/recuperação de dados;
- debugging difícil atravessando múltiplas camadas;
- revisão crítica de implementação complexa;
- decisões com alto custo de reversão;
- tarefas ambíguas em que uma interpretação errada pode contaminar a base do projeto.

### GPT-5.6 Sol — XHigh/Max ou equivalente máximo da interface

Usar excepcionalmente.

Indicado apenas quando houver evidência de que Sol/Alto pode ser insuficiente e a tarefa for realmente uma das mais difíceis do projeto, por exemplo:

- investigação de falha crítica sem causa conhecida;
- revisão final de mudança arquitetural muito extensa;
- problema de concorrência ou consistência especialmente difícil;
- migração crítica com múltiplos invariantes e alto risco de perda de dados.

Não usar como padrão por conveniência.

## Regra de economia

Sempre escolher o menor perfil que ofereça margem adequada para executar a tarefa com segurança.

O objetivo não é escolher o modelo mais barato a qualquer custo nem o mais potente por precaução automática.

A pergunta correta é:

> Qual é a menor capacidade que ainda oferece confiança adequada para esta tarefa específica?

## Regra de escalonamento

Se durante a execução ficar demonstrado que o perfil escolhido foi insuficiente:

1. interromper a tentativa repetitiva;
2. registrar o motivo;
3. subir apenas um nível razoável de capacidade;
4. reenviar a tarefa com o contexto/evidências já obtidos;
5. evitar pular diretamente para a capacidade máxima sem justificativa.

## Regra de redução

Tarefas posteriores não herdam automaticamente a capacidade da tarefa anterior.

Depois de uma tarefa Sol/Alto, a próxima tarefa pode voltar para Luna/Médio se for apenas documentação ou inspeção simples.

A capacidade é escolhida por tarefa, não por fase do projeto.

## Formato obrigatório do pré-flight

Antes de cada prompt do Codex, o Assistente deve mostrar ao PO algo semelhante a:

```text
PRÉ-FLIGHT PARA VOCÊ — NÃO ENVIAR AO CODEX

Modelo recomendado: GPT-5.6 Luna
Raciocínio: Médio
Motivo: inventário de ambiente estritamente delimitado, somente leitura, com alguma interpretação de evidências, sem implementação ou decisão arquitetural.
Escalonar somente se: a inspeção revelar comportamento inconsistente ou debugging não previsto.
```

Depois, em bloco separado, deve aparecer o enunciado destinado ao Codex.

## Registro no repositório

A recomendação de capacidade de cada execução não precisa gerar commit ou entrada no diário por si só.

Registrar somente quando:

- a escolha de capacidade revelar limitação relevante do fluxo;
- houver escalonamento por insuficiência do perfil inicialmente escolhido;
- houver mudança permanente nesta política.

## Princípio final

**Capacidade é um recurso operacional a ser dimensionado, não um padrão fixo.**

Tarefas triviais não justificam o modelo mais potente; tarefas críticas não devem ser subdimensionadas apenas para economizar tokens.