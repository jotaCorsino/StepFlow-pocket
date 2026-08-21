# Categorização, Atendimentos e Equipamentos — StepFlow

**Status:** CONSOLIDADO NO NÚCLEO DE PRODUTO / DETALHES OPERACIONAIS PENDENTES  
**Atualização:** 2026-08-21

## 1. Objetivo

Expandir o StepFlow para documentar procedimentos de diferentes áreas técnicas e, quando houver trabalho real a registrar, separar claramente o **modelo reutilizável** da **ocorrência concreta de serviço**.

O produto deve suportar, entre outros usos:

- manutenção de computadores e notebooks;
- TI geral;
- Service Desk;
- Help Desk;
- infraestrutura e servidores;
- redes de computadores;
- procedimentos internos;
- passo a passos/guias técnicos.

## 2. Modelo de domínio aprovado

Ficam consolidados três conceitos distintos:

1. **Procedimento** — modelo reutilizável/documentação oficial;
2. **Atendimento/Execução** — ocorrência concreta em que um ou mais procedimentos foram realizados;
3. **Equipamento** — ativo físico opcional relacionado ao atendimento quando fizer sentido.

```text
Procedimento oficial
        ↓ usado em
Atendimento/Execução
        ↓ relacionado opcionalmente a
Equipamento
        ↓ registra
procedimentos realizados + resumo + observações
```

Alterar um procedimento oficial no futuro não reescreve o histórico de um atendimento já realizado.

## 3. Categorização dos procedimentos

Fica consolidado:

- categorias configuráveis pela empresa, não hardcoded;
- um procedimento pode pertencer a uma ou mais categorias;
- categorias são pesquisáveis e filtráveis;
- exemplos como `Manutenção`, `TI`, `Service Desk`, `Help Desk`, `Infraestrutura`, `Redes` e `Guias` são exemplos, não enumeração fixa;
- primeira versão usa categorias simples, sem árvore hierárquica complexa ou taxonomia avançada;
- categoria pode ser arquivada sem destruir histórico.

A hierarquia de categorias só será adicionada se uma necessidade real futura justificar.

## 4. Equipamento

Equipamento é opcional. O StepFlow não exige ficha de computador para procedimentos de rede, servidor, Help Desk ou guias gerais.

Para computadores/notebooks, a ficha deve suportar conforme aplicável:

- nome do equipamento;
- tipo do equipamento;
- processador;
- memória RAM;
- armazenamento;
- sistema operacional;
- versão do sistema operacional;
- número de série;
- patrimônio/asset tag;
- um ou mais endereços MAC;
- saúde da bateria quando aplicável;
- cliente/solicitante/responsável relacionado;
- observações.

Campos sem aplicação ao equipamento permanecem vazios/ocultos; não devem se tornar burocracia obrigatória.

## 5. Identidade e busca do equipamento

Fica consolidado que a identidade principal do equipamento não depende exclusivamente de MAC, serial ou outro atributo físico mutável.

Direção aprovada:

- identificador interno estável gerado pelo StepFlow;
- código legível operacional gerado pelo sistema;
- MAC, serial, patrimônio, nome do equipamento, cliente e ordem de serviço/referência são atributos pesquisáveis;
- múltiplos MACs podem existir para o mesmo equipamento.

Isso evita quebrar histórico quando uma interface de rede é trocada, removida ou substituída.

O formato exato do código legível (`EQP-...` ou equivalente) permanece pendente para o Bloco 9/implementação.

## 6. Atendimento / execução

Atendimento é o registro operacional concreto do serviço realizado.

Fica consolidado que ele pode conter:

- identificador interno estável;
- código legível do atendimento;
- ordem de serviço/referência externa opcional;
- cliente/solicitante/responsável quando aplicável;
- equipamento associado opcional;
- técnico/responsável pelo atendimento;
- datas aplicáveis;
- observações;
- resumo do trabalho realizado;
- um ou mais procedimentos utilizados.

Um atendimento pode utilizar múltiplos procedimentos.

O lifecycle exato — estados, conclusão, reabertura e regras de edição — será fechado no Bloco 9.

## 7. Vínculo histórico com o procedimento

Cada procedimento utilizado em um atendimento deve preservar referência à **revisão efetivamente utilizada**, não apenas ao procedimento atual.

Assim, se um atendimento foi executado com a revisão 1.3 e o procedimento depois evoluir para 2.0, o histórico continua refletindo corretamente o que existia quando o trabalho foi feito.

A persistência de checklist/progresso por etapa será fechada no Bloco 9.

## 8. Área operacional `Atendimentos`

Fica aprovado usar **`Atendimentos`** como nome da área operacional principal no Client.

Separação de navegação:

- `Processos` — documentação/modelos oficiais;
- `Atendimentos` — serviços/execuções reais.

Essa separação não impede iniciar um atendimento a partir do leitor de um procedimento.

## 9. Ficha compacta / etiqueta imprimível

Atendimentos com equipamento devem poder gerar uma saída compacta destinada a acompanhamento físico do equipamento.

Conteúdo esperado, quando disponível:

- identidade da empresa/StepFlow;
- código do atendimento;
- ordem de serviço/referência;
- cliente/solicitante;
- código/nome do equipamento;
- processador;
- RAM;
- armazenamento;
- sistema/versão;
- serial/patrimônio/MAC conforme aplicável;
- saúde da bateria quando informada;
- resumo dos procedimentos realizados;
- observações relevantes;
- técnico/responsável;
- data do atendimento/conclusão.

A saída é documento próprio, não captura de tela.

Tamanho físico, paginação, QR/barcode, tecnologia de geração e necessidade de PDF além da impressão direta pertencem ao Bloco 10.

## 10. Cenários suportados

O modelo deve permitir:

```text
procedimento sem atendimento formal
procedimento + atendimento sem equipamento
procedimento + atendimento + equipamento
atendimento com múltiplos procedimentos
mesmo equipamento relacionado a atendimentos diferentes
```

Isso preserva o uso amplo do StepFlow sem transformá-lo em sistema exclusivo de manutenção de PCs.

## 11. Permissões

Já consolidado:

- Funcionário/Técnico pode executar procedimento sem receber permissão para editar documentação oficial;
- autorização real permanece no Host.

Pendente para o Bloco 9:

- quem cria/edita categorias;
- quem cadastra/altera/arquiva equipamento;
- quem inicia/edita/conclui/reabre atendimento;
- quem gera/reimprime ficha.

## 12. Concorrência e histórico

- equipamentos e atendimentos são dados oficiais do Host;
- Clients nunca editam SQLite diretamente;
- alterações concorrentes relevantes usam controle otimista equivalente ao restante do produto;
- atendimento concluído preserva a revisão do procedimento utilizada;
- ficha/relatório reflete estado confirmado pelo Host.

## 13. Impactos na Fase 1

### Bloco 8 — UI/UX

Deve incorporar:

- categorias na lista/editor/leitor de procedimentos;
- `Atendimentos` na navegação;
- superfícies de atendimento/execução;
- ficha de equipamento;
- busca por atendimento/equipamento;
- ação de gerar/imprimir ficha compacta.

### Bloco 9 — Execução operacional e checklist

Fechar:

- lifecycle do atendimento;
- criação/edição/conclusão/reabertura;
- progresso/checklist;
- regras de edição após conclusão;
- concorrência operacional;
- matriz de permissões;
- formato final dos códigos legíveis.

### Bloco 10 — Exportação/impressão

Além de PDF/DOCX/impressão dos procedimentos, fechar a ficha compacta de atendimento/equipamento.

## 14. Fora do escopo inicial

Não transformar automaticamente esta capacidade em:

- CRM completo;
- faturamento/financeiro;
- estoque de peças;
- RMM/inventário automatizado;
- descoberta automática de hardware pela rede;
- sistema de chamados completo com SLA;
- QR/barcode obrigatório;
- taxonomia hierárquica complexa.

Esses itens só entram mediante requisito futuro explícito.

## 15. Pendências vigentes

- formato exato dos códigos legíveis;
- lifecycle do atendimento;
- matriz operacional de permissões;
- persistência/comportamento do checklist;
- regras de edição de equipamento/atendimento após conclusão;
- tamanho/layout físico da ficha;
- necessidade de PDF específico da ficha além da impressão direta;
- uso futuro de QR/barcode somente se houver benefício real.