# Player Bunker Deploy and Rollback Plan

Status: plano operacional para publicacao controlada

Escopo: checklist go/no-go para publicar o MVP do Player Bunker com rollback por camada

Este documento nao autoriza deploy automatico por si so.

## 1. Objetivo

Publicar o MVP do Player Bunker com seguranca, preservando a separacao de responsabilidades entre ETL, Auth API e Portal.

Garantir rollback rapido por camada:

- Portal;
- Auth API;
- artifact.

## 2. Escopo

Inclui:

- `hsc-cs2-portal`;
- `hsc-auth-api`;
- artifact do `hsc-cs2-etl`;
- Nginx/reverse proxy necessario para `/player/*`.

Nao inclui:

- componentizacao do Bunker;
- Angular Material;
- cron/timer definitivo;
- `gen-all-v2` definitivo;
- ranking/elegibilidade/avatar Steam no Player Hero;
- billing/premium.

## 3. Arquitetura alvo de publicacao

Fluxo alvo:

1. ETL gera artifact.
2. Auth API le o artifact em modo read-only.
3. Portal consome a Auth API.

Responsabilidades:

- Nginx serve o Portal Angular estatico em `/portal/cs2-next/`.
- Nginx faz reverse proxy de `/player/*` para a Auth API.
- Auth API emite e limpa o cookie proprio `hsc_player_session`.
- Auth API le o artifact read-only via variaveis de ambiente.
- Auth API nao calcula stats competitivas.
- Portal nao le o artifact diretamente.
- ETL e dono da geracao/materializacao do artifact.
- ETL gera o artifact em root controlado fora de webroot.

## 4. Pre-requisitos go/no-go

Antes de publicar:

- [ ] Portal `main` atualizado e build passando.
- [ ] Auth API `main` atualizado e `/health` ok.
- [ ] ETL `main` atualizado e geracao de artifact validada.
- [ ] MatchZy DB/snapshot definido.
- [ ] Artifact root persistente definido.
- [ ] `PLAYER_BUNKER_ARTIFACT_ROOT` definido.
- [ ] `PLAYER_BUNKER_ACTIVE_SEASON_SLUG` definido.
- [ ] `PLAYER_STEAM_AUTH_ENABLED` definido.
- [ ] `PLAYER_STEAM_RETURN_URL` definido.
- [ ] `PLAYER_STEAM_REALM` definido.
- [ ] `PLAYER_AUTH_CALLBACK_REDIRECT_ENABLED` definido.
- [ ] `PLAYER_AUTH_SUCCESS_REDIRECT_URL` definido.
- [ ] `PLAYER_AUTH_FAILURE_REDIRECT_URL` definido.
- [ ] Nginx `/player/*` proxy definido.
- [ ] Cookie seguro validado conforme dominio/protocolo.
- [ ] Rollback pronto antes de deploy.

## 5. Plano de deploy - ETL artifact

Gerar o artifact a partir do MatchZy DB/snapshot definido para a publicacao.

Checklist:

- [ ] Usar root persistente fora de `/var/www`.
- [ ] Validar `players-manifest.json`.
- [ ] Validar `player/<steamid64>.json`.
- [ ] Confirmar que o artifact nao sera publicado no webroot.
- [ ] Confirmar que o artifact nao contem secrets.
- [ ] Confirmar que o artifact estatistico nao inclui avatar/profile Steam.

Observacao operacional:

`/tmp` foi aceito apenas para validacao local controlada. Ele nao deve ser tratado como root definitivo de producao.

O runbook antigo cita `/var/tmp`, mas o gerador experimental atual bloqueia paths dentro de `/var`. Antes de producao real, definir um root persistente controlado fora de `/var/www` e compativel com o guardrail do ETL, ou corrigir/documentar o guardrail.

## 6. Plano de deploy - Auth API

Configurar a Auth API com as envs necessarias para Player Auth, leitura do artifact e redirects do Portal.

Checklist:

- [ ] Configurar `PLAYER_BUNKER_ARTIFACT_ROOT`.
- [ ] Configurar `PLAYER_BUNKER_ACTIVE_SEASON_SLUG`.
- [ ] Configurar `PLAYER_STEAM_AUTH_ENABLED`.
- [ ] Configurar `PLAYER_STEAM_RETURN_URL`.
- [ ] Configurar `PLAYER_STEAM_REALM`.
- [ ] Configurar `PLAYER_AUTH_CALLBACK_REDIRECT_ENABLED`.
- [ ] Configurar `PLAYER_AUTH_SUCCESS_REDIRECT_URL`.
- [ ] Configurar `PLAYER_AUTH_FAILURE_REDIRECT_URL`.
- [ ] Reiniciar o servico controladamente.
- [ ] Validar `/health`.
- [ ] Validar que `GET /player/auth/steam/start` retorna redirect Steam.
- [ ] Validar callback Steam real.
- [ ] Validar `GET /player/me` autenticado.
- [ ] Validar `GET /player/bunker/summary` autenticado com `statsAvailable: true`.

Endpoints principais:

- `GET /player/me`
- `GET /player/bunker/summary`
- `POST /player/auth/logout`
- `GET /player/auth/steam/start`
- `GET /player/auth/steam/callback`

## 7. Plano de deploy - Portal

Publicar o build estatico do Portal para `/portal/cs2-next/`.

Checklist:

- [ ] Build Angular com base href `/portal/cs2-next/`.
- [ ] Publicar somente `dist/browser`.
- [ ] Nunca executar `git pull` dentro de webroot.
- [ ] Validar assets estaticos.
- [ ] Validar rotas do Portal.
- [ ] Validar `/portal/cs2-next/bunker`.
- [ ] Validar que `/player/*` funciona no mesmo host via reverse proxy.

## 8. Validacao pos-deploy

Checklist:

- [ ] `/health` Auth API ok.
- [ ] `GET /player/auth/steam/start` redireciona.
- [ ] Login Steam real conclui.
- [ ] `GET /player/me` retorna HTTP 200 com `authenticated: true`.
- [ ] `GET /player/bunker/summary` retorna HTTP 200 com `statsAvailable: true`.
- [ ] `/portal/cs2-next/bunker` renderiza Player Hero.
- [ ] Season Hero renderiza.
- [ ] Overview/rings renderiza.
- [ ] Performance por mapa renderiza.
- [ ] Ultimos mapas renderiza.
- [ ] Timeline renderiza.
- [ ] Logout funciona.
- [ ] Console sem erro relevante.

Validacoes recentes que embasam este plano:

- `/bunker` local aprovado visualmente com Player Hero, Season Hero, overview/rings, `byMap`, `recentMaps` e timeline.
- Steam real E2E local aprovado com redirect final para `/bunker`, `GET /player/me` autenticado e SteamID real exibido.
- Artifact staging local gerado com MatchZy DB real local:
  - root local: `/tmp/hsc-cs2-etl-build/season-player`;
  - season: `staging-s01-2026`;
  - players discovered/written: 26;
  - player real `76561198104061526` / `14vosx`;
  - `mapsPlayed`: 11;
  - `byMapCount`: 5;
  - `recentMapsCount`: 11;
  - `timelineCount`: 11.
- Auth API apontada para artifact staging local:
  - `PLAYER_BUNKER_ARTIFACT_ROOT=/tmp/hsc-cs2-etl-build/season-player`;
  - `PLAYER_BUNKER_ACTIVE_SEASON_SLUG=staging-s01-2026`;
  - `GET /player/bunker/summary` autenticado retornou `statsAvailable: true`.
- Portal build estatico validado sem Angular dev proxy:
  - base href `/portal/cs2-next/`;
  - servidor estatico local em `localhost:4300`;
  - reverse proxy `/player/*` para Auth API `localhost:3010`;
  - `GET /player/bunker/summary` via proxy retornou HTTP 200;
  - `/bunker` renderizou dados reais.

Essas evidencias sao historico de validacao local e nao substituem a validacao pos-deploy no host de publicacao.

## 9. Rollback - Portal

Se o problema estiver restrito ao Portal:

- [ ] Restaurar release/symlink anterior ou assets anteriores.
- [ ] Nao mexer no artifact.
- [ ] Nao mexer na DB.
- [ ] Validar a rota anterior.

## 10. Rollback - Auth API

Se o problema estiver na Auth API, Player Auth, envs ou proxy:

- [ ] Reverter envs da API ou release da API.
- [ ] Se Steam Auth quebrar, desabilitar `PLAYER_STEAM_AUTH_ENABLED` ou reverter redirect/proxy conforme plano preparado.
- [ ] Validar `/health`.
- [ ] Validar que endpoints publicos nao foram afetados.

## 11. Rollback - artifact

Se o problema estiver no artifact ou na season ativa:

- [ ] Trocar `PLAYER_BUNKER_ACTIVE_SEASON_SLUG` para uma season anterior valida, se existir.
- [ ] Ou apontar `PLAYER_BUNKER_ARTIFACT_ROOT` para snapshot anterior.
- [ ] Ou remover envs do Bunker para fallback seguro com `statsAvailable: false`.
- [ ] Nao apagar o artifact atual antes de backup.

## 12. Evidencias permitidas

Podem ser registradas:

- HTTP status;
- nomes de endpoints;
- counts agregados;
- commit/PR IDs;
- artifact slug;
- root sem secrets;
- prints sem DevTools/cookies;
- summaries `jq` sem cookie/token.

## 13. Evidencias proibidas

Nao registrar:

- `Cookie`;
- `Set-Cookie`;
- token;
- `.env` real;
- callback Steam completo;
- headers sensiveis;
- screenshots DevTools com sessao;
- DB dumps;
- secrets;
- API keys.

## 14. Criterio final para publicar

Publicar somente quando:

- [ ] Todos os itens go/no-go estiverem aprovados.
- [ ] Rollback tiver sido testado mentalmente e comandos estiverem preparados.
- [ ] Responsavel humano confirmar o deploy.
- [ ] Publicacao ocorrer em janela controlada.

## 15. Pendencias conhecidas pos-MVP

- Definir root persistente final do artifact.
- Corrigir/documentar inconsistencia `/var/tmp` vs guardrail do gerador.
- Cron/timer ETL.
- `gen-all-v2`, se decidido.
- Componentizacao Bunker.
- Avatar Steam/ranking/elegibilidade quando contrato existir.
