# Admin API Contracts

## Objetivo

Documentar os contratos administrativos consumidos pelo Backoffice Admin do HSC, registrando a fronteira entre a SPA administrativa e a Auth API.

Este documento existe para registrar, de forma estável e auditável:

- quais superfícies administrativas o Backoffice consome
- quais contratos já são reais
- quais contratos ainda são alvo de evolução
- como o frontend deve interpretar autenticação, autorização e erros
- quais superfícies são normais de produto e quais são apenas auxiliares de desenvolvimento
- como preservar a separação entre frontend administrativo e backend autoritativo

---

## Escopo

Este documento cobre:

- contratos administrativos consumidos pelo Backoffice
- contrato real de introspecção de sessão
- contratos administrativos conhecidos de `news`
- contratos administrativos conhecidos de `seasons`
- interpretação de respostas HTTP relevantes
- diferença entre superfícies reais, auxiliares e futuras
- postura de consumo do frontend diante da Auth API

Este documento não cobre em profundidade:

- implementação interna da Auth API
- modelagem completa do banco
- detalhes de Nginx, systemd ou MariaDB
- fluxo final de autenticação administrativa de produção
- contratos públicos do portal
- documentação detalhada de Events além do estado atual conhecido

Esses tópicos vivem em outros documentos do ecossistema.

---

## Papel deste documento

Este arquivo define a leitura contratual do Backoffice sobre a Auth API.

Ele existe para responder perguntas como:

- qual endpoint o shell admin usa para resolver sessão?
- quais contratos admin já podem ser usados pelo frontend?
- o que é contrato normal de produto e o que é rota dev-only?
- como interpretar `401`, `403`, `404`, `409` e `5xx`?
- quais superfícies ainda dependem de ampliação no backend?

Regra importante:

- o frontend administrativo não inventa autoridade
- o backend continua sendo a fonte final de autenticação, autorização e invariantes de domínio

---

## Estado atual

O estado atual reconciliado do contexto é:

- o Backoffice Admin já consome sessão administrativa real
- a introspecção canônica da sessão atual já existe
- a Auth API continua sendo a autoridade final para autenticação e autorização
- o modelo administrativo continua sendo session-first com fallback break-glass no backend
- `news` já possui superfícies administrativas reais
- `seasons` já possui superfícies administrativas reais de escrita e ações
- a superfície administrativa de leitura para `seasons` ainda pode exigir evolução adicional
- `events` ainda não está reconciliado como contrato admin completo

Regra importante:

- este documento deve separar com clareza:
  - contratos reais
  - contratos auxiliares de desenvolvimento
  - contratos futuros ou ainda incompletos

---

## Relação com outros documentos

Este documento deve ser lido em conjunto com:

- `docs/05-backoffice-admin/README.md`
- `docs/05-backoffice-admin/architecture-runtime.md`
- `docs/05-backoffice-admin/frontend-structure.md`
- `docs/05-backoffice-admin/auth-rbac-and-guards.md`
- `docs/05-backoffice-admin/operational-runbooks.md`
- `docs/05-backoffice-admin/references-inventory.md`
- `docs/04-infra-aws-lightsail/auth-api-operations.md`

Leitura correta:

- este documento descreve o contrato consumido pelo admin
- ele não substitui a documentação operacional da Auth API
- quando houver conflito, a release real da Auth API e sua documentação operacional prevalecem

---

## Princípios contratuais do Backoffice

O consumo do backend pelo Backoffice deve obedecer a estes princípios:

- session-first
- backend como autoridade final
- frontend sem permissividade inventada
- ações sensíveis sem optimistic update ingênuo
- erros de auth claramente diferenciados
- lacuna explícita melhor do que contrato imaginário

Regra canônica:

**o Backoffice consome contratos reais; ele não deduz backend por conveniência**

---

## Contrato canônico de sessão administrativa

A superfície canônica de introspecção da sessão atual é:

- `GET /auth/session`

Esta superfície já deve ser tratada como **contrato real** do Backoffice.

### Papel do endpoint

`GET /auth/session` existe para:

- resolver a sessão atual no bootstrap do frontend
- sustentar guards assíncronos
- sustentar o shell administrativo
- permitir refresh de página sem perda de identidade
- servir como source of truth da sessão atual no frontend

### Requisição esperada

- método: `GET`
- credenciais/cookie: necessários
- sem body
- consumo normal com `withCredentials: true` no frontend quando aplicável ao ambiente

### Resposta esperada com sessão válida

```json
{
  "authenticated": true,
  "user": {
    "id": "1",
    "email": "admin@local.hsc",
    "name": "HSC Local Admin"
  },
  "role": "admin"
}
```

### Resposta esperada sem sessão válida

* `401 Unauthorized`

```json
{
  "authenticated": false
}
```

### Leitura contratual do frontend

O frontend deve interpretar assim:

* `200` + `authenticated: true`

  * sessão válida
  * shell pode ser liberado
  * guards podem permitir navegação

* `401`

  * sessão ausente, inválida, expirada ou não reconhecida
  * frontend deve convergir para estado `unauthenticated`

Regra importante:

* `GET /auth/session` representa sessão real
* ele não deve ser substituído semânticamente por `x-admin-key`

---

## Rota dev-only de bootstrap local

A superfície auxiliar de desenvolvimento validada localmente é:

* `POST /auth/dev/bootstrap-session`

### Papel da rota

Essa rota existe para:

* criar ou garantir usuário admin local
* criar sessão local de desenvolvimento
* devolver cookie admin
* destravar o Backoffice local sem depender do fluxo final de login administrativo

### Leitura correta

Esta rota é:

* útil para desenvolvimento local
* auxiliar
* controlada por flag de ambiente
* **não** parte do contrato normal de produto

### Resposta esperada com bootstrap ativo

* `200 OK`
* `Set-Cookie`
* body com `authenticated: true`, `user`, `role`

### Resposta esperada quando desabilitada

* `404` ou equivalente de indisponibilidade intencional

### Regra contratual importante

* o Backoffice pode usar essa rota para desenvolvimento local
* o Backoffice **não deve** modelar essa rota como login final de produção

---

## Contratos administrativos reais de News

As superfícies administrativas de `news` já são tratadas como reais no contexto atual.

### Leitura administrativa

* `GET /admin/news`

Função:

* listar conteúdo administrativo de news
* popular listagem admin
* suportar navegação inicial do domínio

### Escrita administrativa

Superfícies conhecidas:

* criação
* atualização
* publicação
* despublicação
* exclusão

Leitura contratual:

* essas operações exigem autenticação administrativa
* o backend continua sendo a autoridade final de permissão e invariantes
* a UI deve considerar essas ações sensíveis

### Expectativa do frontend

Para `news`, o frontend deve estar preparado para:

* loading
* success
* empty
* `401`
* `403`
* `404`
* `409` quando aplicável
* `5xx`

---

## Contratos administrativos reais de Seasons

As superfícies administrativas de `seasons` já existem no backend para escrita e ações de lifecycle.

### Escrita administrativa

Superfícies conhecidas:

* criação
* edição
* activate
* close

Leitura contratual:

* `seasons` já possui base real para mutações administrativas
* activate/close são ações sensíveis e devem ser tratadas como tal
* o backend impõe invariantes de domínio

### Leitura administrativa

A leitura admin de `seasons` deve ser tratada como:

* contrato parcialmente materializado
* ou contrato sujeito a ampliação, dependendo da release ativa

Leitura correta para o frontend:

* o domínio `seasons` já pode ser tratado como primeiro domínio forte do admin
* mas a superfície de leitura pode exigir complementação fina antes de um CRUD/admin ideal completo

Regra importante:

* o frontend não deve assumir que toda leitura admin de `seasons` já está no mesmo grau de maturidade que o lifecycle de escrita

---

## Contratos administrativos de Events

No estado atual, `events` ainda deve ser tratado como domínio administrativo em construção.

Leitura correta:

* não assumir contrato admin completo ainda
* não misturar fluxo público de presença com fluxo staff/admin
* só promover `events` a contrato real quando a superfície backend estiver reconciliada

---

## Interpretação contratual de status HTTP

O Backoffice deve tratar as respostas da Auth API com semântica explícita.

### `200` / `201`

Leitura:

* operação bem-sucedida
* estado final confirmado pelo backend

### `401 Unauthorized`

Leitura:

* sessão ausente, inválida ou expirada
* frontend deve convergir para estado `unauthenticated`
* guard deve bloquear rota protegida

### `403 Forbidden`

Leitura:

* operador autenticado sem escopo suficiente
* frontend deve exibir superfície de acesso negado
* não mascarar como erro genérico

### `404 Not Found`

Leitura:

* recurso inexistente ou deep link inválido
* distinguir de `403` quando possível

### `409 Conflict`

Leitura:

* conflito de domínio ou invariante
* exemplo: transição inválida de lifecycle
* não tratar automaticamente como bug de frontend

### `5xx`

Leitura:

* falha sistêmica do backend
* frontend não deve sinalizar falso sucesso
* deve expor erro legível e manter consistência visual

---

## Papel de `withCredentials` e cookie

No contexto do Backoffice, o consumo da sessão administrativa real exige que o frontend trate cookie/sessão de forma explícita quando aplicável ao ambiente.

Leitura contratual:

* chamadas de introspecção de sessão devem considerar credenciais
* ambientes locais podem usar proxy para evitar fricção de CORS/cookie
* ambientes publicados exigirão reconciliação real de origem, cookie e política de borda

Regra importante:

* o frontend não deve assumir que “funcionou localmente com proxy” resolve automaticamente o cenário publicado
* a publicação do admin exige reconciliação própria de cookie/CORS

---

## Relação entre sessão real e break-glass

O backend continua aceitando caminho break-glass por `x-admin-key` em superfícies administrativas relevantes.

Leitura contratual para o Backoffice:

* `x-admin-key` **não** é contrato de login do produto
* `x-admin-key` existe como fallback operacional e contingência backend
* o Backoffice normal deve operar por sessão real

Isso implica:

* o frontend consome `GET /auth/session`
* o frontend não deve estruturar fluxo de usuário em torno de `x-admin-key`
* o break-glass continua sendo assunto prioritariamente operacional da Auth API

---

## Modelos mínimos esperados no frontend

O frontend do Backoffice deve continuar operando com um modelo mínimo equivalente a:

### SessionStatus

```text
unknown | authenticated | unauthenticated | error
```

### AppRole

```text
viewer | editor | admin
```

### AuthSession mínima

```json
{
  "authenticated": true,
  "user": {
    "id": "1",
    "email": "admin@local.hsc",
    "name": "HSC Local Admin"
  },
  "role": "admin"
}
```

Esses modelos já conversam com o contrato real de `GET /auth/session`.

---

## Contratos reais vs auxiliares vs futuros

Para evitar ambiguidade, o contexto deve separar explicitamente:

### Contratos reais

* `GET /auth/session`
* superfícies admin reais de `news`
* superfícies admin reais de escrita/lifecycle de `seasons`

### Contratos auxiliares de desenvolvimento

* `POST /auth/dev/bootstrap-session`

### Contratos futuros ou ainda incompletos

* superfícies admin completas de `events`
* fluxo final de login administrativo de produção
* eventual ampliação da leitura admin de `seasons`

Regra importante:

* rota auxiliar de desenvolvimento não deve ser promovida a contrato normal de produto
* contrato futuro não deve ser documentado como já disponível

---

## Implicações para guards e shell

Como `GET /auth/session` já é contrato real, o frontend administrativo deve operar assim:

* rota protegida aguarda resolução assíncrona da sessão
* shell não precisa “adivinhar” a auth
* refresh de página deve poder recuperar a identidade atual
* `/login` local pode oferecer bootstrap controlado de sessão para desenvolvimento

Essa leitura já está alinhada com o estado implementado do Backoffice.

---

## Critérios de aceite contratuais do PR-1

Do ponto de vista de contratos, o PR-1 do Backoffice pode ser lido como aceitável quando:

* `GET /auth/session` está integrado
* o frontend resolve sessão antes de decidir guards
* a rota dev-only de bootstrap destrava o desenvolvimento local
* refresh em rota protegida mantém sessão válida
* `401` leva a estado `unauthenticated`
* `403` leva a superfície de acesso negado
* o frontend não trata `x-admin-key` como login de produto

---

## Gaps ainda abertos

Mesmo com a sessão real já materializada, ainda permanecem gaps relevantes:

### 1. Fluxo final de login administrativo de produção

Estado:

* bootstrap local existe
* login final de produção ainda não está documentado como fechado

### 2. Publicação real do Backoffice com cookie/CORS reconciliados

Estado:

* local funciona com proxy
* publicação real ainda exige fechamento operacional

### 3. Contrato admin completo de Events

Estado:

* ainda não reconciliado como superfície madura

### 4. Leitura admin de Seasons pode exigir ampliação fina

Estado:

* escrita e lifecycle já estão fortes
* leitura admin ainda pode pedir ajuste complementar por release

---

## Limites deste documento

Este documento não detalha:

* implementação interna do middleware de auth no backend
* schema completo do banco
* configuração de cookie linha a linha
* política detalhada de CORS no runtime publicado
* fluxo final de login administrativo de produção
* detalhes completos de domínio de `news`, `seasons` e `events`

Esses tópicos vivem em documentos complementares.

---

## Critério de pronto deste documento

Este documento pode ser considerado maduro quando:

* `GET /auth/session` estiver tratado como contrato real sem ambiguidade
* a rota dev-only estiver claramente delimitada como auxiliar de desenvolvimento
* `news` e `seasons` estiverem posicionados corretamente quanto à maturidade contratual
* a diferença entre sessão real e break-glass estiver explícita
* os códigos de resposta relevantes estiverem semanticamente claros
* ele puder servir como referência confiável para o frontend administrativo sem depender de hipótese sobre o backend

---

## Última revisão

* Status: ativo
* Classificação: canônico
* Contexto: backoffice admin / contratos com a Auth API
* Última revisão: 2026-03-19
