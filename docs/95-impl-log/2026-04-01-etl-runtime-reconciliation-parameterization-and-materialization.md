# 2026-04-01 — ETL Runtime Reconciliation, Parameterization and Materialization

## Navegação rápida

- [Home da documentação](../README.md)
- [Impl Log](./README.md)
- [Master Index](../00-governance/99-master-index.md)

---
## Objetivo

Registrar, de forma estável e auditável, o checkpoint técnico construído nesta frente de trabalho, cobrindo:

- reconciliação do runtime ETL real da VPS com o repositório `hsc-cs2-etl`
- import da baseline ativa do host para o Git
- validação local mínima do ETL sem Docker e sem AMP em execução
- desacoplamento interno entre scripts irmãos
- compatibilidade rootless do mirror de news
- parametrização dos paths-base do núcleo do ETL
- introdução da camada de materialização para `/usr/local/bin`
- validação final do runtime materializado via `systemd`
- criação da tag operacional `etl-v0.3.0`

Este impl-log existe para preservar a trilha incremental desta entrega sem substituir os documentos canônicos dos contextos `01-infra-hostinger` e `03-portal-estatico`.

---

## Escopo

Este documento cobre:

- o estado inicial do runtime ETL antes da reconciliação
- a sequência de reconciliação da baseline da VPS
- a evolução do repositório ETL durante a frente
- a validação local mínima em Ubuntu 22.04
- a materialização staged e live do runtime ETL
- a validação final via `gen-all-v2.service` e `gen-health.service`

Este documento não cobre em profundidade:

- contratos JSON campo a campo
- SQL detalhado da v2
- operação do AMP
- topologia completa do Nginx
- backlog funcional futuro do portal

---

## Estado inicial do checkpoint

No início desta frente, o ETL do portal apresentava o seguinte quadro:

- o runtime ativo existia na VPS Hostinger
- `git` já existia no host e `/opt/cs2-portal` já era um repositório válido
- esse repositório não representava integralmente o runtime real em produção
- scripts críticos viviam em `/usr/local/bin`
- base operacional, `sql`, `locks` e `state` viviam em `/opt/cs2-portal`
- `systemd` já apontava para `/usr/local/bin/gen-all-v2.sh` e `/usr/local/bin/gen-health.sh`
- havia necessidade real de equalizar runtime, repositório e processo de deploy

A pergunta central desta frente era:

- como transformar o ETL real da VPS em fonte versionada confiável sem quebrar o contrato atual do host?

---

## Source of truth / evidências

As evidências diretas deste impl-log são:

- runtime real da VPS Hostinger
- hashes dos scripts `gen-*` em `/usr/local/bin`
- units reais em `/etc/systemd/system/`
- repositório `14vosx/hsc-cs2-etl`
- staging remoto em `/tmp/hsc-etl-materialize-staging`
- backup live em `/tmp/hsc-etl-live-backup-20260401T200736Z`
- validação local mínima em Ubuntu 22.04
- validação final de `health.json` com `ok: true`

Repositório e branches envolvidos:

- `hsc-cs2-etl`
  - `chore/reconcile-vps-runtime-2026-04-01`
  - `chore/materialize-etl-runtime-2026-04-01`

Marco operacional final desta frente:

- tag `etl-v0.3.0`

---

## Relações com outros documentos

Este impl-log deve ser reconciliado principalmente com:

- `docs/03-portal-estatico/etl-bash-pipeline.md`
- `docs/03-portal-estatico/etl-runtime-reconciliation.md`
- `docs/03-portal-estatico/etl-runtime-materialization-runbook.md`
- `docs/03-portal-estatico/portal-estatico-operational-runbooks.md`
- `docs/03-portal-estatico/portal-estatico-references-inventory.md`
- `docs/01-infra-hostinger/systemd-automation.md`
- `docs/01-infra-hostinger/filesystem-paths-permissions.md`
- `docs/01-infra-hostinger/infra-hostinger-references-inventory.md`

---

## Linha incremental do checkpoint

## 1. Reconciliação da baseline ativa da VPS

A primeira fase da frente foi de auditoria e import da baseline ativa do host.

Conquistas relevantes:

- confirmação de que `git` já existia na VPS
- confirmação de que `/opt/cs2-portal` já era repositório válido
- identificação de que o runtime real estava fragmentado entre `/usr/local/bin`, `/opt/cs2-portal` e `systemd`
- captura da baseline real do host
- comparação de hashes para validar a integridade da cópia reconciliada
- entrada dos scripts ativos em `bin/`
- entrada das units ativas em `deploy/systemd/`
- preservação da evidência bruta em `_reconcile/2026-04-01/vps-runtime/raw/`

Resultado:

- o repositório ETL passou a conter o runtime real da VPS de forma rastreável
- o comportamento de produção não foi alterado nesta fase inicial

---

## 2. Validação local mínima do ETL

A segunda fase validou uma reprodução local mínima do ETL em Ubuntu 22.04, sem Docker e sem AMP em execução.

Foi usado um snapshot local do `matchzy.db` para montar o sandbox mínimo.

Smoke tests validados com sucesso:

- `gen-ranking.sh`
- `gen-player.sh`
- `gen-matches.sh`
- `gen-players-from-ranking.sh`
- `gen-health.sh`
- `gen-all-v2.sh` (núcleo do pipeline)
- `gen-content-news-cache.sh`
- `gen-content-news-items-cache.sh`

Resultado:

- a reprodução local mínima do ETL foi validada
- o gap remanescente deixou de ser execução básica e passou a ser contrato de paths e materialização

---

## 3. Desacoplamento interno e rootless local

A terceira fase atacou dois pontos:

- chamadas internas entre scripts ainda dependentes de `/usr/local/bin`
- incompatibilidade rootless local dos scripts de mirror de news

Conquistas relevantes:

- chamadas entre scripts irmãos passaram a usar `SCRIPT_DIR`
- o mirror de news ganhou fallback rootless sem quebrar o host real
- o `gen-all-v2.sh` passou a rodar no sandbox local sem depender do `/usr/local/bin` manual para chamadas internas

Resultado:

- o núcleo do ETL ficou mais portátil sem trocar ainda o contrato host-facing

---

## 4. Parametrização dos paths-base

A quarta fase parametrizou o núcleo do ETL com defaults compatíveis com o host atual.

Variáveis consolidadas nesta frente:

- `MATCHZY_DB`
- `ETL_BASE_DIR`
- `API_DIR`
- `API_V1_DIR`
- `TMP_DIR`

Scripts parametrizados e validados em default + override:

- `gen-ranking.sh`
- `gen-player.sh`
- `gen-players-from-ranking.sh`
- `gen-matches.sh`
- `gen-health.sh`
- `gen-maps.sh`
- `gen-players-incremental.sh`
- `gen-match-details-incremental.sh`
- `gen-all-v2.sh`
- `gen-content-news-cache.sh`
- `gen-content-news-items-cache.sh`

Resultado:

- o runtime do host foi preservado
- o ETL passou a ter portabilidade inicial real

---

## 5. Decisão provisória de deploy/materialização

A quinta fase fixou a leitura operacional correta:

- `bin/` no repositório ETL é a fonte versionada
- `/usr/local/bin/*.sh` permanece como camada materializada de runtime no host
- `systemd` continua apontando para `/usr/local/bin/*`
- `deploy/systemd/*` continua refletindo o contrato atual do host

Resultado:

- o runtime live deixa de ser tratado como lugar de autoria manual
- a próxima fase passou a ser materialização controlada, não mudança direta em `systemd`

---

## 6. Materialização staged e live

Foi criada a branch de materialização e o script:

- `scripts/materialize-etl-runtime.sh`

Também foram gerados:

- inventário de materialização
- runbook operacional de staging, backup, materialização live e rollback

Etapas validadas:

1. materialização em diretório temporário local
2. staging remoto em `/tmp/hsc-etl-materialize-staging`
3. comparação de hashes staged vs live
4. smoke manual dos scripts staged na VPS como `amp`
5. backup live em `/tmp/hsc-etl-live-backup-20260401T200736Z`
6. materialização real em `/usr/local/bin`

Resultado:

- os hashes live passaram a bater 1:1 com os hashes staged
- a camada `/usr/local/bin` foi promovida a runtime materializado real

---

## 7. Validação final via systemd

Após a materialização live, a validação final do contrato host-facing foi concluída com sucesso:

- `gen-all-v2.service` => `status=0/SUCCESS`
- `gen-health.service` => `status=0/SUCCESS`
- `health.json` final => `ok: true`

Também foi observado que:

- `ranking.ok: true`
- `matches.ok: true`
- `content.news.ok: true`
- `rankingPlayersMissing.count: 0`

Resultado:

- o host passou a operar com o novo runtime materializado sem exigir alteração de unit file

---

## 8. Fechamento em Git

Ao final da frente:

- a branch completa foi publicada e promovida para `main`
- o repositório ETL recebeu a tag:
  - `etl-v0.3.0`
- foi adicionado `README.md` mínimo explicando a estrutura e o contrato de runtime do repositório ETL

Leitura final do checkpoint:

- runtime reconciliado
- portabilidade inicial validada
- materialização ativa e documentada
- baseline operacional novo consolidado em Git e no host

---

## Estado final ao fim do checkpoint

Ao final desta frente, o estado consolidado ficou assim:

### Repositório ETL
- passou a representar o runtime real da VPS
- contém `bin/` como fonte versionada
- preserva `_reconcile/` como evidência bruta
- contém script e docs de materialização

### Host Hostinger
- continua com `systemd` apontando para `/usr/local/bin/*`
- recebeu materialização live validada
- manteve o contrato host-facing estável
- preservou rollback operacional imediato

### Portal Estático / Static API v2
- ETL principal segue aggregator-first
- `gen-all-v2` e `gen-health` seguem operacionais
- `health.json` final permanece saudável

---

## Limites do checkpoint

Este impl-log não substitui:

- os documentos canônicos do Portal Estático
- os documentos estruturais da Infra Hostinger
- os runbooks do portal
- o inventário estrutural do host

Ele existe para preservar a trilha incremental desta reconciliação e materialização.

---

## Última revisão

- Status: ativo
- Classificação: impl-log
- Contexto: portal-estatico / infra-hostinger / ETL runtime
- Última revisão: 2026-04-01
