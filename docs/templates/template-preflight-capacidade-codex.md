# Template — Pré-flight de Capacidade do Codex

> **ESTE BLOCO É PARA O PO/USUÁRIO. NÃO ENVIAR AO CODEX.**

## Modelo recomendado

`GPT-5.6 <Luna | Terra | Sol>`

## Nível de raciocínio

`<Baixo | Médio | Alto | XHigh/Max ou equivalente disponível>`

## Motivo

Descrever em 1–3 frases por que essa é a menor capacidade adequada para executar a tarefa com margem suficiente de segurança.

Considerar:

- complexidade;
- risco;
- tamanho do escopo;
- quantidade de domínios/arquivos;
- ambiguidade;
- impacto arquitetural;
- concorrência;
- segurança;
- migrations/dados;
- dificuldade de debugging;
- reversibilidade.

## Escalonar somente se

Registrar a condição objetiva que justificaria subir a capacidade durante uma nova tentativa.

Exemplo:

```text
PRÉ-FLIGHT PARA VOCÊ — NÃO ENVIAR AO CODEX

Modelo recomendado: GPT-5.6 Luna
Raciocínio: Médio
Motivo: tarefa de inspeção local, somente leitura, com comandos já definidos e baixa consequência. Exige alguma interpretação de evidências, mas nenhuma decisão arquitetural ou implementação.
Escalonar somente se: surgirem erros inconsistentes que exijam debugging além do inventário previsto.
```

---

Somente depois deste bloco deve ser apresentado, separadamente, o **PROMPT / ENUNCIADO PARA O CODEX**.
