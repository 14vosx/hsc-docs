# Player Bunker MVP2 Enrichment Plan

Status: plano de escopo aprovado para proxima iteracao

Escopo: enriquecer o Player Bunker usando dados reais ja disponiveis no ecossistema HSC

Este documento nao autoriza cutoff de `/portal/cs2`, billing, subdominio, migracao para Angular Material ou novo servico.

## 1. Objetivo

O objetivo do MVP2 e transformar o Bunker publicado em `/portal/cs2-next/bunker` em uma experiencia mais rica, clara e util para o jogador autenticado.

A meta nao e criar uma nova feature grande. A meta e usar melhor os dados que ja existem hoje:

- identidade Steam;
- profile publico de player na Static API v2;
- artifact Season/player consumido pela Auth API;
- sessao Steam-first ja validada;
- UI Bunker ja publicada em canary.

## 2. Estado atual validado

O MVP atual ja validou o fluxo principal:

1. Portal Angular publicado em `/portal/cs2-next/`.
2. Rota `/portal/cs2-next/bunker` publicada.
3. Nginx Hostinger faz reverse proxy de `/player/*` para a Auth API na AWS.
4. Auth API emite sessao player via Steam.
5. Auth API le artifact Season/player em root persistente.
6. `GET /player/bunker/summary` autenticado retorna `statsAvailable: true`.
7. Portal renderiza Player Hero, Season Hero, overview/rings, `byMap`, `recentMaps` e `timeline`.
8. `/portal/cs2` continua preservado como portal oficial atual.
9. Nao houve cutoff.

## 3. Decisoes congeladas para o MVP2

Para esta iteracao:

- Nao fazer cutoff de `/portal/cs2-next` para `/portal/cs2`.
- Nao criar subdominio `bunker`.
- Nao iniciar billing, assinatura ou premium.
- Nao migrar para Angular Material.
- Nao criar novo backend.
- Nao duplicar formula de ranking.
- Nao calcular estatisticas competitivas na Auth API.
- Nao expor artifact do ETL no webroot.
- Nao fazer o Portal logado consumir diretamente o artifact do ETL.
- Manter Auth API como gateway autenticado do Bunker.
- Manter ETL como dono das stats competitivas materializadas.
- Manter Portal como camada de apresentacao.

## 4. Principio arquitetural

Fluxo alvo do MVP2:

```text
Portal Bunker
  -> /player/me
  -> /player/bunker/summary
       Auth API
         -> sessao player
         -> SteamID64 autorizado
         -> Steam profile/cache quando disponivel
         -> Season/player artifact
         -> Static API v2 player profile, se configurado/necessario
```

O Portal deve continuar chamando somente `/player/*` para dados autenticados do Bunker.

A Auth API pode atuar como agregador/gateway defensivo de fontes ja existentes, desde que nao calcule stats competitivas complexas e nao crie nova fonte de verdade.

## 5. Fontes de dados existentes

### 5.1 Auth API / Player Auth

Fonte de identidade autenticada:

* sessao player;
* SteamID64 autenticado;
* conta player;
* logout;
* Steam callback;
* cache Steam Profiles quando disponivel.

Endpoints principais:

* `GET /player/me`
* `GET /player/bunker/summary`
* `POST /player/auth/logout`
* `GET /player/auth/steam/start`
* `GET /player/auth/steam/callback`

### 5.2 Season/player artifact

Fonte autenticada de stats no recorte da Season ativa.

Layout esperado:

```text
<root>/season/<slug>/players-manifest.json
<root>/season/<slug>/player/<steamid64>.json
```

Campos do artifact Season/player:

* `generatedAt`
* `season`
* `steamid64`
* `name`
* `summary`
* `periods`
* `byMap`
* `recentMaps`
* `timeline`

O objeto `summary` representa o recorte da Season. Ele nao deve ser tratado como lifetime.

Campos que nao devem ser adicionados ao artifact Season/player:

* `lifetime`
* `avatarMedium`
* `steamProfileUrl`
* dados de sessao;
* dados de Auth API;
* permissoes;
* billing;
* entitlements.

### 5.3 Static API v2 player profile

Endpoint publico existente:

```text
/api/cs2/v2/player/{steamid64}.json
```

Shape disponivel observado:

* `generatedAt`
* `steamid64`
* `name`
* `avatarMedium`
* `steamProfileUrl`
* `lifetime`
* `periods`
* `byMap`
* `recentMaps`
* `timeline`

Esse endpoint e uma fonte forte para enriquecer o Bunker, principalmente para:

* avatar Steam real;
* link de perfil Steam;
* resumo lifetime;
* metricas avancadas;
* historico recente;
* timeline;
* performance por mapa.

## 6. Problema atual da UI

A UI atual do Bunker ja funciona, mas ainda subutiliza os dados existentes.

Pontos a melhorar:

* Player Hero ainda pode depender de monograma/fallback.
* Avatar Steam real ainda nao esta consolidado na experiencia.
* Perfil Steam ainda nao aparece como link claro.
* O usuario nao ve `generatedAt` de forma forte.
* Metricas como HS%, accuracy, utility, entry, clutch e multikill ainda nao aparecem.
* `recentMaps` poderia contar melhor a historia de cada mapa.
* `timeline` poderia funcionar como grafico/tendencia, nao apenas lista.
* A pagina ainda precisa diferenciar melhor:

  * identidade do jogador;
  * resumo da Season;
  * performance lifetime;
  * mapa recente;
  * timeline.

## 7. Contrato recomendado para o MVP2

Manter o endpoint:

```text
GET /player/bunker/summary
```

Evoluir a resposta de forma retrocompativel.

Shape recomendado:

```json
{
  "ok": true,
  "data": {
    "player": {
      "playerAccountId": "...",
      "steamid64": "...",
      "displayName": "...",
      "avatarMedium": "...",
      "steamProfileUrl": "..."
    },
    "bunker": {
      "status": "ready",
      "seasonFirst": true,
      "statsAvailable": true
    },
    "currentSeason": {
      "slug": "s01-2026",
      "scope": {
        "startAt": "...",
        "endAt": "..."
      }
    },
    "seasonPlayer": {
      "generatedAt": "...",
      "season": {},
      "steamid64": "...",
      "name": "...",
      "summary": {},
      "periods": {},
      "byMap": [],
      "recentMaps": [],
      "timeline": []
    },
    "competitiveProfile": {
      "generatedAt": "...",
      "steamid64": "...",
      "name": "...",
      "avatarMedium": "...",
      "steamProfileUrl": "...",
      "lifetime": {},
      "periods": {},
      "byMap": [],
      "recentMaps": [],
      "timeline": []
    }
  }
}
```

Notas:

* `seasonPlayer` continua representando recorte da Season ativa.
* `competitiveProfile` representa o profile publico/lifetime disponivel hoje.
* `player` representa identidade autenticada.
* `avatarMedium` e `steamProfileUrl` podem vir do Steam Profiles cache ou do profile publico, mas nao devem ser adicionados ao artifact Season/player.
* Se `competitiveProfile` estiver indisponivel, o Bunker ainda deve funcionar com `seasonPlayer`.
* Se `seasonPlayer` estiver indisponivel, o Bunker deve cair para `statsAvailable: false` sem quebrar a sessao.

## 8. Derivacoes permitidas no frontend

O Portal pode calcular apenas metricas simples de apresentacao, sem alterar formula competitiva canonica.

Permitido:

* K/D em um item de `recentMaps`: `kills / deaths`;
* ADR em um item de `recentMaps`: `damage / rounds`;
* accuracy em um item de `recentMaps`: `shots_on_target_total / shots_fired_total`;
* HS% em um item de `recentMaps`: `head_shot_kills / kills`;
* label win/loss a partir de `isWin`;
* score visual a partir de `team1_score` e `team2_score`;
* melhor mapa por ADR;
* mapa mais jogado;
* melhor mapa por winRate com volume minimo;
* melhor mapa recente por `impactRating`;
* maior dano utilitario recente;
* tendencia simples dos ultimos N mapas contra media geral.

Nao permitido nesta fase:

* recalcular ranking;
* recalcular score competitivo canonico;
* recalcular elegibilidade de premiacao;
* inferir Season membership;
* criar estatistica nao presente nem derivavel diretamente;
* fazer N+1 para endpoints de detalhe sem aprovacao.

## 9. Enriquecimentos de UI esperados

### 9.1 Player Hero melhorado

Adicionar:

- avatar Steam real;
- nome real do player;
- SteamID64 em label menor;
- link para Steam profile;
- data de atualizacao dos dados;
- botao de logout;
- estado de sessao;
- fallback visual quando avatar estiver ausente.

Fora do escopo imediato:

- data real de criacao da conta HSC, se ainda nao existir no contrato;
- ranking geral;
- ranking da Season;
- elegibilidade de premiacao.

Esses itens so entram se o contrato fornecer os campos de forma canonica.

### 9.2 Season Overview Hero

Manter a separacao:

- Season atual;
- slug;
- periodo;
- status;
- disponibilidade de stats;
- mensagem clara sobre recorte de Season.

Nao misturar lifetime com Season sem label explicito.

### 9.3 Competitive Summary

Adicionar cards:

- K/D;
- ADR;
- Win rate;
- Impact;
- HS%;
- Accuracy;
- Utility damage per round;
- Entry win rate;
- kills/round;
- assists/round;
- deaths/round;
- 1v1 win rate;
- 1v2 win rate;
- 2K/3K/4K/5K;
- score;
- sampleWeight.

### 9.4 Performance por mapa

Expandir `byMap` para exibir:

- mapa;
- jogos;
- rounds;
- win rate;
- K/D;
- ADR;
- HS%;
- accuracy;
- utility damage per round;
- entry win rate;
- multikills.

Adicionar destaque automatico:

- mapa mais jogado;
- melhor mapa por ADR;
- melhor mapa por winRate com volume minimo;
- mapa de atencao para treino.

### 9.5 Ultimos mapas jogados

Transformar cada item de `recentMaps` em card narrativo:

- mapa;
- data;
- match id;
- resultado win/loss;
- score;
- K/A/D;
- ADR derivado;
- HS kills;
- utility damage;
- entry count/wins;
- clutch 1v1/1v2;
- multikills;
- impact;
- link futuro para match detail quando a rota/contrato estiver estavel.

### 9.6 Timeline

Melhorar leitura da timeline:

- ordenar cronologicamente;
- manter destaque por mapa;
- exibir ADR, K/D e impact;
- permitir sparkline simples sem biblioteca pesada;
- destacar melhor e pior ponto recente;
- usar CSS/SVG simples antes de introduzir lib de graficos.

## 10. Impacto por repositorio

### 10.1 hsc-docs

Primeira entrega.

Criar este plano e registrar o escopo MVP2.

### 10.2 hsc-auth-api

Escopo provavel:

- preservar contrato atual de `/player/bunker/summary`;
- enriquecer resposta com `player.avatarMedium` e `player.steamProfileUrl`, quando disponiveis;
- avaliar fonte de `competitiveProfile`;
- se usar Static API v2 server-side, criar config explicita e timeout curto;
- falhar aberto: se profile publico estiver indisponivel, manter `seasonPlayer` funcionando;
- sanitizar qualquer payload retornado;
- nao expor cookie/token/hash;
- nao calcular stats competitivas complexas.

Possivel env futura:

```text
PLAYER_BUNKER_STATIC_API_BASE_URL=https://haxixesmokeclub.com/api/cs2/v2
PLAYER_BUNKER_STATIC_API_TIMEOUT_SECONDS=5
```

### 10.3 hsc-cs2-portal

Escopo provavel:

* expandir DTOs do Bunker;
* preservar normalizacao defensiva;
* adicionar helpers de derivacao simples;
* reorganizar visualmente Player Hero, Season Hero, summary, map performance, recent maps e timeline;
* evitar aumento descontrolado do CSS de `bunker-page.css`;
* manter build passando;
* aceitar warning de budget apenas se ainda estiver dentro da politica temporaria ja conhecida.

### 10.4 hsc-cs2-etl

Escopo provavel:

* nao alterar artifact Season/player para carregar `lifetime`, `avatarMedium` ou `steamProfileUrl`;
* manter contrato Season/player limpo;
* se necessario, revisar se o profile publico `/api/cs2/v2/player/{steamid64}.json` esta completo e atualizado;
* nao integrar ao `gen-all-v2` neste MVP2, salvo decisao separada.

### 10.5 hsc-backoffice-admin

Fora do escopo.

### 10.6 hsc-brand-hub

Fora do escopo.

## 11. Ordem recomendada de PRs

### PR 1 - docs

Repositorio: `hsc-docs`

Objetivo:

* registrar plano MVP2;
* congelar escopo;
* definir fronteiras;
* registrar campos permitidos;
* registrar fora de escopo.

### PR 2 - Auth API contract enrichment

Repositorio: `hsc-auth-api`

Objetivo:

* enriquecer `/player/bunker/summary`;
* preservar retrocompatibilidade;
* adicionar testes/smoke;
* manter Auth API como gateway;
* nao calcular stats.

Validacoes:

```bash
npm test
npm run db:migrate
node/ops smoke local
curl /health
curl /player/bunker/summary autenticado
```

### PR 3 - Portal UI MVP2

Repositorio: `hsc-cs2-portal`

Objetivo:

* consumir novo contrato defensivamente;
* enriquecer Hero;
* enriquecer cards;
* melhorar recent maps;
* melhorar timeline;
* manter `/portal/cs2-next` como canary.

Validacoes:

```bash
cd frontend/angular
npm run build
git diff --check
```

### PR 4 - canary publish checkpoint

Repositorio: `hsc-docs`

Objetivo:

* registrar validacao canary;
* registrar screenshots seguros;
* registrar HTTP status;
* registrar rollback;
* nao fazer cutoff.

## 12. Riscos

### 12.1 Misturar lifetime com Season

Risco:

* UI sugerir que dados lifetime sao dados da Season.

Mitigacao:

* usar labels explicitas:

  * "Resumo da Season";
  * "Historico competitivo";
  * "Lifetime";
  * "Ultimos mapas".

### 12.2 Contaminar artifact Season/player

Risco:

* adicionar avatar/profile/lifetime ao artifact e quebrar contrato.

Mitigacao:

* manter artifact Season/player focado em stats do recorte;
* tratar identidade Steam e lifetime em outra camada.

### 12.3 Duplicar calculo competitivo

Risco:

* frontend ou Auth API virar segunda fonte de verdade.

Mitigacao:

* permitir apenas derivacoes simples por item;
* ranking, score canonico e elegibilidade ficam fora do MVP2.

### 12.4 Aumentar CSS e complexidade visual

Risco:

* `bunker-page.css` crescer de novo e transformar warning em erro.

Mitigacao:

* manter UI enxuta;
* criar helpers pequenos;
* evitar bibliotecas de grafico nesta etapa;
* considerar componentizacao somente em fase posterior se necessario.

### 12.5 Dependencia da Static API v2 pela Auth API

Risco:

* Auth API depender de HTTP externo para montar Bunker.

Mitigacao:

* timeout curto;
* fallback sem quebrar sessao;
* cache futuro se necessario;
* documentar indisponibilidade parcial.

## 13. Fora de escopo

* Cutoff de `/portal/cs2-next` para `/portal/cs2`.
* Billing.
* Assinaturas.
* Entitlements.
* Subdominio Bunker.
* Angular Material.
* Backoffice de usuarios player.
* Ranking geral do jogador, salvo contrato canonico existente.
* Ranking da Season do jogador, salvo contrato canonico existente.
* Elegibilidade de premiacao, salvo contrato canonico existente.
* Novo servico.
* Novo banco para stats.
* Recalculo de ranking ou score no frontend.
* Publicacao definitiva do artifact via `gen-all-v2`.

## 14. Criterios de aceite do MVP2

O MVP2 sera considerado pronto quando:

* `/portal/cs2-next/bunker` continuar funcionando com Steam real.
* `/player/me` continuar autenticado.
* `/player/bunker/summary` retornar dados enriquecidos de forma retrocompativel.
* Player Hero exibir avatar Steam quando disponivel.
* Player Hero exibir link de perfil Steam quando disponivel.
* UI diferenciar Season e Lifetime.
* Summary exibir metricas adicionais relevantes.
* Recent maps mostrarem score, resultado, KDA e metricas derivadas simples.
* Timeline tiver leitura visual melhor.
* Build do Portal passar.
* Auth API passar smokes.
* Sem secrets/cookies em logs ou docs.
* `/portal/cs2` permanecer intocado.

## 15. Proxima acao apos este documento

Apos merge deste documento:

1. Abrir frente no `hsc-auth-api` para auditar e enriquecer `/player/bunker/summary`.
2. Confirmar a melhor fonte para `avatarMedium` e `steamProfileUrl`:

   * Steam Profiles cache;
   * Static API v2 player profile;
   * ambos com precedencia documentada.
3. Definir contrato final do payload enriquecido.
4. Implementar smoke local.
5. So depois abrir frente no Portal.

## 16. Prompt operacional recomendado para o proximo bloco

Objetivo:

Enriquecer o contrato autenticado do Player Bunker no `hsc-auth-api`, preservando Auth API como gateway e sem calcular stats competitivas.

Regras:

* ler `AGENTS.md`;
* validar estado Git;
* trabalhar em micro-passos;
* nao alterar admin auth;
* nao expor secrets;
* nao mexer em deploy;
* nao alterar `/portal/cs2`;
* nao fazer cutoff;
* nao iniciar billing;
* nao migrar Angular Material;
* manter `/player/bunker/summary` retrocompativel;
* adicionar fallback seguro se a fonte adicional estiver indisponivel;
* adicionar smoke/teste pequeno;
* commitar em PR pequeno.

Foco tecnico:

* resolver SteamID64 autenticado;
* ler seasonPlayer artifact como hoje;
* enriquecer identidade com avatar/profile quando disponivel;
* opcionalmente buscar profile publico em Static API v2 por config;
* retornar `competitiveProfile` sem calcular stats;
* sanitizar resposta;
* validar HTTP 200 autenticado.
