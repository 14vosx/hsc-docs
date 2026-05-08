# HSC User Bunker — Product and Architecture Plan

Status: draft

Escopo: planejamento de produto e arquitetura para a área logada player-facing do HSC

Codinome: Bunker do usuário

Repositórios impactados: hsc-docs, hsc-cs2-portal, hsc-auth-api, hsc-cs2-etl, hsc-backoffice-admin, hsc-brand-hub

Data base: 2026-05-08

## Decisões fechadas da fase de planejamento

Itens 1-6 fechados nesta fase:

- MVP Steam-first: o provider inicial é Steam, sem senha própria e sem magic link player-facing como fundação.
- Google/OIDC fica como fase futura planejada, como identidade complementar de uma Conta HSC que poderá ter múltiplos providers.
- A UI deve explicar que login Steam confirma SteamID e carrega stats HSC, sem acesso a senha, inventário, trocas ou dados sensíveis da conta Steam.
- Conta HSC é a entidade canônica interna do usuário player-facing; no MVP, ela nasce a partir do login Steam.
- Cada Conta HSC terá 1 SteamID64 competitivo principal, e cada SteamID64 pode estar vinculado a no máximo 1 Conta HSC.
- Não haverá troca/desvinculação self-service de Steam no MVP.
- Portal público continua como vitrine: rankings, listas, cards resumidos, partidas, mapas, Seasons e notícias.
- O dossiê individual completo migra de forma faseada para o Bunker: lifetime completo, periods, byMap detalhado, recentMaps, timeline, Season/player stats e insights futuros.
- Bunker MVP usa rota principal `/bunker`, UI privada por camada autenticada e estados obrigatórios de não autenticado, autenticado sem dados, dados parciais, dados disponíveis e erro de dados.
- Bunker é Season-first: Season atual é o destaque principal da UI, e lifetime entra como contexto histórico.
- Stats por Season/player entram no MVP, com ETL como dono conceitual do cálculo/materialização e Auth API como identidade, sessão, autorização e gateway autenticado.
- Não fechar contrato exato agora; endpoints e rotas autenticadas descritos neste documento permanecem conceituais até fase técnica posterior.
- Data minimization é princípio central; SteamID64, persona/nickname Steam, avatar Steam, Steam profile URL e stats vinculadas ao jogador são tratados como dados pessoais.
- Exclusão da Conta HSC revoga sessão, remove/desvincula dados privados da conta e bloqueia acesso ao Bunker, mas não apaga automaticamente histórico competitivo necessário para partidas, rankings, Seasons, auditoria e integridade do ecossistema.
- Antes do lançamento público, HSC precisa de Termos de Uso, Política de Privacidade, texto claro no login Steam e caminho de exclusão/desativação.
- Billing fica fora do MVP; Bunker MVP é gratuito para usuário autenticado via Steam.
- Arquitetura deve ficar preparada para entitlements futuros; Stripe e Mercado Pago são candidatos futuros, sem decisão nesta fase.
- Acesso futuro deve ser modelado por entitlements, não diretamente por `subscription_status`; HSC não deve armazenar dados de cartão.

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
- Manter o Portal público como vitrine e migrar o dossiê individual completo para o Bunker de forma faseada.
- Fazer o MVP como Season-first, com lifetime como contexto histórico.
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
- lifetime stats entram como contexto histórico do Bunker;
- backend inicial pode focar em identidade, sessão, consentimento, vínculo Steam, autorização e gateway autenticado;
- experiência oficial do dossiê individual completo deve ser consumida por rota/camada autenticada quando o Bunker existir;
- Portal pode buscar Player JSON após sessão identificar `steamid64`, sem assumir contrato definitivo nesta fase.

## 7. Lacuna de Season/player

Ainda não existem endpoints específicos de player por Season:

- `/api/cs2/v2/season/{slug}/player/{steamId64}.json`
- `/api/cs2/v2/player/{steamId64}/season/{slug}.json`
- `/api/cs2/v2/player/{steamId64}/seasons.json`

Decisão preliminar:

- Season/player stats entram no MVP.
- Bunker é Season-first, com Season atual como destaque principal.
- Lifetime entra como contexto histórico.
- ETL é o dono conceitual do cálculo/materialização de Season/player stats, porque já é dono da Static API v2 e das regras de pertencimento à Season.
- Auth API não deve ser motor estatístico; deve atuar como identidade, sessão, autorização e gateway autenticado.
- Portal não deve calcular Season/player stats no frontend como solução oficial.
- Contrato exato e implementação ficam para fase técnica posterior.

## 8. Endpoints reutilizáveis e faltantes

Endpoints reutilizáveis:

- `/api/cs2/v2/player/{steamId64}.json`
- `/api/cs2/v2/seasons.json`
- `/api/cs2/v2/season/{slug}.json`
- `/api/cs2/v2/season/{slug}/ranking.json`
- `/api/cs2/v2/season/{slug}/matches.json`
- `/api/cs2/v2/season/{slug}/maps.json`

Endpoints faltantes/candidatos futuros:

- `/api/cs2/v2/season/{slug}/player/{steamId64}.json`, candidato necessário para MVP, ainda não contrato definitivo.
- `/api/cs2/v2/player/{steamId64}/season/{slug}.json`
- `/api/cs2/v2/player/{steamId64}/seasons.json`
- rotas autenticadas conceituais `/player/me/seasons/current` e `/player/me/seasons/:slug`, ainda não definitivas.
- eventual `/player/me` ou `/player/auth/session` no backend dinâmico, ainda não definitivo.

Não criar endpoint novo sem decisão explícita de fase técnica.

Reutilizar Player JSON como base existente e contexto lifetime, mas planejar Season/player stats como parte do MVP.

## 9. Identidade do usuário

Decisões e perguntas que orientam a fase seguinte:

- Conta HSC é a entidade canônica interna do usuário player-facing.
- No MVP, a Conta HSC nasce a partir do login Steam.
- SteamID64 é a identidade competitiva principal.
- Cada Conta HSC terá 1 SteamID64 competitivo principal.
- Cada SteamID64 pode estar vinculado a no máximo 1 Conta HSC.
- Login Steam é mecanismo de claim/ownership do perfil competitivo.
- Google/OIDC futuro será identidade complementar da mesma Conta HSC.
- Não haverá troca/desvinculação self-service de Steam no MVP.
- Ainda precisa ser detalhado como suporte/admin tratará casos excepcionais de vínculo indevido.
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
- Dossiê competitivo: conjunto de stats pessoais detalhadas, com acesso oficial pela camada autenticada do Bunker quando a migração estiver concluída.

## 11. Autenticação candidata

Opções candidatas:

- Steam login é o mecanismo inicial do MVP para provar posse do SteamID.
- Google/OIDC fica como fase futura planejada para identidade geral, contato ou recuperação.
- Produto final desejado: Conta HSC canônica com múltiplos providers, permitindo Steam + Google.
- Sem senha própria no MVP.
- Magic link não deve ser fundação do Bunker.
- UI deve explicar que login Steam serve para confirmar SteamID e carregar stats HSC, não para acessar senha, inventário, trocas ou dados sensíveis da conta Steam.

Documento técnico auxiliar:

- [Player Auth Architecture — Bunker](./player-auth-architecture.md)

## 12. MVP sugerido

MVP de produto sugerido:

- login/cadastro player-facing Steam-first;
- claim/vínculo SteamID64;
- Bunker privado em `/bunker`;
- UI privada por camada autenticada;
- Season atual como hero/destaque principal;
- Season/player stats no MVP, com contrato exato a definir em fase técnica posterior;
- resumo do jogador;
- lifetime stats usando Player JSON como contexto histórico;
- cards de mapa usando `byMap`;
- histórico recente usando `recentMaps`;
- timeline simples;
- hard gate faseado do dossiê pessoal completo para área logada;
- estado vazio para usuário sem histórico;
- estado autenticado sem dados;
- estado de dados parciais;
- estado de erro de dados;
- logout;
- settings mínimos de privacidade/conta.

Fora do MVP:

- billing;
- assinatura;
- premium;
- Google/OIDC;
- senha própria;
- magic link player-facing como fundação;
- troca/desvinculação self-service de Steam;
- perfil público editável;
- comunidade;
- notificações;
- Backoffice para usuários públicos;
- motor dinâmico novo de cálculo de stats;
- gateway premium/avançado de analytics.

## 13. LGPD e dados mínimos

Diretrizes iniciais:

- coletar o mínimo necessário;
- manter Conta HSC privada separada de stats competitivas;
- tratar SteamID64, persona/nickname Steam, avatar Steam, Steam profile URL e stats vinculadas ao jogador como dados pessoais;
- armazenar SteamID64, persona, avatar e profile URL quando necessários ao vínculo e à experiência;
- armazenar e-mail só se provider exigir ou se houver necessidade de comunicação;
- não coletar no MVP senha, CPF, telefone, endereço, inventário Steam, dados de troca, wallet, dados sensíveis, tokens externos persistentes sem necessidade explícita ou dados financeiros;
- não armazenar OAuth tokens sem necessidade;
- registrar consentimento;
- exclusão da Conta HSC deve revogar sessão, remover/desvincular dados privados da conta e bloquear acesso ao Bunker;
- exclusão da Conta HSC não apaga automaticamente histórico competitivo necessário para partidas, rankings, Seasons, auditoria e integridade do ecossistema;
- prever anonimização/desvinculação;
- manter retenção mínima de logs;
- marketing, comunicações promocionais e billing futuro exigem decisões e consentimentos próprios;
- antes do lançamento público, produzir Termos de Uso, Política de Privacidade, texto claro no login Steam e caminho de exclusão/desativação.

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

Bunker MVP será gratuito para usuário autenticado via Steam.

Stripe/Mercado Pago ainda não está decidido.

O MVP não deve ser acoplado a plano comercial incerto.

O acesso futuro deve ser modelado por entitlements, não diretamente por `subscription_status`.

HSC não deve armazenar dados de cartão.

Possíveis entidades futuras:

- `subscriptions`
- `entitlements`
- `feature_access`

Webhooks, cancelamento, inadimplência, reembolso e fiscal ficam para análise própria.

Fora do MVP:

- checkout;
- webhook de pagamento;
- planos pagos;
- subscriptions reais;
- customer portal;
- trial;
- cupom;
- admin de assinatura;
- bloqueio premium por pagamento.

## 16. Arquitetura por repo

Impacto esperado por fase:

- hsc-docs: planejamento e decisões.
- hsc-auth-api: futuro player auth, sessão, conta, consentimento, autorização e gateway autenticado; não deve ser motor estatístico.
- hsc-cs2-portal: rota `/bunker`, login, dashboard, consumo autenticado de dados do jogador e UI Season-first.
- hsc-cs2-etl: dono conceitual do cálculo/materialização de Season/player stats.
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
- contrato season/player fechado sem análise técnica suficiente;
- gateway autenticado acoplado demais ao shape público da Static API.

### Riscos de produto

- construir dashboard antes de validar valor para o jogador;
- misturar Bunker privado, perfil público e comunidade em um único escopo;
- antecipar billing antes de haver proposta clara.

### Riscos LGPD

- coletar e-mail sem necessidade real;
- tratar SteamID64 e stats vinculadas como se não fossem dados pessoais;
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
- quebrar Portal público durante a migração faseada do dossiê individual completo para o Bunker.

## 19. Perguntas em aberto

Produto:

- Como comunicar o hard gate faseado sem piorar a experiência atual do Portal público?
- Quais dados resumidos permanecem públicos em rankings, cards e listas?
- Quais cards entram no MVP?

Auth:

- Como tratar casos excepcionais de SteamID vinculado indevidamente?
- Qual suporte/admin mínimo será necessário para vínculo Steam no futuro?
- Quando Google/OIDC deve entrar como provider complementar?

Stats por Season:

- Qual shape mínimo de `/api/cs2/v2/season/{slug}/player/{steamId64}.json` atende o MVP?
- Como a camada autenticada deve mapear Season atual em `/player/me/seasons/current`?
- Qual shape mínimo de season/player seria útil?

Billing:

- Haverá assinatura, compra avulsa ou entitlements manuais?
- Quais features seriam premium?
- Quando decidir entre Stripe, Mercado Pago ou outro provedor?

Privacidade:

- Quais dados o usuário pode ocultar?
- Como funciona exclusão de conta?
- Como funciona desativação de conta?
- Como comunicar que exclusão da Conta HSC não apaga automaticamente histórico competitivo?

Backoffice:

- Quando será necessário suporte/admin para contas públicas?
- Quais ações administrativas seriam permitidas?
- Como separar suporte de usuário público de permissões administrativas internas?

## 20. Plano em fases

Fase 0: planejamento e decisão.

Fase 1: Player Auth/claim Steam.

Fase 2: Portal Bunker Season-first MVP via camada autenticada mínima.

Fase 3: hard gate progressivo e consolidação do dossiê individual completo atrás da camada autenticada.

Fase 4: settings, privacidade e suporte.

Fase 5: billing/entitlements.

Fase 6: comunidade/conteúdo premium.

## 21. Critérios para sair do planejamento

- decisões 1-6 registradas e aceitas como base do MVP;
- recorte do MVP Steam-first, Season-first e gratuito confirmado;
- dados mínimos, consentimento e exclusão/desativação detalhados para lançamento público;
- decisão de arquitetura hsc-auth-api vs serviço novo confirmada para a fase técnica;
- contrato inicial de sessão;
- contrato inicial de Bunker UI;
- decisão técnica posterior para endpoints Season/player e rotas autenticadas conceituais;
- riscos LGPD documentados;
- fora de escopo aceito.
