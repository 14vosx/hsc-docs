# 2026-05-07 — Steam Profiles e avatar no Season Ranking

## Status

`publicado e validado em runtime/prod`.

Este registro documenta decisões, contratos, smokes locais anteriores e a publicação runtime/prod da Prioridade 2 — Avatar Steam no ranking/pódio.

## Repositórios e PRs

- `hsc-auth-api` PR #51: `feat(steam): add Steam profiles cache`
- `hsc-cs2-etl` PR #9: `feat(seasons): add Steam avatar URL to season ranking`
- `hsc-docs` PR #34: `docs(seasons): document Steam avatars in season ranking`

## Decisão arquitetural

- `hsc-auth-api` é o dono canônico de Steam Profiles.
- `hsc-auth-api` mantém a tabela `steam_profiles`.
- `hsc-auth-api` expõe `POST /internal/steam/profiles/resolve` como endpoint interno protegido por `X-Internal-Key`/`INTERNAL_API_KEY`.
- `hsc-auth-api` usa `STEAM_API_KEY` para consultar Steam Web API.
- `hsc-auth-api` aplica cache, TTL, batch de até 100 SteamIDs, timeout e fallback para cache.
- `hsc-cs2-etl` é consumidor interno da Auth API e materializador do campo na Static API.
- `hsc-cs2-etl` não chama Steam Web API diretamente.
- `hsc-cs2-etl` não usa `STEAM_API_KEY`.
- `hsc-cs2-portal` apenas exibe o campo publicado.
- `hsc-backoffice-admin` está fora desta fatia.

## Contrato público

Recurso:

- `/api/cs2/v2/season/{slug}/ranking.json`

Campo novo em cada item de `players[]`:

- `steam_avatar_url`

Tipo:

- `string|null`

Fallback:

- `null` quando o perfil, avatar, chave interna, endpoint ou resposta válida não estiverem disponíveis.

Falha, timeout, ausência de `INTERNAL_API_KEY`, resposta inválida ou perfil ausente não bloqueiam a publicação do ranking.

## Riscos e limites

- dependência da Steam API externa pela Auth API
- necessidade de `STEAM_API_KEY` na Auth API
- necessidade de `INTERNAL_API_KEY` para consumidores internos
- risco de cache stale dentro da política de TTL
- valores reais de chaves não devem ser registrados em documentação
- rotação futura de `INTERNAL_API_KEY`/`STEAM_API_KEY` ainda não está automatizada

## Smokes locais executados

Os smokes abaixo foram funcionais e locais no `hsc-cs2-etl`, usando fake Auth API local, SQLite fixture temporário e `API_DIR` temporário.

Eles não tocaram produção, `/var/www` ou runtime real.

Smoke 1, com `INTERNAL_API_KEY`:

- fake Auth API local serviu `/content/seasons`
- fake Auth API local serviu `POST /internal/steam/profiles/resolve`
- `gen-season-rankings.sh` gerou `season/s01-2026/ranking.json`
- player `76561198000000001` recebeu `steam_avatar_url = "https://cdn.example.test/avatar-medium.jpg"`
- player `76561198000000002` recebeu `steam_avatar_url = null`
- assertions passaram com `OK steam_avatar_url preenchido e fallback null validado`

Smoke 2, sem `INTERNAL_API_KEY`:

- fake Auth API local serviu apenas `/content/seasons`
- `POST /internal/steam/profiles/resolve` não deveria ser chamado
- `gen-season-rankings.sh` gerou `season/s01-2026/ranking.json`
- player `76561198000000001` recebeu `steam_avatar_url = null`
- fake auth log ficou vazio
- assertions passaram com `OK sem INTERNAL_API_KEY ranking gerado com steam_avatar_url null`

## Publicação runtime/prod

Auth API / Lightsail:

- código em `/opt/hsc/hsc-auth-api` atualizado para `7b61cd7 feat(steam): add Steam profiles cache (#51)`
- snapshot de segurança criado em `/home/ubuntu/hsc-snapshots/auth-api-pre-steam-profiles-20260508T015849Z`
- migration `0006_steam_profiles.sql` aplicada
- serviço `hsc-auth-api.service` reiniciado
- health público `https://auth-api.haxixesmokeclub.com/health` retornou `200` com `db.ready=true`
- env real configurada para `STEAM_API_KEY`, `INTERNAL_API_KEY`, `STEAM_PROFILE_CACHE_TTL_SECONDS` e `STEAM_API_TIMEOUT_SECONDS`, sem documentar valores
- endpoint interno sem key retornou `401 invalid_internal_key`
- endpoint interno autenticado retornou `200 OK`, `profiles_count=2` e `missing_count=0`
- tabela `steam_profiles` validada com `count=2` e `lastFetchedAt=2026-05-08T02:11:02Z`

ETL / Hostinger VPS:

- host runtime: `srv1353392`
- clone runtime `/opt/cs2-portal` atualizado para `31ce4fc feat(seasons): add Steam avatar URL to season ranking (#9)`
- snapshot de segurança criado em `/root/hsc-snapshots/etl-pre-steam-avatar-20260508T014952Z`
- snapshots de ranking criados antes dos smokes em `/root/hsc-snapshots/season-ranking-before-steam-avatar-null-20260508T015325Z.json` e `/root/hsc-snapshots/season-ranking-before-steam-avatar-real-20260508T021444Z.json`
- `/usr/local/bin/gen-season-rankings.sh` materializado via `/opt/cs2-portal/scripts/materialize-etl-runtime.sh`
- hash source/runtime de `gen-season-rankings.sh` confirmado igual: `78db539257451215394aae1d7c3dea338873cb70df254243168804d9f158b8f5`
- arquivo protegido `/etc/hsc/cs2-etl.env` configurado sem documentar valores sensíveis
- drop-in systemd `/etc/systemd/system/gen-all-v2.service.d/steam-profiles-env.conf` configurado com `EnvironmentFile=/etc/hsc/cs2-etl.env`
- variáveis ETL configuradas: `AUTH_INTERNAL_BASE=https://auth-api.haxixesmokeclub.com`, `INTERNAL_API_KEY` e `STEAM_PROFILES_RESOLVE_TIMEOUT_SECONDS=8`

## Smokes runtime/prod executados

Auth API:

- `POST /internal/steam/profiles/resolve` sem key validou proteção com `401 invalid_internal_key`
- `POST /internal/steam/profiles/resolve` autenticado validou resposta `200 OK`, `profiles_count=2` e `missing_count=0`
- tabela `steam_profiles` populada com `count=2`
- um db quick check avulso falhou depois porque o script Node não carregou `.env`; o serviço e o endpoint estavam OK, e o quick check foi corrigido com `dotenv.config`

ETL:

- primeiro smoke sem `INTERNAL_API_KEY` gerou ranking com 32 players, `missing_steam_avatar_url_field=0` e `non_null_steam_avatar_url=0`, validando fallback `steam_avatar_url: null`
- smoke manual com Auth API real gerou ranking com 32 players, `missing_steam_avatar_url_field=0` e `non_null_steam_avatar_url=32`
- execução via `systemctl start gen-all-v2.service` concluiu com status `0/SUCCESS`
- `STEP gen-season-rankings` executou no serviço systemd
- validação pós-systemd confirmou 32 players, `missing_steam_avatar_url_field=0` e `non_null_steam_avatar_url=32`

Static API v2 pública:

- `/api/cs2/v2/season/s01-2026/ranking.json` validou 32 players, `missing_field=0` e `non_null=32`
- HTTP público preservou avatars reais após execução systemd

Portal CS2 Next:

- smoke visual validado em `https://haxixesmokeclub.com/portal/cs2-next/seasons/current`
- smoke visual validado em `https://haxixesmokeclub.com/portal/cs2-next/seasons/current/ranking`
- pódio/top players, tabela preview e tabela completa exibiram avatares reais
- 32 jogadores foram renderizados
- layout e colunas de ranking foram preservados

## Documentos canônicos relacionados

- [JSON Contracts](../03-portal-estatico/json-contracts.md)
- [Static API v2](../03-portal-estatico/static-api-v2.md)
- [ETL Bash Pipeline](../03-portal-estatico/etl-bash-pipeline.md)
- [Auth API Operations](../04-infra-aws-lightsail/auth-api-operations.md)
