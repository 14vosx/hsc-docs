# Backoffice Admin

## Objetivo

Documentar o contexto canônico do Backoffice Admin do ecossistema HSC.

Este documento existe para:

- definir a fronteira documental do produto administrativo
- registrar a função do Backoffice no ecossistema HSC
- explicitar a separação entre camada pública e camada administrativa
- registrar a dependência estrutural da Auth API
- orientar a leitura dos canônicos do contexto `05-backoffice-admin`
- evitar que o contexto volte a ser tratado como apêndice difuso do Portal ou da Auth API

---

## Navegação rápida

### Entrada
- [Home da documentação](../README.md)
- [Governance](../00-governance/README.md)
- [Master Index](../00-governance/99-master-index.md)

### Documentos deste contexto
- [Architecture Runtime](./backoffice-admin-architecture-runtime.md)
- [Frontend Structure](./backoffice-admin-frontend-structure.md)
- [Auth, RBAC and Guards](./auth-rbac-and-guards.md)
- [Admin API Contracts](./admin-api-contracts.md)
- [News Admin API Contracts](./news-admin-api-contracts.md)
- [News Admin Integration and Evolution](./news-admin-integration-and-evolution.md)
- [News Admin Feature Implementation Spec](./news-admin-feature-implementation-spec.md)
- [News Admin Frontend Implementation Runtime](./news-admin-frontend-implementation-runtime.md)
- [News Functional Smoke Guide](./news-functional-smoke-guide.md)
- [Seasons Admin Functional Smoke Guide](./seasons-admin-list-functional-smoke-guide.md)
- [Backoffice UI Material Foundation](./backoffice-ui-material-foundation.md)
- [Operational Runbooks](./backoffice-admin-operational-runbooks.md)
- [References and Inventory](./backoffice-admin-references-inventory.md)

### Relações com outros contextos
- [Infra AWS Lightsail](../04-infra-aws-lightsail/README.md)
- [Portal Estático](../03-portal-estatico/README.md)
- [Infra Hostinger](../01-infra-hostinger/README.md)

### Trilhas operacionais relevantes
- [Auth API Operations](../04-infra-aws-lightsail/auth-api-operations.md)
- [Nginx Reverse Proxy](../04-infra-aws-lightsail/nginx-reverse-proxy.md)
- [Deploy / Release / Rollback](../04-infra-aws-lightsail/deploy-release-rollback.md)
- [Documentation System](../00-governance/documentation-system.md)

---

## Escopo

Este documento cobre:

- a função do Backoffice como produto administrativo próprio
- sua relação com os demais contextos canônicos
- a leitura correta do estado atual do contexto
- a ordem recomendada de leitura dos documentos do contexto
- a próxima fase natural de evolução do Backoffice

Este documento não cobre em profundidade:

- estrutura detalhada da SPA
- contratos HTTP endpoint a endpoint
- estratégia completa de auth/RBAC/guards
- runbooks operacionais detalhados
- detalhes de host, deploy ou banco

Esses tópicos vivem nos documentos especializados do próprio contexto ou em contextos adjacentes.

---

## Estado atual do contexto

O estado atual conhecido do contexto `05-backoffice-admin` é:

- o Backoffice Admin já é um contexto canônico formal do HSC
- a camada administrativa permanece separada do Portal público
- a Auth API publicada é a dependência dinâmica central do contexto
- o modelo de auth do admin é session-first
- o login administrativo real publicado usa magic link
- a SPA já possui jornada real publicada:
  - `/login`
  - `/auth/callback`
  - `/dashboard`
- a sessão administrativa publicada já é cookie-based e cross-subdomain
- o frontend administrativo já opera com sessão real e guards assíncronos
- o domínio `news` atravessa Backoffice Admin, Auth API, ETL Hostinger e Portal Estático
- `GET /admin/news/:id` já é superfície admin real, publicada e reconciliada para edição
- a edição de News usa detalhe administrativo com `content` como fonte primária
- o lifecycle PROD de News já foi validado para edição, refresh/deep link, publish, unpublish e delete
- `seasons` já possui listagem administrativa funcional em `/seasons`
- `seasons` já possui criação administrativa funcional em `/seasons/new`
- `seasons` já possui edição administrativa funcional em `/seasons/:slug/edit`
- `seasons` já possui ações administrativas de lifecycle em `/seasons` para ativar e fechar Seasons
- a leitura admin canônica de Seasons já está disponível na Auth API por `GET /admin/seasons` e `GET /admin/seasons/:slug`
- a criação admin de Seasons já está disponível por `POST /admin/seasons`, criando registros em `draft`
- a edição admin de Seasons já está disponível por `PATCH /admin/seasons/:slug`, sem alterar `slug`
- activate e close de Seasons já estão disponíveis na UI por `POST /admin/seasons/:slug/activate` e `POST /admin/seasons/:slug/close`
- a Auth API já possui contrato protegido de upload administrativo por `POST /admin/uploads`
- o Backoffice News já integra visualmente upload de imagem por `POST /admin/uploads`, com campo de URL, upload, preview e limpeza de imagem validados em smoke manual local
- a Auth API já possui contrato de `cover_image_url` para Seasons nas superfícies administrativas e públicas de Seasons
- o Backoffice Seasons já integra visualmente upload de capa por `POST /admin/uploads`, com URL da capa, upload, preview e limpeza validados em smoke manual local
- ETL/Static API v2 e Portal ainda estão pendentes para publicar ou consumir `cover_image_url` nos JSONs estáticos quando aplicável
- o Backoffice já possui fundação UI Material compartilhada para feedback transitório, feedback persistente, confirmação e input simples em fluxos principais de `news`, `seasons` e `users`
- dark mode, toggle de tema, design system completo e padronização visual total do Backoffice ainda não devem ser tratados como implementados
- `seasons`, `news` e `events` continuam sendo os domínios iniciais
- o contexto já não deve ser tratado como “pré-implementação pura”

Regra importante:

- o Backoffice deve ser tratado como produto administrativo próprio
- ele não deve ser tratado como camada improvisada acoplada ao Portal público

---

## Papel do Backoffice no ecossistema

O Backoffice existe para sustentar a camada administrativa do HSC.

Sua função é:

- operar fluxos administrativos autenticados
- materializar shell, sessão, guards e RBAC do admin
- sustentar gestão de `news`, `seasons` e `events`
- consumir a Auth API como backend dinâmico central
- manter fronteira explícita entre produto público e produto administrativo

Leitura correta:

- o Portal continua sendo a superfície pública do ecossistema
- a Auth API continua sendo a camada dinâmica central
- o Backoffice é a superfície administrativa do ecossistema

---

## Princípios centrais do contexto

Os princípios centrais deste contexto são:

- separação explícita entre público e administrativo
- integração por contrato com a Auth API
- session-first no fluxo administrativo
- backend como autoridade final de autenticação e autorização
- break-glass apenas como contingência controlada
- UI subordinada à autorização do backend
- shell administrativo explícito
- evolução incremental por domínio
- MVP antes de sofisticação
- baixo acoplamento entre features administrativas
- documentação canônica viva desde o início

Regra importante:

- o Backoffice deve nascer e evoluir como produto administrativo disciplinado
- ele não deve degenerar em coleção solta de telas CRUD

---

## Relação com outros contextos

Este contexto se relaciona diretamente com:

### `docs/00-governance/`

Relação estrutural obrigatória.

A governança documental define:
- como o contexto existe
- como seus documentos devem ser mantidos
- como novas superfícies devem ser reconciliadas com o runtime real

### `docs/03-portal-estatico/`

Relação forte por separação de produto.

O Portal público:
- continua sendo camada pública
- não absorve responsabilidades administrativas
- pode consumir outputs de domínios administrados em outra camada, mas não substitui o Backoffice

### `docs/04-infra-aws-lightsail/`

Relação estrutural crítica.

A Infra AWS Lightsail sustenta:
- a Auth API publicada
- o modelo session-first
- o fluxo real de magic link
- a sessão cookie-based cross-subdomain
- a postura fail-closed para mutações administrativas sensíveis
- o runtime real consumido pelo Backoffice

### `docs/98-legacy/`

Relação de reconciliação histórica.

O legado segue sendo fonte importante para:
- roadmap do Backoffice
- separação de camadas futura
- RBAC inicial
- superfícies administrativas previstas
- prioridades de domínio

O legado não governa este contexto, mas sustenta sua reconciliação histórica.

---

## Documentos canônicos deste contexto

Os documentos canônicos diretos de `05-backoffice-admin` são:

- `docs/05-backoffice-admin/README.md`
- `docs/05-backoffice-admin/backoffice-admin-architecture-runtime.md`
- `docs/05-backoffice-admin/backoffice-admin-frontend-structure.md`
- `docs/05-backoffice-admin/auth-rbac-and-guards.md`
- `docs/05-backoffice-admin/admin-api-contracts.md`
- `docs/05-backoffice-admin/news-admin-api-contracts.md`
- `docs/05-backoffice-admin/news-admin-integration-and-evolution.md`
- `docs/05-backoffice-admin/news-admin-feature-implementation-spec.md`
- `docs/05-backoffice-admin/news-admin-frontend-implementation-runtime.md`
- `docs/05-backoffice-admin/news-functional-smoke-guide.md`
- `docs/05-backoffice-admin/seasons-admin-list-functional-smoke-guide.md`
- `docs/05-backoffice-admin/backoffice-ui-material-foundation.md`
- `docs/05-backoffice-admin/backoffice-admin-operational-runbooks.md`
- `docs/05-backoffice-admin/backoffice-admin-references-inventory.md`

Este `README.md` funciona como porta de entrada e índice local do contexto.

---

## Ordem de leitura recomendada deste contexto

A ordem recomendada de leitura é:

1. `docs/05-backoffice-admin/README.md`
2. `docs/05-backoffice-admin/backoffice-admin-architecture-runtime.md`
3. `docs/05-backoffice-admin/backoffice-admin-frontend-structure.md`
4. `docs/05-backoffice-admin/auth-rbac-and-guards.md`
5. `docs/05-backoffice-admin/admin-api-contracts.md`
6. `docs/05-backoffice-admin/news-admin-api-contracts.md`
7. `docs/05-backoffice-admin/news-admin-integration-and-evolution.md`
8. `docs/05-backoffice-admin/news-admin-feature-implementation-spec.md`
9. `docs/05-backoffice-admin/news-admin-frontend-implementation-runtime.md`
10. `docs/05-backoffice-admin/news-functional-smoke-guide.md`
11. `docs/05-backoffice-admin/seasons-admin-list-functional-smoke-guide.md`
12. `docs/05-backoffice-admin/backoffice-ui-material-foundation.md`
13. `docs/05-backoffice-admin/backoffice-admin-operational-runbooks.md`
14. `docs/05-backoffice-admin/backoffice-admin-references-inventory.md`

---

## Próxima fase natural do contexto

A próxima fase natural deste contexto é:

- expandir implementação administrativa por domínio
- evoluir `seasons` com ranking, partidas e integrações futuras sem tratar essas lacunas como estado implementado
- consolidar leituras administrativas canônicas de `events`
- amadurecer superfícies e lifecycle de `events`
- manter auth, guards e callback publicados como espinha dorsal estável
- preservar coerência entre SPA administrativa, Auth API e documentação canônica

Leitura importante:

- a principal pendência estrutural grande de auth base já foi vencida
- o valor agora está em expandir domínio e reduzir gaps contratuais remanescentes

---

## Resumo executivo

O contexto `05-backoffice-admin` existe para registrar e governar a camada administrativa do HSC como produto próprio.

Sua função é:

- separar formalmente o administrativo do público
- documentar o Backoffice como SPA administrativa real
- ancorar sua integração na Auth API publicada
- sustentar a evolução de `news`, `seasons` e `events`
- manter a camada administrativa previsível, auditável e coerente com o restante do ecossistema
