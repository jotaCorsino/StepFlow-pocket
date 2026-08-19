# Arquitetura Inicial — StepFlow Pocket

**Status:** DIREÇÃO ARQUITETURAL INICIAL. Alguns detalhes ainda exigem validação técnica antes da implementação.

## 1. Objetivo

Definir a separação mínima necessária para manter o StepFlow simples para o usuário, seguro para uso simultâneo em rede e fácil de manter.

## 2. Visão geral

```text
                Compartilhamento de rede
        \\192.168.5.7\Arquivos\StepFlow\
                         │
                  ponto de entrada
                         │
          ┌──────────────┼──────────────┐
          │              │              │
       PC Técnico     PC Gerência     PC ADM
          │              │              │
          └──────────────┼──────────────┘
                         │
                  StepFlow Client
                         │
                 HTTP + canal de eventos
                         │
                  StepFlow Host
                         │
          ┌──────────────┼──────────────┐
          │              │              │
       SQLite         arquivos       backups
```

## 3. Componentes

### 3.1. StepFlow Client

Responsabilidades:

- renderizar a interface;
- manter sessão local do usuário;
- solicitar dados ao Host;
- enviar comandos de criação/edição/remoção permitidos;
- apresentar conflitos e erros;
- receber eventos de atualização;
- executar interações locais de UX, como copiar texto e manipular checklist temporário;
- solicitar exportações ou gerar documentos conforme a estratégia final.

Não é responsabilidade do Client:

- abrir diretamente o SQLite do compartilhamento;
- decidir autorização apenas pela UI;
- serializar alterações globais;
- manter a fonte de verdade persistente do sistema.

### 3.2. StepFlow Host

Responsabilidades:

- autenticação;
- autorização;
- usuários e perfis;
- API/contratos do sistema;
- acesso exclusivo/coordenado ao SQLite;
- transações;
- controle de revisão;
- detecção de conflito;
- fila/serialização de comandos de escrita quando necessária;
- auditoria;
- emissão de eventos para clientes;
- operações de backup/restauração;
- acesso aos arquivos persistentes administrados pelo sistema.

### 3.3. StepFlow Data

Persistência local à máquina do Host.

Estrutura conceitual:

```text
StepFlowData/
├── stepflow.sqlite
├── company/
│   └── logo.png
├── avatars/
├── exports/
├── backups/
└── attachments/        # futuro, se necessário
```

A localização física final será definida antes da implementação.

## 4. Por que não compartilhar o SQLite diretamente

O requisito de múltiplas estações exige um ponto coordenador.

A arquitetura não permitirá que cada Client trate `\\servidor\...\stepflow.sqlite` como banco local próprio.

Toda persistência oficial passa pelo Host. Isso também centraliza autorização, controle de revisão, auditoria e eventos.

## 5. Modelo de concorrência

### 5.1. Escritas

A escrita deve ocorrer em transação no Host.

Operações que alterem estado persistente devem carregar contexto suficiente para validar a revisão/base esperada.

### 5.2. Revisão otimista

Exemplo conceitual:

```text
Cliente A lê documento na revision 12
Cliente B lê documento na revision 12
Cliente A salva -> revision 13
Cliente B tenta salvar usando base_revision 12
Host detecta que a atual é 13
Host rejeita sobrescrita silenciosa
Client B recebe conflito e precisa recarregar/revisar
```

### 5.3. Fila

A fila não substitui o controle de revisão.

Ela existe para ordenar comandos de escrita quando operações simultâneas precisam de serialização previsível. Mesmo em fila, uma operação baseada em estado antigo pode ser inválida e deve ser recusada.

### 5.4. Soft lock/presença

Pode existir indicador de que outro usuário está editando determinada documentação.

Esse indicador é informativo. A integridade continua dependendo de revisão e validação no Host.

## 6. Atualização em tempo real

O Host deve possuir um canal para informar clientes conectados sobre eventos relevantes, por exemplo:

- `process.created`;
- `process.updated`;
- `process.deleted`;
- `process.version.created`;
- `user.updated`;
- `company.settings.updated`.

O protocolo final pode ser WebSocket ou alternativa equivalente e será validado tecnicamente antes da implementação.

Eventos não devem carregar dados sensíveis desnecessários. O cliente pode receber o evento e buscar novamente o recurso autorizado.

## 7. Autenticação e sessão

Direção inicial:

1. usuário informa login e senha;
2. Client envia credenciais ao Host pela rede interna;
3. Host valida hash da senha;
4. Host cria sessão/token;
5. Client usa essa sessão nas chamadas seguintes;
6. Host verifica autorização em cada operação protegida.

A estratégia exata de sessão e hash ainda será fechada antes do código.

## 8. Autorização

Perfis ajudam a configurar permissões, mas autorização deve ser verificada por operação.

Exemplos de capacidades conceituais:

```text
process.read
process.create
process.update
process.delete
process.export
users.read
users.create
users.update
users.delete
company.read
company.update
backup.create
backup.restore
```

A matriz final será documentada separadamente.

## 9. Organização do Client

Direção conceitual:

```text
src/
├── app.js
├── core/
│   ├── App.js
│   ├── Router.js
│   ├── ApiClient.js
│   ├── Session.js
│   ├── EventBus.js
│   └── Permissions.js
├── components/
├── modules/
│   ├── auth/
│   ├── processes/
│   ├── users/
│   ├── company/
│   └── export/
├── styles/
└── assets/
```

Essa árvore é direção, não autorização para criar arquivos vazios antes da definição oficial da estrutura do repositório.

## 10. Organização por domínio

Evitar duas formas de monólito:

- um único arquivo gigante;
- uma classe `God Object` que concentra todo o sistema mesmo estando em arquivo separado.

Módulos devem possuir responsabilidade clara e API interna pequena.

## 11. Launcher e distribuição

Experiência alvo:

```text
\\192.168.5.7\Arquivos\StepFlow\StepFlow.exe
```

Direção proposta:

- o arquivo de rede funciona como ponto de entrada/launcher;
- verifica a versão publicada;
- mantém uma cópia local do Client quando necessário;
- inicia a cópia local;
- usuário continua percebendo apenas o duplo clique no arquivo de rede.

Motivações:

- evitar execução pesada contínua sobre SMB;
- facilitar atualização central;
- reduzir arquivos bloqueados no compartilhamento;
- permitir rollback/versionamento futuro.

Essa estratégia exige protótipo técnico antes de ser consolidada definitivamente.

## 12. Localização do Host

O Client precisa localizar o Host sem configuração manual do técnico.

Alternativas a avaliar na fase arquitetural:

- endereço fixo configurado no build/arquivo de configuração publicado;
- arquivo de descoberta no compartilhamento;
- hostname interno estável;
- descoberta local controlada.

A opção escolhida deve favorecer simplicidade e manutenção.

## 13. Falhas esperadas

O Client deve distinguir ao menos:

- Host indisponível;
- sessão expirada;
- usuário sem permissão;
- conflito de revisão;
- falha de validação;
- erro de persistência;
- arquivo/logo inválido;
- exportação com falha;
- versão do Client incompatível com Host.

Mensagens devem ser operacionais e não expor detalhes internos desnecessários.

## 14. Persistência e transações

O SQLite será operado somente pelo Host.

Regras iniciais:

- migrations versionadas;
- transações em operações compostas;
- foreign keys habilitadas;
- índices definidos a partir de consultas reais;
- timestamps e identificadores consistentes;
- exclusão física vs. arquivamento definida por entidade;
- nenhum dado real de operação no repositório.

## 15. Exportação

A exportação deve ter uma camada própria e não depender da estrutura visual da tela de leitura.

O modelo exportável recebe dados normalizados do processo e monta PDF/DOCX com identidade da empresa.

## 16. Backup

Backup deve ser coordenado pelo Host para impedir cópia inconsistente durante escrita.

Estratégia exata será definida depois de validar a forma de backup do SQLite e dos arquivos associados.

## 17. Decisões ainda não fechadas

- Tauri e versão final;
- formato final do Host;
- protocolo Client/Host;
- formato de sessão;
- mecanismo de eventos;
- descoberta do Host;
- launcher/update;
- paths finais de dados;
- estratégia de migrations;
- empacotamento e inicialização automática do Host;
- bibliotecas de exportação.

Esses pontos devem virar decisões consolidadas antes do bloco de código correspondente.
