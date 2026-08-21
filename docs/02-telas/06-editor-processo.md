# Tela 06 — Editor de Processo + Categorias

**Status:** CONSOLIDADO / APROVADO PELO PO  
**Bloco:** Fase 1 — Bloco 8 (UI/UX)  
**Última consolidação:** 2026-08-21

## 1. Objetivo

Permitir que usuários autorizados criem e mantenham procedimentos oficiais de forma estruturada, sem HTML livre e sem workflow editorial burocrático.

O Editor reflete o mesmo modelo consumido pelo Leitor: metadados, categorias, etapas ordenadas e blocos tipados.

## 2. Atores e permissões

Normalmente:

- ADM;
- Gerência.

Funcionário/Técnico não recebe edição oficial por padrão. O Client oculta controles sem permissão, mas o Host valida todas as operações.

## 3. Entrada

```text
Processos → Novo processo
Processos → menu do item → Editar
Leitor → menu contextual → Editar
```

Ao editar procedimento existente, o Client mantém a revisão-base para controle otimista.

## 4. Estrutura aprovada

O Editor possui duas áreas internas:

- **Informações** — identidade/metadados gerais;
- **Etapas** — estrutura e conteúdo técnico.

Trocar entre as áreas não salva automaticamente.

### Informações

Campos conforme capacidade/contrato:

- Código;
- Título;
- Área/Departamento;
- Responsável;
- Categorias;
- Versão exibida;
- Objetivo;
- Observações;
- Pré-requisitos.

`Status` editorial só aparece quando houver regra real para ele; não criar dropdown genérico por conveniência.

### Etapas

A área possui painel contextual `Estrutura`, com lista ordenada das etapas e conteúdo da etapa selecionada.

Ações:

- criar etapa;
- editar título/introdução;
- selecionar etapa;
- reordenar;
- remover com confirmação proporcional quando houver conteúdo.

A numeração deriva da ordem, não é digitada manualmente.

## 5. Categorias

Decisão consolidada:

- categorias são múltiplas e configuráveis;
- multi-select simples, com busca quando necessário;
- categorias associadas aparecem como chips removíveis;
- o Editor seleciona categorias existentes;
- criação/arquivamento de categoria acontece fora do Editor, em Configurações;
- sem taxonomia hierárquica na primeira versão.

## 6. Blocos tipados

`Adicionar bloco` oferece somente:

- Parágrafo (`paragraph`);
- Passos numerados (`numbered_steps`);
- Checklist (`checklist`);
- Observação (`note`);
- Alerta (`warning`);
- Comando (`command`);
- Código (`code`).

Não usar HTML arbitrário como fonte de verdade.

Cada bloco possui tipo, conteúdo, ordem e ações contextuais de mover/remover.

## 7. Reordenação

- drag-and-drop pode existir como atalho;
- sempre existe alternativa acessível `Mover para cima` / `Mover para baixo`;
- ordem só vira oficial após save aceito pelo Host;
- reordenação local não cria lock.

## 8. Passos numerados

Permitir:

- passos;
- subpassos em hierarquia limitada/compreensível;
- editar;
- reordenar;
- remover.

A numeração é derivada da estrutura.

## 9. Checklist documental

No Editor, checklist define apenas os itens do procedimento.

Não existem aqui campos de execução como concluído, executado por, data ou percentual. Esses conceitos pertencem ao Bloco 9/Atendimento.

## 10. Comando e Código

- fonte monoespaçada;
- preservar espaços/quebras relevantes;
- área de edição apropriada;
- preview final é o Leitor;
- StepFlow não executa comandos diretamente.

## 11. Salvamento explícito

Decisão consolidada: **sem autosave inicial**.

Cada save aceito cria nova revisão imutável. Enquanto houver alterações locais, mostrar `Alterações não salvas` de forma discreta.

Fluxo:

```text
Client envia estado estruturado + base_revision
→ Host valida autorização/estrutura/revisão
→ writer/fila coordenada
→ transação
→ nova process_revision imutável
→ current_revision_id atualizado
→ resposta + evento pós-commit
```

Revisão antiga nunca é editada in-place.

## 12. Concorrência e conflito

Se outro usuário salvar primeiro:

- Host responde `409 REVISION_CONFLICT`;
- não sobrescrever automaticamente;
- manter alterações locais visíveis/em memória;
- avisar que existe versão mais recente;
- não oferecer overwrite forçado silencioso;
- recarregar versão atual só após ação consciente;
- confirmar antes de descartar alterações locais.

Não implementar merge automático na primeira versão.

Evento WebSocket durante edição não substitui campos locais. Ele apenas sinaliza mudança e provoca reconciliação segura quando necessário.

## 13. Saída com alterações não salvas

Ao voltar, trocar de rota relevante ou fechar o Client com mudanças locais:

`Há alterações não salvas. Deseja sair e descartá-las?`

Não persistir rascunho local oculto sem requisito aprovado.

## 14. Visualizar

`Visualizar` abre o Leitor usando a **última revisão já salva**.

Não prometer preview de conteúdo ainda não salvo na primeira versão.

## 15. Publicação

`Salvar alterações` e `Publicar revisão atual` são ações distintas.

Regras:

- save cria nova revisão atual;
- save não publica automaticamente;
- usuário com capacidade apropriada pode publicar a revisão atual;
- publicação não cria workflow complexo de revisão/aprovação;
- versão publicada é identificável no Leitor e Histórico.

## 16. Novo processo

- formulário começa sem identidade persistida;
- primeira gravação válida cria `process_id` estável e primeira revisão;
- não publicar automaticamente apenas por criar;
- cancelar antes do primeiro save não cria registro oficial.

## 17. Validações

Host é autoridade para:

- unicidade do Código;
- campos obrigatórios;
- limites/tipos de conteúdo;
- categorias válidas/ativas;
- capacidade do ator;
- revisão-base;
- integridade de etapas/blocos.

Client replica validações simples apenas para UX.

## 18. Estados

### Loading

Não mostrar dados antigos como atuais.

### Novo/vazio

Formulário limpo e primeira ação orientada.

### Erro de validação

Marcar campos/blocos e preservar restante da edição.

### Sem permissão

Não entrar em modo editável.

### Host indisponível

Não salvar nem fingir confirmação; manter estado local enquanto a tela permanecer aberta.

### Conflito

Aplicar a seção de concorrência.

### Processo arquivado durante edição

Reconsultar Host e bloquear save incompatível com o estado atual.

## 19. Autorização

Capacidades podem separar:

- criar procedimento;
- editar conteúdo/metadados;
- publicar;
- arquivar;
- gerenciar categorias.

Perfil é preset; autorização final é Host-side.

## 20. Persistência e comunicação

- somente saves aceitos pelo Host são oficiais;
- nenhum acesso direto ao SQLite;
- nenhuma fila local offline;
- nenhum estado “aceito mas ainda não commitado”;
- eventos somente após commit.

Contratos conceituais:

1. carregar processo/revisão editável;
2. listar categorias autorizadas;
3. criar processo;
4. salvar revisão com `base_revision`;
5. publicar revisão atual;
6. receber capacidades/eventos;
7. arquivar por contrato próprio quando aplicável.

Nomes finais de endpoints pertencem à implementação.

## 21. Acessibilidade e janela

- labels reais;
- foco previsível/visível;
- mover etapa/bloco por teclado;
- drag-and-drop nunca é único meio;
- erros associados aos campos;
- chips de categoria acessíveis;
- menus fecham com Escape;
- foco preservado após adicionar/remover blocos.

Desktop Windows é prioridade. Em janela menor, Informações pode virar uma coluna e o painel Estrutura pode reduzir largura sem esconder controles essenciais.

## 22. Fora do escopo

- HTML/WYSIWYG genérico;
- Markdown obrigatório como fonte oficial;
- autosave a cada tecla;
- merge automático;
- colaboração caractere a caractere;
- criação inline de categorias;
- execução de comandos;
- checklist operacional;
- workflow editorial complexo;
- código Tauri/Host.

## 23. Decisões consolidadas nesta tela

1. separar `Informações` e `Etapas`;
2. usar painel contextual `Estrutura`;
3. salvamento explícito, sem autosave inicial;
4. cada save aceito gera nova revisão imutável;
5. blocos tipados, sem HTML livre;
6. categorias selecionadas no Editor e gerenciadas fora dele;
7. drag-and-drop apenas como complemento a ações acessíveis;
8. conflito preserva edição local e nunca faz overwrite automático;
9. `Visualizar` mostra última revisão salva;
10. `Salvar` e `Publicar revisão atual` são ações distintas.

## 24. Pendências menores

- limites finais por campo/bloco;
- microcopy final;
- aparência exata do seletor múltiplo;
- apresentação visual final do estado publicado/rascunho;
- eventual preview local futuro.

## 25. Critérios de aceite

- [x] layout Informações/Etapas aprovado;
- [x] painel Estrutura aprovado;
- [x] salvamento explícito aprovado;
- [x] revisão imutável preservada;
- [x] blocos tipados aprovados;
- [x] categorias integradas;
- [x] checklist documental separado do operacional;
- [x] conflito sem overwrite silencioso;
- [x] Visualizar usa revisão salva;
- [x] Salvar/Publicar separados;
- [x] nenhum código de produção criado.

## 26. Casos de teste futuros

1. criar processo e cancelar antes de salvar;
2. criar primeira revisão;
3. editar metadados/categorias;
4. adicionar/reordenar/remover etapas;
5. adicionar cada tipo de bloco;
6. reordenar por teclado;
7. sair com alterações não salvas;
8. salvar com base atual;
9. receber `409` após edição concorrente;
10. receber evento durante edição;
11. visualizar última revisão salva;
12. publicar com/sem permissão;
13. perder Host antes do save;
14. validar janela menor/acessibilidade.
