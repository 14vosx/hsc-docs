# Player Bunker Local Integration Validation

Status: checkpoint de validação local

Escopo: validação local integrada do Player Bunker entre ETL, Auth API e Portal

Data base: 2026-05-11

Contexto: validação local após as frentes cross-repo:

- hsc-cs2-portal #33-#39
- hsc-auth-api #77
- hsc-cs2-etl #44

Repositórios envolvidos:

- hsc-cs2-etl
- hsc-auth-api
- hsc-cs2-portal

## 1. Objetivo

Registrar o checkpoint factual da validação local integrada do Player Bunker.

Este documento não autoriza deploy, staging, alteração de contrato, mudança de arquitetura ou operação em infraestrutura.

## 2. Arquitetura validada localmente

A validação reforçou o fluxo arquitetural esperado para o Bunker:

1. ETL gera artifact persistente com dados materializados do jogador.
2. Auth API lê o artifact em modo read-only, configurado por variáveis de ambiente.
3. Portal consome a Auth API para renderizar a experiência autenticada do Bunker.

Separação de responsabilidades:

- hsc-cs2-etl é dono da geração/materialização do artifact.
- hsc-auth-api atua como camada autenticada e gateway de leitura.
- hsc-cs2-portal renderiza a UI do Bunker a partir da Auth API.

## 3. Artifact local validado

Artifact local:

```text
root: /tmp/hsc-bunker-integrated-season-player
season: fixture-s02
players gerados: 3
player validado: 76561198000000001
```

O root validado fica em `/tmp`, não em `/var/www`.

## 4. Payload validado

Payload retornado para o jogador validado:

```text
statsAvailable: true
summaryMaps: 90
summaryWins: 45
byMapCount: 6
recentMapsCount: 20
timelineCount: 60
```

## 5. Validação do Portal

O Portal renderizou visualmente a rota `/bunker` com:

- jogador autenticado;
- summary cards;
- `byMap` com 6 mapas;
- `recentMaps` com 5 mapas recentes.

A timeline estava presente na resposta da Auth API, com `timelineCount: 60`, mas ainda não foi renderizada na UI do Portal.

## 6. Caminho de teste executado

A validação local passou por:

1. ETL gerando artifact persistente local.
2. Auth API local configurada com:
   - `PLAYER_BUNKER_ARTIFACT_ROOT`
   - `PLAYER_BUNKER_ACTIVE_SEASON_SLUG`
3. `GET /player/bunker/summary` retornando HTTP 200.
4. Angular dev proxy retornando HTTP 200.
5. Portal `/bunker` renderizando a UI.

## 7. Segurança e higiene de evidências

Não versionar prints ou evidências que contenham cookies, headers, sessões ou identificadores sensíveis.

Não colar em documentação:

- `Cookie`
- `Set-Cookie`
- token
- `.env` real
- callback Steam real

O artifact root desta validação é local e temporário em `/tmp`. Ele não deve ser tratado como caminho de produção, staging ou webroot.

## 8. Limitações

Esta validação cobre apenas ambiente local.

Ficaram fora do escopo:

- deploy, staging ou produção;
- Steam real end-to-end;
- cron/timer do ETL;
- `gen-all-v2`;
- renderização da timeline no Portal;
- componentização futura do Bunker para reduzir o CSS budget.

## 9. Próximas frentes sugeridas

Frentes candidatas para evolução posterior:

- timeline UI;
- staging controlado do artifact;
- Steam real end-to-end;
- componentização do Bunker.
