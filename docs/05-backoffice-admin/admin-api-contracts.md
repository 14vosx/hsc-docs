# Auth, RBAC and Guards

## Objetivo

Documentar o modelo real de autenticação, autorização, guards e persistência de sessão do Backoffice Admin, refletindo o estado publicado session-first e a dependência explícita da Auth API como autoridade final.

Este documento existe para registrar, de forma estável e auditável:

- o modelo publicado session-first
- o uso de cookie administrativo cross-subdomain
- o papel dos guards do frontend
- a dependência de `reloadSession()`
- o comportamento do callback com retry curto
- a autoridade final do backend para auth e RBAC

---

## Escopo

Este documento cobre:

- autenticação administrativa session-first
- cookie-based auth entre subdomínios
- guards e shell do frontend
- store de sessão do Backoffice
- uso de `reloadSession()`
- callback administrativo e revalidação de sessão
- papel do backend na decisão final de permissão

Este documento não cobre em profundidade:

- contratos administrativos de domínio
- implementação detalhada do sistema de menu
- detalhes de layout do frontend

---

## Estado atual

O Backoffice publicado já opera no modelo session-first real.

O estado reconciliado inclui:

- login administrativo por magic link
- sessão administrativa persistida por cookie
- `GET /auth/session` como contrato real
- guards protegendo shell e rotas privadas
- callback `/auth/callback` consolidando o login
- retry curto no callback para reduzir race condition de primeiro acesso
- autoridade final da Auth API sobre autenticação e permissão

---

## Relações com outros documentos

Este arquivo deve ser lido junto com:

- `docs/05-backoffice-admin/admin-api-contracts.md`
- `docs/05-backoffice-admin/operational-runbooks.md`
- `docs/04-infra-aws-lightsail/auth-api-operations.md`
- `docs/04-infra-aws-lightsail/references-inventory.md`

---

## Modelo publicado: session-first

O modelo administrativo real do Backoffice é session-first.

Fluxo funcional resumido:

1. operador solicita magic link em `/login`
2. backend envia email real
3. operador consome o link no browser
4. Auth API cria sessão e emite cookie
5. callback do Backoffice reexecuta `GET /auth/session`
6. frontend navega para `/dashboard`

Regra importante:

- o frontend só considera o operador autenticado após introspecção real da sessão

---

## Cookie-based auth cross-subdomain

A sessão administrativa publicada depende de cookie HTTP-only emitido pela Auth API.

Topologia publicada:

- Auth API: `auth-api.haxixesmokeclub.com`
- Backoffice: `backoffice.haxixesmokeclub.com`

Para esse fluxo funcionar, o cookie publicado foi reconciliado para política compatível com produção HTTPS e navegação cross-subdomain.

Consequências práticas:

- o frontend precisa enviar requests com `withCredentials: true`
- o backend precisa aceitar credenciais via CORS
- o navegador precisa persistir e reenviar o cookie no fluxo de introspecção

---

## Papel da sessão no frontend

O frontend mantém uma store de sessão administrativa.

Essa store é responsável por:

- saber se a sessão está resolvida
- saber se o operador está autenticado
- armazenar identidade e papel atuais
- permitir guards e shell coerentes com o backend

### Regra fundamental

A store não é fonte autônoma de verdade.

Ela é um espelho do contrato de `GET /auth/session`.

---

## Papel de `reloadSession()`

`reloadSession()` é a operação explícita que reconsulta o backend para resolver a sessão atual.

Ela é usada em pontos críticos como:

- boot do shell administrativo
- callback do login após `status=ok`
- refresh em rota protegida
- ações explícitas de “resolver sessão atual” no ambiente administrativo

### Consequência operacional

- `reloadSession()` é o ponto que reconcilia frontend e backend
- se `reloadSession()` falha ou retorna não autenticado, o frontend não deve fingir que a sessão existe

---

## Guards do frontend

Os guards do Backoffice têm papel de contenção de navegação, não de autoridade final.

### Responsabilidades esperadas

- impedir entrada em rotas protegidas quando a sessão não está autenticada
- aguardar resolução assíncrona do contrato de sessão quando necessário
- redirecionar para `/login` quando não houver sessão válida
- permitir shell e dashboard apenas quando o backend confirmou a sessão

### O que os guards não fazem

- não substituem a autorização final do backend
- não tornam o operador autorizado só porque a rota abriu
- não eliminam a necessidade de tratar `403` reais de mutações administrativas

---

## Callback do login e retry curto

O callback publicado do Backoffice recebe o redirect do consume com:

- `status=ok`
- ou `error=<code>`

### Comportamento atual reconciliado

Em `status=ok`, o callback:

1. tenta resolver a sessão
2. se necessário, aplica retry curto
3. só então conclui erro ou navega para `/dashboard`

### Motivo do retry curto

No checkpoint do login publicado, foi observado um race condition leve entre:

- emissão do cookie no consume
- primeira tentativa de `GET /auth/session`

O retry curto foi introduzido para reduzir esse timing gap sem exigir `F5` do operador.

---

## Backend como autoridade final

A Auth API continua sendo a autoridade final para:

- autenticação
- resolução da identidade atual
- validação de papel
- validação de permissão
- bloqueio de mutações indevidas
- auditoria administrativa

Consequências para o frontend:

- um item de menu visível não implica autorização garantida
- um guard aprovado não implica mutação garantida
- `403` do backend deve ser tratado como decisão final

---

## Modelo inicial de RBAC

O Backoffice deve se manter preparado para RBAC explícito.

Hierarquia administrativa inicial coerente:

```text
viewer
editor
admin
```

### Leitura operacional

- `viewer`: leitura administrativa
- `editor`: edição controlada
- `admin`: ações administrativas sensíveis e lifecycle crítico

### Reconciliação com o backend

O papel que chega da Auth API é a referência real que governa o frontend.

A UI pode adaptar apresentação, mas não deve divergir semanticamente da autoridade do backend.

---

## Invariantes do modelo auth/rbac

1. o Backoffice published é session-first
2. a sessão administrativa é cookie-based
3. `withCredentials: true` é obrigatório nos contratos de sessão
4. `reloadSession()` é a operação principal de reconciliação
5. guards restringem navegação, não substituem autorização final
6. callback com retry curto faz parte do comportamento normal do login publicado
7. backend é a autoridade final de autenticação e permissão

---

## Critério de pronto deste documento

Este documento pode ser considerado reconciliado quando:

- session-first estiver explicitado como modelo real
- cookie cross-subdomain estiver descrito
- papel dos guards estiver claro
- dependência de `reloadSession()` estiver explícita
- callback com retry curto estiver registrado
- backend como autoridade final estiver inequívoco

Neste checkpoint, esse critério está atendido.
