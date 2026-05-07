# Seasons Admin Functional Smoke Guide

## Objetivo

Documentar o estado canônico da administração funcional de `seasons` no Backoffice Admin do HSC.

Este documento existe para:

- registrar a definição de produto de Season como ciclo competitivo oficial
- documentar a leitura, criação, edição e lifecycle administrativo básico disponíveis na Auth API
- registrar o comportamento atual das telas `/seasons`, `/seasons/new` e `/seasons/:slug/edit`
- preservar o smoke funcional local de listagem, criação, edição, ativação e fechamento
- separar estado implementado, intenção de produto e lacunas futuras

---

## Navegação

### Entrada
- [Home da documentação](../README.md)
- [Backoffice Admin](./README.md)
- [Master Index](../00-governance/99-master-index.md)

### Documentos diretamente relacionados
- [Admin API Contracts](./admin-api-contracts.md)
- [Frontend Structure](./backoffice-admin-frontend-structure.md)
- [References and Inventory](./backoffice-admin-references-inventory.md)
- [Auth API Operations](../04-infra-aws-lightsail/auth-api-operations.md)

---

## Escopo

Este guia cobre:

- definição funcional de Season no HSC
- contratos administrativos de leitura, criação, edição, ativação e fechamento de `seasons`
- tela administrativa `/seasons`
- tela administrativa `/seasons/new`
- tela administrativa `/seasons/:slug/edit`
- ações administrativas de lifecycle na listagem: `Editar`, `Ativar` e `Fechar`
- estados de loading, erro, vazio e tabela
- smoke local com Auth API, sessão dev, Backoffice, criação de dados `draft`, edição, ativação e fechamento validados
- decisão de timezone adotada no backend e na UI
- adoção de Angular Material/CDK como base gradual de UI no Backoffice
- lacunas futuras conhecidas

Este guia não cobre:

- ranking por season
- associação automática de partidas à season
- snapshot histórico ou hall of fame
- consumo rico de Seasons no Portal público
- materialização runtime/prod do ETL atualizado para Seasons
- validação pública real da Static API v2 para Seasons
- auditoria do Portal como consumidor visual de capa de Season
- geração ETL de ranking ou partidas por season além do contrato estático já documentado
- monetização, billing ou regras comerciais

---

## Definição de produto

Uma Season HSC é um ciclo competitivo oficial do servidor, não um CRUD genérico.

Formulação aprovada:

> Durante esta janela, as partidas oficiais do servidor contam para uma disputa própria, com ranking, destaques e fechamento histórico.

Leitura correta:

- a Season organiza uma janela competitiva oficial
- a Auth API governa os metadados e o lifecycle da Season
- o Backoffice apresenta as superfícies administrativas de leitura, criação e edição inicial
- ranking, partidas por season e fechamento histórico ainda não existem no estado atual documentado

---

## Estado atual documentado

O estado funcional documentado é:

- a Auth API expõe leitura, criação, edição, ativação e fechamento administrativo de Seasons
- o Backoffice Admin possui página funcional em `/seasons`
- o Backoffice Admin possui página funcional de criação em `/seasons/new`
- o Backoffice Admin possui página funcional de edição em `/seasons/:slug/edit`
- `/seasons` deixou de redirecionar para `/dashboard`
- a tela carrega dados da Auth API por contrato administrativo
- a tela possui estados de loading, erro, vazio e tabela
- a tabela exibe:
  - Nome
  - Slug
  - Status
  - Início
  - Fim
  - Atualizado em
- a listagem administrativa recebe `cover_image_url`
- a tela posiciona Seasons como ciclos competitivos oficiais do servidor HSC
- `/seasons/new` cria uma Season em `draft` por `POST /admin/seasons`
- `/seasons/new` permite informar URL da capa ou fazer upload de capa por `POST /admin/uploads`
- ao criar com sucesso, a UI navega de volta para `/seasons`
- a Season criada aparece na listagem com status `Draft`
- a listagem `/seasons` possui ações condicionais de lifecycle
- a ação `Editar` aparece para Seasons que não estão `closed`
- a ação `Ativar` aparece para Seasons que não estão `active` nem `closed`
- a ação `Fechar` aparece para Seasons `active`
- Seasons `closed` não apresentam ações úteis de lifecycle
- `/seasons/:slug/edit` carrega o detalhe por `GET /admin/seasons/:slug`
- `/seasons/:slug/edit` salva alterações por `PATCH /admin/seasons/:slug`
- o formulário é reutilizado em modo de edição
- o `slug` é somente leitura no modo de edição
- a edição permite alterar `name`, `description`, `start_at`, `end_at` e `cover_image_url`
- o formulário de Seasons exibe URL da capa, upload, preview e ação `Limpar capa`
- Seasons `closed` continuam read-only, com upload e limpeza de capa bloqueados
- a edição não altera o `slug`
- a UI trata Season inexistente como not-found
- `POST /admin/seasons/:slug/activate` é exposto pela ação `Ativar`
- `POST /admin/seasons/:slug/close` é exposto pela ação `Fechar`
- a UI confirma antes de ativar ou fechar
- após `activate` ou `close`, a listagem é atualizada
- a UI mapeia erros administrativos conhecidos de Seasons:
  - `season_already_active`
  - `no_active_season`
  - `season_closed`
  - `season_not_found`

Observação sobre mutações:

- `POST /admin/seasons`, `PATCH /admin/seasons/:slug`, `POST /admin/seasons/:slug/activate` e `POST /admin/seasons/:slug/close` existem no contrato administrativo
- `POST /admin/seasons` e `PATCH /admin/seasons/:slug` aceitam `cover_image_url`
- `PATCH /admin/seasons/:slug` com `cover_image_url: null` limpa a capa
- o código-fonte do ETL em `hsc-cs2-etl` já propaga `cover_image_url` nos JSONs estáticos de Seasons da Static API v2
- a validação do ETL citada para essa propagação foi local/temporária, com fake Auth API local, SQLite fixture temporário e `API_DIR` temporário, sem tocar `/var/www` e sem rodar produção
- este documento descreve a leitura admin, a listagem, a criação em `draft`, a edição inicial e o lifecycle básico no Backoffice
- a UI atual expõe `POST /admin/seasons` pela rota `/seasons/new`
- a UI atual expõe `PATCH /admin/seasons/:slug` pela rota `/seasons/:slug/edit`
- a UI atual expõe `POST /admin/seasons/:slug/activate` e `POST /admin/seasons/:slug/close` pela listagem `/seasons`

---

## Contratos consumidos da Auth API

### `GET /admin/seasons`

Objetivo:

- listar Seasons para uso administrativo no Backoffice

Resposta esperada:

```json
{
  "ok": true,
  "count": 1,
  "items": [
    {
      "id": 1,
      "slug": "season-smoke-local",
      "name": "Season Smoke Local",
      "description": "Season temporaria para smoke local.",
      "cover_image_url": null,
      "start_at": "2026-05-04T00:00:00.000Z",
      "end_at": "2026-05-31T23:59:59.000Z",
      "status": "draft",
      "created_at": "2026-05-04T12:00:00.000Z",
      "updated_at": "2026-05-04T12:00:00.000Z"
    }
  ]
}
```

### `GET /admin/seasons/:slug`

Objetivo:

- carregar uma Season específica por `slug` para uso administrativo

Resposta esperada:

```json
{
  "ok": true,
  "item": {
    "id": 1,
    "slug": "season-smoke-local",
    "name": "Season Smoke Local",
    "description": "Season temporaria para smoke local.",
    "cover_image_url": null,
    "start_at": "2026-05-04T00:00:00.000Z",
    "end_at": "2026-05-31T23:59:59.000Z",
    "status": "draft",
    "created_at": "2026-05-04T12:00:00.000Z",
    "updated_at": "2026-05-04T12:00:00.000Z"
  }
}
```

### `POST /admin/seasons`

Objetivo:

- criar uma Season administrativa em estado `draft`

Payload esperado:

```json
{
  "slug": "season-junho-2026",
  "name": "Season Junho 2026",
  "description": "Ciclo competitivo oficial de junho de 2026.",
  "cover_image_url": "http://127.0.0.1:3000/uploads/season-junho-2026.png",
  "start_at": "2026-06-01T03:00:00.000Z",
  "end_at": "2026-07-01T02:59:00.000Z"
}
```

Resposta esperada:

```json
{
  "ok": true,
  "id": 1,
  "slug": "season-junho-2026",
  "status": "draft"
}
```

Campos do payload:

- `slug`: obrigatório
- `name`: obrigatório
- `description`: texto opcional ou `null`
- `cover_image_url`: URL opcional ou `null`
- `start_at`: data/hora UTC em ISO com `Z`
- `end_at`: data/hora UTC em ISO com `Z`

Regra importante:

- o backend continua sendo a fonte final de validação e invariantes de domínio

### `PATCH /admin/seasons/:slug`

Objetivo:

- atualizar metadados editáveis de uma Season administrativa existente

Payload esperado:

```json
{
  "name": "Season Junho 2026",
  "description": "Ciclo competitivo oficial de junho de 2026 ajustado.",
  "cover_image_url": "http://127.0.0.1:3000/uploads/season-junho-2026-ajustada.png",
  "start_at": "2026-06-01T03:00:00.000Z",
  "end_at": "2026-07-02T02:59:00.000Z"
}
```

Para limpar capa:

```json
{
  "cover_image_url": null
}
```

Resposta esperada:

```json
{
  "ok": true,
  "slug": "season-junho-2026",
  "updated": true
}
```

Campos aceitos no payload:

- `name`: opcional
- `description`: texto opcional ou `null`
- `cover_image_url`: URL opcional ou `null`
- `start_at`: opcional, data/hora UTC em ISO com `Z`
- `end_at`: opcional, data/hora UTC em ISO com `Z`

Regras importantes:

- o endpoint é endereçado pelo `slug`
- o payload de edição não altera o `slug`
- `cover_image_url: null` limpa a capa
- Season `closed` é estado terminal no backend atual e não pode ser editada
- Season `closed` não permite upload ou limpeza de capa pela UI
- o backend continua sendo a fonte final de validação e invariantes de domínio
- quando o backend retorna erro `season_closed`, a UI apresenta `Season fechada não pode ser alterada.`

### `POST /admin/seasons/:slug/activate`

Objetivo:

- ativar uma Season administrativa como ciclo competitivo oficial em andamento

Payload esperado:

- sem corpo relevante

Resposta esperada:

```json
{
  "ok": true,
  "slug": "season-junho-2026",
  "status": "active"
}
```

Regras importantes:

- o endpoint é endereçado pelo `slug`
- a ativação pode rebaixar outra Season `active` para `draft`
- o backend continua sendo a fonte final de validação, autorização e invariantes de domínio

### `POST /admin/seasons/:slug/close`

Objetivo:

- fechar uma Season administrativa ativa

Payload esperado:

- sem corpo relevante

Resposta esperada:

```json
{
  "ok": true,
  "slug": "season-junho-2026",
  "status": "closed"
}
```

Regras importantes:

- o endpoint é endereçado pelo `slug`
- Season `closed` é estado terminal no backend atual
- uma Season fechada não pode mais ser editada
- o backend continua sendo a fonte final de validação, autorização e invariantes de domínio

### Campos do item administrativo

O item administrativo real é:

- `id`
- `slug`
- `name`
- `description`
- `start_at`
- `end_at`
- `status`
- `created_at`
- `updated_at`

### Status reconhecidos

Status administrativos reconhecidos:

- `draft`
- `active`
- `closed`

Regra importante:

- estes status já existem no contrato
- a tela atual exibe o status e apresenta ações condicionais de lifecycle
- transições de lifecycle continuam governadas pelo backend

---

## Comportamento atual da tela `/seasons`

A rota protegida `/seasons` carrega a listagem administrativa inicial de Seasons.

Comportamentos esperados:

- ao entrar, a página dispara carregamento da lista
- enquanto a chamada está em andamento, exibe loading state
- em falha de request, exibe error state mapeado
- quando a lista vem vazia, exibe empty state
- quando há itens, exibe tabela com dados administrativos
- datas são apresentadas ao usuário em horário local do navegador
- a tabela apresenta ações condicionais por status:
  - `Editar` para Seasons que não estão `closed`
  - `Ativar` para Seasons que não estão `active` nem `closed`
  - `Fechar` para Seasons `active`
  - nenhuma ação útil de lifecycle para Seasons `closed`
- antes de ativar, a UI solicita confirmação com o texto:

```text
Ativar esta season como ciclo competitivo oficial em andamento? Se houver outra season ativa, ela poderá deixar de ser ativa.
```

- antes de fechar, a UI solicita confirmação com o texto:

```text
Fechar esta season? Seasons fechadas não podem mais ser editadas.
```

- após ativar ou fechar, a listagem é carregada novamente

Leitura de produto da tela:

- a tela existe para dar visibilidade administrativa ao ciclo competitivo oficial
- ela oferece entrada para criação e edição inicial de metadados
- ela oferece lifecycle administrativo básico para ativar e fechar Seasons
- ela é a superfície de administração funcional de Seasons no Backoffice

---

## Comportamento atual da tela `/seasons/new`

A rota protegida `/seasons/new` permite criar uma Season administrativa em estado `draft`.

Comportamentos esperados:

- a tela apresenta formulário baseado em Angular Material
- o formulário envia `POST /admin/seasons`
- o formulário permite URL da capa, upload, preview e limpeza de capa
- o upload de capa usa `POST /admin/uploads` com `FormData` e `withCredentials`
- uploads aceitos: `image/jpeg`, `image/png` e `image/webp`
- URL vazia ou apenas whitespace normaliza para `null`
- a entrada de data usa `MatDatepicker`
- a entrada de hora usa campo textual `HH:mm`
- a tela não usa `input type="datetime-local"`
- a combinação de data + `HH:mm` usa o horário local do operador
- o payload enviado para a Auth API usa UTC ISO com `Z`
- ao criar com sucesso, a UI navega de volta para `/seasons`
- a Season criada aparece na listagem com status `Draft`

Motivo da entrada separada de data e hora:

- o controle nativo `datetime-local` mostrou experiência ruim no Firefox/Linux
- `MatDatepicker` + campo textual `HH:mm` mantém a UI alinhada ao Angular Material e deixa o comportamento de timezone explícito

Validações de UI:

- `slug` obrigatório
- nome obrigatório
- data de início obrigatória
- hora de início obrigatória
- data de fim obrigatória
- hora de fim obrigatória
- hora em formato `HH:mm`, entre `00:00` e `23:59`
- fim precisa ser depois do início

Regra importante:

- as validações de UI ajudam a experiência do operador
- o backend continua sendo a fonte final de validação

---

## Comportamento atual da tela `/seasons/:slug/edit`

A rota protegida `/seasons/:slug/edit` permite editar metadados administrativos de uma Season existente.

Comportamentos esperados:

- a listagem `/seasons` apresenta ação `Editar` para Seasons que não estão `closed`
- a tela de edição carrega o detalhe por `GET /admin/seasons/:slug`
- a tela reutiliza o formulário de Season em modo edit
- o campo `slug` é apresentado como somente leitura
- o formulário salva alterações por `PATCH /admin/seasons/:slug`
- a edição permite atualizar:
  - `name`
  - `description`
  - `cover_image_url`
  - `start_at`
  - `end_at`
- a tela recarrega URL e preview da capa quando `cover_image_url` existe
- a ação `Limpar capa` salva `cover_image_url: null`
- Seasons `closed` permanecem read-only, com upload e limpeza de capa bloqueados
- a edição não altera o `slug`
- a tela trata Season inexistente como not-found
- a tela apresenta erro `season_closed` como `Season fechada não pode ser alterada.`
- a entrada de data usa `MatDatepicker`
- a entrada de hora usa campo textual `HH:mm`
- a tela não usa `input type="datetime-local"`
- a combinação de data + `HH:mm` usa o horário local do operador
- o payload enviado para a Auth API usa UTC ISO com `Z`

Fronteiras explícitas:

- a edição não altera lifecycle
- a edição não associa partidas, ranking, Portal ou ETL à Season
- a edição de capa no Backoffice/Auth API não confirma publicação em Static API v2 ou consumo no Portal
- activate e close ficam na listagem `/seasons`, não no formulário de edição

Regra importante:

- a UI melhora a ergonomia da edição administrativa, mas o backend continua sendo a autoridade final sobre validação, autorização e invariantes de domínio

---

## Base de UI do Backoffice

Angular Material/CDK foi adotado como base de UI do Backoffice.

Leitura correta:

- Seasons Create já usa Angular Material como base visual e interativa
- Seasons Edit reutiliza a mesma base de formulário com `slug` somente leitura
- a listagem de Seasons usa ações administrativas condicionais com confirmação para mutações de lifecycle
- essa adoção orienta a evolução gradual das telas administrativas
- este documento não é um guia de design system completo
- este documento não promete migração visual completa de todas as telas

---

## Smoke funcional

### Capa de Season — smoke manual local

Validação já executada localmente no Backoffice:

1. criar Season em `/seasons/new` com upload de capa válido
2. confirmar que a URL da capa é preenchida automaticamente após upload
3. confirmar que o preview aparece no formulário
4. salvar a Season
5. abrir `/seasons/:slug/edit`
6. confirmar que a edição recarrega URL e preview da capa
7. acionar `Limpar capa`
8. salvar
9. recarregar `/seasons/:slug/edit`
10. confirmar que a capa permanece limpa e o preview não aparece
11. selecionar arquivo inválido
12. confirmar erro local

Critério esperado:

- `SeasonFormValue` inclui `cover_image_url`
- `CreateSeasonPayload` envia `cover_image_url`
- `UpdateSeasonPayload` permite `cover_image_url?: string | null`
- `SeasonsImageUploadApiService` usa `FormData` em `POST /admin/uploads` com `withCredentials`
- upload aceita `image/jpeg`, `image/png` e `image/webp`
- URL vazia ou apenas whitespace normaliza para `null`
- Seasons `closed` continuam read-only, com upload e limpeza bloqueados
- arquivo inválido mostra erro local

Leitura importante:

- este smoke manual local valida a fatia Backoffice/Auth API
- ele não inventa validação de produção
- um `404` observado em `/api/cs2/v2/season/<local-season>.json` pertence à Static API v2/Portal e não bloqueia a fatia Backoffice/Auth API

---

O smoke funcional preserva a cobertura de listagem, criação e edição já documentada e registra a validação de lifecycle para ativação e fechamento.

Ambiente do smoke:

- Auth API acessível ao Backoffice no ambiente validado
- sessão administrativa válida
- Backoffice Admin aberto na rota `/seasons`

Fluxo de validação funcional:

1. abrir `/seasons`
2. criar duas Seasons temporárias em `/seasons/new`
3. confirmar que ambas começam como `draft`
4. acionar `Ativar` na primeira Season temporária
5. confirmar que a UI solicita confirmação antes da ativação
6. confirmar refresh da listagem após ativação
7. confirmar que a primeira Season temporária fica `active`
8. acionar `Ativar` na segunda Season temporária
9. confirmar que a UI solicita confirmação antes da ativação
10. confirmar refresh da listagem após ativação
11. confirmar que a segunda Season temporária fica `active`
12. confirmar que a primeira Season temporária volta para `draft`
13. acionar `Fechar` na segunda Season temporária
14. confirmar que a UI solicita confirmação antes do fechamento
15. confirmar refresh da listagem após fechamento
16. confirmar que a segunda Season temporária fica `closed`
17. confirmar que a Season `closed` perde ações úteis de lifecycle
18. validar o estado final dos dados temporários
19. limpar os dados temporários ao final do smoke por mecanismo seguro

Observações:

- a criação temporária exercita a tela `/seasons/new` e a mutação administrativa `POST /admin/seasons`
- a edição administrativa segue documentada na tela `/seasons/:slug/edit` e na mutação `PATCH /admin/seasons/:slug`
- a ativação temporária exercita a ação `Ativar` e a mutação administrativa `POST /admin/seasons/:slug/activate`
- o fechamento temporário exercita a ação `Fechar` e a mutação administrativa `POST /admin/seasons/:slug/close`
- este guia não assume endpoint de delete de Season
- dados temporários de smoke devem usar `slug` claramente descartável e ser removidos após validação
- cookies, tokens, chaves e secrets não devem ser registrados na documentação

Exemplo de nomenclatura segura para dado temporário:

```text
season-smoke-local-YYYYMMDD-HHMM
```

Smoke funcional validado:

- foram criadas duas Seasons temporárias:
  - `hsc-ui-lifecycle-a`
  - `hsc-ui-lifecycle-b`
- ambas começaram como `draft`
- `hsc-ui-lifecycle-a` foi ativada
- `hsc-ui-lifecycle-b` foi ativada em seguida
- a listagem foi atualizada após as mutações
- `hsc-ui-lifecycle-a` voltou para `draft` quando `hsc-ui-lifecycle-b` ficou `active`
- `hsc-ui-lifecycle-b` foi fechada
- `hsc-ui-lifecycle-b` ficou `closed` e perdeu ações úteis de lifecycle
- estado final validado no banco:
  - `hsc-ui-lifecycle-a = draft`
  - `hsc-ui-lifecycle-b = closed`
- os dados temporários foram removidos após a validação

---

## Decisão de timezone

A decisão funcional documentada é:

- o backend trafega datas em UTC
- a configuração `mysql2` usa `timezone: "Z"`
- `timezone: "Z"` evita deslocamento indevido ao ler `DATETIME`
- o Backoffice combina data + `HH:mm` no horário local do operador ao criar Seasons
- o Backoffice combina data + `HH:mm` no horário local do operador ao editar Seasons
- o Backoffice envia datas de criação e edição para a Auth API como UTC ISO com `Z`
- o Backoffice exibe datas para o usuário em horário local
- para operação humana do HSC, a referência usual é `America/Sao_Paulo`

Leitura correta:

- UTC é a referência de transporte e persistência operacional da API
- a UI não deve inventar outro timezone de backend
- a apresentação local deve respeitar o navegador do usuário
- em comunicação operacional humana do HSC, usar São Paulo como referência explícita quando necessário

Exemplo validado:

- operador em São Paulo informa início `01/06/2026 00:00`
- payload/banco registra `start_at = 2026-06-01 03:00:00`
- operador em São Paulo informa fim `30/06/2026 23:59`
- payload/banco registra `end_at = 2026-07-01 02:59:00`

Exemplo validado de edição:

- API/banco original registra `start_at = 2026-06-01T03:00:00.000Z`
- UI em São Paulo mostra início `01/06/2026 00:00`
- operador em São Paulo altera fim para `01/07/2026 23:59`
- banco registra `end_at = 2026-07-02 02:59:00`
- dado local de smoke removido após validação

---

## Lacunas explícitas

Lacunas futuras conhecidas:

- não há administração visual de ranking por season no Backoffice neste guia
- não há associação automática de partidas à season
- não há snapshot histórico ou hall of fame
- o Portal CS2 ainda não exibe visão rica de Season
- o código-fonte do ETL já propaga `cover_image_url` para `seasons.json`, `season/{slug}.json` e `season/{slug}/ranking.json`
- seguem pendentes a materialização runtime/prod do ETL atualizado, a validação pública real da Static API v2 e a auditoria do Portal como consumidor visual da capa

Regra importante:

- estas lacunas não devem ser documentadas como estado implementado
- qualquer evolução de ranking, partidas, Portal ou ETL deve ser reconciliada em documento próprio quando existir evidência

---

## Critério de pronto deste documento

Este documento está pronto quando:

- a definição de produto de Season estiver explícita
- a leitura, criação, edição e lifecycle administrativo básico estiverem documentados
- a tela `/seasons` estiver descrita com ações condicionais de lifecycle
- a tela `/seasons/new` estiver descrita como criação em `draft`
- a tela `/seasons/:slug/edit` estiver descrita como edição inicial de metadados
- o smoke funcional de activate/close estiver registrado sem expor dados sensíveis
- a decisão de timezone estiver registrada
- lacunas futuras estiverem separadas do estado atual
