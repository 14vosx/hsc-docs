# Data Sources — MatchZy SQLite

## Objetivo

Documentar a fonte de dados primária do contexto Portal Estático do ecossistema HSC, registrando o papel do banco SQLite do MatchZy, seu path canônico no runtime real e sua relação estrutural com a Static API v2.

Este documento existe para registrar, de forma estável e auditável:

- qual é a fonte de dados primária da camada pública de stats
- onde vive o `matchzy.db` no ambiente real
- quais tabelas principais sustentam ranking, matches, maps e players
- por que essa fonte pertence ao lado jogo, mas impacta diretamente o portal
- quais riscos existem quando path, schema ou instância divergem do esperado
- como validar rapidamente se a fonte continua íntegra

---

## Navegação

### Entrada
- [Home da documentação](../README.md)
- [Portal Estático](./README.md)
- [Master Index](../00-governance/99-master-index.md)

### Origem operacional
- [Game Panel](../02-game-panel/README.md)
- [MatchZy](../02-game-panel/matchzy.md)
- [CS2 Server Configuration](../02-game-panel/cs2-server-configuration.md)
- [Plugins Installed](../02-game-panel/plugins-installed.md)

### Estrutura e transformação dos dados
- [SQL Queries and Views](./sql-queries-and-views.md)
- [ETL Bash Pipeline](./etl-bash-pipeline.md)
- [Static API v2](./static-api-v2.md)
- [JSON Contracts](./json-contracts.md)

### Infraestrutura e suporte
- [Docker Host](../01-infra-hostinger/docker-host.md)
- [Filesystem Paths and Permissions](../01-infra-hostinger/filesystem-paths-permissions.md)
- [Operational Runbooks](./operational-runbooks.md)
- [Observability and Troubleshooting](./observability-troubleshooting.md)

---

## Escopo

Este documento cobre:

- a fonte de dados principal da Static API v2
- o papel do SQLite do MatchZy
- o path canônico do `matchzy.db`
- o path de backup identificado no ambiente
- as tabelas principais consumidas pelo ETL
- a relação entre Game Panel e Portal Estático
- validações mínimas da fonte de dados
- riscos e problemas comuns ligados a essa camada

Este documento não cobre em profundidade:

- queries SQL linha a linha
- implementação completa do ETL Bash
- contratos JSON campo a campo
- operação detalhada do AMP
- troubleshooting detalhado do servidor CS2
- MariaDB da Auth API no Lightsail

Esses tópicos vivem em documentos próprios dos contextos `02-game-panel`, `03-portal-estatico` e `04-infra-aws-lightsail`.

---

## Estado atual

O estado operacional conhecido e reconciliado desta camada é:

- a fonte de dados primária da Static API v2 é o banco SQLite do MatchZy
- o banco ativo e canônico é o `matchzy.db`
- esse banco vive sob a árvore da instância oficial `MixHAXIXE01`
- o ETL do portal depende diretamente desse arquivo
- as tabelas estruturais principais já reconciliadas são:
  - `matchzy_stats_matches`
  - `matchzy_stats_maps`
  - `matchzy_stats_players`

Também ficou reconciliado que:

- existe um arquivo de backup histórico do `matchzy.db`
- esse backup não deve ser tratado como fonte ativa
- a leitura canônica do portal deve apontar para o banco vivo do runtime, e não para cópias de backup

---

## Source of truth / evidências

As principais evidências deste documento, nesta fase de reconciliação, são:

- documentação consolidada atual do ecossistema HSC
- blueprint técnico consolidado do HSC
- documentação reconciliada do Game Panel e do Portal Estático
- runtime real da Hostinger
- validação direta do path do `matchzy.db` no host
- validação direta das tabelas principais do banco no ambiente real

Enquanto novas mudanças não ocorrerem no runtime, o path validado diretamente no host prevalece como source of truth operacional.

---

## Relações com outros documentos

Este arquivo é complementar a:

- `docs/03-portal-estatico/README.md`
- `docs/03-portal-estatico/portal-estatico-architecture-runtime.md`
- `docs/03-portal-estatico/static-api-v2.md`
- `docs/03-portal-estatico/etl-bash-pipeline.md`
- `docs/03-portal-estatico/sql-queries-and-views.md`
- `docs/03-portal-estatico/references-inventory.md`
- `docs/02-game-panel/instance-mixhaxixe01.md`
- `docs/02-game-panel/matchzy.md`
- `docs/02-game-panel/references-inventory.md`

Este documento descreve a fonte SQLite do portal.  
Ele não substitui os documentos de ETL, SQL, MatchZy ou inventário da instância.

---

## Papel do `matchzy.db` no ecossistema

No HSC, o `matchzy.db` é a ponte entre:

- a operação real do servidor CS2
- a persistência estatística do MatchZy
- a geração pública da Static API v2

Em termos práticos:

- o Game Panel produz a fonte
- o Portal Estático consome essa fonte
- o browser nunca consome o banco diretamente
- o público consome apenas os JSONs derivados da v2

Regra canônica:

**o `matchzy.db` é a fonte estrutural da camada pública de stats, mas não é artefato público**

---

## Path canônico do banco ativo

O path canônico ativo do `matchzy.db`, validado no runtime real, é:

```bash
/home/amp/.ampdata/instances/MixHAXIXE01/counter-strike2/730/game/csgo/addons/counterstrikesharp/plugins/MatchZy/matchzy.db
```

Leitura canônica:

- este é o banco vivo do runtime
- este é o path que deve ser tratado como fonte principal da v2
- qualquer referência documental diferente precisa ser tratada como stale até validação contrária no ambiente real

---

## Path de backup identificado

Também foi identificado no ambiente um arquivo de backup do banco:

```bash
/home/amp/.ampdata/instances/MixHAXIXE01/counter-strike2/730/game/csgo/backup-hsc/MatchZy.live-dir.pre-upstream.20260317-205843/matchzy.db
```

Leitura correta desse path:

- é backup histórico
- não é o banco vivo do runtime
- não deve ser tratado como source of truth ativo do portal
- só deve ser usado para investigação histórica, contingência ou comparação

Regra importante:

- o ETL canônico deve mirar o path ativo
- backup não deve entrar na documentação como fonte principal

---

## Relação com a instância `MixHAXIXE01`

O `matchzy.db` vive sob a árvore da instância oficial:

- `MixHAXIXE01`

Isso é estrutural porque:

- o path do banco depende do nome e da árvore da instância
- renomeação ou migração dessa instância pode quebrar o ETL
- a camada pública do portal depende indiretamente da estabilidade dessa identidade operacional

Regra canônica:

- mudança de path do banco ou da instância é mudança estrutural real
- não é detalhe cosmético de organização interna

---

## Relação com MatchZy

O `matchzy.db` é produzido pelo MatchZy como persistência estatística do lado jogo.

Isso significa que:

- o banco não é “do portal”
- o banco nasce do runtime do servidor
- o portal apenas o consome por ETL
- se o MatchZy falhar em persistir, a v2 fica stale depois

Regra importante:

- problema na v2 pode nascer no lado game
- troubleshooting do portal precisa considerar essa dependência

---

## Tabelas principais já reconciliadas

As tabelas principais já reconciliadas como estruturais para a camada pública são:

### `matchzy_stats_matches`

Papel:
- base da coleção pública de partidas
- base de detalhe por match

Impacto na v2:
- `matches.json`
- `match/{id}.json`

---

### `matchzy_stats_maps`

Papel:
- base dos agregados por mapa
- base de detalhe por mapa

Impacto na v2:
- `maps.json`
- `map/{map}.json`

---

### `matchzy_stats_players`

Papel:
- base do ranking público
- base do dossiê público de jogador

Impacto na v2:
- `ranking.json`
- `player/{steamid64}.json`

---

## O que a v2 herda dessa fonte

A Static API v2 herda do `matchzy.db`:

- universo de partidas
- universo de jogadores
- universo de mapas
- contagens e agregados centrais
- identificadores estruturais como:
  - id de match
  - `steamid64`
  - nome/identidade de mapa

Isso significa que:

- drift de schema impacta a v2
- ausência de novas gravações impacta a v2
- mudança de path impacta a v2
- dano ou corrupção do banco impacta a v2

---

## O que o portal não deve fazer

O contexto do Portal Estático não deve:

- servir o `matchzy.db` publicamente
- tratar o banco como artefato da borda
- ler backup como se fosse banco vivo
- documentar MariaDB do lado game como fonte principal da v2
- assumir que a fonte é estável sem validar path e schema

Regra canônica:

**a fonte principal do portal é SQLite do MatchZy, não MariaDB e não backup manual**

---

## Dependências operacionais da fonte

Esta camada depende, no mínimo, de:

- Infra Hostinger saudável
- árvore `amp` íntegra
- instância `MixHAXIXE01` íntegra
- runtime do servidor CS2 saudável
- MatchZy funcional
- permissões corretas de leitura do ETL
- ausência de drift de path
- ausência de drift de schema destrutivo

Dependências cruzadas importantes:

- o Portal Estático depende desta fonte
- a saúde pública do portal não pode ser avaliada ignorando o lado jogo
- o path canônico do banco é dependência estrutural do ETL

---

## Comandos de validação

Os comandos abaixo representam a validação mínima desta fonte de dados.

### Validar presença do banco ativo

```bash
ls -l /home/amp/.ampdata/instances/MixHAXIXE01/counter-strike2/730/game/csgo/addons/counterstrikesharp/plugins/MatchZy/matchzy.db
```

### Validar path absoluto resolvido

```bash
realpath /home/amp/.ampdata/instances/MixHAXIXE01/counter-strike2/730/game/csgo/addons/counterstrikesharp/plugins/MatchZy/matchzy.db
```

### Validar tabelas principais

```bash
sqlite3 /home/amp/.ampdata/instances/MixHAXIXE01/counter-strike2/730/game/csgo/addons/counterstrikesharp/plugins/MatchZy/matchzy.db ".tables"
```

### Validar volume básico da fonte

```bash
sqlite3 /home/amp/.ampdata/instances/MixHAXIXE01/counter-strike2/730/game/csgo/addons/counterstrikesharp/plugins/MatchZy/matchzy.db "SELECT COUNT(*) FROM matchzy_stats_matches;"
sqlite3 /home/amp/.ampdata/instances/MixHAXIXE01/counter-strike2/730/game/csgo/addons/counterstrikesharp/plugins/MatchZy/matchzy.db "SELECT COUNT(*) FROM matchzy_stats_players;"
sqlite3 /home/amp/.ampdata/instances/MixHAXIXE01/counter-strike2/730/game/csgo/addons/counterstrikesharp/plugins/MatchZy/matchzy.db "SELECT COUNT(*) FROM matchzy_stats_maps;"
```

### Validar presença do backup identificado

```bash
ls -l /home/amp/.ampdata/instances/MixHAXIXE01/counter-strike2/730/game/csgo/backup-hsc/MatchZy.live-dir.pre-upstream.20260317-205843/matchzy.db
```

Regra prática:

- a validação operacional principal deve sempre começar pelo banco ativo
- o backup só entra na conversa quando o assunto for contingência ou comparação

---

## Problemas comuns

### 1. ETL aponta para path errado

Causas comuns:

- path antigo preservado em script
- renomeação ou reorganização da árvore da instância
- documentação stale

Impacto:
- a v2 deixa de atualizar
- troubleshooting pode culpar ETL quando a raiz é path

---

### 2. ETL lê backup em vez do banco ativo

Causas comuns:

- script ou operador usando path incorreto
- troubleshooting feito sobre arquivo errado
- confusão entre fonte viva e cópia histórica

Impacto:
- geração de dados stale
- falsa percepção de saúde da cadeia pública

---

### 3. Banco existe, mas tabelas não batem com o esperado

Causas comuns:

- drift de schema
- update do MatchZy
- leitura do banco errado
- banco parcialmente corrompido ou divergente

Impacto:
- SQLs quebram
- ETL quebra
- recursos específicos da v2 deixam de ser gerados

---

### 4. Partidas acontecem, mas a fonte não evolui

Causas comuns:

- MatchZy não está persistindo
- problema de escrita no runtime do jogo
- plugin carregado, mas funcionalmente degradado
- problema de permissões ou contexto da instância

Impacto:
- portal fica stale depois
- problema parece “do portal”, mas nasce no lado game

---

### 5. Banco ativo muda de lugar sem atualização documental

Causas comuns:

- migração manual
- rebuild da instância
- reorganização de árvore
- mudança operacional não registrada

Impacto:
- quebra indireta da cadeia pública
- drift entre docs, ETL e runtime

---

## Invariantes operacionais

Os invariantes conhecidos desta camada incluem:

- a fonte principal da v2 é SQLite
- o banco principal é o `matchzy.db`
- o banco ativo vive sob a árvore da instância `MixHAXIXE01`
- o path ativo reconciliado é o path em `addons/counterstrikesharp/plugins/MatchZy/`
- backup não é fonte ativa
- as tabelas principais consumidas pela v2 são `matches`, `maps` e `players`
- a v2 depende do estado real do lado jogo para continuar atualizando

Esses invariantes ajudam a preservar a leitura correta da cadeia de dados do portal.

---

## Limites deste documento

Este documento não detalha:

- todas as queries SQL usadas pelo ETL
- schema linha a linha de cada tabela
- comportamento interno do MatchZy
- troubleshooting aprofundado do runtime do CS2
- recuperação de corrupção de banco
- restore operacional a partir do backup identificado

Esses tópicos pertencem a documentos específicos ou exigem runbooks adicionais.

---

## Critério de pronto deste documento

Este documento pode ser considerado maduro quando:

- o path canônico do banco ativo estiver explícito sem ambiguidade
- o path de backup estiver corretamente separado do path vivo
- a relação entre `matchzy.db`, MatchZy e v2 estiver clara
- as tabelas principais estiverem formalizadas
- ele puder ser usado como referência confiável da fonte SQLite do portal sem depender do master legado

---

## Última revisão

- Status: ativo
- Classificação: canônico
- Contexto: portal estático / data sources — MatchZy SQLite
- Última revisão: 2026-03-18