# Player Bunker MVP2 Local Validation Checkpoint

## Contexto

Este documento registra o checkpoint factual da validação local do **Bunker — MVP2 / Bunker Melhorado**.

Este checkpoint não autoriza deploy, publicação em canary, rollback, cutoff, mudança de Nginx, mudança de systemd, alteração de `/var/www`, billing, subdomínio, nem migração para Angular Material.

## Escopo validado

Validação local integrada envolvendo:

- `hsc-auth-api`;
- `hsc-cs2-portal`;
- MariaDB local;
- artifact local Season/player;
- mock local da Static API v2;
- Angular local com `npm run start:proxy`.

## Repositórios envolvidos

### `hsc-auth-api`

PR validado:

```text
#79 feat(player-bunker): add optional competitive profile enrichment
```

Commit em `main` observado:

```text
c65a5ee feat(player-bunker): add optional competitive profile enrichment (#79)
```

Responsabilidade validada:

- manter Auth API como gateway autenticado do Bunker;
- preservar `/player/bunker/summary` retrocompatível;
- ler `seasonPlayer` a partir de artifact Season/player;
- buscar `competitiveProfile` opcional na Static API v2;
- popular `player.avatarMedium` e `player.steamProfileUrl` quando a fonte segura existir;
- manter `seasonPlayer` separado de `competitiveProfile`;
- manter `lifetime: null` no nível legado por retrocompatibilidade;
- falhar de forma segura se `competitiveProfile` estiver indisponível.

### `hsc-cs2-portal`

PR validado:

```text
#44 feat(player-bunker): display competitive profile
```

Commit em `main` observado:

```text
92a0e2d feat(player-bunker): display competitive profile (#44)
```

Responsabilidade validada:

- consumir defensivamente o contrato enriquecido de `/player/bunker/summary`;
- renderizar avatar Steam quando `avatarMedium` existir;
- manter monograma local como fallback;
- exibir link `Ver perfil Steam` quando `steamProfileUrl` existir;
- exibir seção separada `Perfil competitivo lifetime`;
- manter dados de Season em `seasonPlayer`;
- não misturar lifetime com Season;
- não recalcular ranking, score, elegibilidade ou membership de Season.

## Ambiente local validado

Componentes locais usados:

```text
MariaDB local via Docker
Auth API local na porta 3010
Static API v2 mock local na porta 8080
Angular local com npm run start:proxy
```

Fluxo local validado:

```text
Angular /bunker
  -> proxy /player/* para Auth API local
  -> Auth API valida cookie player local
  -> Auth API lê artifact Season/player local
  -> Auth API busca Static API v2 mock local
  -> Auth API retorna /player/bunker/summary enriquecido
  -> Portal renderiza Bunker MVP2
```

## Dados locais de validação

SteamID64 fake/local usado:

```text
76561198000000000
```

Season local usada:

```text
smoke-season
```

Artifact local usado:

```text
/tmp/hsc-player-bunker-artifact-local
```

Static API mock local usado:

```text
http://127.0.0.1:8080/api/cs2/v2
```

Auth API local usada:

```text
http://127.0.0.1:3010
```

Angular local usado:

```text
http://localhost:4200/bunker
```

Nenhum cookie, token, hash, segredo, valor real de `.env` ou credencial foi registrado neste documento.

## Contrato observado

O endpoint autenticado:

```text
GET /player/bunker/summary
```

retornou `ok: true` e `data.bunker.statsAvailable: true`.

Campos relevantes observados:

```text
data.player.avatarMedium
data.player.steamProfileUrl
data.seasonPlayer.summary
data.seasonPlayer.byMap
data.seasonPlayer.recentMaps
data.seasonPlayer.timeline
data.competitiveProfile.lifetime
```

Notas observadas:

```text
real_player_identity_connected
season_player_artifact_connected
competitive_profile_connected
```

## UI observada

A tela local `/bunker` renderizou:

- jogador autenticado;
- avatar vindo de `avatarMedium`;
- nome do player;
- SteamID64;
- link `Ver perfil Steam`;
- seção `Temporada atual`;
- seção `Perfil competitivo lifetime`;
- seção `Visão geral da temporada`;
- seção `Performance por mapa`;
- seção `Últimos mapas jogados`;
- seção `Timeline da temporada`.

Após restart do `npm run start:proxy` e hard reload do navegador, a aparente duplicação visual em `Últimos mapas jogados` não persistiu. Nenhum patch adicional foi necessário.

## Validações executadas

### `hsc-auth-api`

Validações executadas durante a implementação do PR #79:

```text
node --check src/config/playerBunker.js
node --check src/services/player-bunker/competitiveProfile.js
node --check src/routes/player/bunker.summary.js
git diff --check
ops/player-bunker-artifact-summary-smoke.sh
```

Smoke local observado:

```text
Player Bunker artifact summary local smoke passed
```

### `hsc-cs2-portal`

Validações executadas durante a implementação do PR #44:

```text
npm run build
git diff --check
```

Build Angular observado como concluído com sucesso.

Warning conhecido observado:

```text
src/app/features/bunker/bunker-page.css exceeded maximum budget
```

Esse warning já é tratado como exceção temporária conhecida no contexto MVP/Canary do Bunker. Ele não bloqueou a validação local, mas deve ser reduzido em uma frente futura de limpeza de CSS.

## Estado após este checkpoint

Estado factual:

- MVP2 validado localmente;
- Auth API `main` contém o enriquecimento opcional de `competitiveProfile`;
- Portal `main` contém a UI consumindo `competitiveProfile`;
- `/portal/cs2` permanece preservado;
- não houve cutoff;
- não houve deploy canary neste checkpoint;
- `/portal/cs2-next` ainda precisa de publicação controlada futura para receber a UI atualizada;
- billing, premium gates, subdomínio Bunker e Angular Material continuam fora de escopo.

## Próximo movimento recomendado

Próximo passo recomendado, em frente separada e com aprovação explícita:

```text
Planejar deploy canary controlado do hsc-cs2-portal para /portal/cs2-next
```

Esse próximo passo deve preservar:

- build com base href `/portal/cs2-next/`;
- publicação somente de `dist/hsc-cs2-portal-angular/browser/*`;
- nenhum `git pull` em `/var/www`;
- nenhum cutoff para `/portal/cs2`;
- validação de `/portal/cs2-next/bunker`;
- rollback documentado antes de executar.
