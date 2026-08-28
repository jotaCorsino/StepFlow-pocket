# Bloco 10 — Etapa 10 — Nomes de arquivo + artefatos temporários — Proposta para análise

**Status:** PROPOSTA / AGUARDANDO APROVAÇÃO DO PO  
**Fase:** Fase 1 — Fechamento arquitetural e especificação  
**Abertura:** 2026-08-28  
**Base consolidada:** Bloco 10 / Etapas 1–9  
**Base Git:** `main` em `708ca5970a2f18b42c3c6a85279f4c9a15ac809a`

## 1. Objetivo

Fechar o contrato de:

- nomes sugeridos para arquivos exportados pelo usuário;
- sanitização segura desses nomes no Windows;
- diferença entre arquivo salvo pelo usuário e artefato transitório interno;
- materialização local necessária para preview/impressão;
- localização controlada dos temporários;
- isolamento entre instâncias do Client;
- limpeza normal e recuperação após encerramento anormal;
- comportamento em conflito, falha e arquivo bloqueado.

A Etapa 10 **não** altera templates, conteúdo documental, política de uma A4, formato PDF/DOCX, fluxo de impressão, modelo de dados ou histórico funcional.

## 2. Contratos herdados

Permanecem vigentes:

```text
Host
→ gera a partir de snapshot consistente
→ devolve artefato pela API autenticada

Client
→ preview / salva / imprime
```

- Host não grava em caminho arbitrário da workstation;
- artefatos de exportação não viram histórico, backup ou registro funcional por padrão;
- impressão usa o PDF oficial no Client Windows;
- WebView2 de impressão é dedicada/transitória;
- Ficha: Salvar e Imprimir reutilizam os mesmos bytes PDF associados à prévia/source_version;
- preview e impressão não autorizam regeneração silenciosa de outro documento;
- não existe área global de `Exportações` nem fila persistente de jobs na v1.

## 3. Dois tipos de arquivo

É obrigatório distinguir:

### 3.1 Arquivo escolhido pelo usuário

É o PDF/DOCX que o usuário decidiu salvar de forma persistente.

```text
artefato gerado
→ Salvar
→ diálogo nativo do Windows
→ usuário escolhe pasta/nome
→ arquivo persistente fora da gestão de temporários do StepFlow
```

Esse arquivo pertence ao usuário após o salvamento. O StepFlow não o apaga posteriormente.

### 3.2 Artefato transitório interno

Existe apenas quando uma integração local realmente precisa de um recurso de filesystem, por exemplo WebView2 de impressão.

```text
bytes canônicos
→ materialização transitória controlada
→ consumo local
→ descarte best-effort
```

Não é exportação do usuário, histórico, cache documental permanente ou backup.

## 4. Nome sugerido — Procedimento

Para PDF e DOCX de Procedimento, usar nome humano, determinístico e ligado à revisão exata.

Formato-base:

```text
{codigo} - {titulo} - v{display_version} - r{revision_no}.{ext}
```

Exemplo:

```text
PR-014 - Configuração de VLAN - v2.0 - r18.pdf
PR-014 - Configuração de VLAN - v2.0 - r18.docx
```

Se `display_version` não existir:

```text
{codigo} - {titulo} - r{revision_no}.{ext}
```

A revisão técnica permanece no nome porque duas revisões diferentes não devem receber silenciosamente a mesma sugestão de arquivo apenas por compartilharem versão editorial.

Não incluir por padrão:

- usuário que exportou;
- timestamp de geração;
- UUID interno;
- hash;
- nome da empresa;
- estado do checklist;
- dados do Atendimento.

O próprio documento continua sendo a autoridade para metadados editoriais completos.

## 5. Nome sugerido — Ficha

A Ficha usa nome curto e previsível:

```text
{service_code} - Ficha.pdf
```

Exemplo:

```text
AT-000142 - Ficha.pdf
```

Não colocar no filename por padrão:

- nome do cliente/solicitante;
- nome do equipamento;
- serial/patrimônio/MAC;
- resumo do serviço;
- técnico;
- status;
- timestamp.

Motivos:

- reduz exposição de informação operacional em Explorer, Recent Files, logs de shell e compartilhamentos;
- mantém nome legível e estável;
- o código do Atendimento já é a referência operacional adequada.

Se o usuário quiser outro nome, o diálogo nativo continua permitindo edição antes de salvar.

## 6. Sanitização de filename

Sanitização afeta **somente o nome do arquivo**, nunca conteúdo do documento.

Regras:

- remover/substituir caracteres inválidos para filename Windows;
- impedir `/`, `\\`, `:`, `*`, `?`, `"`, `<`, `>`, `|` e controles;
- remover espaços/pontos inválidos no final;
- impedir segmentos `.` e `..`;
- evitar nomes reservados do Windows (`CON`, `PRN`, `AUX`, `NUL`, `COM1` etc.);
- preservar letras Unicode/acentos quando válidos;
- nunca permitir que título/campo de domínio injete diretório ou caminho;
- extensão é definida pelo tipo de artefato e não vem de texto livre;
- limitar o segmento variável do título quando necessário para manter nome manejável no Windows.

Se o título sanitizado ficar vazio, usar somente os identificadores seguros aplicáveis, por exemplo:

```text
PR-014 - v2.0 - r18.pdf
```

O corte de um **segmento do filename** é permitido e não equivale a truncar conteúdo documental.

## 7. Conflito com arquivo existente

O StepFlow não deve sobrescrever silenciosamente arquivo persistente já existente.

Baseline:

```text
Salvar
→ diálogo nativo
→ sistema informa/conduz conflito de nome
→ usuário confirma substituição ou escolhe outro nome
```

Não criar automaticamente sequências ocultas do tipo `(1)`, `(2)` sem que o usuário veja o nome final.

Não considerar arquivo salvo até a gravação terminar com sucesso.

Em falha, reportar erro e tentar remover artefato parcial criado pelo próprio StepFlow, sem apagar arquivo preexistente do usuário.

## 8. Materialização transitória no Client

Regra principal:

> Se a próxima etapa consegue consumir bytes/memória com segurança, não criar arquivo temporário apenas por conveniência.

Materializar em filesystem somente quando necessário para a integração local.

Casos esperados:

- preview da Ficha: preferir consumo em memória/custom protocol controlado quando a implementação permitir;
- PDF para impressão Windows/WebView2: pode exigir recurso local transitório;
- DOCX salvo pelo usuário: não precisa de temporário de impressão;
- PDF salvo pelo usuário: o arquivo final escolhido não é considerado temporário.

Nenhum temporário deve ser criado no Host para simular acesso à workstation.

## 9. Diretório transitório controlado

Artefatos transitórios do Client ficam em diretório temporário **por usuário**, resolvido por API do sistema/Tauri, nunca por path hard-coded de máquina.

Estrutura lógica proposta:

```text
<temp-dir-do-usuario>/StepFlow/artifacts/<client-instance-id>/
```

`<client-instance-id>` é opaco e único por instância do Client.

Não usar para temporários:

- pasta de instalação do StepFlow;
- diretório de trabalho atual;
- compartilhamento SMB da implantação central;
- pasta de dados SQLite do Host;
- pasta de backup;
- Documents/Desktop/Downloads;
- diretório escolhido pelo usuário para exportação, salvo temporário técnico necessário à escrita atômica do próprio arquivo final.

A implementação deve usar APIs de path do sistema/Tauri e não depender literalmente da string `%TEMP%`.

## 10. Nome de artefato temporário

Temporários internos usam nome **opaco**, sem dados de negócio.

Exemplos conceituais:

```text
print-<opaque-id>.pdf
preview-<opaque-id>.svg
write-<opaque-id>.tmp
```

O identificador opaco deve evitar colisão entre operações concorrentes.

Não incluir em nome temporário:

- cliente;
- título do Procedimento;
- equipamento;
- serial/MAC;
- resumo/observações;
- nome do técnico.

O path temporário não é interface de produto e não precisa ser bonito.

## 11. Isolamento entre instâncias

Cada processo/instância do Client usa seu próprio subdiretório transitório.

Consequências:

- duas instâncias não reutilizam o mesmo arquivo temporário;
- uma operação não sobrescreve recurso de outra;
- limpeza normal da instância atual não percorre nem apaga indiscriminadamente artefatos de outra instância ativa;
- IDs de temporário não são identidade de documento e não saem para o domínio.

## 12. Lifecycle normal

Para recurso materializado exclusivamente para preview/impressão:

```text
criar
→ concluir escrita
→ disponibilizar à WebView2/consumidor
→ manter enquanto houver uso local
→ fechar consumidor
→ remover best-effort
```

Não apagar enquanto WebView2/Windows ainda puder precisar do recurso.

Ao encerrar normalmente o Client:

- fechar consumidores transitórios;
- tentar apagar arquivos do diretório da própria instância;
- tentar remover o diretório vazio da instância;
- falha de limpeza não transforma uma impressão já entregue ao Windows em falha funcional retroativa.

## 13. Arquivo bloqueado

Windows/WebView2/antivírus podem manter handle temporariamente aberto.

Nesse caso:

- limpeza falha de forma controlada;
- não forçar unlock, kill de processo, alteração de ACL ou workaround agressivo;
- registrar diagnóstico técnico mínimo;
- tentar novamente em momento seguro dentro do lifecycle do Client;
- se ainda permanecer, deixar para scavenging posterior.

O StepFlow nunca apaga outro arquivo do usuário para resolver lock.

## 14. Recuperação após crash / temporários órfãos

Como um encerramento abrupto pode impedir cleanup, o Client realiza scavenging **best-effort** de diretórios StepFlow transitórios antigos que consiga identificar como não pertencentes à instância atual/ativa.

Regras:

- atuar somente dentro do namespace transitório próprio do StepFlow;
- nunca varrer/apagar o diretório temporário geral do usuário;
- não seguir symlink/reparse point para fora da raiz gerenciada;
- não apagar diretório que possa pertencer a instância ativa;
- ignorar item bloqueado e continuar;
- falha de scavenging não impede login/uso normal do Client;
- manter logging técnico suficiente para diagnóstico, sem conteúdo documental.

O mecanismo exato para reconhecer instância ativa/stale pode ser fechado na implementação/Etapa 12, desde que preserve essas garantias.

Não criar daemon, serviço, scheduled task ou watchdog apenas para limpeza.

## 15. Escrita do arquivo persistente

Ao salvar PDF/DOCX em path escolhido pelo usuário, a implementação deve evitar declarar sucesso sobre arquivo incompleto.

Direção proposta:

```text
bytes completos disponíveis
→ preparar gravação no diretório escolhido
→ escrever completamente
→ finalizar/substituir conforme semântica segura do filesystem
→ só então confirmar sucesso
```

Quando a plataforma/filesystem permitir, preferir gravação por arquivo auxiliar no **mesmo diretório de destino** e promoção/replace ao final, para evitar final parcialmente escrito.

Esse auxiliar:

- usa nome opaco;
- existe somente durante a operação de save;
- é removido em falha best-effort;
- não vira artefato gerenciado permanente;
- não autoriza substituir arquivo preexistente sem a confirmação já dada no fluxo de save.

Detalhes específicos de atomicidade entre NTFS/SMB e APIs utilizadas ficam para validação técnica da Etapa 12.

## 16. Memória versus disco

Não transformar `cache em disco` em requisito apenas para evitar manter bytes em memória.

A política inicial é:

- Host gera e devolve bytes;
- Client mantém artefato associado à operação/source_version enquanto necessário;
- filesystem transitório é usado apenas pelas integrações que precisarem;
- limites de tamanho/memória/concurrency ficam para Etapa 12.

## 17. Logging e privacidade

Logs podem registrar:

- tipo do artefato;
- operação (`save`, `preview`, `print`);
- resultado;
- código de erro técnico;
- identificador opaco da operação quando necessário.

Evitar em log técnico por padrão:

- conteúdo do documento;
- resumo/observações;
- nome de cliente;
- serial/MAC;
- path completo escolhido pelo usuário quando não for necessário ao diagnóstico.

Se path precisar ser diagnosticado, preferir informação minimizada/redigida conforme a política de logging a ser fechada na implementação.

## 18. O que não criar

A Etapa 10 não autoriza:

- pasta global permanente `Exports` mantida pelo Host;
- histórico de exportações no SQLite;
- `export_jobs` persistente;
- cache permanente de PDFs/Fichas por Atendimento;
- limpeza via Windows Service, Task Scheduler ou daemon;
- temporários no compartilhamento central por conveniência;
- filename temporário com dados client-facing;
- regeneração silenciosa apenas para salvar/imprimir;
- retenção automática de cópia de cada arquivo salvo pelo usuário.

## 19. Falhas e semântica de sucesso

### Save

Sucesso significa que o arquivo escolhido foi gravado integralmente conforme o fluxo aceito pelo usuário.

### Preview

Sucesso significa que a prévia correspondente ao artefato/source_version foi carregada.

### Print

Permanece a decisão da Etapa 4: sucesso do StepFlow é entrega do fluxo ao diálogo/sistema de impressão Windows, não confirmação física de folha impressa.

### Cleanup

Cleanup é best-effort. Falha de limpeza é problema técnico observável, mas não altera retroativamente o resultado de save/preview/print já concluído.

## 20. Pontos para validação da Etapa 12

Validar concretamente:

- API Tauri/Windows escolhida para diretório temporário por usuário;
- comportamento real da WebView2 com PDF local e momento seguro de remoção;
- escrita/promote/replace em NTFS e SMB;
- symlink/reparse-point protections na limpeza;
- concorrência de múltiplas instâncias do Client;
- limites de memória/tamanho/concurrency;
- comportamento com antivírus/EDR bloqueando arquivo transitório;
- path longo e nomes Unicode no Windows 10/11 alvo;
- política final de retry/scavenging sem daemon.

## 21. Decisões propostas para aprovação

1. arquivos persistentes são escolhidos pelo usuário via diálogo nativo e nunca entram na rotina de cleanup;
2. Procedimento sugere `{codigo} - {titulo} - v{display_version} - r{revision_no}.{ext}`;
3. Ficha sugere `{service_code} - Ficha.pdf`, sem dados pessoais no filename;
4. filename é sanitizado para Windows sem alterar o conteúdo documental;
5. conflito de nome não causa overwrite silencioso;
6. temporário só existe quando uma integração precisa de filesystem;
7. temporários pertencem ao Client, em raiz temporária por usuário e subdiretório por instância;
8. nome temporário é opaco e não contém dados do domínio;
9. cleanup normal ocorre após o consumidor liberar o recurso e no encerramento do Client;
10. lock gera cleanup best-effort/retry seguro, nunca workaround agressivo;
11. crash pode deixar órfão, tratado por scavenging futuro restrito ao namespace StepFlow;
12. não criar serviço/daemon/scheduled task para limpeza;
13. save deve evitar sucesso sobre arquivo parcial; promoção atômica quando suportada é preferida;
14. temporários/exportações não entram em SQLite, histórico ou backup por padrão;
15. limites concretos e APIs finais ficam para validação da Etapa 12.

## 22. Fora do escopo desta proposta

- QR/barcode — Etapa 11;
- limites de recursos/APIs finais — Etapa 12;
- implementação de produção;
- sincronização do checkout local;
- Backup/Restore técnico do Bloco 11.
