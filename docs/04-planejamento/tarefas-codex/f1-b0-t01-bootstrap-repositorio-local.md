# F1-B0-T01 — Bootstrap do Repositório Local

**Fase:** 1 — Fechamento arquitetural e especificação  
**Bloco:** 0 — Preparação do ambiente de trabalho  
**Status:** PRONTA PARA EXECUÇÃO

## 1. Objetivo

Preparar `C:\dev\StepFlow` como cópia local íntegra do repositório oficial `jotaCorsino/StepFlow-pocket`, sem iniciar qualquer implementação do produto.

Esta tarefa existe antes de todas as demais tarefas Codex da Fase 1.

## 2. Contexto

O repositório remoto já foi inicializado e contém a documentação oficial do projeto. O ambiente local ainda não foi preparado.

Como o repositório é público, leitura e clone por HTTPS não exigem autenticação GitHub. Credenciais serão necessárias apenas quando uma tarefa futura exigir `push` para o remoto.

Repositório remoto:

`https://github.com/jotaCorsino/StepFlow-pocket.git`

Pasta local oficial:

`C:\dev\StepFlow`

Branch principal:

`main`

## 3. Estado inicial esperado

Pode ocorrer um destes cenários:

- `C:\dev\StepFlow` não existe;
- `C:\dev\StepFlow` existe e está vazia;
- `C:\dev\StepFlow` existe com arquivos não versionados.

O terceiro cenário exige parada antes de apagar ou sobrescrever qualquer conteúdo.

## 4. Escopo incluído

- verificar disponibilidade do Git;
- verificar o estado de `C:\dev\StepFlow`;
- clonar o repositório oficial quando a pasta não existir ou estiver vazia;
- confirmar branch `main`;
- confirmar `origin` apontando para o repositório oficial;
- confirmar que o checkout contém `README.md`, `AGENTS.md` e `docs/`;
- confirmar que o working tree está limpo;
- registrar evidências do bootstrap.

## 5. Fora do escopo

Não fazer nesta tarefa:

- instalar Node.js, Rust, Tauri ou qualquer SDK;
- criar `package.json`;
- criar `src-tauri`;
- criar código do Client ou Host;
- criar SQLite;
- criar launcher;
- editar documentação de produto ou arquitetura;
- criar nova branch;
- realizar commit;
- realizar push;
- configurar credenciais GitHub;
- apagar arquivos preexistentes em `C:\dev\StepFlow`.

## 6. Procedimento esperado

### 6.1. Verificar Git

Executar:

```powershell
git --version
```

Se Git não estiver disponível, parar e reportar o bloqueio. Não instalar automaticamente.

### 6.2. Verificar a pasta local

Inspecionar `C:\dev\StepFlow`.

Se não existir, clonar diretamente para ela:

```powershell
git clone https://github.com/jotaCorsino/StepFlow-pocket.git C:\dev\StepFlow
```

Se existir e estiver completamente vazia, o mesmo comando pode ser utilizado.

Se existir e contiver qualquer arquivo ou subpasta, **não apagar, mover ou sobrescrever nada automaticamente**. Registrar o conteúdo encontrado e interromper para decisão.

### 6.3. Validar checkout

Dentro de `C:\dev\StepFlow`, executar:

```powershell
git status
git branch --show-current
git remote -v
git log -1 --oneline
```

Também confirmar a existência de:

```text
README.md
AGENTS.md
docs/
```

## 7. Critérios de aceite

- [ ] Git disponível no ambiente;
- [ ] `C:\dev\StepFlow` corresponde ao repositório oficial;
- [ ] branch atual é `main`;
- [ ] `origin` aponta para `https://github.com/jotaCorsino/StepFlow-pocket.git` ou URL GitHub equivalente do mesmo repositório;
- [ ] `README.md` existe;
- [ ] `AGENTS.md` existe;
- [ ] `docs/` existe;
- [ ] `git status` não apresenta alterações locais inesperadas;
- [ ] nenhuma dependência de aplicação foi instalada;
- [ ] nenhum arquivo existente foi apagado para forçar o clone.

## 8. Validações obrigatórias

Incluir no relatório final as saídas relevantes de:

```powershell
git --version
git status
git branch --show-current
git remote -v
git log -1 --oneline
```

E uma listagem resumida da raiz de `C:\dev\StepFlow`.

## 9. Documentação a atualizar

Nenhuma atualização documental é obrigatória nesta tarefa, porque seu único objetivo é preparar a cópia local da fonte de verdade já existente.

Não editar o diário ou changelog apenas para registrar o clone local.

## 10. Relatório final obrigatório

Informar:

1. versão do Git;
2. estado inicial encontrado em `C:\dev\StepFlow`;
3. se o clone foi realizado;
4. branch atual;
5. URL(s) de `origin`;
6. commit atual (`HEAD`);
7. estado do working tree;
8. confirmação de `README.md`, `AGENTS.md` e `docs/`;
9. qualquer bloqueio encontrado.

## 11. Regra de parada

Parar sem alterar conteúdo se:

- Git não estiver instalado;
- `C:\dev\StepFlow` contiver arquivos preexistentes;
- o clone apontar para repositório diferente;
- houver erro de rede que impeça acesso ao GitHub;
- houver qualquer situação que exija apagar ou sobrescrever arquivos locais.

Não improvisar solução destrutiva.
