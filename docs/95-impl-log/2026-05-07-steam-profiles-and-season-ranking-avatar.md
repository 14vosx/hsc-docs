# 2026-05-07 — Steam Profiles e avatar no Season Ranking

## Status

`source-ready` / mergeado em código-fonte.

Este registro documenta decisões e contratos já concluídos nos repositórios de implementação. Ele não afirma que a produção já foi configurada, materializada ou validada com smoke público.

## Repositórios e PRs

- `hsc-auth-api` PR #51: `feat(steam): add Steam profiles cache`
- `hsc-cs2-etl` PR #9: `feat(seasons): add Steam avatar URL to season ranking`

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
- produção ainda requer configuração, materialização e smoke específicos

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

## Smokes pendentes de runtime/prod

- Auth API runtime/prod com variáveis reais configuradas
- ETL runtime/prod materializado
- JSON público validado em `/api/cs2/v2/season/{slug}/ranking.json` contendo `players[].steam_avatar_url`
- Portal público validado em `/seasons/current` e `/seasons/current/ranking` exibindo os dados materializados

## Documentos canônicos relacionados

- [JSON Contracts](../03-portal-estatico/json-contracts.md)
- [Static API v2](../03-portal-estatico/static-api-v2.md)
- [ETL Bash Pipeline](../03-portal-estatico/etl-bash-pipeline.md)
- [Auth API Operations](../04-infra-aws-lightsail/auth-api-operations.md)
