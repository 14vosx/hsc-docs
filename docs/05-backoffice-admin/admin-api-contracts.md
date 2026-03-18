# Admin API Contracts

## Objetivo

Documentar os contratos administrativos entre o Backoffice Admin do HSC e a Auth API, registrando quais superfícies o frontend administrativo deve consumir, quais invariantes precisam ser respeitadas e quais princípios governam leitura, mutação, autorização, erro e evolução contratual.

Este documento existe para registrar, de forma estável e auditável:

- as superfícies administrativas conhecidas e/ou esperadas da Auth API
- a relação contratual entre Backoffice e backend dinâmico
- as regras mínimas de leitura e mutação por domínio
- as expectativas de autenticação, autorização e auditoria
- o tratamento esperado de respostas, erros e invariantes de domínio
- os limites entre contrato funcional, implementação backend e UX administrativa

---

## Escopo

Este documento cobre:

- contratos administrativos consumidos pelo Backoffice
- superfícies de autenticação/identidade relevantes ao admin
- superfícies administrativas de `news`, `seasons` e `events`
- expectativas de payload e resposta em alto nível
- tratamento esperado de 401, 403, 404, 409 e 5xx
- regras de fail-closed e auditabilidade para mutações sensíveis
- princípios de versionamento e evolução contratual

Este documento não cobre em profundidade:

- implementação interna do backend
- SQL, transações e repositórios
- detalhes visuais da UI
- design detalhado de formulários
- runbooks operacionais completos
- contratos públicos do Portal estático
- shape final de todos os DTOs reais de release, quando ainda não reconciliados

Esses tópicos vivem em outros documentos do contexto ou em contextos canônicos adjacentes.

---

## Estado atual

O estado contratual conhecido, nesta fase, é:

- a Auth API é a camada dinâmica central do Backoffice
- o Backoffice deve consumir superfícies administrativas via contrato explícito
- `news` e `seasons` já possuem superfícies administrativas mais maduras
- `events` já aparece como domínio previsto com estrutura inicial
- o modelo administrativo é session-first, com break-glass mantido como contingência backend
- mutações administrativas relevantes devem respeitar postura fail-closed
- auditoria administrativa transacional é requisito crítico para writes sensíveis
- o frontend deve ser construído para contratos estáveis, mas sem assumir mais maturidade do que a release ativa realmente oferece

Regra importante:

- este documento distingue entre:
  - superfície conhecida/reconciliada
  - superfície-alvo esperada para o Backoffice
- quando houver lacuna, ela deve ser tratada como pendência explícita, não como fato concluído

---

## Source of truth / evidências do contrato

Este documento se ancora principalmente em:

- `docs/04-infra-aws-lightsail/auth-api-operations.md`
- `docs/98-legacy/HSC_MASTER_DOCUMENTATION.md`
- `docs/98-legacy/HSC_Master_Blueprint.md`
- `docs/05-backoffice-admin/architecture-runtime.md`
- `docs/05-backoffice-admin/auth-rbac-and-guards.md`

Leitura importante:

- o legado já formaliza superfícies administrativas relevantes para `news` e `seasons`
- o contexto da Auth API já registra operação session-first, trilha administrativa e fail-closed
- o Backoffice deve consumir essas superfícies sem contornar a autoridade do backend

---

## Princípios contratuais do contexto

Os contratos administrativos do Backoffice devem obedecer aos seguintes princípios:

- backend como autoridade final
- frontend consumindo contrato explícito
- session-first como fluxo principal
- break-glass fora da UX principal
- mutações relevantes sob fail-closed
- auditoria obrigatória para writes sensíveis
- respostas previsíveis e úteis para a UI
- invariantes de domínio preservadas no backend
- evolução contratual incremental
- sem acoplamento do Backoffice ao Portal para mutações

Princípio importante:

- o frontend pode otimizar UX, mas não pode “relaxar” contrato, autorização ou invariantes

---

## Topologia contratual

A relação contratual do Backoffice deve ser lida assim:

```text
Backoffice SPA
  -> Auth API
    -> superfícies de sessão/identidade
    -> superfícies administrativas de domínio
    -> respostas de sucesso/erro para leitura e mutação
```

O Backoffice não deve:

- mutar dados administrativos via Portal
- depender de ETL Bash
- depender da Static API v2 para writes
- contornar a Auth API em ações administrativas

---

## Classes de contrato do Backoffice

Os contratos administrativos do Backoffice se dividem em quatro grupos:

### 1. Contratos de sessão e identidade

Usados para:

- descobrir se existe sessão válida
- obter operador atual
- obter papel efetivo
- sustentar guards e UI por permissão

### 2. Contratos de leitura administrativa

Usados para:

- listar entidades
- carregar dados para edição
- carregar contexto administrativo

### 3. Contratos de mutação administrativa

Usados para:

- criar
- editar
- publicar
- ativar
- fechar
- excluir
- cancelar
- demais ações sensíveis de lifecycle

### 4. Contratos de erro e rejeição

Usados para:

- diferenciar autenticação ausente de falta de permissão
- refletir invariantes de domínio
- impedir falsa confirmação de sucesso na UI

---

## Estratégia geral de autenticação do contrato

O fluxo contratual do Backoffice deve ser session-first.

Leitura recomendada:

- a sessão válida sustenta o acesso normal à API administrativa
- a identidade atual deve ser resolvida por contrato explícito
- o papel efetivo deve ser lido da camada dinâmica
- a UI deve derivar comportamento a partir dessa resolução

Regra importante:

- o contrato normal do Backoffice não deve ser desenhado como se o operador precisasse enviar break-glass manualmente
- o break-glass continua como contingência operacional do backend

---

## Superfícies de sessão e identidade

### Estado conhecido

A documentação existente já indica superfícies de autenticação administrativa, verificação de identidade/sessão e diagnóstico administrativo.

A nomenclatura exata de cada rota deve ser reconciliada com a release ativa.

### Contrato funcional esperado

O Backoffice precisa de pelo menos uma superfície que responda:

- se há sessão válida
- quem é o operador atual
- qual é o papel efetivo
- se existe acesso administrativo válido

Superfícies-alvo naturais para esse papel podem assumir formas como:

```text
GET /auth/session
GET /auth/me
GET /me
GET /admin/whoami
```

Regra importante:

- este documento não fixa a rota final sem reconciliação de release
- mas fixa a necessidade contratual dessa superfície para o Backoffice

### Resposta mínima esperada

A resposta de identidade/sessão deve permitir ao frontend derivar, no mínimo:

- `authenticated`
- identidade do operador
- papel efetivo
- flags administrativas mínimas

Exemplo conceitual:

```json
{
  "authenticated": true,
  "user": {
    "id": "usr_123",
    "email": "staff@hsc.local",
    "name": "HSC Staff"
  },
  "role": "editor"
}
```

Esse shape é ilustrativo.

Regra importante:

- o contrato final deve privilegiar clareza e estabilidade
- o frontend não deve depender de shape ambíguo ou excessivamente acoplado à implementação interna

---

## Convenções de resposta para a UI administrativa

O contrato administrativo deve favorecer respostas que sejam simples para o frontend consumir.

Padrões desejáveis:

- leitura retorna payload claro
- mutação retorna recurso atualizado ou confirmação útil
- erros de domínio retornam mensagem interpretável
- 401 e 403 são distinguíveis
- 404 é limpo e sem semântica ambígua
- 409 ou equivalente é usado quando houver conflito de estado/invariante

O frontend deve evitar depender de:

- mensagens livres sem código ou sem estrutura mínima
- payloads heterogêneos demais entre endpoints equivalentes
- side effects implícitos não refletidos na resposta

---

## Contratos administrativos — News

## Estado conhecido

As superfícies administrativas de `news` já aparecem com maturidade razoável no legado reconciliado.

Superfícies conhecidas:

```text
GET    /admin/news
POST   /admin/news
PATCH  /admin/news/:id
POST   /admin/news/:id/publish
POST   /admin/news/:id/unpublish
DELETE /admin/news/:id
```

Leitura importante:

- `news` já é domínio administrativo explícito
- publish/unpublish e delete são ações sensíveis
- writes relevantes devem respeitar trilha transacional de auditoria

---

## Objetivo funcional de cada rota de News

### `GET /admin/news`

Responsável por:

- listar itens de notícia no contexto administrativo
- incluir drafts, publicados e outros estados internos úteis ao admin

Resposta esperada em alto nível:

- coleção administrativa de notícias
- campos suficientes para listagem, status e navegação de edição

Campos conceitualmente úteis:

- `id`
- `slug`
- `title`
- `status`
- `publishedAt`
- `updatedAt`

### `POST /admin/news`

Responsável por:

- criar item de notícia em estado inicial compatível com workflow administrativo

Entrada esperada em alto nível:

- slug
- título
- resumo/descrição curta, quando aplicável
- corpo/conteúdo
- metadados editoriais mínimos

Regra importante:

- o contrato final deve distinguir campos obrigatórios de opcionais
- o backend deve validar unicidade e consistência do domínio

### `PATCH /admin/news/:id`

Responsável por:

- editar item existente
- atualizar metadados ou conteúdo permitidos

### `POST /admin/news/:id/publish`

Responsável por:

- publicar item
- aplicar transição de lifecycle compatível com domínio

Regra importante:

- publish é ação sensível
- `published_at` ou equivalente deve ser controlado pelo backend

### `POST /admin/news/:id/unpublish`

Responsável por:

- reverter item para estado não publicado, conforme regra do domínio

### `DELETE /admin/news/:id`

Responsável por:

- excluir item sob política administrativa adequada

Regra importante:

- delete é ação sensível
- a UI deve exigir confirmação explícita
- a confirmação visual depende da resposta do backend

---

## Invariantes contratuais — News

As invariantes mínimas de `news` devem incluir:

- slug válido
- unicidade de slug, quando aplicável
- publish só em item apto para publicação
- unpublish só em item em estado compatível
- delete respeitando autorização adequada
- writes sensíveis sob audit transacional

A UI pode antecipar algumas validações básicas, mas:

- a validação final é sempre do backend

---

## Contratos administrativos — Seasons

## Estado conhecido

As superfícies administrativas de `seasons` já aparecem formalizadas no legado reconciliado.

Superfícies conhecidas:

```text
POST  /admin/seasons
PATCH /admin/seasons/:slug
POST  /admin/seasons/:slug/activate
POST  /admin/seasons/:slug/close
```

Superfície de leitura administrativa explícita ainda pode precisar de reconciliação de release para listagem/detail admin.

Para o Backoffice, a existência de leitura administrativa própria é natural e recomendada.

Superfícies-alvo esperadas para suporte pleno ao admin:

```text
GET /admin/seasons
GET /admin/seasons/:slug
```

Regra importante:

- se a release ativa ainda não expuser essas leituras, isso deve entrar como gap contratual explícito
- o Backoffice não deve fingir que esse contrato já existe se ele ainda não estiver materializado

---

## Objetivo funcional de cada rota de Seasons

### `GET /admin/seasons` (superfície-alvo esperada)

Responsável por:

- listar seasons no contexto administrativo
- expor status suficiente para operação e tomada de decisão

Campos conceitualmente úteis:

- `slug`
- `name`
- `status`
- `startsAt`
- `endsAt`
- `isActive`
- `updatedAt`

### `GET /admin/seasons/:slug` (superfície-alvo esperada)

Responsável por:

- carregar season para edição/admin detail

### `POST /admin/seasons`

Responsável por:

- criar season

Entrada esperada em alto nível:

- slug
- nome
- descrição, quando aplicável
- janela temporal, quando aplicável
- metadados administrativos mínimos

### `PATCH /admin/seasons/:slug`

Responsável por:

- editar campos permitidos da season

### `POST /admin/seasons/:slug/activate`

Responsável por:

- ativar a season
- executar a transição crítica de lifecycle

Regra importante:

- activate é ação sensível
- o backend deve preservar a invariante de “uma única season ativa”

### `POST /admin/seasons/:slug/close`

Responsável por:

- fechar a season ativa ou apta a fechamento, conforme regra do domínio

Regra importante:

- close é ação sensível
- o backend deve preservar coerência de estado

---

## Invariantes contratuais — Seasons

As invariantes mínimas de `seasons` devem incluir:

- slug válido e estável
- transições de status válidas
- apenas uma season ativa por vez
- activate e close exigem autorização adequada
- writes sensíveis sob audit transacional
- mutações inválidas devem falhar de forma explícita

Erros naturais deste domínio podem incluir:

- season não encontrada
- slug duplicado
- season já ativa
- transição inválida de estado
- operação não permitida para o papel atual

---

## Contratos administrativos — Events

## Estado conhecido

`events` já aparece no legado como estrutura inicial de domínio.

Superfícies registradas:

```text
GET    /content/events
POST   /admin/events
PATCH  /admin/events/:id
DELETE /admin/events/:id
POST   /admin/events/:id/confirm
```

Leitura importante:

- o domínio existe como intenção e estrutura inicial
- a separação entre gestão administrativa e interação pública ainda pede formalização cuidadosa

Regra importante:

- para o Backoffice, o foco inicial deve ser a gestão administrativa de eventos
- confirmação de presença do usuário final não deve ser confundida com operação administrativa da staff UI

---

## Superfícies-alvo esperadas para o Backoffice em Events

Para o Backoffice funcionar de forma saudável, as superfícies administrativas mínimas esperadas são:

```text
GET    /admin/events
GET    /admin/events/:id
POST   /admin/events
PATCH  /admin/events/:id
DELETE /admin/events/:id
```

Opcionalmente, o domínio pode evoluir para:

```text
POST /admin/events/:id/cancel
POST /admin/events/:id/reopen
```

Mas essas superfícies só devem entrar no contrato quando o lifecycle estiver reconciliado.

---

## Objetivo funcional de cada rota de Events

### `GET /admin/events` (superfície-alvo esperada)

Responsável por:

- listar eventos no contexto administrativo

Campos conceitualmente úteis:

- `id`
- `title`
- `type`
- `status`
- `scheduledAt`
- `createdAt`
- `updatedAt`

### `GET /admin/events/:id` (superfície-alvo esperada)

Responsável por:

- carregar um evento para edição/admin detail

### `POST /admin/events`

Responsável por:

- criar evento administrativo

Entrada esperada em alto nível:

- título/nome
- tipo
- data/hora
- metadados mínimos do evento

### `PATCH /admin/events/:id`

Responsável por:

- editar evento existente

### `DELETE /admin/events/:id`

Responsável por:

- excluir evento, quando essa for a política do domínio inicial

Regra importante:

- se o domínio migrar de delete físico para cancelamento lógico, o contrato deve ser revisto explicitamente

---

## Separação contratual em Events

O domínio `events` precisa de separação clara entre:

### Gestão administrativa

Consumida pelo Backoffice.

Exemplos:

- criar
- editar
- excluir/cancelar
- listar eventos administrativos

### Interação de usuário final

Consumida por Portal ou Account.

Exemplos:

- confirmar presença
- eventualmente cancelar presença
- ver meus confirms

Regra importante:

- `POST /admin/events/:id/confirm` é semanticamente ambíguo para o Backoffice
- a recomendação é não adotar esse endpoint como centro do admin
- confirmação de presença deve tender a uma superfície orientada ao usuário final, fora do fluxo principal do Backoffice

---

## Contratos de erro e rejeição

O Backoffice precisa de contratos de erro previsíveis.

### 401 — Não autenticado

Usar quando:

- não há sessão válida
- a sessão expirou
- a identidade não pode ser reconhecida

Comportamento esperado da UI:

- interromper fluxo protegido
- redirecionar para login
- limpar estado local inconsistente

### 403 — Sem permissão

Usar quando:

- o operador está autenticado
- mas não possui permissão suficiente para a rota ou ação

Comportamento esperado da UI:

- mostrar acesso negado ou erro contextual
- não mascarar como erro genérico

### 404 — Recurso ausente

Usar quando:

- item não existe
- slug/id não é encontrado
- recurso já foi removido ou nunca existiu

### 409 — Conflito de estado / invariante

Usar quando:

- season já ativa
- transição inválida
- slug duplicado sob política de conflito
- ação incompatível com o estado atual

### 422 ou erro equivalente de validação

Usar quando:

- payload é semanticamente inválido
- campos obrigatórios estão ausentes
- formato ou regra de negócio falha

### 5xx — Falha interna

Usar quando:

- houve falha real de backend
- a operação não pôde ser concluída
- fail-closed negou persistência por pré-requisito interno não atendido

Regra importante:

- a UI deve distinguir falha de validação, falha de permissão e falha sistêmica

---

## Fail-closed e auditoria administrativa

As mutações administrativas relevantes do Backoffice devem assumir explicitamente a política:

```text
No audit, no write
```

Leitura correta:

- se a mutação sensível não puder registrar auditoria conforme exigido
- a operação deve falhar
- a UI não deve tratar essa falha como sucesso parcial

Isso impacta especialmente:

- publish/unpublish/delete de news
- activate/close de season
- demais writes sensíveis que o domínio passe a adotar

Consequência para o frontend:

- sem optimistic update ingênuo para ações críticas
- confirmação visual só após resposta de sucesso
- erro deve ser comunicado de forma clara ao operador

---

## Shape contratual mínimo recomendado

Sem fixar DTO final prematuramente, o contrato administrativo deve privilegiar shapes previsíveis.

### Recomendação para leitura de coleção

```json
{
  "items": []
}
```

ou shape equivalente estável, como:

```json
{
  "data": []
}
```

### Recomendação para leitura de item único

```json
{
  "item": {}
}
```

ou shape equivalente estável.

### Recomendação para mutação bem-sucedida

Uma das opções abaixo:

```json
{
  "item": {}
}
```

ou

```json
{
  "ok": true
}
```

ou

```json
{
  "ok": true,
  "item": {}
}
```

Regra importante:

- escolher um padrão e manter consistência
- o frontend deve evitar acoplamento a múltiplos shapes arbitrários no mesmo domínio

---

## Campos administrativos mínimos por domínio

Sem fixar schema final, o frontend administrativo deve esperar os seguintes eixos de informação por domínio.

### News

- identidade (`id`)
- slug
- título
- status
- timestamps administrativos relevantes

### Seasons

- identidade natural (`slug`)
- nome
- status
- indicadores de atividade
- timestamps administrativos relevantes

### Events

- identidade (`id`)
- título/nome
- tipo
- status
- agenda (`scheduledAt` ou equivalente)
- timestamps administrativos relevantes

---

## Estratégia de mapeamento no frontend

O Backoffice não deve espalhar shape cru de API em toda a árvore de componentes.

A recomendação é:

- `services/` fazem chamada HTTP
- `models/` definem tipos do domínio
- `store/` consolida estado consumível pela UI
- o mapeamento da resposta deve acontecer perto da feature

Estrutura recomendada:

```text
features/<dominio>/
  services/
    <dominio>-api.service.ts
  models/
    <dominio>.model.ts
    <dominio>-dto.model.ts
  store/
    <dominio>.store.ts
```

Regra importante:

- DTO da API e model de tela não precisam ser a mesma coisa
- a separação ajuda a proteger a UI de mudanças contratuais localizadas

---

## Contratos e paginação

No MVP, paginação avançada não precisa ser obrigatória.

Mas o contrato deve permanecer compatível com evolução futura.

Abordagem recomendada:

- no início, aceitar coleções simples
- quando houver necessidade real, evoluir para paginação explícita
- não introduzir paginação complexa sem pressão real do volume

Se paginação existir, o contrato deve manter clareza mínima, por exemplo:

- `items`
- `page`
- `pageSize`
- `total`

---

## Contratos e filtros

No MVP, filtros devem ser simples.

Exemplos razoáveis:

- status
- busca textual
- slug ou identificador
- tipo de evento

Regra importante:

- o frontend não deve depender de filtros complexos inexistentes
- filtros só entram no contrato conforme necessidade real de operação

---

## Evolução contratual

A evolução dos contratos administrativos do Backoffice deve seguir estas regras:

- mudanças devem ser documentadas
- o runtime real deve prevalecer sobre hipótese antiga
- superfícies novas devem nascer com objetivo e papel claro
- mudanças breaking devem ser reconciliadas com docs e frontend
- o contexto `05-backoffice-admin` deve ser atualizado junto da mudança relevante

Mudanças que exigem atualização documental imediata:

- nova rota administrativa
- alteração de payload de mutação
- alteração de shape de identidade/sessão
- alteração de regra de RBAC refletida na resposta
- mudança de lifecycle de `news`, `seasons` ou `events`

---

## Riscos contratuais a evitar

### Risco 1 — o frontend assumir endpoints que ainda não existem

Efeito:
- documentação fictícia
- retrabalho
- gap entre produto e runtime

### Risco 2 — shapes inconsistentes entre rotas semelhantes

Efeito:
- mapeamento frágil
- store mais complexa
- componentes contaminados com exceções de contrato

### Risco 3 — confundir gestão administrativa com superfície pública

Efeito:
- acoplamento incorreto
- erosão da separação entre Backoffice e Portal/Account

### Risco 4 — tratar erro de permissão como erro genérico

Efeito:
- UX opaca
- troubleshooting pior
- autorização mal compreendida

### Risco 5 — ignorar fail-closed no desenho do frontend

Efeito:
- optimistic update indevido
- falso sucesso visual
- colisão com a política transacional do backend

### Risco 6 — consolidar cedo demais um contrato ainda imaturo de Events

Efeito:
- rigidez precoce
- documentação desalinhada da realidade
- aumento de custo de mudança

---

## Gaps contratuais atuais a observar

Os gaps mais prováveis, nesta fase, são:

- rota final canônica de sessão/identidade ainda a reconciliar
- superfícies admin de leitura de `seasons` ainda a confirmar no runtime
- superfícies admin de leitura de `events` ainda a formalizar
- separação definitiva entre confirmação pública e gestão admin de `events`
- shape final dos DTOs administrativos ainda a consolidar por release

Esses gaps não impedem o desenho do Backoffice, mas:

- precisam ser tratados explicitamente no backlog
- não devem ser escondidos no documento

---

## Ordem de implementação recomendada

A ordem mais saudável para materializar contratos no Backoffice é:

1. reconciliar contrato de sessão/identidade
2. reconciliar leituras admin mínimas
3. implementar `seasons`
4. implementar `news`
5. formalizar leituras admin de `events`
6. implementar mutações admin de `events`
7. revisar gaps, erros e invariantes por domínio

---

## Resumo executivo

Os contratos administrativos do Backoffice Admin do HSC devem ser construídos sobre uma regra simples:

- o Backoffice fala com a Auth API por contrato explícito
- a Auth API continua sendo a autoridade final
- mutações relevantes respeitam autorização, invariantes e fail-closed
- `news` e `seasons` já oferecem base mais madura
- `events` deve evoluir de forma incremental, sem confundir gestão admin com interação pública

O documento deve ser lido como:

- mapa contratual do admin
- registro das superfícies conhecidas
- declaração das superfícies-alvo necessárias
- guia para implementação disciplinada do frontend administrativo