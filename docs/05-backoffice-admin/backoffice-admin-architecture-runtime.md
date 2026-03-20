# Architecture Runtime

## Objetivo

Documentar a topologia funcional e a relação de runtime do Backoffice Admin do HSC.

Este documento existe para registrar, de forma objetiva e auditável:

- a topologia de alto nível do Backoffice como SPA administrativa
- a relação entre browser, Backoffice e Auth API
- a fronteira entre superfícies públicas, administrativas e backend dinâmico
- o papel do shell administrativo, do callback de auth e da sessão real
- o que já é realidade reconciliada e o que ainda permanece como expansão incremental

---

## Escopo

Este documento cobre:

- a topologia lógica do runtime do Backoffice
- a relação entre SPA administrativa e Auth API publicada
- o fluxo administrativo real de login por magic link
- a superfície administrativa básica publicada
- a distinção entre fluxo local de desenvolvimento e fluxo publicado
- os principais failure domains do runtime do admin

Este documento não cobre em profundidade:

- estrutura detalhada de diretórios do frontend
- contratos endpoint a endpoint
- runbooks detalhados de troubleshooting
- detalhes de host, Nginx e systemd da Auth API
- modelagem completa de domínio por feature

Esses tópicos vivem em documentos especializados.

---

## Estado atual

O estado arquitetural conhecido do contexto é:

- o Backoffice Admin já é uma superfície administrativa real do ecossistema
- a aplicação é uma SPA administrativa separada do Portal público
- a Auth API publicada sustenta a identidade, a sessão e os contratos administrativos base
- o modelo administrativo do backend já é session-first
- o login administrativo real publicado usa magic link
- a SPA publicada já opera com:
  - `/login`
  - `/auth/callback`
  - `/dashboard`
- a sessão administrativa publicada já é cookie-based e cross-subdomain
- o callback da SPA já resolve sessão real antes de liberar o dashboard
- `news`, `seasons` e `events` continuam sendo os domínios administrativos iniciais

Regra importante:

- este documento registra o runtime funcional do Backoffice
- onde a implementação de domínio ainda não estiver completa, isso deve ser tratado como expansão incremental, não como ausência do produto administrativo

---

## Source of truth / evidências

As principais evidências deste documento, nesta fase, são:

- `docs/00-governance/documentation-system.md`
- `docs/00-governance/99-master-index.md`
- `docs/04-infra-aws-lightsail/auth-api-operations.md`
- `docs/05-backoffice-admin/backoffice-admin-frontend-structure.md`
- `docs/05-backoffice-admin/auth-rbac-and-guards.md`
- `docs/05-backoffice-admin/admin-api-contracts.md`
- `docs/05-backoffice-admin/backoffice-admin-operational-runbooks.md`
- impl-log do checkpoint de auth admin magic link e cutover de migrations

O desenho do Backoffice já não é apenas alvo abstrato.
Ele nasce da reconciliação entre:

- a separação de camadas prevista no ecossistema
- o runtime real publicado da Auth API
- a jornada publicada de login administrativo
- a necessidade de uma superfície administrativa própria e estável

---

## Relações com outros documentos

Este arquivo é complementar a:

- `docs/05-backoffice-admin/README.md`
- `docs/05-backoffice-admin/backoffice-admin-frontend-structure.md`
- `docs/05-backoffice-admin/auth-rbac-and-guards.md`
- `docs/05-backoffice-admin/admin-api-contracts.md`
- `docs/05-backoffice-admin/backoffice-admin-operational-runbooks.md`
- `docs/05-backoffice-admin/backoffice-admin-references-inventory.md`

Este documento descreve a topologia e os fluxos do runtime do Backoffice.
Ele não substitui:

- o documento de estrutura frontend
- o documento de RBAC e guards
- o documento de contratos administrativos
- os runbooks operacionais

---

## Visão arquitetural de alto nível

A topologia funcional do Backoffice Admin é:

```text
Browser
  -> Backoffice SPA
    -> Auth API
      -> MariaDB / serviços internos do backend
```

Em paralelo, o restante do ecossistema preserva sua separação:

```text
Browser
  -> Portal Estático
    -> Static API v2
```

Leitura correta:

- o Backoffice não substitui o Portal
- o Portal não substitui o Backoffice
- a Auth API é a camada dinâmica comum que sustenta o admin

---

## Superfície publicada do Backoffice

A superfície publicada do produto administrativo é:

- `https://backoffice.haxixesmokeclub.com`

Superfícies reais já reconciliadas no runtime publicado:

- `/login`
- `/auth/callback`
- `/dashboard`

Regra importante:

- o Backoffice é uma superfície publicada distinta da Auth API
- a SPA administrativa não deve ser confundida com a borda pública do backend

---

## Relação com a Auth API publicada

A superfície pública canônica da Auth API é:

- `https://auth-api.haxixesmokeclub.com`

Do ponto de vista do Backoffice, essa Auth API sustenta:

- resolução de sessão
- fluxo de magic link admin
- callback de consumo do token
- contratos administrativos base
- decisão final de autenticação e autorização

Leitura correta:

- o frontend consome a Auth API como backend dinâmico central
- o frontend não é autoridade da sessão
- o frontend não é autoridade de permissão

---

## Fluxo real de autenticação administrativa

A jornada publicada e reconciliada do login administrativo é:

1. operador acessa `/login`
2. frontend solicita magic link
3. frontend chama `POST /auth/magic-link/request`
4. operador recebe email real
5. link aponta para `GET /auth/magic-link/consume?token=...`
6. backend consome o token, cria a sessão e emite cookie
7. backend redireciona para `/auth/callback?status=ok`
8. a SPA resolve a sessão via backend
9. a SPA navega para `/dashboard`

Em caso de falha, a SPA trata códigos como:

- `invalid_or_expired_link`
- `consume_failed`
- `missing_token`
- `forbidden`

---

## Sessão administrativa publicada

A sessão administrativa publicada possui as seguintes características:

- session-first
- cookie-based
- HttpOnly
- cross-subdomain entre Auth API e Backoffice
- autoridade final no backend

Consequências arquiteturais:

- a SPA precisa consultar o backend para validar sessão
- o callback não pode assumir autenticação sem rechecagem
- guards e shell dependem de resolução assíncrona da sessão
- a navegação protegida depende de sessão e papel efetivos

---

## Fluxo local de desenvolvimento

No desenvolvimento local, a SPA usa proxy para integração ergonômica com a Auth API local.

Leitura correta:

- o proxy local é ferramenta de desenvolvimento
- ele não é parte da arquitetura publicada

Resumo:

- **local:** proxy
- **publicado:** base URL real da Auth API

---

## Failure domains principais do runtime do admin

Os principais failure domains desta camada são:

### 1. Auth API indisponível

Efeito:
- sessão não resolve
- callback falha
- contratos administrativos podem falhar

### 2. Cookie de sessão não persistido ou não enviado

Efeito:
- `/auth/session` responde como não autenticado
- callback não conclui a jornada
- dashboard não é liberado

### 3. CORS entre Backoffice e Auth API

Efeito:
- chamadas de auth bloqueadas pelo navegador
- login admin fica inviável mesmo com backend funcional

### 4. SMTP / entrega de magic link

Efeito:
- request de login responde com sucesso genérico, mas email não chega
- operador não fecha a jornada de acesso

### 5. Drift de contrato ou de schema no backend

Efeito:
- auth base pode falhar em runtime
- callback pode retornar erro técnico
- superfícies administrativas podem degradar

---

## Domínios administrativos já previstos

Os domínios administrativos que já estruturam o runtime do produto são:

- `dashboard`
- `seasons`
- `news`
- `events`

Leitura correta:

- o shell e a auth base já foram materializados
- a expansão natural agora é por domínio
- `seasons` continua sendo o melhor primeiro domínio forte
- `news` tende a seguir logo depois
- `events` ainda depende de reconciliação contratual adicional

---

## Critério de pronto deste runtime

O runtime do Backoffice pode ser considerado minimamente consolidado quando:

- a SPA publicada carrega
- `/login` funciona
- o email de magic link é entregue
- `consume` fecha com sucesso
- `/auth/callback` resolve a sessão
- `/dashboard` abre autenticado
- a Auth API publicada permanece autoridade final da sessão
- a fronteira entre Backoffice e Portal permanece preservada

---

## Última revisão

- Revisado após a materialização do fluxo real de magic link admin publicado
- Revisado após a estabilização da sessão cookie-based cross-subdomain
- Revisado após a publicação do callback `/auth/callback`
- Revisado após o cutover da Auth API para migrations SQL explícitas
