# HSC User Bunker — Product and Architecture Plan

Status: draft

Escopo: planejamento de produto e arquitetura para a área logada player-facing do HSC

Codinome: Bunker do usuário

Repositórios impactados: hsc-docs, hsc-cs2-portal, hsc-auth-api, hsc-cs2-etl, hsc-backoffice-admin, hsc-brand-hub

Data base: 2026-05-08

## 1. Contexto

O HSC já possui Portal CS2 público, Static API v2, Seasons, Backoffice, Auth API, Steam Profiles e Brand Hub.

Hoje não existe área logada player-facing.

A auth atual é administrativa/backoffice.

O Bunker ainda não existe e deve ser planejado antes de qualquer implementação.

## 2. Objetivo do Bunker

O Bunker é a área logada para usuário comum/player do Portal CS2.

Ele deve funcionar como dashboard privado ou semi-privado com dados personalizados do jogador.

A evolução futura pode incluir perfil, stats, Seasons, assinatura, comunidade e conteúdo exclusivo.

## 3. Personas iniciais

- Jogador com histórico competitivo: já aparece no ranking/Player JSON e quer consultar evolução, mapas, partidas recentes e timeline.
- Jogador novo ou sem histórico: pode criar conta, vincular Steam e ver estado vazio/guia de próximos passos.
- Membro/assinante futuro: pode ter acesso a badges, recursos premium ou conteúdo exclusivo, mas isso fica fora do MVP.
- Admin/suporte futuro: pode precisar apoiar contas públicas, vínculo Steam e assinatura, mas não é consumidor do MVP player-facing.

## 4. Princípios da frente

- Produto antes de implementação.
- Não quebrar Portal público.
- Não misturar Admin Auth com Player Auth.
- Reutilizar Static API v2 e Player JSON existente antes de criar backend novo.
- Segurança e LGPD desde o desenho.
- Billing como fase separada.
- Evolução incremental.

## 5. Capacidades existentes

### hsc-cs2-portal

Capacidades já existentes relacionadas ao Bunker:

- ranking
- matches
- maps
- news
- seasons
- season ranking
- season matches
- season maps
- api-smoke

### hsc-cs2-etl

Capacidades já existentes relacionadas ao Bunker:

- Static API v2
- Player JSON
- season ranking
- season matches
- season maps
- Steam avatar materializado

### hsc-auth-api

Capacidades já existentes relacionadas ao Bunker:

- admin auth
- content APIs
- seasons
- uploads
- Steam Profiles cache

### hsc-backoffice-admin

O hsc-backoffice-admin é admin/internal e não é player-facing.

### hsc-brand-hub

O hsc-brand-hub é referência visual e institucional.

### hsc-docs

O hsc-docs é a documentação canônica do ecossistema HSC.

## 6. Player JSON existente

Já existe o endpoint público:

```text
/api/cs2/v2/player/{steamId64}.json
```

Shape conhecido:

- `generatedAt`
- `steamid64`
- `name`
- `avatarMedium`
- `steamProfileUrl`
- `lifetime`
- `periods`
- `byMap`
- `recentMaps`
- `timeline`

Isso muda o MVP:

- não precisa criar motor novo de stats no começo;
- lifetime stats podem ser renderizados pelo Portal;
- backend inicial pode focar em identidade, sessão, consentimento e vínculo Steam;
- Portal pode buscar Player JSON após sessão identificar `steamid64`.

## 7. Lacuna de Season/player

Ainda não existem endpoints específicos de player por Season:

- `/api/cs2/v2/season/{slug}/player/{steamId64}.json`
- `/api/cs2/v2/player/{steamId64}/season/{slug}.json`
- `/api/cs2/v2/player/{steamId64}/seasons.json`

Decisão preliminar:

- Season-specific player stats ficam fora do MVP ou como fase 2.
- Não assumir solução sem análise.
- Opções futuras incluem endpoint ETL, agregação frontend, agregação Auth API ou cache backend.

## 8. Endpoints reutilizáveis e faltantes

Endpoints reutilizáveis:

- `/api/cs2/v2/player/{steamId64}.json`
- `/api/cs2/v2/seasons.json`
- `/api/cs2/v2/season/{slug}.json`
- `/api/cs2/v2/season/{slug}/ranking.json`
- `/api/cs2/v2/season/{slug}/matches.json`
- `/api/cs2/v2/season/{slug}/maps.json`

Endpoints faltantes/candidatos futuros:

- `/api/cs2/v2/season/{slug}/player/{steamId64}.json`
- `/api/cs2/v2/player/{steamId64}/season/{slug}.json`
- `/api/cs2/v2/player/{steamId64}/seasons.json`
- eventual `/player/me` ou `/player/auth/session` no backend dinâmico, ainda não definitivo.

Não criar endpoint novo no MVP sem decisão explícita.

Reutilizar Player JSON para lifetime MVP.

## 9. Identidade do usuário

Perguntas e hipóteses que precisam ser resolvidas antes de implementação:

- O que é uma conta HSC?
- SteamID64 é identidade primária competitiva?
- Google pode ser identidade complementar?
- Pode haver conta sem Steam vinculado?
- Pode haver múltiplos providers?
- Pode trocar/desvincular Steam?
- Como impedir claim indevido de SteamID?
- Perfil público e conta privada são entidades separadas?

## 10. Modelagem conceitual

Esta modelagem é conceitual e não autoriza criação de tabelas, migrations, endpoints ou contratos definitivos.

- Conta HSC: entidade interna player-facing, separada de `users`/admin.
- Identidade externa: provider vinculado, como Steam, Google/OIDC ou e-mail futuro.
- Vínculo Steam: relação verificada entre Conta HSC e SteamID64 competitivo.
- Sessão player-facing: sessão separada da sessão admin, com cookie próprio.
- Consentimento: registro de aceite/revogação para privacidade e uso de dados.
- Assinatura/entitlement futuro: fase separada, não requisito do MVP.
- Audit log mínimo: eventos de login, logout, vínculo/desvínculo Steam e ações sensíveis.

## 11. Autenticação candidata

Opções candidatas:

- Steam login como mecanismo forte para provar posse do SteamID.
- Google/OIDC como identidade geral, contato ou recuperação.
- Ambos vinculados a uma conta HSC canônica.
- Magic link não deve ser fundação do Bunker.

Documento técnico auxiliar:

- [Player Auth Architecture — Bunker](./player-auth-architecture.md)

## 12. MVP sugerido

MVP de produto sugerido:

- login/cadastro player-facing;
- claim/vínculo Steam;
- Bunker privado em `/bunker`;
- resumo do jogador;
- lifetime stats usando Player JSON;
- cards de mapa usando `byMap`;
- histórico recente usando `recentMaps`;
- timeline simples;
- estado vazio para usuário sem histórico;
- logout;
- settings mínimos de privacidade/conta.

Fora do MVP:

- billing;
- assinatura;
- premium;
- Google se não for necessário no primeiro corte;
- stats por Season/player;
- perfil público editável;
- comunidade;
- notificações;
- Backoffice para usuários públicos;
- gateway dinâmico de stats.

## 13. LGPD e dados mínimos

Diretrizes iniciais:

- coletar o mínimo necessário;
- armazenar SteamID64, persona, avatar e profile URL quando necessários ao vínculo e à experiência;
- armazenar e-mail só se provider exigir ou se houver necessidade de comunicação;
- não armazenar OAuth tokens sem necessidade;
- registrar consentimento;
- prever exclusão de conta;
- prever anonimização/desvinculação;
- manter retenção mínima de logs;
- produzir política de privacidade futura antes de operação pública.

## 14. Segurança

Requisitos mínimos de segurança:

- separar admin/user auth;
- usar cookie player separado;
- configurar HttpOnly/Secure/SameSite;
- usar state/CSRF em login externo;
- validar redirect de forma estrita;
- armazenar hash de token de sessão;
- permitir revogação;
- não confiar no frontend para SteamID;
- não expor segredos;
- manter auditoria mínima.

## 15. Billing e assinaturas futuras

Billing deve ser fase separada.

Stripe/Mercado Pago ainda não está decidido.

O MVP não deve ser acoplado a plano comercial incerto.

Possíveis entidades futuras:

- `subscriptions`
- `entitlements`
- `feature_access`

Webhooks, cancelamento, inadimplência, reembolso e fiscal ficam para análise própria.

## 16. Arquitetura por repo

Impacto esperado por fase:

- hsc-docs: planejamento e decisões.
- hsc-auth-api: futuro player auth, sessão, conta, consentimento e possível billing.
- hsc-cs2-portal: rotas `/bunker`, login, dashboard e consumo de Player JSON.
- hsc-cs2-etl: possível fase 2 com endpoints season/player.
- hsc-backoffice-admin: futuro suporte/admin de contas públicas, não MVP.
- hsc-brand-hub: referência visual, não implementação funcional.

## 17. Opções arquiteturais

### A. hsc-auth-api também hospeda Player Auth

Vantagens:

- reutiliza infraestrutura dinâmica existente;
- aproveita Steam Profiles cache;
- reduz número inicial de serviços;
- facilita primeira evolução incremental.

Riscos:

- pode aumentar acoplamento entre domínios admin e player;
- exige separação forte de rotas, tabelas, cookies, middlewares e políticas.

### B. Futuro hsc-user-api separado

Vantagens:

- separa domínio player-facing desde a origem;
- pode escalar produto, billing e comunidade de forma mais isolada.

Riscos:

- aumenta complexidade operacional;
- duplica infraestrutura dinâmica cedo demais;
- exige contratos claros com Auth API, Portal e Static API.

### C. Provider externo gerenciado com backend mínimo

Vantagens:

- reduz superfície própria de autenticação;
- pode acelerar login social, recuperação e segurança operacional.

Riscos:

- cria dependência externa forte;
- pode complicar SteamID competitivo, billing local e LGPD;
- exige análise de custo, privacidade e portabilidade.

Recomendação preliminar:

- usar hsc-auth-api com separação forte de domínio para MVP/primeira evolução;
- reavaliar serviço separado se billing, escala ou complexidade justificar.

Esta recomendação não fecha implementação absoluta.

## 18. Riscos

### Riscos técnicos

- acoplamento indevido entre Admin Auth e Player Auth;
- duplicação de stats fora da Static API v2;
- contratos de sessão definidos cedo demais;
- ausência de endpoint season/player no MVP.

### Riscos de produto

- construir dashboard antes de validar valor para o jogador;
- misturar Bunker privado, perfil público e comunidade em um único escopo;
- antecipar billing antes de haver proposta clara.

### Riscos LGPD

- coletar e-mail sem necessidade real;
- reter logs por tempo excessivo;
- não definir exclusão/desvinculação;
- misturar dados públicos de Steam com dados privados de conta HSC.

### Riscos de segurança

- confiar no frontend para SteamID;
- expor tokens ao JS;
- validar redirect de forma frouxa;
- compartilhar cookie ou middleware com admin;
- não registrar revogação e auditoria mínima.

### Riscos operacionais

- criar serviço novo sem necessidade inicial;
- depender de fluxos manuais para suporte de conta;
- não ter runbook antes de abrir login público.

### Riscos de acoplamento com Static API

- tratar Static API v2 como backend dinâmico;
- exigir stats em tempo real antes de existir necessidade;
- criar gateway dinâmico de stats sem análise.

## 19. Perguntas em aberto

Produto:

- Qual é a proposta inicial do Bunker para o player?
- O Bunker é privado, semi-privado ou também origem de perfil público?
- Quais cards entram no MVP?

Auth:

- Steam basta no primeiro corte?
- Google/OIDC entra no MVP ou fica para fase posterior?
- Conta HSC pode existir sem Steam?

Stats por Season:

- Season-specific player stats ficam fora do MVP?
- O endpoint deve nascer no ETL, no Portal ou no backend dinâmico?
- Qual shape mínimo de season/player seria útil?

Billing:

- Haverá assinatura, compra avulsa ou entitlements manuais?
- Stripe, Mercado Pago ou outro provedor?
- Quais features seriam premium?

Privacidade:

- Quais dados o usuário pode ocultar?
- Como funciona exclusão de conta?
- Como funciona desvinculação de Steam?

Backoffice:

- Quando será necessário suporte/admin para contas públicas?
- Quais ações administrativas seriam permitidas?
- Como separar suporte de usuário público de permissões administrativas internas?

## 20. Plano em fases

Fase 0: planejamento e decisão.

Fase 1: Player Auth/claim Steam.

Fase 2: Portal Bunker lifetime MVP.

Fase 3: Season-specific player stats.

Fase 4: settings, privacidade e suporte.

Fase 5: billing/entitlements.

Fase 6: comunidade/conteúdo premium.

## 21. Critérios para sair do planejamento

- decisão sobre Steam/Google/ambos;
- decisão de MVP;
- decisão de dados mínimos;
- decisão de arquitetura hsc-auth-api vs serviço novo;
- contrato inicial de sessão;
- contrato inicial de Bunker UI;
- riscos LGPD documentados;
- fora de escopo aceito.
