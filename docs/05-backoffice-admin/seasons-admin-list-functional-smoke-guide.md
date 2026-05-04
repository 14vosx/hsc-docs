# Seasons Admin List Functional Smoke Guide

## Objetivo

Documentar o estado canônico da listagem administrativa inicial de `seasons` no Backoffice Admin do HSC.

Este documento existe para:

- registrar a definição de produto de Season como ciclo competitivo oficial
- documentar a leitura administrativa real disponível na Auth API
- registrar o comportamento atual da tela `/seasons`
- preservar o smoke funcional local da listagem
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
- contrato administrativo de leitura de `seasons`
- tela administrativa `/seasons`
- estados de loading, erro, vazio e tabela
- smoke local com Auth API, sessão dev e Backoffice
- decisão de timezone adotada no backend e na UI
- lacunas futuras conhecidas

Este guia não cobre:

- criação, edição, ativação ou fechamento pela UI do Backoffice
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
- o Backoffice apresenta a superfície administrativa inicial de leitura
- ranking, partidas por season e fechamento histórico ainda não existem no estado atual documentado

---

## Estado atual documentado

O estado funcional documentado é:

- a Auth API expõe leitura administrativa de Seasons
- o Backoffice Admin possui página funcional em `/seasons`
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

Lacunas da UI atual:

- create pela UI
- edit pela UI
- activate pela UI
- close pela UI
- ações de lifecycle na tela
- rota `/seasons/new`
- rota `/seasons/:slug/edit`

Observação sobre mutações:

- `POST /admin/seasons`, `PATCH /admin/seasons/:slug`, `POST /admin/seasons/:slug/activate` e `POST /admin/seasons/:slug/close` já existiam previamente no contrato administrativo
- este documento descreve a leitura admin e a listagem no Backoffice
- a UI atual não expõe essas mutações

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
- ela não deve ser interpretada como CRUD completo de Season
- ela é a superfície inicial de administração de Seasons no Backoffice

---

## Smoke funcional local

O smoke funcional local exercita a integração entre Auth API local e Backoffice local.

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
6. criar uma Season local temporária via `POST /admin/seasons`
7. recarregar `/seasons`
8. confirmar tabela renderizando o item criado
9. limpar o dado local temporário ao final do smoke por mecanismo local seguro

Observações:

- este smoke é local/dev
- este smoke é uma validação funcional local
- a criação temporária usa mutação administrativa preexistente apenas para preparar dado local de smoke
- este guia não assume endpoint de delete de Season
- dados temporários de smoke devem usar `slug` claramente descartável
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
- Backoffice e Portal exibem datas para o usuário em horário local
- para operação humana do HSC, a referência usual é `America/Sao_Paulo`

Leitura correta:

- UTC é a referência de transporte e persistência operacional da API
- a UI não deve inventar outro timezone de backend
- a apresentação local deve respeitar o navegador do usuário
- em comunicação operacional humana do HSC, usar São Paulo como referência explícita quando necessário

---

## Lacunas explícitas

Lacunas futuras conhecidas:

- o Backoffice ainda não cria Seasons pela UI
- o Backoffice ainda não edita Seasons pela UI
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
- a leitura administrativa real estiver documentada
- a tela `/seasons` estiver descrita sem prometer CRUD completo
- o smoke local estiver registrado como local/dev
- a decisão de timezone estiver registrada
- lacunas futuras estiverem separadas do estado atual
