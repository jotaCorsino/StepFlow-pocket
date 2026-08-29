# Bloco 11 — Backup / Restauração técnico

**Status:** PROPOSTA / EM ANÁLISE — NÃO CONSOLIDADO COMO BLOCO  
**Fase:** 1 — Fechamento arquitetural e especificação  
**Abertura:** 2026-08-29  
**Última análise:** 2026-08-29

## Objetivo

Fechar o contrato técnico de Backup/Restore antes da estrutura oficial e da Fase 2, sem criar implementação funcional nesta etapa.

## Contratos já consolidados

A UX permanece definida em `../02-telas/13-backup-restauracao.md`.

Este bloco não reabre, sem bloqueador técnico concreto:

- Backup/Restore coordenado pelo Host;
- Client sem acesso direto ao SQLite;
- autorização Host-side;
- Restore normal somente com backup elegível e confirmação reforçada;
- safety backup obrigatório antes da etapa destrutiva do Restore normal;
- falha do safety backup bloqueia o Restore normal;
- recuperação sem Host funcional pertence a disaster recovery local/controlado;
- sucesso somente após confirmação do Host;
- Backup separado de exportação documental;
- ausência de scheduler periódico por inferência;
- Restore de Gerência não autorizado; Gerência × Backup permanece pendente;
- contrato Pocket como gate superior.

## Tópicos que o Bloco 11 deve fechar

1. conjunto exato de dados e arquivos que formam o estado recuperável;
2. snapshot consistente de SQLite + arquivos administrados;
3. formato e identidade do backup;
4. manifesto, verificação e compatibilidade entre versões/schema;
5. escrita completa, promoção e tratamento de backup parcial;
6. catálogo e retenção inicial;
7. coordenação com mutações e operações administrativas;
8. Restore normal e safety backup;
9. restart, reconexão e sessões após Restore;
10. falhas parciais e resultado incerto;
11. disaster recovery local quando o Host não inicia;
12. capacidades e auditoria;
13. validação técnica final.

## Ordem de análise

1. estado recuperável + envelope do backup;
2. consistência + escrita/promoção/verificação;
3. catálogo + retenção + coordenação;
4. Restore + safety backup + compatibilidade;
5. restart/sessões/reconexão + falhas;
6. disaster recovery + capacidades/auditoria;
7. validação técnica final.

A ordem organiza o trabalho; não aprova antecipadamente alternativa técnica ainda não revisada.

---

# Análise 1 — estado recuperável + envelope do backup

**Status:** APROVADA PELO PO em 2026-08-29.

## 1.1 Fronteira do estado recuperável

A arquitetura separa:

```text
StepFlow\
├── app\
├── config\
├── data\
│   ├── stepflow.sqlite
│   ├── company\
│   └── avatars\
├── logs\
└── backups\
```

O Backup normal do StepFlow é **backup de estado da aplicação**, não imagem completa da implantação/servidor.

Entram no estado recuperável inicial:

```text
data\stepflow.sqlite
data\company\**
data\avatars\**
```

Regras consolidadas:

- `stepflow.sqlite` contém o estado relacional oficial;
- `company/` contém arquivos administrados da identidade da empresa que não estejam no SQLite;
- `avatars/` contém arquivos administrados de usuários;
- novo tipo de arquivo persistente só entra no backup após inclusão explícita no contrato/allowlist do Host;
- não copiar recursivamente qualquer conteúdo apenas por estar abaixo da pasta StepFlow.

Ficam fora do Backup normal inicial:

- `app/` — binários substituíveis;
- `config/` — configuração operacional/de implantação;
- `logs/` — diagnóstico, não estado funcional a restaurar;
- `backups/` — nunca incluir backups dentro de backup;
- exportações PDF/DOCX/Ficha salvas pelo usuário;
- temporários do Host/Client;
- cópias locais do Client em `%LOCALAPPDATA%`.

`config/` fica fora porque um Restore de dados não deve reverter silenciosamente endereço, porta, paths ou outras escolhas específicas da implantação e tornar o Host inacessível. Configuração funcional que precise acompanhar o estado restaurado deve residir no banco/arquivos administrados ou ser adicionada explicitamente ao contrato futuro.

Consequência: o Backup permite reconstruir o **estado StepFlow** dentro de uma implantação compatível; não é backup bare-metal do Windows nem da pasta de deployment inteira.

## 1.2 Envelope físico

Um backup confirmado é **um único arquivo imutável por snapshot**, administrado pelo Host em `backups/`.

Formato lógico:

```text
backup-<utc>-<backup_id>.stepflow-backup

manifest.json
payload/
├── stepflow.sqlite
├── company/
└── avatars/
```

Container consolidado:

- ZIP padrão sob extensão própria `.stepflow-backup`;
- método `Stored` sem compressão no baseline;
- compressão futura exige nova versão/capacidade explícita do formato;
- pacote é lido/escrito pelo StepFlow; nenhuma ferramenta externa é requisito operacional;
- extensão própria identifica semântica StepFlow e não autoriza edição manual do conteúdo.

Motivos para arquivo único:

- unidade operacional simples para listar, mover e validar;
- evita backup confirmado espalhado por vários arquivos finais;
- facilita safety backup e disaster recovery;
- permite promoção do pacote como uma unidade de filesystem;
- mantém o conteúdo lógico explícito por manifesto.

## 1.3 Manifesto mínimo

`manifest.json` registra pelo menos:

- `format_version`;
- `backup_id` opaco;
- `created_at` em UTC;
- `origin` (`manual` ou `system`);
- ator responsável quando aplicável;
- motivo técnico quando `origin = system`, quando aplicável;
- versão StepFlow de origem;
- versão/schema ou migration de origem necessária à compatibilidade;
- lista ordenada de entradas do payload;
- path lógico normalizado de cada entrada;
- tamanho em bytes;
- SHA-256 de cada entrada.

SHA-256 é verificação de integridade/corrupção, não assinatura/autenticidade do pacote.

O manifesto não carrega senha, token reutilizável ou conteúdo de negócio desnecessário.

## 1.4 Paths e segurança do envelope

- somente paths relativos e normalizados;
- rejeitar entrada absoluta ou que escape do namespace lógico;
- não seguir reparse points/symlinks durante coleta ou restauração;
- Restore mapeia nomes lógicos conhecidos para destinos controlados pelo Host; nunca confia em path arbitrário vindo do pacote;
- entrada desconhecida em versão de formato não suportada torna o pacote incompatível, não é copiada por conveniência.

## 1.5 Lifecycle físico

Backup em construção não aparece como backup válido.

```text
staging interno
→ materializar snapshot
→ construir pacote
→ finalizar escrita
→ verificar manifesto + payload
→ promover para filename final
→ somente então confirmar/listar como válido
```

- staging não usa filename final;
- falha antes da promoção deixa somente resíduo não confirmado para cleanup conservador;
- colisão nunca sobrescreve backup confirmado existente;
- promoção final ocorre no mesmo filesystem sempre que possível.

## 1.6 SQLite

A **SQLite Online Backup API**, exposta pelo `rusqlite` pela feature `backup`, é a direção escolhida para criar o snapshot do banco.

Razões:

- API oficial para copiar banco ativo de forma consistente;
- evita cópia crua de `stepflow.sqlite` enquanto WAL/transações estão ativos;
- integra-se ao Host Rust já escolhido;
- não exige ferramenta externa;
- não exige compactar/reorganizar a base a cada backup.

`VACUUM INTO` permanece referência técnica válida, mas não é o mecanismo baseline do Backup StepFlow.

## 1.7 Decisões aprovadas — D11.1 a D11.10

- **D11.1:** Backup normal protege estado da aplicação, não a implantação inteira;
- **D11.2:** payload inicial = `stepflow.sqlite` + `company/**` + `avatars/**`;
- **D11.3:** `app/`, `config/`, `logs/`, `backups/`, exportações, temporários e Client local ficam fora;
- **D11.4:** inclusão futura de novo arquivo persistente exige allowlist/contrato explícito;
- **D11.5:** backup confirmado é um único pacote imutável `.stepflow-backup`;
- **D11.6:** container = ZIP padrão, método `Stored` no baseline;
- **D11.7:** pacote contém `manifest.json` + `payload/` em paths lógicos controlados;
- **D11.8:** manifesto versionado registra origem, compatibilidade, tamanho e SHA-256 por entrada;
- **D11.9:** pacote parcial nunca é listado como válido; staging precede promoção;
- **D11.10:** SQLite Online Backup API é o mecanismo baseline para snapshot do banco.

---

# Análise 2 — consistência + escrita/promoção/verificação

**Status:** PROPOSTA PARA REVISÃO DO PO — NÃO CONSOLIDADA.

## 2.1 Objetivo de consistência

O backup precisa representar **um único ponto lógico do estado StepFlow**.

Não basta o SQLite ser consistente isoladamente. Uma linha do banco pode referenciar logo/avatar/arquivo administrado, portanto banco e arquivos precisam pertencer à mesma janela lógica.

A proposta evita depender de comportamento sutil de snapshot enquanto mutações seguem ocorrendo: o Host cria um **barrier de captura** sobre mutações, materializa banco + arquivos em staging e libera as mutações antes das etapas pesadas de hash/ZIP/verificação.

## 2.2 Barrier de captura no Host

Fluxo proposto:

```text
Backup aceito
→ Host entra em BACKUP_CAPTURE
→ parar aceitação de novas mutações normais
→ drenar mutações já aceitas até commit/promoção de arquivo concluídos
→ atingir ponto quiescente
→ capturar SQLite + arquivos administrados em staging
→ liberar mutações
→ empacotar/verificar/promover fora do barrier
```

Regras:

- o barrier pertence ao Host, não ao Client;
- atinge **mutações de estado**, incluindo alterações de arquivos administrados;
- consultas read-only podem continuar quando seguras;
- operação administrativa incompatível não entra em paralelo;
- requests mutantes que chegarem depois da entrada no barrier não ficam acumulados indefinidamente: recebem resultado semântico temporário/retryable de operação em andamento;
- mutação já aceita antes do barrier termina de forma determinística antes da captura ou impede a captura;
- nenhuma mutação fica em estado “talvez entrou no backup”.

A UX já admite pequena janela de mutações temporariamente indisponíveis. Nenhum tempo máximo é fixado sem benchmark.

## 2.3 Ponto quiescente

Antes de copiar qualquer elemento do snapshot, o Host confirma:

- writer lógico sem mutação aceita ainda não concluída;
- transação mutante ativa inexistente;
- promoção/substituição de arquivo administrado inexistente;
- nenhuma operação de Backup/Restore/migration incompatível em paralelo;
- estado confirmado pós-commit é o estado que será capturado.

Se o ponto quiescente não puder ser alcançado com segurança, o backup falha sem produzir pacote confirmado.

## 2.4 Staging da captura

Usar namespace privado administrado pelo Host, preferencialmente dentro de `backups/` e no mesmo volume do destino final:

```text
backups/
├── .staging/
│   └── <backup_id>/
│       ├── snapshot.sqlite
│       ├── company/
│       └── avatars/
└── <backups confirmados>
```

- nome de staging não é backup válido;
- diretório é opaco para a UX;
- não reutilizar staging de operação anterior;
- não seguir reparse points;
- criação/uso deve evitar overwrite de outro `backup_id`.

## 2.5 Captura do SQLite / WAL

Durante o barrier:

1. abrir destino SQLite novo em staging;
2. executar Online Backup API do banco oficial para esse destino;
3. concluir `sqlite3_backup_finish`/equivalente;
4. fechar conexão de destino corretamente;
5. garantir que o payload final usa banco SQLite autocontido.

Regras:

- **não copiar** `stepflow.sqlite` cru;
- **não incluir** `stepflow.sqlite-wal` nem `stepflow.sqlite-shm` no pacote;
- WAL/SHM pertencem ao mecanismo runtime, não ao estado lógico do backup;
- destino de staging precisa poder ser aberto sozinho como banco SQLite antes do empacotamento;
- `SQLITE_BUSY`/`SQLITE_LOCKED` podem receber tratamento bounded/retryable conforme API; erro fatal aborta a captura;
- nenhum retry cego infinito.

Como as mutações StepFlow estão barradas durante a captura, a cópia do banco não precisa perseguir mudanças concorrentes do próprio produto. Reads podem continuar conforme isolamento/WAL.

## 2.6 Captura de `company/` e `avatars/`

Ainda dentro do mesmo barrier:

- enumerar somente roots allowlisted;
- copiar arquivos regulares para staging;
- preservar conteúdo, não depender de timestamp como prova de identidade;
- recusar/rejeitar reparse point, symlink/junction ou path que escape do root esperado;
- erro de leitura em qualquer arquivo necessário invalida a captura completa;
- não aceitar edição externa/manual desses arquivos como mecanismo suportado durante operação normal.

A ordem SQLite ↔ arquivos não altera a consistência enquanto o barrier estiver ativo e o ponto quiescente tiver sido alcançado.

## 2.7 Liberação do barrier

O barrier pode ser liberado assim que existirem em staging:

- `snapshot.sqlite` autocontido e fechado;
- cópias completas de `company/**` e `avatars/**` pertencentes ao mesmo ponto quiescente.

Depois disso, mutações comuns podem voltar a ser aceitas. As etapas seguintes trabalham somente sobre staging imutável:

```text
staging bruto concluído
→ liberar mutações
→ montar manifest
→ calcular hashes
→ construir .stepflow-backup
→ verificar
→ promover
```

Portanto o tempo de ZIP, SHA-256 e checks de integridade **não amplia** a janela de indisponibilidade de mutações.

## 2.8 Verificação antes da promoção

O pacote candidato precisa ser reaberto e validado pelo próprio código de leitura antes de virar confirmado.

Verificações propostas:

### Envelope

- ZIP abre e termina corretamente;
- exatamente um `manifest.json` válido;
- `format_version` suportado;
- `backup_id` consistente com a operação;
- paths normalizados/allowlisted;
- nenhuma entrada absoluta, `..`, duplicada ou desconhecida para a versão;
- tamanhos coerentes;
- SHA-256 de **todas** as entradas do payload confere.

### SQLite

Sobre a cópia de staging/payload:

- abrir como banco isolado;
- `PRAGMA quick_check` deve retornar `ok`;
- `PRAGMA foreign_key_check` não deve retornar violações;
- schema/migration registrada no manifesto deve corresponder ao banco capturado.

`quick_check` é proposto para criação do backup por ser uma verificação estrutural O(N) mais barata que `integrity_check`. A política de `integrity_check` completo antes de Restore será fechada na análise específica de Restore/compatibilidade.

Falha em qualquer verificação impede promoção/confirmação.

## 2.9 Construção e flush do pacote

O `.stepflow-backup` candidato é criado em staging, nunca diretamente no filename final.

Antes da promoção:

- finalizar o ZIP;
- propagar qualquer erro de fechamento/finalização;
- executar flush/sincronização do arquivo equivalente a `File::sync_all()`;
- fechar o handle antes da promoção;
- reabrir/validar o candidato conforme seção anterior.

`sync_all()` é requisito de tentativa explícita de levar conteúdo/metadata do arquivo ao filesystem antes de reportar sucesso; não é justificativa para prometer resistência absoluta a qualquer falha de hardware/controladora.

## 2.10 Promoção final sem overwrite

Proposta:

```text
candidato íntegro em staging
→ filename final único em backups/
→ move/rename no mesmo volume
→ sem replace de destino existente
→ reabrir destino final
→ confirmar estrutura/identidade
→ somente então BACKUP_CONFIRMED
```

Regras:

- staging e destino final devem ficar no mesmo filesystem/volume no baseline;
- colisão no filename final é erro, nunca overwrite;
- em Windows, o adapter de promoção deve usar semântica **no-replace**; não depender de `std::fs::rename` se isso puder substituir destino existente;
- promoção cross-volume não é baseline;
- após promoção, o Host reabre o arquivo final e confirma que o pacote reconhecido é o mesmo candidato verificado;
- sucesso da UI somente após essa confirmação.

No Windows, a implementação futura pode usar `MoveFileExW` sem `MOVEFILE_REPLACE_EXISTING` ou primitive equivalente que preserve a política no-replace. O detalhe fica encapsulado em adapter de filesystem.

## 2.11 Crash/falhas durante criação

Classificação proposta:

### Antes do ponto quiescente

- nenhum snapshot iniciado;
- liberar estado administrativo;
- nenhum backup novo.

### Durante captura bruta

- liberar barrier;
- staging incompleto permanece não confirmado;
- cleanup best-effort/conservador;
- nenhum filename final.

### Depois da liberação do barrier, antes da promoção

- estado ativo continua normal;
- staging/candidato pode ser descartado posteriormente;
- nenhum backup válido é anunciado.

### Durante/depois da promoção

- filename final por si só **não prova validade**;
- na próxima leitura/startup, pacote final é verificado antes de ganhar estado íntegro/elegível;
- pacote final inválido não é apagado silenciosamente; fica classificável como inválido/corrompido para diagnóstico conforme catálogo futuro.

Nenhuma falha de criação do backup altera o estado funcional oficial do StepFlow.

## 2.12 Resultado incerto

O Host não faz retry cego de criação quando houve erro de I/O depois de operações potencialmente concluídas.

Após falha:

- reconsulta filesystem/staging/final pelo `backup_id`;
- classifica se não houve promoção, se existe candidato final verificável ou se o resultado ficou inválido/incerto;
- somente pacote final verificado pode ser promovido semanticamente a sucesso;
- auditoria registra resultado real observado, não intenção.

## 2.13 Startup e resíduos

Na inicialização:

- `.staging` nunca é fonte de backup válido;
- resíduos de staging podem receber scavenging conservador conforme política a fechar com catálogo/retenção;
- arquivos finais são candidatos a catálogo, não automaticamente íntegros;
- pacote corrompido/incompatível é preservado/identificado conforme regra do catálogo, não usado silenciosamente;
- nenhum cleanup atravessa reparse point ou sai do root de backup administrado.

## 2.14 Propostas resultantes da Análise 2

Para revisão do PO:

- **P11.11:** consistência de backup é definida no nível `SQLite + arquivos administrados`, não apenas no banco;
- **P11.12:** Host usa barrier de captura para mutações; leituras seguras podem continuar;
- **P11.13:** mutações já aceitas drenam antes da captura; novas mutações durante o barrier recebem estado temporário/retryable em vez de acumular indefinidamente;
- **P11.14:** ponto quiescente inclui writer, transações e promoções de arquivos administrados;
- **P11.15:** captura bruta ocorre em staging privado no mesmo volume de `backups/`;
- **P11.16:** SQLite é copiado pela Online Backup API para banco novo; `-wal`/`-shm` não entram no payload;
- **P11.17:** `company/**` e `avatars/**` são copiados sob o mesmo barrier e pela allowlist;
- **P11.18:** barrier termina após snapshot bruto completo; hashes/ZIP/verificação/promoção ficam fora;
- **P11.19:** candidato exige validação integral do envelope + SHA-256 por entrada;
- **P11.20:** criação exige `PRAGMA quick_check = ok` + `foreign_key_check` vazio; `integrity_check` completo fica para análise de Restore/compatibilidade;
- **P11.21:** pacote candidato recebe flush explícito (`sync_all`/equivalente) antes da promoção;
- **P11.22:** promoção final é same-volume, no-replace e nunca sobrescreve backup existente;
- **P11.23:** sucesso só ocorre após arquivo final reaberto/confirmado; filename sozinho não comprova validade;
- **P11.24:** crash/falha deixa staging ou pacote inválido não confirmado; nunca transforma parcial em válido;
- **P11.25:** nenhum timeout, tamanho máximo ou duração de barrier é congelado sem benchmark da fase executável.

## Referências técnicas da Análise 2

- SQLite Online Backup API: `https://www.sqlite.org/backup.html`
- SQLite backup C API: `https://www.sqlite.org/c3ref/backup_finish.html`
- SQLite isolation/WAL snapshot: `https://www.sqlite.org/isolation.html`
- SQLite `quick_check` / `integrity_check`: `https://www.sqlite.org/pragma.html#pragma_quick_check`
- Rust `File::sync_all`: `https://doc.rust-lang.org/std/fs/struct.File.html#method.sync_all`
- Windows `MoveFileExW`: `https://learn.microsoft.com/windows/win32/api/winbase/nf-winbase-movefileexw`

---

## Critérios de fechamento do Bloco 11

O bloco só pode ser considerado concluído quando as decisões permitirem implementação futura sem escolhas críticas deixadas ao executor e quando:

- UX da Tela 13 continuar coerente;
- modelo de dados/migrations souber quais impactos precisará absorver na fase executável;
- contrato Pocket permanecer intacto;
- nenhum backup parcial puder ser tratado como válido;
- Restore tiver estados de falha e recuperação definidos;
- disaster recovery possuir fronteira clara em relação ao Restore normal;
- decisões aprovadas forem sincronizadas nas fontes específicas.

## Fora do escopo do Bloco 11

- implementação funcional;
- migrations oficiais;
- scheduler periódico;
- serviço persistente de backup;
- backup em nuvem;
- integração com destino externo específico;
- nova UX sem bloqueador técnico;
- números finais de performance sem evidência.

## Próxima análise

Após revisão das propostas P11.11–P11.25, avançar para **catálogo + retenção + coordenação de operações administrativas**, preservando a separação entre pacote físico, estado de verificação e política de retenção.
