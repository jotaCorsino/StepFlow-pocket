# Categorização, Atendimentos e Equipamentos — StepFlow

**Status:** REQUISITOS DE PRODUTO CONFIRMADOS / MODELAGEM PROPOSTA PARA APROVAÇÃO  
**Atualização:** 2026-08-21

## 1. Objetivo

Expandir o StepFlow para documentar procedimentos de diferentes áreas técnicas e, em cenários que exigem rastreabilidade do trabalho realizado, registrar informações específicas do serviço/equipamento e gerar uma ficha compacta imprimível.

O produto deve suportar, entre outros usos:

- manutenção de computadores e notebooks;
- TI geral;
- Service Desk;
- Help Desk;
- infraestrutura e servidores;
- redes de computadores;
- procedimentos internos;
- passo a passos/guias técnicos.

## 2. Requisitos confirmados pelo PO

Estão confirmados:

1. sistema de categorização de procedimentos;
2. categorias devem permitir organizar procedimentos de diferentes áreas de atuação;
3. em manutenção de computadores/notebooks deve existir uma área para registrar informações específicas do equipamento;
4. a ficha deve suportar, conforme aplicável: nome, processador, RAM, armazenamento, versão do sistema, MAC ou outro identificador útil, saúde da bateria e observações;
5. deve ser possível relacionar o registro a cliente e/ou ordem de serviço/referência útil para busca;
6. o sistema deve facilitar busca pelos identificadores operacionais disponíveis;
7. deve existir resumo do que foi feito/procedimentos realizados;
8. a ficha resumida precisa poder ser extraída/gerada para impressão e anexação física ao equipamento;
9. a funcionalidade de manutenção de computadores não pode limitar o uso do StepFlow em redes, infraestrutura, Service Desk, Help Desk, guias ou outros procedimentos.

## 3. Modelagem recomendada — ainda sujeita à aprovação do PO

Para atender aos requisitos sem misturar documentação com registros de serviço, a direção recomendada é separar:

1. **Procedimento** — modelo reutilizável/documentação oficial;
2. **Atendimento/Execução** — ocorrência concreta em que um ou mais procedimentos foram realizados;
3. **Equipamento** — ativo físico opcional relacionado ao atendimento quando fizer sentido.

Exemplo recomendado:

```text
Procedimento oficial
"Manutenção preventiva de notebook"
        ↓ usado em
Atendimento/registro de serviço
        ↓ relacionado opcionalmente a
Equipamento
        ↓ registra
procedimentos realizados + resumo + observações
```

Essa separação é **PROPOSTA DE MODELAGEM**, não autorização de implementação até aprovação do PO e fechamento do Bloco 9.

## 4. Categorização dos procedimentos

### Requisito confirmado

Procedimentos precisam ser organizáveis por categorias adequadas aos diferentes contextos de uso.

### Proposta recomendada

- categorias configuráveis pela empresa, não hardcoded;
- procedimento pode pertencer a uma ou mais categorias;
- categorias pesquisáveis/filtráveis;
- exemplos como `Manutenção`, `TI`, `Service Desk`, `Help Desk`, `Infraestrutura`, `Redes` e `Guias` são exemplos, não enumeração fixa;
- começar sem árvore hierárquica complexa, tags paralelas ou taxonomia avançada;
- permitir arquivamento preservando histórico.

A necessidade real de múltiplas categorias versus categoria única/hierárquica deve ser validada antes da implementação.

## 5. Ficha de equipamento

### Campos confirmados pelo requisito

Para manutenção de computadores/notebooks, a ficha deve suportar:

- nome do equipamento;
- processador;
- memória RAM;
- armazenamento;
- sistema operacional/versão;
- endereço MAC ou identificador equivalente útil;
- saúde da bateria quando aplicável;
- observações;
- vínculo/referência com cliente e/ou ordem de serviço para facilitar localização.

### Campos adicionais recomendados

- tipo do equipamento;
- número de série;
- patrimônio/asset tag;
- técnico/responsável;
- código interno legível do equipamento.

Esses campos adicionais são propostas e podem ser ajustados pelo PO.

## 6. Identidade e busca

### Requisito confirmado

O usuário precisa localizar facilmente o registro usando informações reais do trabalho/equipamento.

### Recomendação técnica

Não usar MAC como única identidade canônica. A direção recomendada é:

- identificador interno estável gerado pelo StepFlow;
- código legível para uso operacional;
- serial, patrimônio, MAC, nome, cliente e OS/referência como atributos de busca.

Motivo: um equipamento pode ter múltiplas interfaces, adaptadores substituídos ou endereços alteráveis. A identidade interna evita que uma mudança física quebre o histórico.

Busca proposta, conforme dados disponíveis:

- código interno do equipamento;
- código do atendimento/registro;
- nome do equipamento;
- cliente/solicitante/responsável;
- ordem de serviço/referência externa;
- número de série;
- patrimônio;
- MAC normalizado.

O formato de códigos como `EQP-...`/`ATD-...` é pendente.

## 7. Registro do serviço realizado

### Requisito confirmado

A ficha precisa guardar resumo do que foi feito e os procedimentos realizados.

### Modelagem recomendada

Criar um registro de atendimento/execução com:

- identificador interno;
- código legível;
- OS/referência externa opcional;
- cliente/solicitante/responsável quando aplicável;
- equipamento opcional;
- técnico/responsável;
- datas aplicáveis;
- observações;
- resumo do trabalho realizado;
- um ou mais procedimentos usados.

Para preservar histórico, recomenda-se vincular o atendimento à **revisão exata** do procedimento que foi utilizada, não apenas ao procedimento atual.

Lifecycle, estados e regras de conclusão/reabertura permanecem pendentes do Bloco 9.

## 8. Ficha compacta / etiqueta imprimível

### Requisito confirmado

Precisa existir uma saída compacta que possa ser impressa e anexada fisicamente ao equipamento.

Conteúdo esperado, quando disponível:

- identidade da empresa/StepFlow;
- cliente/solicitante;
- OS/referência;
- identificação do equipamento;
- processador;
- RAM;
- armazenamento;
- sistema/versão;
- identificadores úteis como serial/patrimônio/MAC;
- saúde da bateria quando aplicável;
- resumo dos procedimentos realizados;
- observações relevantes;
- técnico/responsável;
- data.

A saída deve ser documento próprio, não captura de tela.

Tamanho físico, paginação, PDF, QR/barcode e mecanismo de geração pertencem ao Bloco 10 e continuam pendentes.

## 9. Generalidade do produto

O StepFlow não deve exigir ficha de computador para todo procedimento.

A direção recomendada deve permitir cenários como:

```text
procedimento sem registro formal de serviço
procedimento + registro de serviço sem equipamento
procedimento + registro de serviço + equipamento
registro de serviço com mais de um procedimento
```

Essa flexibilidade é proposta para preservar o uso em Service Desk, servidores, redes, guias e manutenção física.

## 10. Permissões

Confirmado:

- Funcionário/Técnico pode executar procedimento sem receber permissão para editar documentação oficial;
- autorização real permanece no Host.

Pendente:

- quem cria/edita categorias;
- quem cadastra/altera equipamento;
- quem inicia/edita/conclui atendimento;
- quem pode reabrir atendimento;
- quem gera/reimprime ficha.

## 11. Concorrência e histórico

Se a modelagem recomendada for aprovada:

- equipamento e atendimento serão dados oficiais do Host;
- Clients nunca editarão SQLite diretamente;
- alterações concorrentes relevantes usarão revisão/controle otimista;
- atendimento concluído preservará a revisão de procedimento utilizada;
- ficha impressa refletirá estado confirmado pelo Host.

## 12. Impacto na Fase 1

### Bloco 8 — UI/UX

Deve incorporar os **requisitos confirmados**:

- categorias na experiência de procedimentos;
- área de registro do serviço/equipamento;
- busca operacional;
- ação de gerar/imprimir ficha compacta.

A estrutura exata das telas depende da aprovação da modelagem recomendada.

### Bloco 9 — execução operacional e checklist

Se a separação Atendimento/Equipamento for aprovada, o Bloco 9 fecha:

- entidades/regras operacionais finais;
- lifecycle;
- vínculo com procedimento/revisão;
- checklist/progresso;
- concorrência/histórico;
- permissões.

### Bloco 10 — exportação/impressão

Fechará a estratégia da ficha compacta além dos documentos completos de procedimentos.

## 13. Fora do escopo por enquanto

Não transformar automaticamente esta necessidade em:

- CRM completo;
- faturamento/financeiro;
- estoque de peças;
- inventário/RMM completo;
- descoberta automática de hardware;
- sistema de chamados completo com SLA;
- QR/barcode obrigatório;
- taxonomia hierárquica complexa.

## 14. Pontos para aprovação do PO

1. aprovar separação **Procedimento × Atendimento/Execução × Equipamento**;
2. aprovar categorias configuráveis, inicialmente simples e potencialmente múltiplas;
3. aprovar identificador interno/código StepFlow como identidade principal, mantendo MAC/serial/patrimônio como busca;
4. aprovar `Atendimentos` como nome/área operacional ou escolher outro termo;
5. aprovar que um atendimento possa referenciar mais de um procedimento;
6. aprovar vínculo histórico com a revisão do procedimento utilizada;
7. confirmar se ficha compacta precisa de PDF além de impressão direta;
8. lifecycle, permissões e checklist ficam para o Bloco 9.
