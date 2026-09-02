# Rebaseline — StepFlow web estático single-user

**Status:** EM ANÁLISE / REBASELINE ARQUITETURAL  
**Data:** 2026-09-02

## Motivo

A infraestrutura disponível mudou: o ambiente de hospedagem é gerenciado por cPanel e não aceita executáveis próprios do StepFlow no servidor.

Além disso, o produto deixa de perseguir multiusuário central. O novo objetivo é uma aplicação **single-user, local-first, estática e de consumo mínimo**, preservando o domínio e a experiência de uso já aprovados quando continuarem fazendo sentido.

A arquitetura anterior baseada em executáveis, Client/Host/Controller/Launcher, Tauri, SQLite central, WebSocket, autenticação e Backup/Restore deixa de ser vigente.

## Direção consolidada para o rebaseline

```text
cPanel / HTTPS
→ entrega somente HTML + CSS + JavaScript + assets estáticos
→ navegador executa toda a aplicação
→ IndexedDB persiste os dados localmente
→ Service Worker mantém o app disponível offline
→ Exportar dados gera arquivo portátil
→ Importar dados recria/substitui o conjunto local após validação
```

Não existe backend obrigatório do StepFlow.

Não existe banco no cPanel.

Não existe PHP/Node/Python/Rust em runtime do StepFlow no servidor.

## Distribuição

### Principal — site estático hospedado

- arquivos estáticos publicados no cPanel;
- HTTPS obrigatório;
- abre em navegador moderno;
- instalação PWA opcional/recomendada para uso offline e experiência de aplicativo;
- atualizações do código vêm da publicação estática;
- dados permanecem no navegador/perfil do dispositivo.

### Portátil opcional — HTML autocontido

Pode existir um artefato `StepFlow.html` gerado pelo build, contendo CSS/JS/assets necessários para abrir por duplo clique.

Esse modo é uma saída portátil e **não é a arquitetura principal de persistência**, porque comportamento de origem e armazenamento para `file://` varia entre navegadores. Ele deve permitir navegação e operações do app e priorizar Importar/Exportar dados como mecanismo confiável de portabilidade.

A nomenclatura antiga de distribuição não será reutilizada.

## Single-user

O StepFlow passa a considerar um conjunto de dados local por instalação/origem do navegador.

Ficam removidos do produto baseline:

- login;
- senha;
- sessão/token;
- usuários;
- perfis de acesso;
- ADM/Gerência/Funcionário;
- autorização por capacidade;
- presença online;
- WebSocket;
- sincronização entre dispositivos;
- concorrência entre usuários;
- conflitos distribuídos;
- auditoria por identidade autenticada.

Campos como `Responsável` continuam sendo dados de negócio e podem usar texto/lista local de responsáveis, sem transformar essas pessoas em contas do sistema.

## Persistência local

Baseline: **IndexedDB**.

Motivos:

- armazenamento assíncrono;
- objetos estruturados e blobs;
- índices e transações locais;
- não bloqueia a UI como armazenamento síncrono;
- adequado a dados maiores que preferências simples.

`localStorage` fica restrito a preferências pequenas de UI quando útil; não é datastore principal.

### Biblioteca candidata

`dexie/Dexie.js` é candidata preferencial para encapsular IndexedDB, migrations locais, queries e transações com API pequena e madura. A adoção final deve usar apenas o bundle necessário e ser medida no build.

## Modelo de dados

O domínio continua orientado a:

```text
Procedimentos
Atendimentos / Execuções
Equipamentos opcionais
Categorias
Configuração da empresa
Responsáveis locais
```

Revisões imutáveis, histórico e checklist continuam úteis mesmo em single-user.

Identificadores são gerados localmente.

Não existe schema SQL. Evolução de persistência ocorre por versões/migrations do IndexedDB.

## Exportação / Importação de dados

Backup/Restore deixa de existir no produto.

A substituição é **Exportação / Importação de dados**.

### Exportar dados

Gera um arquivo portátil contendo o estado funcional do StepFlow:

- metadados de formato e versão;
- configuração da empresa;
- categorias;
- responsáveis locais;
- procedimentos e revisões;
- atendimentos;
- equipamentos;
- assets administrados, quando aplicável.

Formato deve ser versionado, validável e determinístico.

Preferência: arquivo único `.stepflow` comprimido quando isso trouxer ganho real; JSON legível permanece uma opção de fallback/desenvolvimento.

### Importar dados

Fluxo baseline:

```text
selecionar arquivo
→ validar formato/versão/limites
→ mostrar resumo
→ confirmar substituição ou modo suportado
→ aplicar em transação local
→ recarregar estado
```

Importação nunca aceita conteúdo arbitrário como HTML/script executável.

Não há Restore de filesystem, journal de recovery, safety backup ou operação destrutiva no servidor.

### Biblioteca candidata de compactação

`101arrowz/fflate` pode ser usada para ZIP/deflate em browser quando a medição justificar. Caso APIs nativas sejam suficientes, não adicionar dependência.

## Offline

A forma confiável de offline é a aplicação hospedada em HTTPS + Service Worker/PWA:

```text
primeiro acesso online
→ assets são cacheados
→ usuário pode instalar/adicionar o app
→ uso posterior funciona sem rede
→ dados continuam no IndexedDB local
```

Nenhuma sincronização em background é necessária porque não existe backend de dados.

## Frontend

Tecnologia runtime:

- HTML semântico;
- CSS próprio;
- JavaScript moderno;
- módulos por feature/componente/serviço;
- nenhuma framework SPA obrigatória.

O source permanece modular mesmo se o build gerar um ou poucos arquivos estáticos finais.

Não criar um `index.html` monolítico como fonte de desenvolvimento.

## Build e artefatos

Ferramentas de build podem existir apenas no desenvolvimento/CI.

Objetivos:

- bundle pequeno;
- tree shaking/minificação;
- hashes quando útil;
- `dist/` puramente estático;
- variante hospedada;
- variante `StepFlow.html` autocontida quando suportada;
- nenhum runtime de desenvolvimento enviado ao cPanel.

Um bundler pequeno como `esbuild` pode ser avaliado no início da implementação. Ele não vira requisito de runtime.

## Consumo de recursos

Princípios obrigatórios:

- zero processo de servidor do StepFlow;
- zero banco de dados no servidor;
- zero polling/WebSocket;
- zero framework pesado por padrão;
- carregar somente recursos da tela/feature quando necessário se houver benefício mensurável;
- evitar bibliotecas para funcionalidades triviais oferecidas pelo navegador;
- imagens dimensionadas/comprimidas;
- fonte do sistema por padrão, evitando webfonts obrigatórias;
- ícones SVG locais/inline, sem icon font;
- animações discretas e sem loops permanentes;
- datasets renderizados com paginação/virtualização somente quando necessário.

## Geração documental

A geração anterior dependente de Host/Rust/Typst/OOXML será reavaliada.

Baseline minimalista proposto:

- impressão e PDF pelo próprio navegador usando HTML/CSS de impressão;
- Ficha e Procedimento possuem layouts `@media print` controlados;
- `window.print()` atende impressão e `Salvar como PDF` sem biblioteca pesada;
- DOCX automático sai do baseline inicial, podendo voltar somente se houver necessidade real e biblioteca browser-side pequena/confiável.

## cPanel

O cPanel é tratado apenas como hospedagem de arquivos estáticos.

A publicação não depende de Application Manager, Passenger, PHP, MySQL, MariaDB, SSH ou cron.

Estrutura conceitual:

```text
public_html/stepflow/
├── index.html
├── manifest.webmanifest
├── sw.js
├── assets/
│   ├── app-*.js
│   ├── app-*.css
│   └── ...
└── icons/
```

## Segurança no novo modelo

Sem backend e sem autenticação, a segurança muda de escopo:

- HTTPS protege entrega do código hospedado;
- CSP deve restringir execução de script/origens;
- sem CDN obrigatória em produção;
- dependências são vendorizadas/bundled e pinadas;
- conteúdo digitado pelo usuário é tratado como dados, não HTML confiável;
- importação possui limites de tamanho/estrutura;
- dados locais herdam a segurança do perfil/navegador/dispositivo;
- proteção contra acesso físico ao dispositivo não é responsabilidade do StepFlow baseline.

## Recursos open source

Política:

1. preferir Web Platform nativa;
2. adotar biblioteca somente para reduzir complexidade/risco relevante;
3. preferir projetos maduros, pequenos, permissivos e ativos;
4. fixar versão e licença;
5. medir tamanho real no bundle;
6. evitar frameworks/ecossistemas grandes sem necessidade.

Candidatos iniciais:

- `dexie/Dexie.js` — IndexedDB;
- `101arrowz/fflate` — compactação/exportação quando necessária;
- `evanw/esbuild` — build/dev apenas, se escolhido.

Novas bibliotecas entram por necessidade concreta, não por conveniência genérica.

## Impacto documental

Precisam ser removidos ou reescritos em toda a documentação ativa:

- nomenclatura antiga da distribuição;
- Tauri/WebView2;
- Rust/Tokio/Axum/rusqlite;
- Launcher/Controller/Host;
- executáveis;
- SMB como mecanismo de distribuição;
- SQLite/WAL/migrations SQL;
- HTTP/JSON/WebSocket interno;
- autenticação/sessão/autorização;
- multiusuário/concorrência/fila/eventos distribuídos;
- Backup/Restore/disaster recovery;
- Typst/OOXML server-side;
- gates Win32/EDR/ACL específicos da arquitetura anterior.

O histórico Git continuará contendo o passado do projeto; a árvore documental ativa não deve tratá-lo como arquitetura vigente.

## Nova sequência de implementação — proposta inicial

```text
R1 documentação/rebaseline
→ W1 workspace web + build estático mínimo
→ W2 shell/UI base + routing local
→ W3 IndexedDB + migrations locais
→ W4 Procedimentos + categorias + responsáveis
→ W5 Reader/Atendimentos/Equipamentos
→ W6 Exportar/Importar dados
→ W7 impressão/PDF via browser
→ W8 PWA/offline + build portátil HTML
→ W9 hardening/performance + publicação cPanel
```

Nenhuma implementação deve começar antes de este rebaseline documental ser aprovado e consolidado.