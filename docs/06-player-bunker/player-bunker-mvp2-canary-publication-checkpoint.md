# Player Bunker MVP2 Canary Publication Checkpoint

## Contexto

Este documento registra o checkpoint factual da publicação controlada do **Bunker — MVP2 / Bunker Melhorado** no canary público do Portal CS2 Next.

Este checkpoint não representa cutoff, não promove `/portal/cs2-next` para portal oficial e não altera o status de `/portal/cs2` como portal legado/oficial atual.

## Escopo publicado

Publicação controlada envolvendo:

- `hsc-cs2-portal` em `/portal/cs2-next`;
- `hsc-auth-api` em `auth-api.haxixesmokeclub.com`;
- validação real com sessão Steam do player `14vosx`;
- preservação de `/portal/cs2`.

Fora de escopo neste checkpoint:

- cutoff para `/portal/cs2`;
- billing;
- subdomínio Bunker;
- Angular Material migration;
- novo serviço backend;
- alteração de fórmula de ranking, score ou membership de Season;
- alteração em Admin Auth;
- alteração de Nginx, DNS ou TLS.

## Repositórios envolvidos

### `hsc-cs2-portal`

PR publicado no canary:

```text
#44 feat(player-bunker): display competitive profile
```

Commit em `main` usado como fonte do build:

```text
92a0e2d feat(player-bunker): display competitive profile (#44)
```

Responsabilidade publicada:

- consumir `competitiveProfile` de `/player/bunker/summary`;
- renderizar `avatarMedium` quando disponível;
- exibir link `Ver perfil Steam` quando `steamProfileUrl` existir;
- exibir seção `Perfil competitivo lifetime`;
- manter Season e lifetime separados;
- preservar leitura de `seasonPlayer.summary`, `byMap`, `recentMaps` e `timeline`;
- não recalcular ranking, score, elegibilidade ou membership de Season.

### `hsc-auth-api`

PR publicado:

```text
#79 feat(player-bunker): add optional competitive profile enrichment
```

Commit publicado:

```text
c65a5ee feat(player-bunker): add optional competitive profile enrichment (#79)
```

Tag publicada:

```text
auth-api-bunker-mvp2-20260514T142700Z
```

Responsabilidade publicada:

- manter Auth API como gateway autenticado;
- preservar `/player/bunker/summary` retrocompatível;
- retornar `seasonPlayer` a partir do artifact Season/player;
- enriquecer com `competitiveProfile` opcional vindo da Static API v2;
- preencher `data.player.avatarMedium` quando disponível;
- preencher `data.player.steamProfileUrl` quando disponível;
- manter `data.lifetime = null` no nível legado;
- usar fallback seguro se o enriquecimento opcional falhar.

## Publicação do Portal canary

Destino publicado:

```text
/var/www/portal/cs2-next
```

URL pública validada:

```text
https://haxixesmokeclub.com/portal/cs2-next/
https://haxixesmokeclub.com/portal/cs2-next/bunker
```

Pacote local validado antes da publicação:

```text
/tmp/hsc-cs2-next-bunker-mvp2-20260514T135231Z.tar.gz
```

Hash SHA-256 validado:

```text
584dd6f73647e403b2415ec93f0be181a1c92d71545836533e749765e0d16043
```

Build canary gerado com:

```text
npx ng build hsc-cs2-portal-angular --base-href=/portal/cs2-next/
```

O pacote publicado contém somente o conteúdo de:

```text
frontend/angular/dist/hsc-cs2-portal-angular/browser/*
```

`index.html` publicado confirmou:

```text
<base href="/portal/cs2-next/">
```

Bundle publicado observado:

```text
main-CASCS5VU.js
chunk-4TGVBBAY.js
```

Arquivos proibidos não foram encontrados no pacote/staging/webroot:

```text
.git
.github
.env
.npmrc
node_modules
src
package.json
package-lock.json
angular.json
tsconfig*.json
```

Smoke HTTP público pós-publicação:

```text
/portal/cs2-next/        HTTP 200
/portal/cs2-next/bunker  HTTP 200
/portal/cs2/             HTTP 200
```

O legado `/portal/cs2` não foi sobrescrito e não continha os marcadores do bundle novo de `/portal/cs2-next`.

## Publicação da Auth API

Host validado:

```text
ip-172-26-2-109
```

Serviço systemd:

```text
hsc-auth-api.service
```

Diretório da aplicação:

```text
/opt/hsc/hsc-auth-api
```

Tag de deploy usada:

```text
auth-api-bunker-mvp2-20260514T142700Z
```

Deploy executado via:

```text
/opt/hsc/hsc-auth-api/ops/deploy-auth.sh auth-api-bunker-mvp2-20260514T142700Z
```

Resultado observado:

```text
HEAD em c65a5ee
service active
No pending migrations
health OK
content/news smoke OK
content/seasons smoke OK
content/seasons/active smoke OK
admin/schema smoke OK
```

Configurações não sensíveis adicionadas ao drop-in `player-bunker.conf`:

```text
PLAYER_BUNKER_STATIC_API_BASE_URL=https://haxixesmokeclub.com/api/cs2/v2
PLAYER_BUNKER_STATIC_API_TIMEOUT_MS=1500
```

Configurações já existentes preservadas:

```text
PLAYER_BUNKER_ARTIFACT_ROOT=/srv/hsc/player-bunker-artifacts
PLAYER_BUNKER_ACTIVE_SEASON_SLUG=s01-2026
```

Health validado:

```text
http://127.0.0.1:3000/health
https://auth-api.haxixesmokeclub.com/health
```

Ambos retornaram `ok: true` e `db.ready: true`.

## Validação real end-to-end

Player validado:

```text
14vosx
76561198104061526
```

URL validada no navegador:

```text
https://haxixesmokeclub.com/portal/cs2-next/bunker
```

A tela renderizou:

- sessão conectada;
- avatar Steam real;
- link `Ver perfil Steam`;
- Season atual `s01-2026`;
- seção `Perfil competitivo lifetime`;
- seção `Visão geral da temporada`;
- seção `Performance por mapa`;
- seção `Últimos mapas jogados`;
- seção `Timeline da temporada`.

Contrato observado em `GET /player/bunker/summary`:

```text
ok: true
bunker.status: ready
bunker.seasonFirst: true
bunker.statsAvailable: true
currentSeason.slug: s01-2026
player.steamid64: 76561198104061526
player.avatarMedium: presente
player.steamProfileUrl: presente
seasonPlayer: presente
competitiveProfile: presente
competitiveProfile.lifetime: presente
```

Notas observadas:

```text
real_player_identity_connected
season_player_artifact_connected
competitive_profile_connected
```

Campos relevantes confirmados:

```text
data.player.avatarMedium
data.player.steamProfileUrl
data.seasonPlayer.summary
data.seasonPlayer.byMap
data.seasonPlayer.recentMaps
data.seasonPlayer.timeline
data.competitiveProfile.lifetime
```

## Rollback disponível

### Portal canary

Backup criado antes da publicação:

```text
/root/hsc-cs2-next-before-bunker-mvp2-20260514T140930Z.tar.gz
```

Escopo do rollback do Portal:

```text
/var/www/portal/cs2-next
```

Não envolve `/portal/cs2`.

### Auth API

Tag de rollback criada:

```text
auth-api-before-bunker-mvp2-20260514T142700Z
```

Commit de rollback:

```text
dbae37a docs(player-bunker): document artifact env runbook (#77)
```

State file do deploy registrou a tag anterior:

```text
/opt/hsc/.deploy-auth-last-tag
```

## Incidentes operacionais observados

Durante a publicação da Auth API, foram encontrados e resolvidos os seguintes pontos operacionais:

1. Lock stale em `/tmp/hsc-auth-deploy.lock` bloqueou o deploy inicialmente.
2. Git recusou execução como root por `safe.directory`.
3. Root não tinha chave SSH própria para GitHub.
4. O usuário `hscadmin` tinha chave GitHub funcional, mas não tinha sudo sem senha para concluir o deploy.
5. O deploy foi executado como root usando `GIT_SSH_COMMAND` apontando para a chave SSH do `hscadmin`.

Esses pontos não geraram rollback e não quebraram o serviço. Antes do deploy efetivo, o serviço permaneceu ativo e os health checks continuaram OK.

## Estado final

Estado após a publicação:

```text
hsc-cs2-portal
  /portal/cs2-next publicado com Bunker MVP2

hsc-auth-api
  c65a5ee publicado
  auth-api-bunker-mvp2-20260514T142700Z ativo

/portal/cs2
  preservado

cutoff
  não realizado

billing
  não iniciado

subdomínio Bunker
  não criado
```

## Próximos passos recomendados

1. Registrar este checkpoint em `hsc-docs`.
2. Considerar documentação operacional futura para o deploy da Auth API com usuário `hscadmin`/root e `GIT_SSH_COMMAND`.
3. Planejar uma frente separada de limpeza do CSS budget do Bunker.
4. Avaliar UX incremental do Bunker MVP2 sem mexer no contrato canônico.
5. Não fazer cutoff para `/portal/cs2` sem decisão explícita.
