# Fechamento do Bloco 2 — StepFlow Host Pocket

**Data:** 2026-08-20  
**Status:** BLOCO 2 FECHADO EM NÍVEL ARQUITETURAL

## 1. Decisão tecnológica

O StepFlow Host será implementado em **Rust**, usando:

- runtime assíncrono Tokio;
- Axum para HTTP e, quando fechado no Bloco 4, canal de eventos compatível;
- `rusqlite` com SQLite bundled/embutido no artefato final.

A PoC do Bloco 2 validou que o Host pode ser compilado como executável Windows e executado a partir de pasta própria, sem Node.js, npm, Rust, Cargo, Visual Studio ou SQLite Server no runtime.

## 2. Modelo operacional Pocket

O Host não será Windows Service, daemon, tarefa agendada nem processo residente.

A máquina central receberá uma pasta pronta do StepFlow. O ciclo será orquestrado por um Controller portátil conforme `docs/03-arquitetura/host-controller-pocket.md`.

```text
Controller inicia sob demanda
        ↓
Controller valida ambiente local
        ↓
Controller inicia Host como processo-filho
        ↓
Host abre API/eventos + SQLite local
        ↓
Clients utilizam o sistema
        ↓
Controller coordena encerramento
        ↓
Host fecha banco/conexões
        ↓
Host termina
        ↓
Controller termina
        ↓
zero processo StepFlow residual
```

Nenhuma inicialização automática no boot está prevista.

## 3. Implantação na máquina central

Modelo conceitual:

```text
StepFlow\
├── app\
│   ├── StepFlowController.exe
│   └── StepFlowHost.exe
├── config\
│   └── stepflow-host.toml
├── data\
│   └── stepflow.sqlite
├── logs\
└── backups\
```

O path real permanece pendente do ambiente corporativo.

A implantação normal deverá consistir em:

1. copiar/publicar a pasta pronta;
2. ajustar configuração mínima do ambiente;
3. iniciar o Controller quando o StepFlow for utilizado.

Não instalar:

- Node.js;
- Rust/Cargo;
- Visual Studio/MSVC;
- SQLite Server;
- runtime específico do Host;
- serviço StepFlow;
- tarefa agendada StepFlow.

## 4. Configuração

A configuração operacional será externa aos binários e mantida em arquivo local à implantação, conceitualmente `config/stepflow-host.toml`.

Ela deverá conter somente parâmetros de ambiente, por exemplo:

- endereço/interface de bind;
- porta do Host;
- referências relativas aos diretórios persistentes quando necessário;
- opções operacionais realmente necessárias.

Regras:

- nenhuma configuração corporativa real deve ser hardcoded no binário;
- caminhos relativos à raiz StepFlow são preferidos quando possível;
- credenciais/senhas não devem ser armazenadas em texto puro nesse arquivo;
- valores default seguros podem existir, mas endereço real de rede será decidido no ambiente corporativo;
- alteração de configuração não deve exigir recompilar o produto.

## 5. Paths e persistência

Separação obrigatória:

- `app/`: binários substituíveis;
- `config/`: configuração da implantação;
- `data/`: banco e arquivos persistentes principais;
- `logs/`: diagnóstico operacional;
- `backups/`: backups administrados pelo StepFlow.

O Host resolve paths a partir da raiz da própria implantação, não do diretório de trabalho corrente do processo.

A atualização de `app/` não deve apagar nem substituir `data/`, `config/`, `logs/` ou `backups`.

## 6. Instância única

Somente uma instância central do Host poderá operar sobre uma determinada base.

O Controller deverá:

1. verificar se já existe Host válido;
2. reutilizar/reconhecer a instância existente quando aplicável;
3. impedir abertura concorrente de uma segunda instância sobre os mesmos dados;
4. tratar estado ambíguo como erro seguro, não como autorização para abrir outro Host.

O mecanismo técnico exato de exclusividade será definido na implementação/fundação, mas o comportamento é requisito.

## 7. Readiness e disponibilidade

O Controller não deve declarar o StepFlow disponível imediatamente após criar o processo.

Ele deverá aguardar sinal verificável de readiness do Host, incluindo pelo menos:

- processo iniciado;
- configuração carregada;
- SQLite aberto/inicializado;
- listener de comunicação pronto.

Se o Host não ficar pronto dentro do limite definido, o Controller deverá apresentar diagnóstico e encerrar/limpar o processo iniciado quando seguro.

## 8. Shutdown

Shutdown normal deve ser gracioso.

Ordem conceitual:

1. Controller solicita encerramento;
2. Host deixa de aceitar novas operações que alterem estado quando necessário;
3. operações em andamento são finalizadas ou abortadas transacionalmente de forma segura;
4. conexões/eventos são encerrados;
5. SQLite é fechado;
6. Host termina;
7. Controller confirma a saída e termina.

Não usar `kill` forçado como mecanismo normal.

Se ainda houver Clients conectados, a política de UX/aviso será especificada no Bloco 4, mas o Host nunca deve corromper dados para encerrar rapidamente.

## 9. Falhas

### Host encerra inesperadamente

Controller detecta a saída e registra diagnóstico. Não deve entrar em loop infinito de restart automático.

### Controller encerra inesperadamente

A implementação futura deverá garantir que o Host não se transforme em daemon órfão permanente. A estratégia técnica pode usar vínculo/process supervision local ou mecanismo equivalente, desde que não instale componente residente.

### SQLite/configuração indisponível

Host falha antes de declarar readiness e registra causa operacional.

### Porta indisponível

Host não escolhe silenciosamente outra porta incompatível. Reporta erro de configuração/ocupação.

## 10. Logs e diagnóstico

Logs devem ficar em `logs/` e não depender de terminal aberto.

Cada entrada deverá permitir identificar no mínimo:

- timestamp;
- nível (`INFO`, `WARN`, `ERROR` ou equivalente);
- componente;
- evento/mensagem;
- detalhe técnico útil quando houver erro.

Não registrar:

- senha;
- hash de senha desnecessariamente;
- token de sessão completo;
- conteúdo sensível que não seja necessário ao diagnóstico.

Eventos mínimos de diagnóstico:

- início do Controller;
- início do Host;
- configuração carregada/erro;
- path de dados efetivo sem segredos;
- listener iniciado;
- readiness alcançada;
- falha de startup;
- solicitação de shutdown;
- shutdown concluído;
- erro fatal inesperado.

A política de rotação/retenção será definida sem depender de software externo.

## 11. Atualização

A atualização deve preservar o conceito copy-deploy.

Fluxo inicial:

```text
encerrar Controller/Host
        ↓
confirmar ausência de processos StepFlow
        ↓
backup quando exigido pela versão
        ↓
publicar/ativar novos binários em app/
        ↓
preservar config/data/logs/backups
        ↓
iniciar Controller
        ↓
validar readiness
```

Não atualizar binário em uso.

## 12. Rollback

A estratégia deve permitir retornar aos binários anteriores sem reinstalar o Windows nem runtimes.

Regras:

- versões de `app/` devem poder ser substituídas/controladas independentemente dos dados;
- rollback nunca apaga o banco automaticamente;
- migrations incompatíveis precisam de política específica no Bloco 6;
- se uma migration impedir rollback simples, isso deve ser conhecido antes de ativar a nova versão.

## 13. Diagnóstico para o operador

O Controller deverá oferecer diagnóstico mínimo legível para situações como:

- configuração inválida;
- Host já em execução;
- falha ao iniciar Host;
- porta ocupada;
- banco inacessível;
- falta de permissão na pasta;
- versão/configuração incompatível;
- shutdown não concluído normalmente.

O operador não deve precisar abrir IDE, Rust/Cargo ou ferramentas de desenvolvimento para entender uma falha básica.

## 14. Limitação operacional aceita

Com zero componente StepFlow residente, o primeiro Client remoto não consegue iniciar sozinho um processo na máquina central apenas executando arquivo via SMB.

Portanto, o Controller precisa ser iniciado na máquina central ou por mecanismo corporativo já existente e aprovado.

Essa é uma consequência deliberada do requisito de não instalar nem manter agente/serviço StepFlow residente.

## 15. Gate do Bloco 2

Considera-se fechado em nível arquitetural:

- tecnologia do Host: Rust + Axum/Tokio;
- SQLite bundled e local ao Host;
- artefato Pocket sem runtime global;
- Controller portátil sob demanda;
- nenhum serviço/processo permanente;
- estrutura conceitual de paths;
- configuração externa;
- readiness;
- instância única;
- shutdown gracioso;
- logs/diagnóstico;
- atualização/rollback por pasta.

Ficam para blocos posteriores:

- protocolo Client ↔ Host e eventos: Bloco 4;
- autenticação/sessões: Bloco 5;
- schema/migrations: Bloco 6;
- concorrência detalhada: Bloco 7;
- launcher/distribuição do Client: Bloco 3.

**Resultado: Bloco 2 apto a ser encerrado sem nova PoC.**
