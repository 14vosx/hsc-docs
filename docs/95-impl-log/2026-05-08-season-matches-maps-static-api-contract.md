# 2026-05-08 — Season matches/maps na Static API v2

## Status

`ETL mergeado e validado localmente; produção/runtime pendente`.

Este registro documenta a Prioridade 3 — Season matches/maps, entregue no `hsc-cs2-etl` após merge do PR #10.

## Repositórios e PRs

- `hsc-cs2-etl` PR #10: `feat(seasons): add season matches and maps`
- commit mergeado: `0345b57`
- `hsc-docs`: documentação canônica dos contratos públicos da Static API v2

Não houve alteração registrada nesta entrega em:

- `hsc-auth-api`
- `hsc-cs2-portal`
- `hsc-backoffice-admin`
- produção/runtime
- `systemd`
- `/usr/local/bin`
- `/var/www`

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

## Fora do escopo desta entrega

Esta entrega não publicou nem validou produção para esses endpoints.

Também ficou fora do escopo:

- alteração em `hsc-auth-api`
- alteração em `hsc-cs2-portal`
- alteração em `hsc-backoffice-admin`
- alteração em produção/runtime
- alteração em `systemd`
- alteração em `/usr/local/bin`
- alteração em `/var/www`
- criação de endpoints de detalhe por Season
- afirmação de consumo pelo Portal Angular
- mudança dos contratos globais de partidas e mapas

## Pendências

- materializar o ETL atualizado no runtime quando essa etapa for explicitamente executada
- validar a geração em produção/runtime
- validar a publicação pública dos endpoints sob `/api/cs2/v2/season/{slug}/...`
- documentar eventual consumo pelo Portal Angular somente depois de implementação e validação próprias

## Documentos canônicos relacionados

- [JSON Contracts](../03-portal-estatico/json-contracts.md)
- [Static API v2](../03-portal-estatico/static-api-v2.md)
- [ETL Bash Pipeline](../03-portal-estatico/etl-bash-pipeline.md)
