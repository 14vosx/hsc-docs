# Auth API Operations

## Objetivo

Documentar a operação funcional da Auth API no contexto AWS Lightsail, registrando as superfícies relevantes, os invariantes operacionais e os fluxos essenciais de validação do backend dinâmico do ecossistema HSC.

Este documento existe para registrar, de forma estável e auditável:

- o papel operacional da Auth API no ecossistema
- as superfícies públicas e administrativas mais relevantes
- o modelo administrativo session-first com caminho break-glass
- o contrato operacional de introspecção de sessão administrativa
- os princípios de CORS e consumers autorizados
- a obrigatoriedade de auditoria administrativa
- os smoke tests mínimos de operação
- as dependências cruzadas do runtime

---

## Escopo

Este documento cobre:

- o papel operacional da Auth API
- endpoints públicos operacionais
- endpoints administrativos operacionais
- autenticação administrativa baseada em sessão
- contrato operacional de `GET /auth/session`
- bootstrap local controlado de sessão admin
- caminho break-glass administrativo
- regras operacionais de CORS
- auditoria administrativa
- invariantes de operação
- smoke tests e operações recorrentes

Este documento não cobre em profundidade:

- configuração detalhada do Nginx
- unit file do serviço
- fluxo completo de deploy e rollback
- backup e restore
- modelagem completa de domínio
- login final de produção para operadores
- backlog funcional futuro

Esses assuntos vivem em documentos próprios do contexto.

---

## Estado atual

A Auth API opera como backend dinâmico central do HSC no contexto AWS Lightsail.

O estado operacional conhecido desta camada inclui:

- execução em Node.js via systemd
- exposição pública por Nginx com TLS e reverse proxy
- persistência em MariaDB local
- superfícies públicas de conteúdo
- superfícies administrativas protegidas
- modelo administrativo session-first
- caminho break-glass por chave administrativa
- trilha de auditoria para mutações administrativas relevantes
- política de CORS orientada por allowlist explícita de origens
- base estrutural de sessão administrativa já materializada no código local
- introspecção administrativa por `GET /auth/session`
- bootstrap local controlado de sessão admin para desenvolvimento

A Auth API sustenta a metade dinâmica do ecossistema HSC, enquanto a Hostinger sustenta jogo, ETL e portal estático.

---

## Source of truth / evidências

As principais evidências deste documento, nesta fase, são:

- documentação canônica do contexto `04-infra-aws-lightsail`
- runtime reconciliado do Lightsail
- workflow real do repositório da Auth API em `ops/`
- implementação local validada do fluxo de sessão administrativa
- contrato local validado de:
  - `POST /auth/dev/bootstrap-session`
  - `GET /auth/session`
- release ativa reconciliada no Lightsail
- impl-logs e ajustes locais realizados para suportar o Backoffice Admin

Enquanto a migração canônica não estiver concluída em todos os detalhes, essas fontes seguem sendo usadas para reconciliação do estado operacional real.

---

## Relações com outros documentos

Este arquivo é complementar a:

- `docs/04-infra-aws-lightsail/README.md`
- `docs/04-infra-aws-lightsail/architecture-runtime.md`
- `docs/04-infra-aws-lightsail/node-systemd.md`
- `docs/04-infra-aws-lightsail/mariadb-local.md`
- `docs/04-infra-aws-lightsail/deploy-release-rollback.md`
- `docs/04-infra-aws-lightsail/observability-troubleshooting.md`
- `docs/04-infra-aws-lightsail/references-inventory.md`
- `docs/05-backoffice-admin/admin-api-contracts.md`
- `docs/05-backoffice-admin/auth-rbac-and-guards.md`

Este documento descreve a operação funcional da Auth API.  
Ele não substitui a documentação de runtime do host, banco, edge ou deploy.

---

## Papel operacional da Auth API

Do ponto de vista do ecossistema, a Auth API é o backend central das superfícies que não pertencem ao portal puramente estático.

A camada responde por:

- conteúdo dinâmico público relevante ao ecossistema
- superfícies administrativas protegidas
- autenticação e identidade administrativa
- validação de autorização em mutações sensíveis
- trilha administrativa persistida
- suporte operacional ao Backoffice Admin

Regra importante:

- a Auth API não deve ser tratada apenas como “API de conteúdo”
- ela também é a autoridade dinâmica de autenticação, autorização e write admin

---

## Endpoints públicos operacionais

As superfícies públicas operacionais conhecidas incluem:

### Health

- `/health`

Função:
- validar disponibilidade do runtime
- expor resposta básica de saúde da aplicação
- ajudar a separar falhas entre edge, app e banco

Observação operacional:
- o contexto também admite health local via `127.0.0.1:3000/health`

---

### Conteúdo News

Superfícies públicas relevantes:

- `/content/news`
- `/content/news/:slug`

Função:
- expor conteúdo público gerenciado na camada dinâmica
- servir como origem para consumo externo e, em alguns fluxos, para espelhamento no lado Hostinger

---

### Conteúdo Seasons

Superfícies públicas relevantes:

- `/content/seasons`
- `/content/seasons/active`
- `/content/seasons/:slug`

Função:
- expor catálogo ou estado ativo de temporadas
- sustentar consumo público ou integração com outras camadas do ecossistema

---

### Estruturas futuras ou incrementais

O backend já prevê base estrutural para outras superfícies dinâmicas, como Events, mas este documento não deve assumir maturidade funcional completa sem validação real da release ativa.

Regra editorial:
- documentar apenas o que já está presente no estado operacional reconciliado

---

## Endpoints administrativos operacionais

As superfícies administrativas conhecidas da Auth API incluem operações ligadas a:

- conteúdo News
- conteúdo Seasons
- autenticação administrativa
- introspecção de sessão administrativa
- inspeções técnicas ou administrativas restritas

Superfícies relevantes do estado atual:

- `/admin/schema`
- `/admin/news`
- mutações administrativas de `news`
- mutações administrativas de `seasons`
- `/auth/session`

Pontos operacionais importantes:

- superfícies administrativas não são parte do portal estático
- mutações administrativas devem respeitar autenticação e auditoria
- uma rota administrativa disponível não implica que ela possa ser usada sem sessão ou sem trilha adequada

---

## Session-first + break-glass

O modelo administrativo atual é session-first com caminho break-glass.

### Session-first

O modelo prioritário de operação administrativa é baseado em sessão persistida.

Isso significa:

- o fluxo normal administrativo deve privilegiar autenticação por sessão
- a aplicação deve reconhecer contexto autenticado persistido
- superfícies administrativas devem operar sob identidade autenticada rastreável
- o uso administrativo normal não deve depender exclusivamente de cabeçalho estático

Esse modelo favorece:

- melhor rastreabilidade
- menor dependência de segredo bruto por request
- maior aderência a operação administrativa real

### Estado atual do session-first

A base do modelo session-first já está materializada localmente com:

- coluna `role` em `users`
- tabela `sessions`
- resolução de sessão administrativa por cookie
- introspecção administrativa via `GET /auth/session`

O comportamento operacional esperado é:

- com sessão admin válida:
  - `GET /auth/session` retorna `200`
  - `authenticated: true`
  - `user`
  - `role`
- sem sessão válida:
  - `GET /auth/session` retorna `401`
  - `authenticated: false`

Regra importante:

- `GET /auth/session` deve refletir sessão real
- ele não deve usar `x-admin-key` como substituto semântico de sessão atual

---

### Break-glass administrativo

O contexto mantém um caminho break-glass baseado em chave administrativa para contingência e operação controlada.

Características esperadas:

- uso excepcional, não rotina
- escopo administrativo restrito
- necessidade de cuidado operacional
- relevância alta para troubleshooting, acesso de emergência e operações de continuidade

Regra operacional:

- break-glass existe como fallback controlado
- não deve substituir o fluxo normal session-first

No estado atual, isso significa:

- rotas administrativas ainda aceitam `x-admin-key`
- a camada de auth admin tenta primeiro sessão válida
- na ausência de sessão, pode cair para `x-admin-key`
- o Backoffice não deve modelar `x-admin-key` como jornada normal de login

---

## Contrato operacional de introspecção de sessão

A superfície operacional canônica para introspecção da sessão administrativa atual é:

- `GET /auth/session`

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

### Resposta esperada com sessão válida

- `401 Unauthorized`

```json
{
  "authenticated": false
}
```

### Função operacional

Essa rota existe para:

- sustentar bootstrap de shell administrativo
- permitir guards e UI session-first no Backoffice
- evitar que o frontend precise inferir sessão por tentativa cega de mutação
- centralizar a leitura da identidade administrativa atual

Regra importante:

- a rota de introspecção deve ser legível, pequena e estável
- o frontend administrativo deve consumi-la como source of truth da sessão atual

---

### Bootstrap local controlado de sessão

Para desenvolvimento local, o contexto agora admite uma rota controlada de bootstrap:

- `POST /auth/dev/bootstrap-session`

### Papel operacional

Essa rota existe para:

- criar ou garantir um usuário admin local
- criar uma sessão local válida em `sessions`
- devolver `Set-Cookie` com a sessão admin
- destravar desenvolvimento local do Backoffice sem depender de fluxo final de login

### Regras operacionais

- a rota é somente para desenvolvimento local
- ela depende de flag explícita no ambiente
- ela não deve ficar habilitada em produção
- ela não substitui o fluxo final de autenticação administrativa

### Controle esperado

O comportamento deve ser condicionado por:

- `AUTH_DEV_BOOTSTRAP_ENABLED=true`

Variáveis auxiliares esperadas:

- `AUTH_DEV_ADMIN_EMAIL`
- `AUTH_DEV_ADMIN_NAME`
- `ADMIN_SESSION_COOKIE`
- `ADMIN_SESSION_TTL_HOURS`

### Resultado esperado

Com a flag ativa, a rota deve:

- responder `200`
- garantir usuário admin local
- criar sessão válida
- devolver cookie `HttpOnly`

Sem a flag ativa, a rota deve:

- responder `404` ou equivalente de indisponibilidade intencional

---

### Fluxos de autenticação administrativa

Os fluxos conhecidos do contexto incluem:

- leitura de sessão administrativa atual
- resolução de identidade administrativa corrente
- proteção de rotas administrativas
- fallback break-glass para contingência controlada
- bootstrap local controlado para desenvolvimento

O fluxo local validado neste estágio é:

1. `POST /auth/dev/bootstrap-session`
2. API cria ou garante usuário admin local
3. API cria sessão persistida
4. API devolve cookie admin
5. `GET /auth/session` passa a responder `authenticated: true`
6. rotas admin passam a reconhecer identidade via sessão
7. fallback por x-admin-key continua disponível

Regra importante:

- o fluxo local de bootstrap não deve ser confundido com o fluxo final de produção
- ele existe para destravar desenvolvimento e validação do Backoffice

---

### CORS e consumers autorizados

A política operacional conhecida de CORS da Auth API é orientada por allowlist explícita.

Implicações:

- nem toda origem deve consumir a API
- consumers legítimos precisam estar explicitamente autorizados
- a resposta operacional da aplicação pode refletir as origens permitidas
- o controle de CORS pertence prioritariamente ao runtime da aplicação, e não apenas ao edge

Consumers autorizados podem incluir:

- superfícies oficiais do ecossistema HSC
- portal e domínios/subdomínios operacionais aprovados
- interfaces administrativas autorizadas
- camadas futuras de backoffice ou account, quando formalizadas

Regra operacional:

- alteração de CORS é mudança relevante
- mudanças de allowlist devem ser registradas de forma rastreável

### Observação local

Durante desenvolvimento do Backoffice local, o uso de proxy no frontend é estratégia válida para evitar dependência prematura de ajuste fino de CORS e cookies cross-origin.

---

### Auditoria administrativa

A auditoria administrativa é componente central do contexto.

O estado operacional conhecido já indica:

- existência de trilha administrativa persistida
- tabela operacional `admin_audit_log`
- endurecimento do princípio de auditoria em mutações sensíveis

Princípio operacional:

**mutações administrativas relevantes devem gerar rastreabilidade adequada**

Essa trilha é importante para:

- investigação
- conformidade interna
- reconstrução de ações
- análise de incidentes
- validação de comportamento administrativo

### Relação com `req.admin`

A camada administrativa deve preencher contexto suficiente para auditoria, incluindo quando possível:

- `userId`
- `via`
- papel efetivo
- identidade resolvida

No estado atual, a via administrativa pode ser:

- `session`
- `admin-key`

---

### Invariante fail-closed para mutações

O contexto já evoluiu para postura fail-closed em mutações administrativas sensíveis.

Interpretação operacional:

- não basta a rota estar exposta
- não basta haver autenticação parcial
- não basta haver intenção de escrita

Se a mutação administrativa não puder cumprir os pré-requisitos de segurança e rastreabilidade esperados, o comportamento saudável do sistema é negar a operação.

Em termos práticos, isso significa:

- autenticação, autorização e auditoria precisam estar coerentes
- o backend não deve aceitar mutações críticas em estado operacional ambíguo
- operação “sem audit, sem write” é a referência de endurecimento conhecida deste contexto

---

### Invariantes operacionais

Os invariantes operacionais conhecidos da Auth API incluem:

1. A aplicação deve responder health local e público

A validação mínima inclui:

- `http://127.0.0.1:3000/health`
- `https://auth-api.haxixesmokeclub.com/health`

2. O banco precisa estar pronto para conteúdo e auth

Isso inclui:

- `schema_meta` coerente
- `users` coerente
- `sessions` coerente
- tabelas de domínio operacionais
- readiness de banco antes do boot funcional da app

3. O fluxo admin normal deve preferir sessão

A operação administrativa moderna deve priorizar:

- introspecção de sessão
- identidade resolvida
- trilha rastreável

4. O fallback por chave deve continuar controlado

`x-admin-key` existe como contingência operacional, não como modelo principal de produto.

5. Rotas dev-only não podem vazar para produção

Em especial:

- `POST /auth/dev/bootstrap-session` não deve ficar habilitada no runtime público

---

### Schema mínimo relevante ao modelo admin atual

Sem virar documento completo de banco, o estado operacional atual já exige reconhecer estas estruturas como relevantes:

- `users`
- `profiles`
- `news`
- `seasons`
- `sessions`
- `schema_meta`

#### Campos relevantes adicionados ao modelo admin

`users`:

- `role`

`sessions`:

- `id`
- `user_id`
- `token_hash`
- `expires_at`
- `revoked_at`
- `created_at`
- `updated_at`

Regra importante:

- esse documento não substitui uma documentação detalhada de schema
- mas deve registrar as estruturas que já impactam a operação funcional

---

### Smoke tests mínimos

#### Smoke 1 — health local
```bash
curl -i http://127.0.0.1:3000/health
```

#### Smoke 2 — conteúdo público
```bash
curl -i http://127.0.0.1:3000/content/news
curl -i http://127.0.0.1:3000/content/seasons
```

#### Smoke 3 — admin break-glass
```bash
curl -i -H "x-admin-key: ..." http://127.0.0.1:3000/admin/news
curl -i -H "x-admin-key: ..." http://127.0.0.1:3000/admin/schema
```

#### Smoke 4 — bootstrap local de sessão
```bash
curl -i -c /tmp/hsc-auth-cookie.txt -X POST http://127.0.0.1:3000/auth/dev/bootstrap-session
```

#### Smoke 5 — introspecção da sessão com cookie
```bash
curl -i -b /tmp/hsc-auth-cookie.txt http://127.0.0.1:3000/auth/session
```

#### Smoke 6 — introspecção sem sessão
```bash
curl -i http://127.0.0.1:3000/auth/session
```

#### Smoke 7 — workflow local com scripts
```bash
ENV_FILE=.env.local ./ops/dev.sh
./ops/smoke-local.sh
```

Regra importante:

- smoke local deve permanecer verde antes de release
- sessão local e fallback break-glass devem ser validados explicitamente quando a mudança tocar auth admin

---

### Operação local do repositório

O repositório local da Auth API possui workflow real em `ops/`.

Pontos operacionais importantes:

- `./ops/dev.sh`

  - sobe dependências locais
  - aguarda MariaDB ficar pronto
  - instala deps
  - sobe a API com `.env.local`

  <br>

- `./ops/stop.sh`

  - para ambiente local
  - pode remover volumes
  - pode remover imagens

  <br>

- `./ops/smoke-local.sh`

  - valida baseline local do runtime

  <br>

- `./ops/release.sh <tag>`

  - só deve ser executado a partir de `main`
  - exige árvore limpa
  - exige sync com `origin/main`
  - roda `smoke-local.sh` antes da TAG

Regra importante:

- release de produção continua disciplinada por tag
- bootstrap local de sessão não muda o workflow de release do repo

---

### Problemas comuns

#### 1. Health responde, mas rotas de conteúdo falham

Causas comuns:

- falha de banco
- regressão de release
- schema incompatível
- erro específico do módulo de conteúdo

Impacto:

- contexto aparentemente vivo, mas funcionalmente degradado

---

#### 2. Sessão administrativa não persiste

Causas comuns:

- cookie ausente ou inválido
- sessão expirada ou revogada
- tabela `sessions` incompatível
- regressão em resolução de sessão
- schema antigo local

Impacto:

- fluxo normal administrativo falha
- Backoffice cai em `/login`
- introspecção responde `401`

---

#### 3. Break-glass funciona, mas fluxo normal falha

Causas comuns:

- problema em `sessions`
- cookie não sendo lido
- `GET /auth/session` com regressão
- problema em resolução de identidade admin

Impacto:

- operação administrativa degradada
- contingência ainda disponível

---

#### 4. Mutações administrativas são negadas indevidamente

Causas comuns:

- regra fail-closed atuando sobre contexto inconsistente
- auditoria indisponível
- identidade administrativa não reconhecida
- erro de autorização

Impacto:

- proteção ativa, mas necessidade de investigação operacional

---

#### 5. MariaDB sobe, mas a API falha logo no boot local

Causas comuns:

- banco ainda não pronto no primeiro boot
- readiness insuficiente em script local
- schema local incompatível
- volume antigo preservando tabela obsoleta

Impacto:

- falso negativo de runtime
- troubleshooting local ruidoso
- necessidade de reset controlado do volume

---

#### 6. Bootstrap local de sessão falha

Causas comuns:

- `AUTH_DEV_BOOTSTRAP_ENABLED` ausente ou falso
- tabela `sessions` incompatível
- inconsistência em `users.role`
- regressão no helper de criação de sessão

Impacto:

- impossível destravar fluxo local do Backoffice
- `/auth/session `permanece só em `401`

---

### Limites deste documento

Este documento não detalha:

- todos os contratos HTTP campo a campo
- configuração completa do Nginx
- fluxo detalhado de deploy
- estrutura completa do banco
- backup e restore
- design completo de domínio de News, Seasons ou Events
- login final de produção para operadores administrativos

Esses detalhes devem viver em documentos específicos ou impl-logs próprios.

---

### Critério de pronto deste documento

Este documento pode ser considerado maduro quando:
- as superfícies públicas e administrativas mais relevantes estiverem reconciliadas com a release ativa
- o modelo session-first + break-glass estiver formalizado sem ambiguidade
- `GET /auth/session` estiver tratado como contrato operacional real
- o bootstrap local controlado estiver claramente delimitado como dev-only
- a exigência de auditoria administrativa estiver clara
- a política de CORS estiver descrita em nível operacional
- os smoke tests cobrirem corretamente o núcleo funcional da Auth API
- ele puder ser usado como referência de operação funcional sem depender do master legado

---

### Última revisão

- Status: ativo
- Classificação: canônico
- Contexto: infraestrutura AWS Lightsail / operação funcional da Auth API
- Última revisão: 2026-03-19