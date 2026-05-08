# HSC Repositories Map

## Objetivo

Este documento registra o mapa canônico dos repositórios do ecossistema HSC.

Ele existe para:

- deixar claro o papel de cada repositório
- evitar confusão entre superfícies públicas, administrativas, ETL e documentação
- orientar READMEs locais sem substituir a documentação específica de cada repo
- registrar guardrails comuns de segurança e publicação

Este documento não é plano de deploy, inventário de segredos, fonte de valores de `.env` ou substituto dos READMEs locais.

---

## Guardrails comuns

- nunca publicar `.env`, tokens, credenciais, cookies, SMTP, DB URLs ou chaves
- nunca tratar webroot público como Git working tree
- não fazer deploy por `git pull` em `/var/www`
- READMEs devem explicar o papel do repo sem expor runtime sensível
- comandos de deploy devem ficar em docs/runbooks apropriados, com rollback e notas de segurança, não em README genérico
- `hsc-docs` documenta o ecossistema, mas não substitui os READMEs locais dos repositórios de implementação

---

## Visão geral

| Repositório | Papel principal |
| --- | --- |
| `hsc-cs2-portal` | Portal público CS2 e Angular CS2 Next |
| `hsc-cs2-etl` | ETL que materializa a Static API v2 |
| `hsc-docs` | Documentação canônica do ecossistema |
| `hsc-auth-api` | API dinâmica de auth, admin e conteúdo |
| `hsc-backoffice-admin` | SPA administrativa |
| `hsc-brand-hub` | Hub público de marca no apex |

---

## `hsc-cs2-portal`

Nome:
- `hsc-cs2-portal`

URL:
- <https://github.com/14vosx/hsc-cs2-portal>

Descrição curta sugerida para GitHub:
- Portal público CS2 do HSC: Angular CS2 Next, ranking, partidas, mapas, Seasons e consumo da Static API v2.

Papel no ecossistema:
- superfície pública CS2 do HSC
- experiência player-facing de ranking, partidas, mapas, jogadores e Seasons
- consumidor browser da Static API v2

Escopo:
- frontend público CS2
- Angular CS2 Next
- páginas player-facing
- consumo de contratos públicos da Static API v2
- integração com conteúdo público espelhado quando aplicável

Fora de escopo:
- geração da Static API v2
- Auth API administrativa
- backoffice protegido
- operação do Game Panel
- deploy sem runbook apropriado

Relação com outros repos:
- consome a Static API v2 gerada pelo `hsc-cs2-etl`
- consome conteúdo público vindo do `hsc-auth-api` via espelhos/contratos públicos quando aplicável
- é documentado canonicamente neste `hsc-docs`
- não substitui o `hsc-brand-hub` como superfície institucional do apex

README status atual:
- sem `README.md` raiz
- possui `AGENTS.md`
- possui `frontend/angular/README.md`

Guardrails de segurança:
- não documentar valores reais de `.env`
- não publicar raiz do repositório como webroot
- não tratar webroot público como Git working tree
- não fazer deploy por `git pull` em `/var/www`

Documentação relacionada dentro de `hsc-docs`:
- [Portal Estático](../03-portal-estatico/README.md)
- [Portal Estático Frontend Structure](../03-portal-estatico/portal-estatico-frontend-structure.md)
- [Static API v2](../03-portal-estatico/static-api-v2.md)
- [Nginx Publishing and Cache](../03-portal-estatico/nginx-publishing-cache.md)

---

## `hsc-cs2-etl`

Nome:
- `hsc-cs2-etl`

URL:
- <https://github.com/14vosx/hsc-cs2-etl>

Descrição curta sugerida para GitHub:
- Pipeline ETL do HSC CS2 para gerar a Static API v2 a partir de dados MatchZy/SQLite e contratos de conteúdo.

Papel no ecossistema:
- materializar a Static API v2 consumida pelo portal público
- transformar dados operacionais do CS2 em JSONs públicos
- consolidar recortes de domínio como ranking, partidas, mapas, players e Seasons

Escopo:
- scripts ETL versionados
- geração dos artefatos JSON da v2
- integração com MatchZy/SQLite
- integração com contratos públicos/internos necessários do `hsc-auth-api` para Seasons/Steam Profiles
- CI mínima de shell quando aplicável

Fora de escopo:
- frontend público
- backoffice administrativo
- Auth API dinâmica
- operação direta do Game Panel
- exposição de segredos operacionais

Relação com outros repos:
- gera a Static API v2 consumida pelo `hsc-cs2-portal`
- consome MatchZy/SQLite como fonte de dados do lado jogo
- consome alguns contratos públicos/internos do `hsc-auth-api` para Seasons/Steam Profiles
- é documentado canonicamente neste `hsc-docs`

README status atual:
- possui `README.md` raiz
- precisa apenas revisão leve/padronização posterior

Guardrails de segurança:
- não registrar credenciais, tokens ou chaves internas
- não documentar valores reais de `.env`
- não expor caminhos sensíveis além dos já canonicamente documentados em runbooks apropriados
- não transformar README genérico em runbook de deploy

Documentação relacionada dentro de `hsc-docs`:
- [ETL Bash Pipeline](../03-portal-estatico/etl-bash-pipeline.md)
- [ETL Runtime Reconciliation](../03-portal-estatico/etl-runtime-reconciliation.md)
- [ETL Runtime Materialization Runbook](../03-portal-estatico/etl-runtime-materialization-runbook.md)
- [Static API v2](../03-portal-estatico/static-api-v2.md)
- [Data Sources - MatchZy SQLite](../03-portal-estatico/data-sources-matchzy-sqlite.md)

---

## `hsc-docs`

Nome:
- `hsc-docs`

URL:
- <https://github.com/14vosx/hsc-docs>

Descrição curta sugerida para GitHub:
- Documentação canônica do ecossistema HSC: arquitetura, infraestrutura, runbooks, decisões e histórico de implementação.

Papel no ecossistema:
- fonte canônica de documentação transversal do HSC
- registro de arquitetura, operação, contratos, decisões e histórico
- ponte entre repos de implementação e contextos operacionais

Escopo:
- documentação em Markdown
- governança documental
- índices e mapas
- runbooks canônicos
- impl logs e auditoria

Fora de escopo:
- runtime de aplicação
- deploy de produto
- armazenamento de segredos
- automações operacionais
- substituição dos READMEs locais dos repos de implementação

Relação com outros repos:
- documenta todos os contextos do ecossistema HSC
- referencia os repos de implementação quando necessário
- não substitui os READMEs locais
- deve apontar para documentos canônicos em vez de duplicar grandes blocos

README status atual:
- sem `README.md` raiz antes desta frente
- possui `docs/README.md`
- possui READMEs por contexto

Guardrails de segurança:
- não registrar segredos, tokens, credenciais ou valores reais de `.env`
- não tratar documentação como runtime deployável
- não inventar estado operacional
- não alterar decisões de arquitetura sem aprovação explícita

Documentação relacionada dentro de `hsc-docs`:
- [Home da documentação](../README.md)
- [Governance](./README.md)
- [Documentation System](./documentation-system.md)
- [HSC Docs Maintenance Playbook](./hsc-docs-maintenance-playbook.md)
- [Master Index](./99-master-index.md)

---

## `hsc-auth-api`

Nome:
- `hsc-auth-api`

URL:
- <https://github.com/14vosx/hsc-auth-api>

Descrição curta sugerida para GitHub:
- API de autenticação e conteúdo do HSC: Auth, RBAC, Admin APIs, News, Seasons, uploads e perfis Steam.

Papel no ecossistema:
- camada dinâmica de autenticação, autorização e conteúdo
- backend administrativo consumido pelo Backoffice
- fonte de domínios como News, Seasons, uploads e Steam Profiles

Escopo:
- Auth API
- RBAC
- Admin APIs
- News
- Seasons
- uploads
- Steam Profiles
- integrações internas necessárias ao ecossistema

Fora de escopo:
- frontend público CS2
- geração direta da Static API v2
- backoffice frontend
- documentação de valores reais de ambiente
- exposição de credenciais ou infraestrutura sensível

Relação com outros repos:
- é consumido pelo `hsc-backoffice-admin` para administração
- fornece conteúdo público que pode ser espelhado/contratado para consumo pelo `hsc-cs2-portal`
- fornece contratos públicos/internos consumidos pelo `hsc-cs2-etl` para Seasons/Steam Profiles quando aplicável
- é documentado canonicamente neste `hsc-docs`

README status atual:
- sem `README.md` raiz
- possui `AGENTS.md`
- possui `.env.local.example`
- não expor `.env` real

Guardrails de segurança:
- nunca publicar `.env` real
- nunca registrar tokens, credenciais, cookies, SMTP, DB URLs ou chaves
- manter exemplos de ambiente sem valores sensíveis
- documentar deploy apenas em runbooks apropriados com rollback e notas de segurança

Documentação relacionada dentro de `hsc-docs`:
- [Infra AWS Lightsail](../04-infra-aws-lightsail/README.md)
- [Auth API Operations](../04-infra-aws-lightsail/auth-api-operations.md)
- [Admin API Contracts](../05-backoffice-admin/admin-api-contracts.md)
- [News Admin API Contracts](../05-backoffice-admin/news-admin-api-contracts.md)

---

## `hsc-backoffice-admin`

Nome:
- `hsc-backoffice-admin`

URL:
- <https://github.com/14vosx/hsc-backoffice-admin>

Descrição curta sugerida para GitHub:
- Backoffice administrativo Angular do HSC para gestão de conteúdo, usuários, notícias, Seasons e operações internas.

Papel no ecossistema:
- SPA administrativa protegida do HSC
- interface para operar domínios administrativos expostos pela Auth API
- superfície de gestão de conteúdo, usuários, notícias, Seasons e fluxos internos

Escopo:
- frontend administrativo Angular
- autenticação no frontend
- guards e RBAC na interface
- consumo de APIs administrativas
- fluxos administrativos de conteúdo

Fora de escopo:
- Auth API backend
- Portal público CS2
- Static API v2
- ETL
- operação do Game Panel

Relação com outros repos:
- consome `hsc-auth-api` para administração
- é documentado canonicamente neste `hsc-docs`
- não substitui o portal público `hsc-cs2-portal`
- não publica a Static API v2

README status atual:
- possui `README.md` raiz
- precisa auditoria focada antes de reescrever

Guardrails de segurança:
- não registrar tokens, cookies ou credenciais administrativas
- não documentar valores reais de `.env`
- não misturar README genérico com runbook de deploy
- manter política de Auth/RBAC alinhada aos documentos canônicos

Documentação relacionada dentro de `hsc-docs`:
- [Backoffice Admin](../05-backoffice-admin/README.md)
- [Backoffice Admin Architecture Runtime](../05-backoffice-admin/backoffice-admin-architecture-runtime.md)
- [Auth, RBAC and Guards](../05-backoffice-admin/auth-rbac-and-guards.md)
- [Admin API Contracts](../05-backoffice-admin/admin-api-contracts.md)

---

## `hsc-brand-hub`

Nome:
- `hsc-brand-hub`

URL:
- <https://github.com/14vosx/hsc-brand-hub>

Descrição curta sugerida para GitHub:
- Hub público da marca HSC: site estático do apex, identidade visual, assets e superfície institucional.

Papel no ecossistema:
- superfície pública institucional da marca HSC
- site estático do apex
- camada de identidade visual e assets de marca

Escopo:
- Brand Hub público
- páginas institucionais
- identidade visual
- assets de marca
- experiência pública de entrada da marca

Fora de escopo:
- Portal CS2 player-facing
- Backoffice administrativo
- Auth API
- ETL da Static API v2
- operação do Game Panel

Relação com outros repos:
- é superfície pública de marca no apex
- não deve ser confundido com o Portal CS2 (`hsc-cs2-portal`)
- pode apontar para superfícies públicas do ecossistema quando aplicável
- é documentado canonicamente neste `hsc-docs`

README status atual:
- sem `README.md` raiz
- possui `AGENTS.md`

Guardrails de segurança:
- não publicar segredos ou valores reais de ambiente
- não misturar superfície institucional com webroot operacional do Portal CS2
- não documentar deploy sem runbook apropriado
- não tratar assets sensíveis ou internos como públicos sem decisão explícita

Documentação relacionada dentro de `hsc-docs`:
- [Portal Estático](../03-portal-estatico/README.md)
- [Brand Hub Root - Product and Surface Decisions](../03-portal-estatico/brand-hub-root-product-and-surface-decisions.md)
- [Brand Hub Root - Publishing and Cutover Runtime](../03-portal-estatico/brand-hub-root-publishing-and-cutover-runtime.md)

---

## Leitura transversal

Relações principais:

- `hsc-cs2-portal` consome Static API v2 gerada pelo `hsc-cs2-etl`
- `hsc-cs2-portal` consome conteúdo público vindo do `hsc-auth-api` via espelhos/contratos públicos quando aplicável
- `hsc-cs2-etl` consome MatchZy/SQLite e alguns contratos públicos/internos do `hsc-auth-api` para Seasons/Steam Profiles
- `hsc-backoffice-admin` consome `hsc-auth-api` para administração
- `hsc-brand-hub` é superfície pública de marca no apex e não deve ser confundido com Portal CS2
- `hsc-docs` documenta todos os contextos, mas não substitui os READMEs locais

---

## Última revisão

- Status: ativo
- Classificação: canônico
- Contexto: governança / mapa de repositórios
- Última revisão: 2026-05-08
