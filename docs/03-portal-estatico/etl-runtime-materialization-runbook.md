# ETL Runtime Materialization Runbook

## Navegação rápida

- [Home da documentação](../README.md)
- [Portal Estático](./README.md)
- [Master Index](../00-governance/99-master-index.md)

---
## Objetivo

Documentar o runbook canônico de materialização do runtime ETL da Static API v2 no host Hostinger, preservando o contrato atual do `systemd` e reduzindo dependência de edição manual em `/usr/local/bin`.

Este documento existe para registrar, de forma estável e auditável:

- como preparar o staging remoto do runtime ETL
- como materializar os scripts versionados em `/usr/local/bin`
- como validar staging, hashes, backup e publicação live
- como validar o resultado final via `systemd`
- como executar rollback manual do runtime materializado

---

## Escopo

Este documento cobre:

- inventário de scripts materializados em `/usr/local/bin`
- staging remoto do runtime ETL
- smoke manual com os scripts staged
- backup do runtime live anterior
- materialização real em `/usr/local/bin`
- validação final via `systemd`
- rollback manual

Este documento não cobre em profundidade:

- contratos JSON da v2
- SQL da geração
- operação do AMP
- topologia completa do Nginx
- deploy da Auth API

---

## Estado atual

O contrato operacional vigente nesta fase é:

- `bin/` no repositório ETL é a fonte versionada
- `/usr/local/bin/*.sh` é a camada materializada de runtime no host
- `systemd` continua apontando para `/usr/local/bin/*`
- `${ETL_BASE_DIR}` continua sustentando `locks`, `state`, `sql` e scripts de content/news
- a materialização do runtime ETL já foi validada em staging e em live na VPS Hostinger

Leitura canônica:

- a materialização não troca o contrato do `systemd`
- ela substitui o processo manual de copiar/editar scripts diretamente no host

---

## Source of truth / evidências

As evidências diretas deste runbook são:

- `scripts/materialize-etl-runtime.sh` no repositório `hsc-cs2-etl`
- staging remoto em `/tmp/hsc-etl-materialize-staging`
- comparação de hashes staged vs live
- backup operacional em `/tmp/hsc-etl-live-backup-*`
- smoke manual do runtime staged como `amp`
- validação final via `gen-all-v2.service` e `gen-health.service`

---

## Relações com outros documentos

Este arquivo é complementar a:

- `docs/03-portal-estatico/etl-runtime-reconciliation.md`
- `docs/03-portal-estatico/etl-bash-pipeline.md`
- `docs/03-portal-estatico/portal-estatico-operational-runbooks.md`
- `docs/03-portal-estatico/portal-estatico-references-inventory.md`
- `docs/01-infra-hostinger/systemd-automation.md`
- `docs/01-infra-hostinger/filesystem-paths-permissions.md`
- `docs/95-impl-log/2026-04-01-etl-runtime-reconciliation-parameterization-and-materialization.md`

---

## Pré-condições

Antes de materializar o runtime ETL, devem estar disponíveis:

- branch de materialização atualizada localmente
- acesso SSH à VPS Hostinger
- script `scripts/materialize-etl-runtime.sh`
- árvore `bin/` reconciliada no repositório ETL
- permissão para usar staging e backup em `/tmp`

---

## Inventário materializado em `/usr/local/bin`

Os scripts que devem ser materializados em `/usr/local/bin` no contrato atual são:

- `gen-all-v2.sh`
- `gen-health.sh`
- `gen-maps.sh`
- `gen-match-details-incremental.sh`
- `gen-matches.sh`
- `gen-players-from-ranking.sh`
- `gen-player.sh`
- `gen-players-incremental.sh`
- `gen-ranking.sh`

Scripts que permanecem referenciados via `${ETL_BASE_DIR}/bin` no contrato atual:

- `gen-content-news-cache.sh`
- `gen-content-news-items-cache.sh`

---

## Estratégia de materialização

A sequência canônica de materialização é:

1. staging remoto
2. materialização staged em destino temporário
3. comparação de hashes staged vs live
4. smoke manual com runtime staged
5. backup do runtime live
6. materialização live em `/usr/local/bin`
7. validação final via `systemd`

Regra importante:

- o `systemd` não deve ser retargetado para paths do repositório nesta fase
- o runtime live continua sendo `/usr/local/bin/*`

---

## Comando de staging remoto

O staging remoto canônico desta frente foi executado a partir do repositório ETL com estrutura equivalente a:

    STAGING_ROOT="/tmp/hsc-etl-materialize-staging"
    SSH_TARGET="root@187.77.32.7"

    tar -C . -cf - scripts/materialize-etl-runtime.sh bin     | ssh "$SSH_TARGET" "
      set -euo pipefail
      rm -rf '$STAGING_ROOT'
      mkdir -p '$STAGING_ROOT'
      tar -C '$STAGING_ROOT' -xf -
      DEST_BIN='$STAGING_ROOT/usr/local/bin' bash '$STAGING_ROOT/scripts/materialize-etl-runtime.sh'
    "

O resultado esperado do staging é:

- árvore staged em `/tmp/hsc-etl-materialize-staging/usr/local/bin`
- scripts staged com permissão executável
- nenhum impacto ainda sobre `/usr/local/bin` live

---

## Comparação de hashes staged vs live

Após o staging, a validação canônica compara os hashes SHA-256 dos scripts staged com os hashes live atualmente em `/usr/local/bin`.

Objetivo:

- provar exatamente o que será alterado
- evitar publicação cega no runtime live
- registrar se o host vai receber mudança real ou apenas re-materialização equivalente

---

## Smoke manual com os scripts staged

Antes da publicação live, o runtime staged deve ser exercitado na própria VPS.

Sequência validada nesta frente:

    runuser -u amp -- env VERBOSE=1 /tmp/hsc-etl-materialize-staging/usr/local/bin/gen-all-v2.sh
    runuser -u amp -- /tmp/hsc-etl-materialize-staging/usr/local/bin/gen-health.sh
    cat /var/www/api/cs2/v2/health.json

Critério de sucesso:

- `gen-all-v2.sh` staged roda sem erro estrutural
- `gen-health.sh` staged atualiza `health.json`
- `health.json` final permanece com `ok: true`

---

## Backup + materialização live

O backup live deve ser feito antes da materialização real.

Forma validada nesta frente:

    BACKUP_DIR="/tmp/hsc-etl-live-backup-$(date -u +%Y%m%dT%H%M%SZ)"
    mkdir -p "$BACKUP_DIR"

    cp -a /usr/local/bin/gen-*.sh "$BACKUP_DIR"/

    DEST_BIN="/usr/local/bin" bash /tmp/hsc-etl-materialize-staging/scripts/materialize-etl-runtime.sh

Critério de sucesso:

- backup criado antes da substituição live
- hashes live passam a bater com os hashes staged
- `/usr/local/bin` deixa de depender de edição manual arquivo a arquivo

---

## Validação final via systemd

Após a materialização live, a validação final do contrato host-facing deve ocorrer via `systemd`.

Sequência validada nesta frente:

    systemctl start gen-all-v2.service
    systemctl start gen-health.service

    systemctl --no-pager --full status gen-all-v2.service gen-health.service
    journalctl -u gen-all-v2.service -n 40 --no-pager
    journalctl -u gen-health.service -n 20 --no-pager
    cat /var/www/api/cs2/v2/health.json

Critério de sucesso:

- `gen-all-v2.service` finaliza com `status=0/SUCCESS`
- `gen-health.service` finaliza com `status=0/SUCCESS`
- `health.json` final permanece com `ok: true`

---

## Rollback manual

O rollback manual do runtime materializado deve usar o backup live criado antes da substituição.

Forma canônica:

    cp -a /tmp/hsc-etl-live-backup-YYYYMMDDTHHMMSSZ/gen-*.sh /usr/local/bin/
    systemctl start gen-all-v2.service
    systemctl start gen-health.service

Regra importante:

- o rollback devolve a camada `/usr/local/bin`
- ele não muda o contrato do `systemd`
- o rollback deve sempre ser seguido de novo smoke operacional

---

## Limites deste documento

Este documento não substitui:

- a documentação do runtime ETL reconciliado
- o documento da pipeline Bash da v2
- os runbooks gerais do Portal Estático
- a documentação estrutural da Infra Hostinger

Ele existe especificamente para fixar o procedimento de materialização do runtime ETL no host.

---

## Critério de pronto

Este documento pode ser considerado saudável quando:

- explicar como staging, backup, materialização live e rollback devem ser feitos
- deixar claro o inventário materializado em `/usr/local/bin`
- manter coerência com o contrato atual do `systemd`
- reduzir dependência de memória informal para deploy do runtime ETL

---

## Última revisão

- Status: ativo
- Classificação: canônico
- Contexto: portal-estatico / ETL materialization
- Última revisão: 2026-04-01
