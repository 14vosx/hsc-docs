# Seasons Admin Functional Smoke Guide

## Objetivo

Documentar o estado canônico da administração funcional de `seasons` no Backoffice Admin do HSC.

Este documento existe para:

- registrar a definição de produto de Season como ciclo competitivo oficial
- documentar a leitura, criação e edição administrativa reais disponíveis na Auth API
- registrar o comportamento atual das telas `/seasons`, `/seasons/new` e `/seasons/:slug/edit`
- preservar o smoke funcional local de listagem, criação e edição
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
- contratos administrativos de leitura, criação e edição de `seasons`
- tela administrativa `/seasons`
- tela administrativa `/seasons/new`
- tela administrativa `/seasons/:slug/edit`
- estados de loading, erro, vazio e tabela
- smoke local com Auth API, sessão dev, Backoffice, criação de dado draft e edição validada
- decisão de timezone adotada no backend e na UI
- adoção de Angular Material/CDK como base gradual de UI no Backoffice
- lacunas futuras conhecidas

Este guia não cobre:

- ativação ou fechamento pela UI do Backoffice
- ranking por season
- associação automática de partidas à season
- snapshot histórico ou hall of fame
- consumo rico de Seasons no Portal público
- geração ETL de ranking ou partidas por season
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

- a Auth API expõe leitura, criação e edição administrativa de Seasons
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
- a tela posiciona Seasons como ciclos competitivos oficiais do servidor HSC
- `/seasons/new` cria uma Season em `draft` por `POST /admin/seasons`
- ao criar com sucesso, a UI navega de volta para `/seasons`
- a Season criada aparece na listagem com status `Draft`
- a listagem `/seasons` possui ação `Editar`
- `/seasons/:slug/edit` carrega o detalhe por `GET /admin/seasons/:slug`
- `/seasons/:slug/edit` salva alterações por `PATCH /admin/seasons/:slug`
- o formulário é reutilizado em modo de edição
- o `slug` é somente leitura no modo de edição
- a edição permite alterar `name`, `description`, `start_at` e `end_at`
- a edição não altera o `slug`
- a UI trata Season inexistente como not-found
- a UI mapeia erro `season_closed` como `Season fechada não pode ser alterada.`

Lacunas da UI atual:

- activate pela UI
- close pela UI
- ações de lifecycle na tela

Observação sobre mutações:

- `POST /admin/seasons`, `PATCH /admin/seasons/:slug`, `POST /admin/seasons/:slug/activate` e `POST /admin/seasons/:slug/close` já existiam previamente no contrato administrativo
- este documento descreve a leitura admin, a listagem, a criação em `draft` e a edição inicial no Backoffice
- a UI atual expõe `POST /admin/seasons` pela rota `/seasons/new`
- a UI atual expõe `PATCH /admin/seasons/:slug` pela rota `/seasons/:slug/edit`
- a UI atual não expõe activate ou close

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
  "start_at": "2026-06-01T03:00:00.000Z",
  "end_at": "2026-07-02T02:59:00.000Z"
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
- `start_at`: opcional, data/hora UTC em ISO com `Z`
- `end_at`: opcional, data/hora UTC em ISO com `Z`

Regras importantes:

- o endpoint é endereçado pelo `slug`
- o payload de edição não altera o `slug`
- o backend continua sendo a fonte final de validação e invariantes de domínio
- quando o backend retorna erro `season_closed`, a UI apresenta `Season fechada não pode ser alterada.`

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
- a tela atual apenas exibe o status
- a tela atual não executa transições de lifecycle

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

Leitura de produto da tela:

- a tela existe para dar visibilidade administrativa ao ciclo competitivo oficial
- ela oferece entrada para criação e edição inicial de metadados
- ela não deve ser interpretada como lifecycle completo de Season
- ela é a superfície de administração funcional de Seasons no Backoffice

---

## Comportamento atual da tela `/seasons/new`

A rota protegida `/seasons/new` permite criar uma Season administrativa em estado `draft`.

Comportamentos esperados:

- a tela apresenta formulário baseado em Angular Material
- o formulário envia `POST /admin/seasons`
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

- a listagem `/seasons` apresenta ação `Editar` para cada item
- a tela de edição carrega o detalhe por `GET /admin/seasons/:slug`
- a tela reutiliza o formulário de Season em modo edit
- o campo `slug` é apresentado como somente leitura
- o formulário salva alterações por `PATCH /admin/seasons/:slug`
- a edição permite atualizar:
  - `name`
  - `description`
  - `start_at`
  - `end_at`
- a edição não altera o `slug`
- a tela trata Season inexistente como not-found
- a tela apresenta erro `season_closed` como `Season fechada não pode ser alterada.`
- a entrada de data usa `MatDatepicker`
- a entrada de hora usa campo textual `HH:mm`
- a tela não usa `input type="datetime-local"`
- a combinação de data + `HH:mm` usa o horário local do operador
- o payload enviado para a Auth API usa UTC ISO com `Z`

Fronteiras explícitas:

- a edição não implementa activate
- a edição não implementa close
- a edição não altera lifecycle
- a edição não associa partidas, ranking, Portal ou ETL à Season

Regra importante:

- a UI melhora a ergonomia da edição administrativa, mas o backend continua sendo a autoridade final sobre validação, autorização e invariantes de domínio

---

## Base de UI do Backoffice

Angular Material/CDK foi adotado como base de UI do Backoffice.

Leitura correta:

- Seasons Create já usa Angular Material como base visual e interativa
- Seasons Edit reutiliza a mesma base de formulário com `slug` somente leitura
- essa adoção orienta a evolução gradual das telas administrativas
- este documento não é um guia de design system completo
- este documento não promete migração visual completa de todas as telas

---

## Smoke funcional local

O smoke funcional local exercita a integração entre Auth API local e Backoffice local para listagem, criação e edição.

Ambiente do smoke:

- Auth API local
- sessão administrativa local via dev bootstrap session
- Backoffice Admin com `npm run start:dev`
- navegação local até `/seasons`

Fluxo de validação funcional:

1. iniciar a Auth API local
2. criar sessão administrativa local pelo endpoint de dev bootstrap
3. iniciar o Backoffice Admin com `npm run start:dev`
4. abrir `/seasons`
5. confirmar empty state quando não há Seasons locais
6. abrir `/seasons/new`
7. preencher `slug`, nome, descrição opcional, data/hora de início e data/hora de fim
8. criar a Season em `draft`
9. confirmar navegação de volta para `/seasons`
10. confirmar tabela renderizando o item criado com status `Draft`
11. acionar `Editar` na listagem
12. confirmar carregamento de detalhe por `GET /admin/seasons/:slug`
13. alterar campos editáveis de metadados, mantendo `slug` somente leitura
14. salvar a edição por `PATCH /admin/seasons/:slug`
15. confirmar retorno/atualização da listagem com os dados alterados
16. limpar o dado local temporário ao final do smoke por mecanismo local seguro

Observações:

- este smoke é local/dev
- este smoke é uma validação funcional local
- a criação temporária exercita a tela `/seasons/new` e a mutação administrativa `POST /admin/seasons`
- a edição temporária exercita a tela `/seasons/:slug/edit` e a mutação administrativa `PATCH /admin/seasons/:slug`
- este guia não assume endpoint de delete de Season
- dados temporários de smoke devem usar `slug` claramente descartável e ser removidos após validação
- o dado de smoke local validado foi removido após a validação
- cookies, tokens, chaves e secrets não devem ser registrados na documentação

Exemplo de nomenclatura segura para dado temporário:

```text
season-smoke-local-YYYYMMDD-HHMM
```

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

- o Backoffice ainda não ativa Seasons pela UI
- o Backoffice ainda não fecha Seasons pela UI
- não há ações de lifecycle na listagem
- não há ranking por season
- não há associação automática de partidas à season
- não há snapshot histórico ou hall of fame
- o Portal CS2 ainda não exibe visão rica de Season
- o ETL ainda não gera ranking ou partidas por season

Regra importante:

- estas lacunas não devem ser documentadas como estado implementado
- qualquer evolução de ranking, partidas, Portal ou ETL deve ser reconciliada em documento próprio quando existir evidência

---

## Critério de pronto deste documento

Este documento está pronto quando:

- a definição de produto de Season estiver explícita
- a leitura, criação e edição administrativa reais estiverem documentadas
- a tela `/seasons` estiver descrita sem prometer lifecycle completo
- a tela `/seasons/new` estiver descrita como criação em `draft`
- a tela `/seasons/:slug/edit` estiver descrita como edição inicial de metadados
- o smoke local estiver registrado como local/dev
- a decisão de timezone estiver registrada
- lacunas futuras estiverem separadas do estado atual
