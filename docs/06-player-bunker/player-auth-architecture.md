# HSC Player Auth Architecture — Bunker

Status: draft

Escopo: arquitetura de autenticação player-facing para o Bunker do usuário

Repositórios impactados: hsc-auth-api, hsc-cs2-portal, hsc-docs

Data base: 2026-05-08

## 1. Contexto

HSC está evoluindo para plataforma com Portal CS2, Static API v2, Seasons, Backoffice, Auth API, Steam Profiles e futura área logada chamada Bunker.

A auth atual do hsc-auth-api foi criada para o Backoffice, com magic link, sessão server-side e cookie administrativo.

Essa decisão foi correta no passado, mas não deve ser fundação do Bunker.

## 2. Decisão principal

Bunker deve ter Player Auth própria, separada da Admin Auth.

Não reutilizar diretamente:

- `users`
- `sessions`
- `magic_links`
- `hsc_admin_session`
- `adminAuth`
- `/auth/session`
- roles `viewer`, `editor`, `admin`

Novo domínio sugerido na API:

- `/player/auth/*`
- `/player/me`
- `/player/bunker/*`

No Portal, produto/rota pode ser Bunker:

- `/portal/cs2-next/bunker`

## 3. Magic link

Magic link permanece válido para o Backoffice.

Não deve ser base principal do player-facing.

O usuário natural do HSC é o jogador CS2 identificado por SteamID64.

## 4. Modelo recomendado

Provider inicial: Steam.

Provider futuro: Google/OIDC.

E-mail/magic link apenas fallback futuro.

SteamID64 é a identidade competitiva natural para ranking, seasons, partidas, mapas, histórico e avatar.

## 5. Modelo de identidade

Tabelas conceituais:

- `player_accounts`
- `player_identities`
- `player_steam_links`
- `player_sessions`

Campos sugeridos resumidos:

### `player_accounts`

- `id CHAR(36) PK`
- `email nullable unique`
- `display_name`
- `avatar_url`
- `status active/disabled/deleted`
- `created_at`
- `updated_at`
- `deleted_at`

### `player_identities`

- `id CHAR(36) PK`
- `player_account_id`
- `provider steam/google/email`
- `provider_subject`
- `provider_email`
- `provider_display_name`
- `provider_avatar_url`
- `created_at`
- `updated_at`
- `unique(provider, provider_subject)`

### `player_steam_links`

- `player_account_id PK`
- `steamid64 unique`
- `steam_persona_name`
- `steam_avatar_url`
- `steam_profile_url`
- `verified_at`
- `created_at`
- `updated_at`

### `player_sessions`

- `id CHAR(36) PK`
- `player_account_id`
- `token_hash CHAR(64) unique`
- `expires_at`
- `revoked_at`
- `created_at`
- `updated_at`

## 6. Sessão

Cookie próprio: `hsc_player_session`.

Usar server-side session com token opaco em cookie HttpOnly.

Não expor token ao JS.

Usar `Secure` em produção.

Definir `SameSite` conforme topologia de domínio/subdomínio.

Configs sugeridas:

```text
PLAYER_SESSION_COOKIE=hsc_player_session
PLAYER_SESSION_TTL_HOURS=168
PLAYER_PORTAL_URL=https://haxixesmokeclub.com/portal/cs2-next
PLAYER_AUTH_CALLBACK_PATH=/auth/player/callback
```

## 7. Fluxo Sign in with Steam

Fluxo proposto:

1. Usuário clica entrar com Steam.
2. Portal envia para Auth API.
3. Auth API inicia fluxo Steam.
4. Steam autentica.
5. Callback volta para Auth API.
6. Auth API valida.
7. Auth API extrai SteamID64.
8. Auth API cria/localiza `player_account`.
9. Auth API cria/atualiza identity `provider=steam`.
10. Auth API cria/atualiza Steam link.
11. Auth API resolve/cacheia Steam Profile.
12. Auth API cria `player_session`.
13. Auth API seta `hsc_player_session`.
14. Auth API redireciona ao Bunker.

## 8. Rotas propostas

- `GET /player/auth/steam/start`
- `GET /player/auth/steam/callback`
- `GET /player/auth/session`
- `POST /player/auth/logout`
- `GET /player/me`

Exemplo curto de resposta autenticada de `/player/auth/session`:

```json
{
  "authenticated": true,
  "player": {
    "id": "player-account-id",
    "email": null,
    "display_name": "Player",
    "steam": {
      "steamid64": "76561198000000000",
      "persona_name": "Player",
      "avatar_url": "https://...",
      "profile_url": "https://steamcommunity.com/profiles/76561198000000000"
    }
  }
}
```

Resposta não autenticada:

```json
{
  "authenticated": false
}
```

## 9. Integração Steam Profiles

hsc-auth-api já é dono canônico do cache Steam Profiles.

Player Auth deve reutilizar esse domínio.

Portal não deve chamar `/internal/steam/profiles/resolve`.

## 10. Integração Static API v2

Auth API não deve calcular estatísticas competitivas no primeiro corte.

Fluxo:

1. `GET /player/auth/session`
2. Obtém `steamid64`.
3. `GET /api/cs2/v2/player/{steamid64}.json`
4. Renderiza `lifetime`, `periods`, `byMap`, `recentMaps`, `timeline` e dados de Season.

Esse fluxo evita segunda fonte de verdade.

## 11. Entitlements futuros

Fora do MVP:

- billing
- assinatura
- premium
- `player_entitlements`
- `player_subscriptions`
- `player_feature_access`

## 12. Segurança mínima

- Separar admin auth de player auth.
- Usar cookies HttpOnly/Secure.
- Não usar localStorage para tokens.
- Validar callback/redirect.
- Usar `state` temporário.
- Expirar estado.
- Armazenar hash de tokens de sessão.
- Permitir revogação.
- Não logar segredos.
- Validar SteamID64 com regex estrita.
- Não confiar no frontend para identidade competitiva.
- Manter auditoria mínima de login/logout.

## 13. Fora de escopo

- Migrar Backoffice para OAuth/OIDC.
- Remover magic link admin.
- Billing.
- Assinatura.
- Google/OIDC.
- Upload avatar player.
- Perfil público editável.
- Comunidade.
- Feed.
- Notificações.
- Cálculo de stats no Auth API.
- Gateway dinâmico para Static API v2.

## 14. Plano de implementação

Fases:

1. Documentação e decisão.
2. Banco/domínio player.
3. Login Steam.
4. Sessão e Bunker API.
5. Portal Bunker.
6. Providers e entitlements futuros.

## 15. Decisão resumida

- Manter Admin Auth atual como legado funcional.
- Criar Player Auth nova no hsc-auth-api, isolada por domínio, tabelas, cookie, middleware e rotas.
- Steam primeiro provider.
- Magic link não é fundação do Bunker.
