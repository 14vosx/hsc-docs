# Operational Runbooks

## Objetivo

Documentar os runbooks operacionais do Backoffice Admin do HSC, com foco em desenvolvimento local, validação funcional da SPA administrativa e interação controlada com a Auth API.

Este documento existe para registrar, de forma estável e auditável:

- como subir o Backoffice localmente
- como conectar o Backoffice à Auth API local
- como validar o fluxo session-first no ambiente de desenvolvimento
- como diagnosticar falhas comuns de auth, proxy e guards
- como confirmar que a SPA está pronta para receber domínios administrativos reais
- quais fluxos ainda são apenas auxiliares de desenvolvimento

---

## Escopo

Este documento cobre:

- subida local do Backoffice
- operação local com proxy
- fluxo local de sessão administrativa
- bootstrap local de sessão via Auth API
- validação de guards e shell administrativo
- refresh autenticado em rota protegida
- troubleshooting funcional do admin local
- critérios mínimos de saúde operacional do Backoffice

Este documento não cobre em profundidade:

- deploy final do frontend administrativo
- hostname final do Backoffice publicado
- política final de cookies/CORS em produção
- fluxo final de login administrativo de produção
- operação detalhada da Auth API
- runbooks específicos de domínio como `seasons`, `news` ou `events`

Esses assuntos vivem em documentos próprios ou em fases posteriores do produto.

---

## Estado atual

O estado operacional conhecido do Backoffice, nesta fase, é:

- a SPA administrativa já existe como aplicação Angular 21 standalone-first
- a navegação protegida já está funcionando
- a store de sessão já consome introspecção real de sessão
- o fluxo local usa proxy para a Auth API
- a rota `/login` já pode bootstrapar sessão local de desenvolvimento
- o shell administrativo já carrega sob autenticação resolvida
- o refresh em `/dashboard` já preserva a leitura correta da sessão
- a base está pronta para receber o primeiro domínio administrativo real

Regra importante:

- o fluxo local atual é suficiente para desenvolvimento e validação do admin
- ele não deve ser confundido com o fluxo final de produção

---

## Relação com outros documentos

Este documento deve ser lido junto com:

- `docs/05-backoffice-admin/README.md`
- `docs/05-backoffice-admin/architecture-runtime.md`
- `docs/05-backoffice-admin/frontend-structure.md`
- `docs/05-backoffice-admin/auth-rbac-and-guards.md`
- `docs/05-backoffice-admin/admin-api-contracts.md`
- `docs/05-backoffice-admin/references-inventory.md`
- `docs/04-infra-aws-lightsail/auth-api-operations.md`

Leitura correta:

- este documento descreve como operar o Backoffice
- ele não substitui a documentação operacional da Auth API
- ele assume que a Auth API continua sendo a autoridade final de auth admin

---

## Princípios operacionais do Backoffice

Os runbooks do Backoffice seguem estes princípios:

- frontend administrativo separado do Portal
- session-first
- backend como autoridade final
- proxy local para reduzir fricção no desenvolvimento
- fluxo local explícito melhor que “mágica” implícita
- guards assíncronos para evitar navegação ambígua
- troubleshooting por sintoma observável
- falha explícita melhor que estado visual enganoso

Regra importante:

- o objetivo do runbook não é esconder complexidade
- é permitir operação previsível da SPA administrativa

---

## Pré-requisitos para operação local

Antes de subir o Backoffice localmente, estes pré-requisitos devem estar verdadeiros:

### 1. Repositório do Backoffice disponível localmente

Exemplo de diretório:

```text
~/work/hsc/hsc-backoffice-admin
```

### 2. Repositório da Auth API disponível localmente

Exemplo de diretório:

```text
~/work/hsc/hsc-auth-api
```

### 3. Auth API local funcional

A Auth API local deve ser capaz de:

* subir com `.env.local`
* responder `GET /health`
* responder `POST /auth/dev/bootstrap-session`
* responder `GET /auth/session`

### 4. Node e npm funcionais no ambiente local

A SPA do Backoffice depende do ambiente Node funcional para `npm install` e `npm start`.

### 5. Proxy local configurado no Backoffice

A aplicação local do Backoffice deve ter proxy para a Auth API, evitando depender prematuramente de ajuste fino de CORS/cookies cross-origin no navegador.

---

## Fluxo local canônico

O fluxo local canônico do Backoffice é:

1. subir a Auth API local
2. validar que a Auth API responde health
3. subir o Backoffice local com proxy
4. abrir `/login`
5. bootstrapar ou resolver a sessão atual
6. navegar para `/dashboard`
7. validar refresh autenticado
8. só então continuar com desenvolvimento de domínio

Regra importante:

* se a Auth API local não estiver funcional, o Backoffice não deve ser considerado operacionalmente saudável
* a SPA administrativa depende do contrato real de sessão

---

## Subir a Auth API local

No repositório `hsc-auth-api`, a forma canônica de subir o ambiente local é:

```bash
ENV_FILE=.env.local ./ops/dev.sh
```

### Resultado esperado

* MariaDB local sobe
* readiness do banco é aguardada
* dependências Node são instaladas
* a API sobe
* o schema é garantido
* a aplicação passa a escutar em `http://127.0.0.1:3000`

### Smoke mínimo esperado

```bash
curl -i http://127.0.0.1:3000/health
```

Resultado esperado:

* `200 OK`

---

## Subir o Backoffice local

No repositório `hsc-backoffice-admin`, a forma canônica de subir o ambiente local é:

```bash
npm start
```

### Pré-condição importante

O `package.json` local deve usar proxy config no script `start`, por exemplo:

```text
ng serve --proxy-config proxy.conf.json
```

### Resultado esperado

* a aplicação Angular sobe localmente
* chamadas para `/auth`, `/admin`, `/content` e `/health` são encaminhadas para a Auth API local
* o desenvolvimento local evita dependência imediata de CORS publicado

---

## Proxy local do Backoffice

O Backoffice usa proxy local para encaminhar chamadas ao backend.

### Papel do proxy

Ele existe para:

* simplificar desenvolvimento local
* reduzir fricção com CORS
* permitir cookies de sessão no fluxo local
* tornar o comportamento da SPA mais próximo do uso real

### Superfícies típicas proxadas

* `/auth`
* `/admin`
* `/content`
* `/health`

### Regra importante

* o sucesso local via proxy não substitui a futura reconciliação de cookies/CORS no ambiente publicado
* ele é uma ferramenta operacional de desenvolvimento

---

## Fluxo local de sessão administrativa

O fluxo local atualmente validado é:

### 1. Abrir `/login`

A página `/login` funciona como superfície inicial do admin local.

### 2. Criar sessão local de desenvolvimento

A UI local pode chamar a rota auxiliar:

* `POST /auth/dev/bootstrap-session`

Resultado esperado:

* a Auth API garante um usuário admin local
* cria uma sessão válida
* devolve `Set-Cookie`

### 3. Navegar para `/dashboard`

Após sessão válida:

* a store passa a reconhecer estado autenticado
* os guards deixam de bloquear o shell
* o operador entra no dashboard técnico

### 4. Refresh em `/dashboard`

Ao atualizar a página:

* os guards resolvem a sessão antes da navegação
* `GET /auth/session` confirma a identidade
* o shell continua acessível

### 5. Perda de sessão

Sem cookie válido:

* `GET /auth/session` responde `401`
* a store converge para `unauthenticated`
* a rota protegida deve levar de volta ao `/login`

---

## Runbook — validação local mínima do Backoffice

### Passo 1 — subir a Auth API

```bash
cd ~/work/hsc/hsc-auth-api
ENV_FILE=.env.local ./ops/dev.sh
```

### Passo 2 — validar health da Auth API

```bash
curl -i http://127.0.0.1:3000/health
```

### Passo 3 — subir o Backoffice

```bash
cd ~/work/hsc/hsc-backoffice-admin
npm start
```

### Passo 4 — abrir `/login`

Validar:

* a página abre
* o status da store é visível
* os botões de desenvolvimento aparecem

### Passo 5 — bootstrapar sessão local

Na UI:

* clicar em `Criar sessão local de desenvolvimento`

Resultado esperado:

* a sessão é criada
* a navegação vai para `/dashboard`

### Passo 6 — validar `/dashboard`

Confirmar:

* shell carregado
* header com status/role/usuário
* dashboard técnico com status de sessão
* nenhuma tela de erro inesperada

### Passo 7 — validar refresh

Atualizar a página em `/dashboard`.

Resultado esperado:

* a aplicação continua autenticada
* os guards resolvem a sessão corretamente
* não há redirecionamento incorreto para `/login`

---

## Runbook — validar a sessão fora da UI

Quando necessário, a sessão também pode ser validada diretamente na Auth API local.

### Criar cookie local

```bash
curl -i -c /tmp/hsc-auth-cookie.txt -X POST http://127.0.0.1:3000/auth/dev/bootstrap-session
```

### Introspectar sessão com cookie

```bash
curl -i -b /tmp/hsc-auth-cookie.txt http://127.0.0.1:3000/auth/session
```

### Testar ausência de sessão

```bash
curl -i http://127.0.0.1:3000/auth/session
```

Leitura correta:

* com cookie válido: `200` e `authenticated: true`
* sem cookie: `401` e `authenticated: false`

Esse runbook é útil quando você quer separar problema de SPA de problema de backend.

---

## Runbook — confirmar que os guards estão corretos

### Caso 1 — sem sessão

Acesse `/dashboard` sem cookie válido.

Resultado esperado:

* guard bloqueia
* usuário volta para `/login`

### Caso 2 — com sessão válida

Acesse `/dashboard` depois de bootstrapar sessão local.

Resultado esperado:

* guard permite
* shell abre

### Caso 3 — refresh em rota protegida

Atualize `/dashboard`.

Resultado esperado:

* guard aguarda resolução assíncrona da sessão
* rota permanece liberada
* sem flicker destrutivo

### Caso 4 — sessão perdida entre requests

Se a sessão expirar ou desaparecer:

* recarga ou nova resolução deve convergir para `unauthenticated`
* o admin deve voltar para `/login`

---

## Runbook — validar o header e o dashboard técnico

### Header

O header local deve refletir, no mínimo:

* status da sessão
* role atual
* identidade do operador, quando disponível

### Dashboard técnico

O dashboard local deve refletir, no mínimo:

* estado da sessão
* flags derivadas (`authenticated`, `adminAllowed`, etc.)
* operador atual
* erro atual, quando houver
* botão de recarga da sessão

Esse dashboard não é ainda um dashboard de produto final.
Ele funciona como superfície técnica de validação do PR-1.

---

## Runbook — resolver sessão atual pela UI

Na `/login`, o fluxo auxiliar recomendado inclui:

* botão para criar sessão local
* botão para resolver sessão atual

### Quando usar “Resolver sessão atual”

Use quando:

* você já tem cookie válido no navegador
* quer verificar se a store se recompõe corretamente
* quer testar comportamento sem recriar sessão

Resultado esperado:

* se houver cookie válido, navega para `/dashboard`
* se não houver cookie válido, permanece em estado não autenticado ou erro coerente

---

## Runbook — limpar o ambiente local

### Parar o Backoffice

Encerrar o `ng serve` com `Ctrl+C`.

### Parar a Auth API local

No repo `hsc-auth-api`:

```bash
./ops/stop.sh
```

### Parar removendo volumes locais

```bash
REMOVE_VOLUMES=true ./ops/stop.sh
```

Use esse caminho quando:

* o schema local ficou inconsistente
* o volume preservou tabela antiga
* você quer resetar completamente o banco local

---

## Runbook — reset controlado do banco local da Auth API

Quando o banco local estiver incompatível com a evolução do schema, o caminho correto é:

### 1. Parar e limpar volumes

```bash
cd ~/work/hsc/hsc-auth-api
REMOVE_VOLUMES=true ./ops/stop.sh
```

### 2. Subir novamente

```bash
ENV_FILE=.env.local ./ops/dev.sh
```

### 3. Revalidar sessão local

```bash
curl -i -c /tmp/hsc-auth-cookie.txt -X POST http://127.0.0.1:3000/auth/dev/bootstrap-session
curl -i -b /tmp/hsc-auth-cookie.txt http://127.0.0.1:3000/auth/session
```

Esse runbook é especialmente útil quando a tabela `sessions` local ficou com shape obsoleto.

---

## Troubleshooting — `/login` abre, mas o bootstrap falha

### Sintoma

O botão de criar sessão local falha.

### Causas comuns

* Auth API local não está rodando
* proxy do Angular não está ativo
* `AUTH_DEV_BOOTSTRAP_ENABLED` está ausente ou falso
* schema local da Auth API está incompatível
* sessão dev-only local está quebrada

### Como validar

1. testar `GET /health` na Auth API
2. testar `POST /auth/dev/bootstrap-session` direto com `curl`
3. verificar logs da Auth API local
4. revisar `.env.local` da Auth API

---

## Troubleshooting — `/dashboard` redireciona para `/login` mesmo com cookie

### Sintoma

Você já criou sessão local, mas a rota protegida continua caindo em `/login`.

### Causas comuns

* guard ainda não esperou a resolução da sessão
* `GET /auth/session` falhou
* cookie não está sendo reenviado corretamente
* proxy local não está encaminhando a chamada como esperado
* a store ficou em `unauthenticated` ou `error`

### Como validar

1. testar `GET /auth/session` com cookie via `curl`
2. testar recarga explícita da sessão pela UI
3. inspecionar erro atual na store
4. confirmar `withCredentials` no consumo da Auth API

---

## Troubleshooting — refresh em rota protegida quebra a sessão

### Sintoma

Entrou em `/dashboard`, mas ao atualizar a página volta para `/login`.

### Causas comuns

* guards síncronos antigos
* store não resolve a sessão antes da decisão do guard
* `GET /auth/session` falha no refresh
* cookie não está sendo preservado

### Leitura correta

Esse sintoma quase sempre indica falha de resolução de sessão antes do guard decidir.

---

## Troubleshooting — header ou dashboard mostram estado incoerente

### Sintoma

O header mostra `status` inesperado ou o dashboard exibe flags contraditórias.

### Causas comuns

* store não recarregada
* sessão mudou no backend
* erro capturado na store
* falha na interpretação do contrato de `GET /auth/session`

### Como agir

1. usar `Resolver sessão atual`
2. usar `Recarregar sessão`
3. testar `GET /auth/session` diretamente
4. verificar se houve perda de cookie

---

## Troubleshooting — Auth API sobe, mas o Backoffice não conecta

### Sintoma

A SPA sobe, mas chamadas para `/auth` falham.

### Causas comuns

* proxy não configurado no `npm start`
* `proxy.conf.json` ausente ou incorreto
* Auth API local não está ouvindo em `127.0.0.1:3000`
* backend indisponível
* path não proxado

### Como validar

1. revisar `package.json`
2. revisar `proxy.conf.json`
3. testar `curl http://127.0.0.1:3000/health`
4. reiniciar `ng serve`

---

## Troubleshooting — ambiente local da Auth API falha no boot

### Sintoma

O Backoffice não consegue autenticar porque a Auth API local nem sobe corretamente.

### Causas comuns

* MariaDB ainda não pronto
* volume local preservando schema antigo
* `.env.local` inconsistente
* script de operação local desatualizado

### Como agir

1. usar `./ops/dev.sh`
2. validar readiness do banco
3. se necessário, resetar volume com `REMOVE_VOLUMES=true ./ops/stop.sh`
4. subir novamente
5. revalidar `POST /auth/dev/bootstrap-session`

---

## Invariantes operacionais deste contexto

Os invariantes mínimos do Backoffice local, nesta fase, são:

### 1. O Backoffice não opera sem a Auth API

A SPA administrativa depende de contrato real de sessão.

### 2. O shell só deve abrir com sessão resolvida

Navegação protegida não pode depender de “talvez autenticado”.

### 3. O fluxo local normal deve ser previsível

* `/login`
* bootstrap ou resolução da sessão
* `/dashboard`
* refresh preservado

### 4. O proxy local faz parte da ergonomia oficial de desenvolvimento

Ele não é detalhe cosmético; é parte do fluxo local saudável.

### 5. O bootstrap local é auxiliar, não produto final

Ele existe para desenvolvimento e validação, não como jornada final de autenticação administrativa.

---

## Critério de pronto do PR-1 em operação local

Do ponto de vista operacional, o PR-1 do Backoffice pode ser considerado saudável quando:

* a Auth API local sobe de forma consistente
* o Backoffice local sobe de forma consistente
* `/login` abre corretamente
* o bootstrap local de sessão funciona
* `/dashboard` abre sob sessão válida
* refresh em `/dashboard` funciona
* `401` leva a estado não autenticado
* `403` permanece como superfície separada
* o dashboard técnico reflete a store real

---

## Limites deste documento

Este documento não define:

* o hostname final do Backoffice publicado
* a política final de CORS/cookie em produção
* o login administrativo final de produção
* o deploy final da SPA administrativa
* os runbooks específicos de `seasons`, `news` ou `events`

Esses pontos pertencem a documentos próprios ou a fases posteriores.

---

## Última revisão

* Status: ativo
* Classificação: canônico
* Contexto: backoffice admin / operação local e runbooks
* Última revisão: 2026-03-19

```

O próximo micro-passo natural é revisar `docs/05-backoffice-admin/references-inventory.md` para consolidar Angular 21, proxy local, store de sessão e superfícies reais de auth.
```
