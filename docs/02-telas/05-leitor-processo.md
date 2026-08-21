# Tela 05 — Leitor de Processo em Formato Livro

**Status:** CONSOLIDADO / APROVADO PELO PO  
**Bloco:** Fase 1 — Bloco 8 (UI/UX)  
**Última consolidação:** 2026-08-21

## 1. Objetivo

Ser a principal superfície de consumo de procedimentos do StepFlow, permitindo leitura técnica guiada com baixo atrito, mantendo cada etapa como uma página de manual e preservando claramente a revisão que está sendo consultada.

## 2. Atores e permissões

- ADM;
- Gerência;
- Funcionário/Técnico.

Todos podem ler procedimentos autorizados. Ações adicionais aparecem conforme capacidades da sessão, mas a autorização real permanece no Host.

## 3. Entrada e retorno

Fluxo principal:

```text
Processos
→ selecionar procedimento
→ Leitor
```

Também pode ser aberto a partir do Dashboard, Histórico e futuramente de um Atendimento que referencie revisão específica.

Ao retornar para Lista/Pesquisa, busca e filtros anteriores são preservados.

## 4. Estrutura aprovada

```text
← Processos

PR-014  Configuração de VLAN                              [⋯]
Redes  Infraestrutura      TI       Versão 2.0

Etapa 3 de 7                                      [ Sumário ▾ ]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

3. Configurar a VLAN no switch
Breve introdução da etapa...

1. Acesse o equipamento...
2. Entre no modo de configuração...

┌ Observação ────────────────────────────────────────────────┐
│ ...                                                        │
└────────────────────────────────────────────────────────────┘

┌ Comando ─────────────────────────────────────────────── [⧉]┐
│ configure terminal                                        │
└────────────────────────────────────────────────────────────┘

                         [ ← Etapa anterior ] [ Próxima etapa → ]
```

O Shell/sidebar global permanece visível. O leitor não cria segunda sidebar permanente.

## 5. Cabeçalho

Exibir de forma compacta:

- código;
- título;
- categorias;
- Área/Departamento;
- versão exibida;
- status editorial somente quando relevante ao contexto/perfil.

Metadados secundários não devem ocupar permanentemente grande área da tela.

## 6. Visão geral

Antes da Etapa 1 existe uma página lógica **Visão geral**, não numerada como etapa.

Ela apresenta, quando houver:

- Objetivo;
- Pré-requisitos;
- Observações gerais;
- Responsável documental;
- categorias;
- versão e informações editoriais essenciais.

`Visão geral` é apenas uma apresentação dos metadados existentes, não uma nova entidade de domínio.

## 7. Etapas como páginas

Cada `process_stage` corresponde a uma página do manual, contendo:

- número/posição;
- título;
- introdução quando existir;
- blocos ordenados;
- navegação anterior/próxima.

Duas etapas completas não devem ser fundidas em uma página apenas para reduzir cliques.

## 8. Navegação

O leitor possui:

- `Visão geral` antes da Etapa 1;
- indicador `Etapa X de Y` nas etapas;
- barra simples de **posição de navegação**, nunca percentual de conclusão do serviço;
- `Sumário` temporário;
- Anterior/Próxima.

Regras:

- Visão geral → Próxima = Etapa 1;
- Etapa 1 → Anterior = Visão geral;
- última etapa não mostra Próxima inválida;
- nova página volta ao início do conteúdo.

## 9. Sumário

O `Sumário` é dropdown/painel temporário, não segunda sidebar fixa.

Ele:

- lista Visão geral + etapas;
- marca página atual;
- permite salto direto;
- fecha após seleção;
- permite rolagem própria em procedimentos longos;
- é operável por teclado;
- não representa conclusão/progresso operacional.

## 10. Blocos suportados

O leitor renderiza os tipos conceituais consolidados:

- `paragraph` — texto normal;
- `numbered_steps` — passos/subpassos numerados;
- `checklist` — definição documental de itens;
- `note` — observação discreta;
- `warning` — alerta com maior destaque;
- `command` — comando/instrução curta monoespaçada;
- `code` — trecho maior preservando espaços/quebras.

HTML arbitrário não é fonte de verdade.

## 11. Checklist documental

No leitor independente de Atendimento, checklist representa somente a definição documental.

Esta tela não atribui persistência, conclusão, responsável ou estado operacional aos itens. Essas regras pertencem ao Bloco 9.

## 12. Cópia de comandos/código

- controle icon-only discreto;
- nome acessível para leitor de tela;
- conteúdo copiado exatamente como exibido;
- feedback curto como `✓ Copiado`;
- feedback não desloca perceptivelmente a página;
- falha de Clipboard gera mensagem curta e segura.

## 13. Ações contextuais

Conforme capacidade, menu discreto pode conter:

- Editar;
- Histórico;
- Exportar/Imprimir;
- ações documentais futuras explicitamente aprovadas.

Fica reservado também um ponto de entrada futuro **`Iniciar atendimento`**. A posição da ação é aprovada no leitor, mas lifecycle, permissões, criação efetiva e dados obrigatórios permanecem para o Bloco 9.

## 14. Revisões

- Funcionário/Técnico normalmente recebe revisão publicada/autorizada;
- Gerência/ADM podem abrir revisão específica quando permitido;
- revisão histórica/draft deve ser claramente identificada;
- versão editorial e revisão técnica são conceitos distintos.

## 15. Atualização em tempo real

Quando uma nova revisão/publicação surgir enquanto o usuário lê:

- o conteúdo aberto **não é substituído silenciosamente**;
- a revisão aberta permanece estável;
- exibir aviso discreto: `Existe uma versão mais recente deste procedimento.`;
- oferecer atualização consciente para a versão mais recente quando aplicável;
- WebSocket sinaliza e o Client reconsulta o Host.

Isso evita alterar instruções no meio do trabalho.

## 16. Estados

### Loading

Preservar Shell e estrutura do leitor, sem mostrar dados antigos como atuais.

### Processo/revisão indisponível

Mensagem simples e retorno seguro para `Processos`.

### Sem permissão

Não renderizar conteúdo protegido mantido de estado/cache anterior.

### Host indisponível

Seguir estado transversal do Shell; cache não vira fonte oficial silenciosamente.

### Arquivado

Funcionário sem acesso deixa de receber o recurso. Usuário administrativo autorizado pode receber indicação clara de `Arquivado` em contexto histórico.

### Nova versão disponível

Aviso não intrusivo conforme seção 15.

## 17. Dados exibidos

- código;
- título;
- Área/Departamento;
- responsável documental quando útil;
- status quando relevante;
- versão exibida;
- categorias;
- objetivo;
- observações;
- pré-requisitos;
- revisão selecionada;
- etapas e blocos ordenados.

IDs internos não precisam aparecer ao usuário.

## 18. Persistência

No leitor puro:

- navegar não persiste alteração;
- copiar não persiste alteração;
- abrir Sumário não persiste alteração;
- checklist não salva estado nesta fase.

Nenhuma nova persistência é criada pela Tela 05.

## 19. Contratos Client ↔ Host

Conceitualmente, o Client precisa:

1. obter procedimento/revisão autorizada por identidade estável;
2. obter metadados + categorias;
3. obter etapas/blocos ordenados;
4. distinguir atual/publicada/histórica conforme capacidade;
5. receber capacidades da sessão;
6. reconsultar após eventos relevantes.

Endpoints finais pertencem à implementação.

## 20. Concorrência

Leitura não cria lock. Se outra revisão surgir, o leitor não mescla nem troca a revisão automaticamente.

Edição continua regida por revisão otimista na Tela 06.

## 21. Acessibilidade

- foco visível;
- headings semânticos;
- Sumário operável por teclado;
- Anterior/Próxima com nomes acessíveis;
- copiar icon-only com nome acessível;
- blocos de código selecionáveis manualmente;
- alertas não dependem apenas de cor.

## 22. Janelas menores

Desktop Windows é prioridade.

- título/metadados podem quebrar linha;
- categorias podem fluir;
- metadados secundários podem recolher;
- código/comando pode rolar horizontalmente dentro do bloco;
- navegação permanece acessível;
- não criar layout mobile/hamburger sem necessidade demonstrada.

## 23. Direção visual consolidada

- corporativa, clássica e discreta;
- largura confortável de leitura;
- aparência de manual sem skeuomorfismo pesado de papel/livro;
- hierarquia tipográfica clara;
- conteúdo técnico domina a tela;
- categorias e metadados ficam secundários;
- sem segunda sidebar permanente.

## 24. Fora do escopo

- persistência de checklist/progresso;
- lifecycle de Atendimento;
- criação efetiva de Atendimento;
- edição de procedimento;
- geração técnica de PDF/DOCX/ficha;
- implementação Tauri/Host.

## 25. Critérios de aceite

- [x] Visão geral antes das etapas;
- [x] uma etapa por página;
- [x] Sumário temporário;
- [x] progresso separado de conclusão;
- [x] blocos tipados respeitados;
- [x] cópia icon-only com feedback curto;
- [x] checklist documental separado do operacional;
- [x] revisão aberta não muda silenciosamente;
- [x] categorias discretas no cabeçalho;
- [x] ponto futuro `Iniciar atendimento` reservado sem antecipar Bloco 9;
- [x] nenhum código de produção criado.

## 26. Casos de teste futuros

1. abrir Visão geral;
2. navegar todas as etapas;
3. saltar pelo Sumário;
4. copiar comando/código;
5. abrir revisão histórica autorizada;
6. receber evento de nova publicação durante leitura;
7. retornar à Lista preservando filtros;
8. perder Host durante leitura;
9. validar teclado/acessibilidade;
10. validar procedimento longo e janela menor.
