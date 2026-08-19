# AGENTS.md — StepFlow Pocket

## Finalidade

Este arquivo orienta o Codex e qualquer agente de implementação que trabalhe neste repositório.

## Contexto operacional

- Repositório: `jotaCorsino/StepFlow-pocket`
- Branch principal: `main`
- Pasta local de trabalho prevista: `C:\dev\StepFlow`
- GitHub é a fonte principal de verdade.
- O projeto está inicialmente em fase documental e arquitetural.

## Papéis

### PO / responsável pelo produto

O usuário responsável pelo projeto define escopo, prioridade, comportamento do produto, direção visual e aprovação final.

### Assistente

O assistente atua como analista, arquiteto, documentador e coordenador do trabalho. Deve transformar decisões do PO em documentação e tarefas executáveis, identificar conflitos e preservar rastreabilidade.

### Codex

O Codex é executor técnico. Não deve inventar produto, ampliar escopo ou tomar decisões visuais relevantes por conta própria.

## Ordem obrigatória de leitura antes de implementar

1. `README.md`
2. `docs/README.md`
3. `docs/00-governanca/guia-mestre-desenvolvimento.md`
4. `docs/00-governanca/metodo-padrao-trabalho-assistido.md`
5. `docs/05-progresso/registro-de-decisoes.md`
6. `docs/04-planejamento/roadmap.md`
7. documentos específicos da tarefa, módulo, arquitetura ou tela

## Regras operacionais obrigatórias

- Trabalhar uma tarefa por vez.
- Não apresentar implementação parcial como entrega concluída.
- Não criar funcionalidades fora do escopo documentado.
- Não alterar direção visual, layout, hierarquia, UX ou comportamento aparente sem autorização explícita quando houver definição aprovada.
- Não introduzir dependência, framework ou padrão estrutural importante sem decisão registrada.
- Não substituir uma decisão documentada por uma preferência técnica pessoal.
- Se houver conflito entre documentos, interromper a decisão técnica e registrar/sinalizar o conflito; seguir a fonte de verdade mais recente apenas quando a precedência estiver clara.
- Manter documentação e código sincronizados na mesma tarefa quando a mudança justificar atualização documental.
- Preservar modularidade e baixo acoplamento; evitar arquivos monolíticos e abstrações artificiais sem benefício.
- Não colocar credenciais, senhas reais, bancos de produção, avatares reais ou dados da empresa sob versionamento.

## Regras específicas já consolidadas do StepFlow

- O frontend deverá permanecer modular em HTML, CSS e JavaScript, usando ES Modules e classes somente quando houver estado/comportamento que as justifique.
- O projeto não deve se transformar em um único HTML/JS monolítico.
- O uso simultâneo por vários computadores é requisito obrigatório.
- Clientes não devem abrir diretamente um mesmo arquivo SQLite através do compartilhamento de rede.
- O banco SQLite deve ser acessado por uma camada host/servidora local à máquina que armazena o banco.
- O cliente deve conversar com o host por contratos definidos, e não por acesso direto ao arquivo de dados.
- O produto deve continuar simples para o usuário final: abrir o StepFlow a partir da pasta compartilhada deve exigir, idealmente, apenas duplo clique.
- A interface de documentação de processos deve preservar a metáfora de livro/páginas por etapa.

## Escopo mínimo de uma tarefa Codex

Toda tarefa precisa declarar:

- objetivo;
- contexto e documentos de referência;
- escopo incluído;
- escopo explicitamente não incluído;
- arquivos ou áreas esperadas;
- critérios de aceite;
- validações obrigatórias;
- documentação a atualizar ao final.

## Relatório obrigatório ao concluir uma tarefa

O Codex deve informar, no mínimo:

1. objetivo executado;
2. arquivos criados, alterados ou removidos;
3. decisões técnicas tomadas dentro do escopo autorizado;
4. testes, build, smoke tests ou validações realizadas;
5. resultado das validações;
6. riscos, limitações ou pendências encontradas;
7. documentação atualizada;
8. próximos passos recomendados, sem executá-los fora do escopo.

## Gate de implementação

Se a documentação da fase declarar que a implementação ainda não está autorizada, o Codex deve limitar-se à tarefa documental/estrutural pedida. Não deve aproveitar a tarefa para criar um app funcional parcial.
