# Launcher e Distribuição do StepFlow Client

**Data:** 2026-08-20  
**Status:** DIREÇÃO ARQUITETURAL CONSOLIDADA PARA A FASE 1

## 1. Objetivo

Preservar a experiência aprovada para o técnico:

```text
ponto de entrada interno do StepFlow
        ↓
duplo clique
        ↓
Client local atualizado
        ↓
login
```

sem exigir instalação tradicional, comandos, configuração manual complexa ou execução permanente do Client diretamente sobre SMB.

## 2. Princípio

O ponto de entrada publicado na rede deve funcionar como **launcher portátil e descartável**. Seu papel é somente preparar/iniciar a cópia local do Client e então encerrar.

O launcher não é serviço, agente residente, watchdog ou updater em background.

## 3. Local do Client na estação

A cópia operacional do Client deve ficar em pasta por usuário sob `%LOCALAPPDATA%`, conceitualmente:

```text
%LOCALAPPDATA%\StepFlow\Client\
├── versions\
│   ├── <versao-A>\
│   │   └── StepFlow.exe
│   └── <versao-B>\
│       └── StepFlow.exe
├── current\              # marcador/manifesto lógico; mecanismo exato a definir
└── launcher-state\       # somente se realmente necessário
```

Não exigir privilégios administrativos para criar ou atualizar essa cópia local.

## 4. Conteúdo publicado na rede

O compartilhamento/ponto de publicação real ainda não está definido. Usar apenas representação conceitual:

```text
\\<HOST-OU-SERVIDOR-DA-EMPRESA>\<COMPARTILHAMENTO>\<PASTA-STEPFLOW>\client\
├── StepFlowLauncher.exe
├── manifest.json
└── releases\
    └── <versao>\
        ├── StepFlow.exe
        └── <arquivos-privados-do-client-se-houver>
```

Nenhum IP, hostname ou share de exemplo deve ser embutido como fato.

## 5. Manifesto de publicação

O manifesto deve fornecer somente o necessário para distribuição segura, por exemplo:

- versão publicada;
- arquitetura (`x64` inicialmente);
- nome/caminho relativo dos artefatos;
- tamanho quando útil;
- SHA-256 de cada artefato relevante;
- versão mínima de launcher/Client quando necessário;
- referência à configuração de implantação que será fechada no Bloco 4.

Não incluir credenciais, tokens, senhas ou dados sensíveis.

## 6. Fluxo do launcher

Fluxo normal:

```text
duplo clique
   ↓
ler manifesto publicado
   ↓
comparar versão local
   ↓
versão já íntegra? ── sim ──> iniciar Client local
   │
   não
   ↓
copiar para diretório temporário/versionado
   ↓
validar SHA-256
   ↓
ativar versão local
   ↓
iniciar Client local
   ↓
launcher encerra
```

O launcher deve ser pequeno, self-contained e sem runtime global obrigatório.

## 7. Atualização sem conflito com Client em uso

Não substituir em-place o executável que está aberto.

Usar versões lado a lado. Uma nova publicação deve ser copiada para nova pasta versionada e somente novos lançamentos passam a usar essa versão.

Isso evita:

- arquivo em uso bloqueando publicação;
- corrupção por cópia parcial;
- Client executando binários misturados de duas versões;
- necessidade de encerrar à força sessões existentes apenas para publicar atualização.

Sessões já abertas podem continuar na versão anterior até serem encerradas, sujeito à política de compatibilidade Client↔Host que será definida no Bloco 4.

## 8. Ativação e rollback

Uma versão local só pode ser considerada pronta após cópia completa e validação de integridade.

Se a nova versão falhar antes da ativação:

- não destruir a versão local anterior válida;
- manter a versão anterior como fallback;
- apresentar erro operacional simples ao usuário/administrador.

O desenho deve permitir conservar pelo menos uma versão anterior válida para rollback, sem acumular versões indefinidamente.

A política exata de retenção será definida na implementação/fundação.

## 9. Concorrência do launcher na mesma estação

Duas ativações simultâneas no mesmo perfil não podem copiar/ativar a mesma versão de forma concorrente.

A implementação deverá usar mecanismo local de exclusão mútua/lock transitório durante atualização.

Esse lock:

- existe somente enquanto o launcher prepara a versão;
- não é serviço;
- não permanece consumindo recursos depois que o launcher encerra;
- não deve impedir iniciar uma versão já validada quando não há atualização em curso.

## 10. Falhas esperadas

O launcher deve distinguir pelo menos:

- ponto de publicação indisponível;
- manifesto inválido;
- versão/arquitetura incompatível;
- falha de cópia;
- hash divergente;
- falta de permissão/espaço local;
- outra atualização local em andamento;
- Client local ausente/corrompido;
- configuração de implantação inválida.

Mensagens devem orientar o usuário sem expor detalhes técnicos desnecessários.

## 11. Comportamento quando a rede está indisponível

Não consolidar ainda operação offline completa do Client em relação ao Host.

Para o launcher, a direção é:

- se a publicação estiver indisponível e existir uma versão local previamente validada, o launcher **pode** iniciar essa versão somente se a política de compatibilidade/descoberta do Bloco 4 permitir;
- se não existir versão local válida, informar indisponibilidade;
- não inventar versão nem usar artefato parcialmente copiado.

A decisão final depende da forma como o Client localizará e validará o Host.

## 12. Relação com o Host Pocket

O launcher do técnico **não inicia remotamente o Host na máquina central**.

O Host segue o modelo sob demanda definido no Bloco 2 e precisa estar ativo na máquina central para os Clients se conectarem.

Executar `StepFlowLauncher.exe` a partir de um compartilhamento no PC do técnico significa executar o launcher na estação do técnico, não na máquina central.

## 13. Tecnologia recomendada do launcher

Direção recomendada: **executável Rust x64 pequeno e self-contained**, separado do Client Tauri.

Motivos:

- reutiliza o toolchain já aprovado do projeto;
- não exige Node/npm/Rust no computador do técnico em runtime;
- reduz superfície e dependências;
- executa apenas tarefas mecânicas de manifesto, hash, cópia, lock e `CreateProcess`/equivalente;
- encerra imediatamente após iniciar o Client.

Essa escolha não autoriza implementar o launcher definitivo ainda; a Fase 1 apenas fecha a arquitetura.

## 14. O que não será feito

- instalador MSI/NSIS obrigatório para o técnico;
- serviço Windows para o launcher;
- updater residente;
- registro/PATH global;
- execução pesada permanente a partir do compartilhamento;
- hardcode de IP/hostname/share ainda não confirmados;
- abertura de SQLite pelo launcher ou Client;
- tentativa de usar o launcher para executar processo remotamente no servidor.

## 15. Validação pendente ao ambiente corporativo

A arquitetura pode ser fechada agora, mas os seguintes itens só podem ser validados definitivamente na empresa:

- caminho SMB real;
- permissões de leitura no ponto de publicação;
- políticas corporativas para execução do launcher a partir da rede;
- desempenho de cópia;
- comportamento em múltiplas estações reais;
- WebView2 nas estações representativas;
- eventual interferência de antivírus/EDR/políticas locais.

Esses testes são `NÃO APLICÁVEIS NESTE AMBIENTE` enquanto o desenvolvimento estiver fora da LAN corporativa.

## 16. Gate do Bloco 3

O Bloco 3 pode ser considerado arquiteturalmente fechado quando estas regras estiverem registradas:

1. ponto de entrada de duplo clique preservado;
2. launcher portátil e transitório;
3. Client executado localmente em `%LOCALAPPDATA%`;
4. versões lado a lado;
5. integridade por SHA-256;
6. ativação somente após cópia completa;
7. rollback preservando versão anterior;
8. nenhum processo/serviço residente do launcher;
9. endereço real do compartilhamento mantido como configuração futura;
10. validação SMB real explicitamente deferida ao ambiente corporativo.
