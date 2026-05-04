# News Functional Smoke Guide

## Objetivo

Documentar um smoke funcional/dev seguro para validar o domínio `news` de ponta a ponta no ecossistema HSC.

Este documento existe para:

- validar o fluxo administrativo real no Backoffice Admin
- validar a superfície admin publicada da Auth API
- validar a publicação pública via `/content/news`
- validar o espelhamento ETL na Hostinger
- validar o consumo final pelo Portal Estático
- deixar claro como limpar a notícia de smoke ao final do teste

---

## Navegação

### Entrada
- [Home da documentação](../README.md)
- [Backoffice Admin](./README.md)
- [Master Index](../00-governance/99-master-index.md)

### Documentos diretamente relacionados
- [News Admin API Contracts](./news-admin-api-contracts.md)
- [News Admin Integration and Evolution](./news-admin-integration-and-evolution.md)
- [News Admin Feature Implementation Spec](./news-admin-feature-implementation-spec.md)
- [News Admin Frontend Implementation Runtime](./news-admin-frontend-implementation-runtime.md)

### Relações com outros contextos
- [Portal Estático](../03-portal-estatico/README.md)
- [ETL Bash Pipeline](../03-portal-estatico/etl-bash-pipeline.md)
- [Nginx Publishing and Cache](../03-portal-estatico/nginx-publishing-cache.md)
- [Auth API Operations](../04-infra-aws-lightsail/auth-api-operations.md)

---

## Escopo

Este guia cobre:

- criação de draft
- edição
- refresh e deep link de `/news/:id/edit`
- publish
- unpublish
- delete
- validação de `/content/news` na Auth API
- execução controlada dos services ETL de News na Hostinger
- validação do JSON público em disco
- validação do JSON público via domínio
- validação visual no Portal Estático
- limpeza da notícia de smoke

Este guia não cobre:

- provisionamento de credenciais
- bypass de autenticação
- Swagger formal
- alteração de contrato da API
- criação ou alteração de timers systemd
- deploy/rollback da Auth API ou do Portal

---

## Estado validado

O domínio News atravessa:

```text
Backoffice Admin
→ Auth API PROD na AWS Lightsail
→ /content/news
→ ETL Hostinger
→ /var/www/api/cs2/v2/content/news
→ Nginx
→ Portal Estático
```

Estado PROD validado:

- Auth API PROD atualizada para o commit `728299e`
- `GET /admin/news/:id` publicado e reconciliado
- edição carrega conteúdo pelo detalhe administrativo
- refresh em `/news/:id/edit` funciona
- deep link ou nova aba em `/news/:id/edit` funciona
- publish funciona
- unpublish funciona
- delete funciona
- CORS administrativo permite `DELETE`

Header CORS validado:

```http
Access-Control-Allow-Methods: GET,POST,PATCH,DELETE,OPTIONS
```

ETL validado:

- `/opt/cs2-portal/bin/gen-content-news-cache.sh`
- `/opt/cs2-portal/bin/gen-content-news-items-cache.sh`
- `gen-content-news.service`
- `gen-content-news-items.service`
- `gen-all-v2.timer` segue ativo como automação agregada da v2
- antes da ativação dedicada, News dependia do `gen-all-v2.timer` para atualização automática, com latência maior
- `gen-content-news.timer` `enabled/active`
- `gen-content-news-items.timer` `enabled/active`
- `gen-content-news.timer` usa `OnUnitActiveSec=120s` e drop-in `OnActiveSec=120s`
- `gen-content-news-items.timer` usa `OnUnitActiveSec=300s` e drop-in `OnActiveSec=300s`
- expectativa conservadora: até ~2 min para lista e até ~5 min para detalhes
- os drop-ins existem em:
  - `/etc/systemd/system/gen-content-news.timer.d/override.conf`
  - `/etc/systemd/system/gen-content-news-items.timer.d/override.conf`

Smoke público validado:

- JSON público index acessível
- JSON público item acessível
- Portal Estático mostrou a notícia
- console do browser sem erro
- execução automática dos dois timers dedicados de News validada
- impacto observado no momento do teste foi baixo, com load após execução em `0.02, 0.01, 0.00`

Decisão operacional:

- a VPS Hostinger também roda o servidor CS2 via Game Panel/AMP
- estabilidade do jogo tem prioridade sobre latência de News
- não reduzir para 60s antes do upgrade previsto da VPS
- reavaliar a cadência depois do Game Panel 4

---

## Superfícies envolvidas

O smoke passa por três superfícies públicas diferentes. Elas têm papéis distintos e não devem ser tratadas como equivalentes.

### Auth API pública canônica

Fonte dinâmica canônica do conteúdo publicado:

```text
https://auth-api.haxixesmokeclub.com/content/news
https://auth-api.haxixesmokeclub.com/content/news/:slug
```

Uso no smoke:

- validar que a publicação feita pelo Backoffice chegou à Auth API
- validar o contrato público canônico antes do espelhamento

### Static API v2 materializada em disco

Artefatos gerados pelo ETL na Hostinger:

```text
/var/www/api/cs2/v2/content/news/index.json
/var/www/api/cs2/v2/content/news/<slug>.json
```

Uso no smoke:

- validar que o ETL materializou o index público
- validar que o item público por `slug` foi gerado quando aplicável

### Mirror same-origin público via Nginx

Superfície pública consumida pelo Portal Estático no domínio principal:

```text
https://haxixesmokeclub.com/content/news/
https://haxixesmokeclub.com/content/news/<slug>/
```

Uso no smoke:

- validar a entrega pública via Nginx
- validar o consumo same-origin usado pelo Portal Estático

---

## Premissas seguras

Antes de executar o smoke:

- usar uma notícia de teste com `slug` claramente descartável
- não registrar tokens, cookies ou secrets na documentação
- usar sessão administrativa válida obtida pelo fluxo normal do Backoffice
- tratar a UI do Backoffice como fluxo principal do smoke administrativo
- usar `curl` administrativo apenas como opção avançada, quando o operador já tiver sessão/cookie jar obtido por fluxo operacional seguro e documentado
- remover ou despublicar a notícia de smoke ao final

Exemplos abaixo usam placeholders:

```sh
AUTH_API_BASE="https://auth-api.haxixesmokeclub.com"
PUBLIC_BASE="https://haxixesmokeclub.com"
COOKIE_JAR="/tmp/hsc-news-smoke.cookies"
SLUG="news-smoke-YYYYMMDD-HHMM"
```

Regra importante:

- este guia não orienta exportar cookie do browser
- não registrar, copiar ou colar tokens, cookies ou secrets em logs, issues, PRs ou documentação
- se não houver cookie jar administrativo obtido por fluxo operacional seguro, executar as etapas administrativas pela UI

---

## Smoke funcional no Backoffice

### 1. Criar draft

No Backoffice Admin PROD:

1. abrir `/news/new`
2. preencher `slug`, `title` e `content`
3. salvar
4. confirmar navegação ou disponibilidade do item criado

Opção avançada via `curl`, somente com cookie jar administrativo obtido por fluxo operacional seguro e documentado:

```sh
curl -sS \
  -b "$COOKIE_JAR" \
  -c "$COOKIE_JAR" \
  -H "Content-Type: application/json" \
  -X POST "$AUTH_API_BASE/admin/news" \
  --data '{
    "slug": "'"$SLUG"'",
    "title": "News Smoke Test",
    "content": "Conteúdo inicial do smoke funcional de News."
  }'
```

Critério esperado:

- resposta `ok: true`
- retorno de `id`
- status inicial `draft`

### 2. Editar

No Backoffice Admin PROD:

1. abrir `/news/:id/edit`
2. confirmar que o formulário carregou `content`
3. alterar `title` ou `content`
4. salvar

Opção avançada via `curl`, somente com cookie jar administrativo obtido por fluxo operacional seguro e documentado:

```sh
NEWS_ID="<id-retornado-na-criacao>"

curl -sS \
  -b "$COOKIE_JAR" \
  -c "$COOKIE_JAR" \
  -H "Content-Type: application/json" \
  -X PATCH "$AUTH_API_BASE/admin/news/$NEWS_ID" \
  --data '{
    "title": "News Smoke Test Atualizada",
    "content": "Conteúdo atualizado do smoke funcional de News."
  }'
```

Critério esperado:

- resposta `ok: true`
- item retornado com título atualizado
- item segue em `draft`, salvo se já tiver sido publicado antes

### 3. Validar refresh e deep link

No browser:

1. abrir `/news/:id/edit`
2. executar refresh da página
3. abrir a mesma URL em nova aba autenticada

Critério esperado:

- formulário carrega conteúdo em todos os casos
- não aparece estado antigo de `missing-draft`
- a edição não depende de cache/listagem como fonte primária

### 4. Publicar

No Backoffice Admin PROD, acionar publish.

Opção avançada via `curl`, somente com cookie jar administrativo obtido por fluxo operacional seguro e documentado:

```sh
curl -sS \
  -b "$COOKIE_JAR" \
  -c "$COOKIE_JAR" \
  -X POST "$AUTH_API_BASE/admin/news/$NEWS_ID/publish"
```

Critério esperado:

- resposta `ok: true`
- `status: "published"`
- `published_at` preenchido

### 5. Validar `/content/news` na Auth API

```sh
curl -sS "$AUTH_API_BASE/content/news"
curl -sS "$AUTH_API_BASE/content/news/$SLUG"
```

Critério esperado:

- index contém a notícia publicada
- detalhe por `slug` retorna `content`
- contrato público não expõe `id` nem `status`

---

## Smoke controlado na Hostinger

Use esta etapa quando o teste for controlado e houver intenção explícita de materializar o cache público imediatamente, sem esperar a execução automática dos timers dedicados.

Leitura atual:

- `gen-content-news.timer` atualiza a lista em até ~2 min
- `gen-content-news-items.timer` atualiza detalhes em até ~5 min
- execução manual dos services continua útil para smoke controlado imediato

### 6. Rodar services ETL de News

Na Hostinger:

```sh
sudo systemctl start gen-content-news.service
sudo systemctl start gen-content-news-items.service
```

Critério esperado:

- os services concluem sem erro
- os artefatos públicos de News são atualizados

Observação:

- `gen-all-v2.timer` segue ativo como automação agregada da v2
- timers dedicados de News estão `enabled/active` em modo conservador
- o impacto observado no momento do teste foi baixo, mas isso não deve ser lido como garantia de ausência de impacto no servidor CS2

### 7. Validar JSON público em disco

Na Hostinger:

```sh
sudo test -f /var/www/api/cs2/v2/content/news/index.json
sudo find /var/www/api/cs2/v2/content/news -maxdepth 2 -type f | sort
```

Para inspecionar sem registrar conteúdo sensível:

```sh
sudo sed -n '1,120p' /var/www/api/cs2/v2/content/news/index.json
```

Critério esperado:

- `index.json` existe
- item publicado aparece no index
- JSON do item por `slug` existe quando aplicável

### 8. Validar JSON público via domínio

```sh
curl -sS "$PUBLIC_BASE/content/news/"
curl -sS "$PUBLIC_BASE/content/news/$SLUG/"
```

Critério esperado:

- Nginx serve o index público
- Nginx serve o item público
- resposta é JSON válido

### 9. Validar Portal Estático

No browser:

1. abrir o Portal Estático
2. navegar até a superfície pública de News
3. confirmar que a notícia aparece
4. abrir o detalhe da notícia, quando houver rota/superfície pública de detalhe
5. verificar o console do browser

Critério esperado:

- notícia publicada aparece no Portal Estático
- detalhe público carrega quando a UI expõe essa navegação
- console sem erro

---

## Limpeza do smoke

### 10. Remover ou despublicar notícia de smoke

Fluxo principal pela UI:

- despublicar a notícia de smoke; ou
- remover a notícia de smoke quando o teste pedir limpeza total.

Opção avançada A via `curl`: despublicar e preservar registro administrativo:

```sh
curl -sS \
  -b "$COOKIE_JAR" \
  -c "$COOKIE_JAR" \
  -X POST "$AUTH_API_BASE/admin/news/$NEWS_ID/unpublish"
```

Opção avançada B via `curl`: remover notícia de smoke:

```sh
curl -sS \
  -b "$COOKIE_JAR" \
  -c "$COOKIE_JAR" \
  -X DELETE "$AUTH_API_BASE/admin/news/$NEWS_ID"
```

Após a limpeza, quando for teste controlado, regerar o cache público:

```sh
sudo systemctl start gen-content-news.service
sudo systemctl start gen-content-news-items.service
```

Critério esperado:

- notícia deixa de aparecer em `/content/news` após unpublish ou delete
- JSON público deixa de expor a notícia após regeneração do cache
- Portal Estático deixa de exibir a notícia de smoke

### Rollback dos timers dedicados de News

O rollback operacional canônico dos timers dedicados vive em [systemd Automation](../01-infra-hostinger/systemd-automation.md).

Resumo do rollback conservador:

```sh
sudo systemctl disable --now gen-content-news.timer gen-content-news-items.timer
sudo systemctl reset-failed gen-content-news.timer gen-content-news-items.timer gen-content-news.service gen-content-news-items.service
```

Leitura esperada:

- timers dedicados deixam de agendar automaticamente
- services continuam disponíveis para execução manual controlada
- News volta a depender da automação agregada e/ou execução manual controlada, com latência maior

---

## Critérios de validação

O smoke é considerado saudável quando:

1. draft é criado com sucesso
2. edição carrega `content`
3. refresh de `/news/:id/edit` funciona
4. deep link ou nova aba funciona
5. publish funciona
6. Auth API `/content/news` expõe a notícia publicada
7. ETL materializa `/var/www/api/cs2/v2/content/news/index.json`
8. JSON público via domínio é acessível
9. Portal Estático mostra a notícia
10. console do Portal fica sem erro
11. notícia de smoke é removida ou despublicada

---

## Última revisão

- Status: ativo
- Classificação: canônico
- Contexto: `05-backoffice-admin`
- Domínio: `news`
- Última revisão: 2026-05-04
