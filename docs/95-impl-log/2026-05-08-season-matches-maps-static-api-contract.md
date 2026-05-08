# 2026-05-08 — Season matches/maps na Static API v2

## Status

`Publicado e validado em produção/runtime`.

Este registro documenta a Prioridade 3 — Season matches/maps.

Histórico do status:

- a primeira etapa foi entregue no `hsc-cs2-etl` após merge do PR #10, com ETL mergeado e validação local/smoke temporária
- naquele checkpoint, produção/runtime ainda estava pendente
- em 2026-05-08, a etapa runtime/prod foi materializada e validada publicamente

## Repositórios e PRs

- `hsc-cs2-etl` PR #10: `feat(seasons): add season matches and maps`
- commit mergeado PR #10: `0345b57`
- `hsc-cs2-etl` PR #11: `fix(etl): materialize season matches maps script`
- commit mergeado PR #11: `81701e6`
- `hsc-docs`: documentação canônica dos contratos públicos da Static API v2

Não houve alteração registrada nesta frente em:

- `hsc-auth-api`
- `hsc-backoffice-admin`
- Portal Angular

## Contexto de produto

Season não deve ser lida apenas como CRUD administrativo.

No HSC, Season é uma feature competitiva com tema/narrativa, ranking separado, partidas e mapas classificados dentro da temporada e ciclo competitivo próprio.

Nesta entrega:

- Ranking Geral continua diferente de Ranking da Season
- partidas/maps por Season são recortes competitivos da Season
- esses recortes não substituem os contratos globais de partidas e mapas
- a entrega é um contrato incremental novo, não uma restauração acrítica de documentação antiga

## Contratos públicos adicionados

Novos recursos da Static API v2:

- `/api/cs2/v2/season/{slug}/matches.json`
- `/api/cs2/v2/season/{slug}/maps.json`

### `season/{slug}/matches.json`

Payload top-level:

- `generatedAt`
- `season`
- `rules`
- `summary`
- `computed`
- `matches`

`summary`:

- `matches`
- `maps`
- `rounds`
- `players`
- `lastMapEndedAt`

`computed`:

- `firstMapStartedAt`

`matches[]` preserva o shape base de `matches.json` quando possível, incluindo:

- `matchid`
- `start_time`
- `end_time`
- `winner`
- `series_type`
- `team1_name`
- `team1_score`
- `team2_name`
- `team2_score`
- `server_ip`
- `maps[]`

Campos aditivos por partida:

- `seasonMapCount`
- `seasonRounds`
- `seasonFirstMapStartedAt`
- `seasonLastMapEndedAt`

Notas de contrato:

- `matches[].maps[]` é restringido aos mapas válidos da Season
- `matches[].maps[]` inclui `rounds` como campo aditivo

### `season/{slug}/maps.json`

Payload top-level:

- `generatedAt`
- `season`
- `rules`
- `summary`
- `computed`
- `maps`

`summary`:

- `matches`
- `maps`
- `rounds`
- `players`
- `lastMapEndedAt`

`computed`:

- `distinctMaps`

`maps[]` é agregado por `mapname` dentro da Season e contém:

- `map`
- `matches`
- `rounds`
- `avgRoundsPerMatch`
- `lastPlayed`

## Regra de pertencimento

Um mapa pertence à Season quando:

- `matchzy_stats_maps.end_time` está entre `season.start_at` e `season.end_at`
- `winner` está preenchido
- `team1_score + team2_score >= 12`

## Relação com endpoints globais

Os contratos globais abaixo não mudaram nesta entrega:

- `matches.json`
- `maps.json`
- `match/{id}.json`
- `map/{map}.json`

Detalhes continuam nos endpoints globais:

- `/api/cs2/v2/match/{id}.json`
- `/api/cs2/v2/map/{map}.json`

Não foram criados:

- `season/{slug}/match/{id}.json`
- `season/{slug}/map/{map}.json`

## Validação local registrada

Validações executadas no `hsc-cs2-etl`:

- `bash -n bin/gen-season-matches-maps.sh`
- `bash -n bin/gen-all-v2.sh`
- `shellcheck bin/gen-season-matches-maps.sh bin/gen-all-v2.sh`
- `git diff --check`
- `git diff --cached --check`

Smoke local temporário:

- SQLite fixture em `/tmp`
- fake Auth API local em `127.0.0.1`
- `API_DIR` temporário
- execução de `bin/gen-season-matches-maps.sh`
- validação dos JSONs com `python3 -m json.tool`

Resultado do smoke:

- `season/s01-2026/matches.json` gerado
- `season/s01-2026/maps.json` gerado
- `summary.matches = 2`
- `summary.maps = 3`
- `summary.rounds = 67`
- `summary.players = 3`
- `computed.distinctMaps = 3`
- mapa inválido com `winner` vazio e `0` rounds foi excluído

## Publicação e validação runtime/prod

Em 2026-05-08, os contratos de partidas/maps por Season foram materializados no runtime Hostinger e validados publicamente.

Contexto runtime/prod:

- Host ETL: `srv1353392`
- runtime source: `/opt/cs2-portal`
- runtime source final atualizado para `81701e6` (`fix(etl): materialize season matches maps script`)
- scripts runtime: `/usr/local/bin`
- Static API: `/var/www/api/cs2/v2`
- service: `gen-all-v2.service`
- timer: `gen-all-v2.timer`
- snapshot antes da publicação: `/root/hsc-snapshots/etl-pre-season-matches-maps-20260508T143611Z`

Sequência executada:

- `/opt/cs2-portal` foi atualizado inicialmente de `31ce4fc` para `0345b57`
- `scripts/materialize-etl-runtime.sh` foi executado
- foi identificado um gap operacional: `gen-all-v2.sh` foi materializado, mas `gen-season-matches-maps.sh` ainda não foi copiado para `/usr/local/bin`
- a correção imediata no runtime foi feita manualmente com `install`, copiando `/opt/cs2-portal/bin/gen-season-matches-maps.sh` para `/usr/local/bin/gen-season-matches-maps.sh`
- o hash do script foi confirmado como `4b0eabcba08ee08b46c26278d4b1a99128b16d4c3a0fa6bffeea617ee5d01ceb`
- depois disso, a correção definitiva foi aberta e mergeada no `hsc-cs2-etl` PR #11
- `/opt/cs2-portal` foi atualizado para `81701e6`
- `scripts/materialize-etl-runtime.sh` foi reexecutado e passou a materializar automaticamente `/usr/local/bin/gen-season-matches-maps.sh`

Hash final validado para os dois caminhos:

- `/opt/cs2-portal/bin/gen-season-matches-maps.sh`
- `/usr/local/bin/gen-season-matches-maps.sh`
- hash: `4b0eabcba08ee08b46c26278d4b1a99128b16d4c3a0fa6bffeea617ee5d01ceb`

Execução final do service:

- `gen-all-v2.service` executou com `status=0/SUCCESS`
- o journal confirmou os steps:
  - `STEP gen-seasons`
  - `STEP gen-season-matches-maps`
  - `STEP gen-season-rankings`

Endpoints públicos validados:

- `https://haxixesmokeclub.com/api/cs2/v2/season/s01-2026/matches.json`
- `https://haxixesmokeclub.com/api/cs2/v2/season/s01-2026/maps.json`
- `https://haxixesmokeclub.com/api/cs2/v2/season/s01-2026/ranking.json`

Resultado do smoke público final:

- `matches.summary.matches = 15`
- `matches.summary.maps = 15`
- `matches.summary.rounds = 336`
- `matches.summary.players = 32`
- `matches.summary.lastMapEndedAt = 2026-04-16 01:23:43`
- `matches.computed.firstMapStartedAt = 2026-02-09 22:54:25`
- `matches_count = 15`
- `maps.summary.matches = 15`
- `maps.summary.maps = 15`
- `maps.summary.rounds = 336`
- `maps.summary.players = 32`
- `maps.summary.lastMapEndedAt = 2026-04-16 01:23:43`
- `maps.computed.distinctMaps = 7`
- `maps_count = 7`
- regressão de ranking: `players = 32`
- regressão de ranking: `missing_steam_avatar_url_field = 0`
- regressão de ranking: `non_null_steam_avatar_url = 32`

## Fora do escopo desta frente

O checkpoint inicial do PR #10 não publicou nem validou produção para esses endpoints. Essa pendência foi fechada posteriormente pela publicação runtime/prod documentada acima.

Também ficou fora do escopo:

- alteração em `hsc-auth-api`
- alteração em `hsc-backoffice-admin`
- alteração no Portal Angular
- criação de endpoints de detalhe por Season
- afirmação de consumo pelo Portal Angular
- mudança dos contratos globais de partidas e mapas

## Pendências

- não registrar validação visual de staging/prod para Season Matches/Maps até que essa etapa seja autorizada e executada

## Atualização posterior - Portal Angular CS2 Next

Em 2026-05-08, o `hsc-cs2-portal` PR #28 foi mergeado em `main` com o commit `01564d1` (`feat(cs2-next): add season matches and maps pages`).

Esse PR adicionou rotas player-facing no Portal Angular CS2 Next para consumir os contratos documentados da Static API v2:

- `/seasons/current/matches`
- `/seasons/:slug/matches`
- `/seasons/current/maps`
- `/seasons/:slug/maps`

As listagens consomem os recortes prontos:

- `/api/cs2/v2/season/{slug}/matches.json`
- `/api/cs2/v2/season/{slug}/maps.json`

Os detalhes continuam globais:

- `/matches/:matchId`
- `/maps/:map`

Não foram criados endpoints de detalhe por Season.

## Atualização posterior - deploy staging e hardening Nginx

Em 2026-05-08, após o merge do `hsc-cs2-portal` PR #28 (`01564d1`) e do `hsc-docs` PR #38 (`da2743f`), o Portal Angular CS2 Next foi publicado em staging público na VPS `srv1353392`.

Publicação realizada:

- build local publicado a partir de `frontend/angular/dist/hsc-cs2-portal-angular/browser/*`
- webroot de staging: `/var/www/portal/cs2-next`
- Nginx ativo: `/etc/nginx/conf.d/srv1353392.hstgr.cloud.conf`
- backup do build anterior: `/root/hsc-snapshots/cs2-next-before-cs2-next-20260508T161102Z-01564d1.tar.gz`
- backup do Nginx antes do hardening: `/etc/nginx/conf.d/srv1353392.hstgr.cloud.conf.bak.harden-cs2next-20260508T162837Z`

Smoke público validado com `200`:

- `/portal/cs2-next/`
- `/portal/cs2-next/seasons/current`
- `/portal/cs2-next/seasons/current/ranking`
- `/portal/cs2-next/seasons/current/matches`
- `/portal/cs2-next/seasons/current/maps`
- `/portal/cs2-next/matches`
- `/portal/cs2-next/maps`
- `/portal/cs2-next/api-smoke`
- `/portal/cs2/`
- `/api/cs2/v2/season/s01-2026/matches.json`
- `/api/cs2/v2/season/s01-2026/maps.json`

Hardening Nginx validado:

- `/portal/cs2` → `301 /portal/cs2/`
- `/portal/cs2-next` → `301 /portal/cs2-next/`
- `/portal/cs2-next/package.json` → `404`
- `/portal/cs2-next/angular.json` → `404`
- `/portal/cs2-next/src/` → `404`
- `/portal/cs2-next/node_modules/` → `404`
- `/portal/cs2-next/.git/` → `404`

Leitura final:

- `/portal/cs2-next/` ficou concluído em staging público para as páginas player-facing de Season Matches/Maps
- `/portal/cs2/` segue legado e preservado
- esta frente não representa cutover de produção do portal legado
- rollback permanece disponível pelos snapshots registrados acima

## Documentos canônicos relacionados

- [JSON Contracts](../03-portal-estatico/json-contracts.md)
- [Static API v2](../03-portal-estatico/static-api-v2.md)
- [ETL Bash Pipeline](../03-portal-estatico/etl-bash-pipeline.md)
