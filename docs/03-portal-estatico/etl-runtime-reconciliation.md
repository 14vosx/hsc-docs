# ETL Runtime Reconciliation

## Navegação rápida

- [Home da documentação](../README.md)
- [Portal Estático](./README.md)
- [Master Index](../00-governance/99-master-index.md)

---
## Objetivo

Documentar a reconciliação do runtime ETL real da Static API v2 com o repositório versionado `hsc-cs2-etl`, registrando a nova leitura canônica da relação entre código-fonte, runtime materializado, evidência bruta importada e validação operacional do host.

Este documento existe para registrar, de forma estável e auditável:

- como o runtime ETL real foi reconciliado com o Git
- qual passou a ser a fonte versionada oficial dos scripts ETL
- como a evidência bruta da baseline importada foi preservada
- como a execução local mínima foi validada sem Docker e sem AMP em execução
- quais paths-base foram parametrizados sem quebrar o host atual
- como o contrato provisório de materialização foi fixado
- qual tag operacional passou a marcar o novo baseline reconciliado

---

## Escopo

Este documento cobre:

- reconciliação entre runtime da VPS e repositório ETL
- baseline importada de `/usr/local/bin` e `systemd`
- papel de `bin/`, `deploy/systemd/`, `_reconcile/` e `scripts/`
- validação local mínima da pipeline ETL
- parametrização dos caminhos-base do runtime ETL
- contrato provisório de materialização para `/usr/local/bin`
- validação final do runtime materializado via `systemd`

Este documento não cobre em profundidade:

- contratos JSON campo a campo
- SQL detalhado da v2
- operação do AMP e da instância `MixHAXIXE01`
- topologia completa do Nginx
- Auth API no Lightsail
- backlog funcional futuro do portal

Esses tópicos vivem em documentos próprios dos contextos `01-infra-hostinger`, `02-game-panel`, `03-portal-estatico` e `04-infra-aws-lightsail`.

---

## Estado atual

O estado operacional conhecido e reconciliado do runtime ETL da v2 é:

- o runtime ativo da VPS foi reconciliado com o repositório `14vosx/hsc-cs2-etl`
- `bin/` passou a ser a fonte versionada oficial dos scripts ETL
- `deploy/systemd/` passou a refletir o contrato host-facing atual das units ETL
- `_reconcile/2026-04-01/vps-runtime/raw/` passou a preservar a evidência bruta importada da baseline da VPS
- o runtime live do host continua sendo materializado em `/usr/local/bin/`
- o `systemd` continua apontando para `/usr/local/bin/*`
- a camada de materialização foi validada por staging remoto, comparação de hashes, backup, publicação live e smoke via `systemd`
- o baseline operacional reconciliado foi marcado no repositório ETL pela tag `etl-v0.3.0`

Leitura canônica:

- a fonte do ETL agora vive no repositório versionado
- `/usr/local/bin` deixa de ser superfície de autoria manual e passa a ser runtime materializado
- o contrato do host permanece estável enquanto a materialização substitui o antigo processo manual

---

## Source of truth / evidências

As principais evidências desta reconciliação são:

- runtime real da Hostinger
- árvore real de `/usr/local/bin/`
- unit files reais em `/etc/systemd/system/`
- base operacional em `/opt/cs2-portal/`
- comparação de hashes entre runtime live e staging remoto
- backup operacional do runtime anterior em `/tmp/hsc-etl-live-backup-*`
- repositório `14vosx/hsc-cs2-etl`
- documentação auditável registrada no próprio repositório ETL
- validação local mínima executada em Ubuntu 22.04 sem Docker e sem AMP em execução
- validação final via `gen-all-v2.service` e `gen-health.service`

Regra canônica:

- quando houver conflito entre descrição histórica do ETL e runtime real validado, prevalece o runtime real validado
- a reconciliação versionada do repositório ETL passa a ser a leitura oficial do código-fonte desta camada

---

## Relações com outros documentos

Este arquivo é complementar a:

- `docs/03-portal-estatico/README.md`
- `docs/03-portal-estatico/etl-bash-pipeline.md`
- `docs/03-portal-estatico/static-api-v2.md`
- `docs/03-portal-estatico/portal-estatico-operational-runbooks.md`
- `docs/03-portal-estatico/portal-estatico-references-inventory.md`
- `docs/03-portal-estatico/etl-runtime-materialization-runbook.md`
- `docs/01-infra-hostinger/systemd-automation.md`
- `docs/01-infra-hostinger/filesystem-paths-permissions.md`
- `docs/01-infra-hostinger/infra-hostinger-references-inventory.md`
- `docs/95-impl-log/2026-04-01-etl-runtime-reconciliation-parameterization-and-materialization.md`

Este documento não substitui o runbook operacional de materialização nem os documentos de publishing, filesystem, systemd ou troubleshooting.

---

## Repositório reconciliado do runtime ETL

O repositório ETL relevante para esta camada é:

- `14vosx/hsc-cs2-etl`

Após a reconciliação, a estrutura relevante desse repositório passou a ser lida assim:

### `bin/`

Papel:
- fonte versionada dos scripts ETL
- local canônico de autoria dos scripts `gen-*`

### `deploy/systemd/`

Papel:
- units versionadas que refletem o contrato host-facing atual
- referência documental do `systemd`, sem retarget imediato para paths do repositório

### `_reconcile/2026-04-01/vps-runtime/raw/`

Papel:
- evidência bruta da baseline importada da VPS
- trilha auditável do runtime anterior reconciliado

### `scripts/materialize-etl-runtime.sh`

Papel:
- materializar o conjunto de scripts de runtime em `/usr/local/bin`
- sustentar o novo fluxo de deploy do runtime ETL sem editar manualmente cada script no host

---

## Reconciliação da baseline da VPS

A reconciliação executada nesta frente fechou os seguintes pontos:

- `git` já existia na VPS e `/opt/cs2-portal` já era um repositório válido
- o runtime real não estava totalmente representado por esse repositório
- a árvore viva relevante do ETL estava fragmentada entre:
  - `/usr/local/bin/`
  - `/opt/cs2-portal/`
  - `/etc/systemd/system/`
- a baseline real foi importada para o repositório ETL sem alterar o comportamento de produção na fase inicial
- os hashes da cópia local reconciliada foram comparados com os hashes do runtime real da VPS

Leitura canônica:

- o ETL deixou de ser apenas um conjunto de scripts “rodando no host” e passou a ser um runtime reconciliado com fonte versionada explícita

---

## Reprodução local mínima validada

A reconciliação não parou no código importado.

Também foi validada uma reprodução local mínima do runtime ETL em Ubuntu 22.04, usando:

- `bash`
- `git`
- `sqlite3`
- `jq`
- `python3`
- `curl`
- snapshot local do `matchzy.db`

Essa validação confirmou com sucesso a geração local de:

- `ranking.json`
- `matches.json`
- `match/*.json`
- `player/*.json`
- `maps.json`
- `map/*.json`
- `content/news/index.json`
- `health.json`

Também ficou validado que:

- o ETL consegue rodar localmente sem Docker e sem AMP em execução
- o problema residual deixou de ser de execução e passou a ser de contrato de paths e materialização

---

## Parametrização de paths-base

Durante a reconciliação, os scripts centrais do ETL passaram a aceitar variáveis de ambiente para paths-base, mantendo defaults compatíveis com o host atual.

As variáveis-base consolidadas nesta frente incluem:

- `MATCHZY_DB`
- `ETL_BASE_DIR`
- `API_DIR`
- `API_V1_DIR`
- `TMP_DIR`

Essa parametrização foi aplicada de forma incremental aos scripts principais do ETL e validada em dois cenários:

1. modo default, preservando o host atual
2. modo override, usando paths temporários e fixture local

Leitura canônica:

- o runtime do host continua funcionando com os defaults atuais
- o ETL deixou de depender exclusivamente de hardcodes para executar fora do host real

---

## Contrato provisório de materialização

A decisão provisória consolidada para esta fase é:

- `bin/` no repositório é a fonte versionada do ETL
- `/usr/local/bin/*.sh` permanece como camada materializada de runtime no host
- `systemd` continua apontando para `/usr/local/bin/*`
- `${ETL_BASE_DIR}` continua sendo a base para `locks`, `state`, `sql` e scripts de content/news

Motivo da decisão:

- as units atuais rodam como `amp:www-data`
- o host já tem `ReadWritePaths` explícitos e hardening estável
- alterar `ExecStart` diretamente para paths do repositório nesta fase aumentaria risco operacional sem ganho proporcional

---

## Validação final da materialização

A camada materializada foi validada em quatro estágios:

### 1. staging remoto

Os scripts versionados foram materializados em:

- `/tmp/hsc-etl-materialize-staging/usr/local/bin`

### 2. comparação staged vs live

Os hashes dos scripts staged foram comparados com os hashes da árvore live em `/usr/local/bin`.

### 3. backup do runtime anterior

Antes da publicação live, foi criado backup operacional em:

- `/tmp/hsc-etl-live-backup-20260401T200736Z`

### 4. smoke final via systemd

Após a materialização live, foram validados com sucesso:

- `gen-all-v2.service`
- `gen-health.service`
- `health.json` com `ok: true`

---

## Baseline operacional consolidado

O baseline operacional consolidado desta frente passou a ser:

- repositório ETL reconciliado e versionado
- runtime live materializado em `/usr/local/bin`
- `systemd` host-facing preservado
- documentação auditável registrada
- runbook operacional explícito de staging, backup, materialização e rollback
- tag operacional do repositório ETL:
  - `etl-v0.3.0`

---

## Limites deste documento

Este documento não substitui:

- o runbook detalhado de materialização
- os runbooks operacionais do portal
- a documentação do `systemd`
- o inventário estrutural do host
- os detalhes de schema SQL ou contrato JSON

Ele existe para fixar a nova leitura canônica da relação entre repositório ETL, runtime materializado e host real.

---

## Critério de pronto

Este documento pode ser considerado saudável quando:

- explicar de forma clara como o runtime ETL foi reconciliado com o Git
- deixar explícita a diferença entre fonte versionada, runtime materializado e evidência bruta
- registrar a validação local mínima e a validação real no host
- refletir corretamente o contrato atual entre repositório, `/usr/local/bin` e `systemd`
- reduzir ambiguidade operacional real sobre a origem do ETL e sua forma de publicação

---

## Última revisão

- Status: ativo
- Classificação: canônico
- Contexto: portal-estatico / ETL runtime
- Última revisão: 2026-04-01
