# SQL Queries and Views

## Objetivo

Documentar o papel das queries SQL versionadas na geração da Static API v2 do contexto Portal Estático do ecossistema HSC.

Este documento existe para registrar, de forma estável e auditável:

- como a camada SQL participa da geração da v2
- onde vivem as queries operacionais do contexto
- quais grupos de query sustentam ranking, matches, players e maps
- quais regras de negócio já estão embutidas na camada SQL
- quais dependências técnicas, como SQLite JSON1 e compatibilidade de schema, precisam ser preservadas
- quais riscos de drift existem entre fonte de dados, SQL e contratos públicos

---

## Navegação

### Entrada
- [Home da documentação](../README.md)
- [Portal Estático](./README.md)
- [Master Index](../00-governance/99-master-index.md)

### Origem e fonte dos dados
- [Game Panel](../02-game-panel/README.md)
- [MatchZy](../02-game-panel/matchzy.md)
- [Data Sources — MatchZy SQLite](./data-sources-matchzy-sqlite.md)

### Transformação e publicação
- [ETL Bash Pipeline](./etl-bash-pipeline.md)
- [Static API v2](./static-api-v2.md)
- [JSON Contracts](./json-contracts.md)
- [Frontend Structure](./frontend-structure.md)

### Infraestrutura e suporte
- [Docker Host](../01-infra-hostinger/docker-host.md)
- [Filesystem Paths and Permissions](../01-infra-hostinger/filesystem-paths-permissions.md)
- [Operational Runbooks](./portal-estatico-operational-runbooks.md)
- [Observability and Troubleshooting](./portal-estatico-observability-troubleshooting.md)

---

## Escopo

Este documento cobre:

- o papel das queries SQL versionadas no contexto
- a localização operacional conhecida dos SQLs
- os grupos centrais de query usados pela v2
- a dependência de SQLite com suporte a JSON1
- as regras de negócio embutidas na camada SQL
- a relação entre SQL, ETL e contratos JSON
- os riscos de drift de schema
- as validações mínimas esperadas sobre a camada SQL

Este documento não cobre em profundidade:

- dump completo de cada query
- pipeline ETL script por script
- shape campo a campo dos JSONs publicados
- troubleshooting aprofundado do portal
- operação do MatchZy como runtime de jogo
- Auth API no Lightsail

Esses tópicos vivem em documentos específicos deste contexto ou em outros contextos canônicos.

---

## Estado atual

O estado operacional conhecido da camada SQL do Portal Estático é:

- a Static API v2 depende de queries SQL versionadas
- essas queries operam sobre o `matchzy.db`
- a geração pública da v2 depende diretamente da compatibilidade entre schema e SQL
- parte da lógica pública do ranking v2 já está embutida em SQL
- o ETL Bash executa ou consome resultados dessas queries para produzir os JSONs finais
- a camada SQL é uma peça central do contrato público, mesmo não sendo visível diretamente ao usuário final

A arquitetura atual assume que:

- a fonte de verdade de stats vem do SQLite
- a transformação principal acontece por Bash + SQL
- a consistência pública da v2 depende da disciplina dessa camada

---

## Source of truth / evidências

As principais evidências deste documento, nesta fase de migração documental, são:

- documentação consolidada atual do ecossistema HSC
- blueprint técnico consolidado do HSC
- reconciliação documental da Static API v2
- reconciliação da existência de SQL versionado no contexto do portal
- referência documentada ao uso de `ranking_v2.sql`
- reconciliação da dependência de SQLite JSON1 e da relação entre schema, query e output público

Enquanto a migração canônica do contexto não estiver concluída, essas fontes seguem sendo usadas como base de reconciliação do estado real.

---

## Relações com outros documentos

Este arquivo é complementar a:

- `docs/03-portal-estatico/README.md`
- `docs/03-portal-estatico/portal-estatico-architecture-runtime.md`
- `docs/03-portal-estatico/static-api-v2.md`
- `docs/03-portal-estatico/data-sources-matchzy-sqlite.md`
- `docs/03-portal-estatico/etl-bash-pipeline.md`
- `docs/03-portal-estatico/json-contracts.md`
- `docs/03-portal-estatico/portal-estatico-observability-troubleshooting.md`

Este documento descreve a camada SQL da geração da v2.  
Ele não substitui os documentos de ETL, contratos, fonte de dados ou troubleshooting.

---

## Papel dos SQLs versionados

As queries SQL versionadas existem para transformar a fonte SQLite em insumo confiável para a v2.

Seu papel é:

- extrair os dados corretos do `matchzy.db`
- consolidar agregados públicos
- expressar parte das regras de negócio da camada pública
- reduzir duplicação de lógica no Bash
- tornar a transformação mais auditável
- dar previsibilidade à geração do ranking, matches, players e maps

Em termos arquiteturais:

- o SQLite guarda a matéria-prima
- a camada SQL organiza a leitura e a transformação estrutural
- o ETL Bash usa essa camada para gerar os artefatos públicos finais

---

## Localização oficial dos SQLs

A localização operacional reconciliada da camada SQL do contexto inclui a árvore:

- `/opt/cs2-portal/sql/`

Nessa árvore vivem os SQLs versionados usados pela geração da v2.

Regra canônica:

- os SQLs do contexto devem ser tratados como artefatos operacionais versionados
- mudanças nesses arquivos são mudanças relevantes da camada pública
- SQL não deve ficar disperso informalmente em scripts, shell history ou documentos soltos sem reconciliação

---

## Papel do versionamento dos SQLs

O versionamento dos SQLs existe para:

- manter rastreabilidade da lógica de geração
- permitir auditoria das regras públicas
- reduzir drift entre ETL e contratos
- facilitar troubleshooting quando um recurso quebra
- preservar histórico de evolução da v2

Regra importante:

- uma mudança em SQL pode mudar o comportamento público da API estática
- por isso, mudança em SQL relevante deve ser tratada como mudança de contrato operacional, mesmo que o path público da v2 não mude

---

## Queries centrais por artefato

A camada SQL da v2 pode ser entendida por grupos de responsabilidade.

### Ranking

Responsável por sustentar:

- `ranking.json`
- parte da cobertura pública de jogadores

Há referência reconciliada a:

- `ranking_v2.sql`

Papel dessa camada:

- consolidar o ranking público
- embutir regras da versão 2 do ranking
- servir como uma das bases para geração de páginas de player

---

### Matches

Responsável por sustentar:

- `matches.json`

Papel dessa camada:

- consolidar lista pública de partidas
- organizar dados de partida em formato agregável
- preparar o terreno para recursos de detalhe por match

---

### Match detail

Responsável por sustentar:

- `match/{id}.json`

Papel dessa camada:

- extrair detalhe incremental de cada partida
- preservar consistência entre coleção e detalhe
- permitir páginas públicas específicas de match

---

### Players

Responsável por sustentar:

- `player/{steamid64}.json`

Papel dessa camada:

- consolidar visão pública individual por jogador
- alinhar-se com o universo de `ranking.json`
- servir como base de dossiê público do player

---

### Maps

Responsável por sustentar:

- `maps.json`
- `map/{map}.json`

Papel dessa camada:

- consolidar agregados por mapa
- permitir coleções e detalhes públicos por mapa
- manter consistência entre lista e recurso detalhado

---

## Regras de negócio embutidas no SQL

Parte relevante da semântica pública da v2 já vive na camada SQL.

Exemplos do tipo de lógica que tende a estar embutida nessa camada:

- critérios de ordenação de ranking
- agregação por jogador
- agregação por mapa
- seleção de partidas públicas
- projeção de colunas e estrutura de saída intermediária
- compatibilização entre o que existe no schema e o que a v2 precisa publicar

Isso significa que:

- a camada SQL não é neutra
- ela já incorpora decisões de produto e de apresentação pública
- qualquer mudança nela pode alterar a leitura que o portal faz dos dados

Regra importante:

- quando a lógica pública mudar por causa de SQL, isso deve ser reconhecido como mudança real do comportamento do sistema

---

## Relação entre SQL e ETL

A relação entre SQL e ETL é estrutural:

- o SQL define parte importante do que será lido e agregado
- o Bash organiza a orquestração da geração, incremental, locks e publicação
- o JSON final depende dos dois

Em termos práticos:

- SQL errado pode quebrar um recurso mesmo com ETL saudável
- ETL errado pode publicar mal mesmo com SQL correto
- troubleshooting da v2 precisa considerar os dois lados

A camada SQL, portanto, não deve ser tratada como detalhe secundário do pipeline.

---

## Relação entre SQL e contratos JSON

A camada SQL influencia diretamente os contratos JSON publicados.

Isso acontece porque:

- a projeção dos dados já nasce na query
- a disponibilidade de campos no JSON depende do que a query entrega
- agregados públicos dependem da lógica SQL
- mudanças em aliases, seleção de colunas ou joins podem quebrar o shape consumido pelo frontend

Regra canônica:

- mudança relevante em SQL deve ser tratada como potencial mudança de contrato JSON
- por isso, a camada SQL deve permanecer sincronizada com `json-contracts.md`

---

## Dependência de JSON1

A camada SQL da v2 possui dependência conhecida de SQLite com suporte a JSON1.

Isso implica:

- parte da lógica pública depende de funções ou expressões compatíveis com JSON1
- o ambiente que executa as queries precisa suportar esse recurso
- incompatibilidade de SQLite pode quebrar queries sem que o problema esteja no frontend ou no Nginx

Regra operacional:

- suporte a JSON1 não é um detalhe incidental
- ele é uma dependência técnica do pipeline atual

---

## Dependência de schema

As queries SQL da v2 dependem diretamente do schema do `matchzy.db`.

Isso significa:

- tabelas esperadas precisam existir
- colunas esperadas precisam existir
- tipos e estruturas mínimas precisam continuar compatíveis
- evolução do MatchZy ou do banco pode quebrar a geração pública

Regra importante:

- drift de schema quebra a camada SQL antes de quebrar a camada de apresentação
- o problema pode parecer “do portal”, mas a raiz pode estar no banco ou na query

---

## Riscos de drift de schema

Os principais riscos de drift entre schema e queries incluem:

- coluna renomeada sem atualização do SQL
- tabela alterada ou removida
- mudança de shape do dado de origem
- atualização do MatchZy com impacto estrutural
- query assumindo campo que não existe mais
- query antiga aplicada sobre schema novo sem reconciliação

Sintomas típicos:

- geração falha
- JSON não é gerado
- JSON é gerado com lacunas
- recurso específico quebra enquanto outros continuam saudáveis
- health geral parece bom, mas um artefato específico degrada

---

## Estratégia de validação

A camada SQL precisa de validação mínima sempre que houver:

- mudança relevante no ETL
- mudança no MatchZy
- suspeita de drift de schema
- quebra de contrato em `ranking.json`, `matches.json`, `player/{steamid64}.json` ou `map/{map}.json`
- troubleshooting de geração parcial

A validação mínima deve incluir:

- confirmar existência do `matchzy.db`
- validar tabelas principais
- validar suporte a JSON1, quando necessário
- executar leitura simples representativa
- inspecionar schema de tabela afetada
- reconciliar a query afetada com o artefato público quebrado

---

## Comandos de validação

Os comandos abaixo representam verificações típicas da camada SQL.

### Verificar existência do diretório de SQLs

```bash
ls -lah /opt/cs2-portal/sql/
```

### Verificar existência do banco

Substitua pelo path real validado do `matchzy.db`.

```bash
ls -l /CAMINHO/DO/matchzy.db
```

### Verificar tabelas do banco

```bash
sqlite3 /CAMINHO/DO/matchzy.db ".tables"
```

### Verificar schema de tabela principal

Exemplo representativo:

```bash
sqlite3 /CAMINHO/DO/matchzy.db ".schema matchzy_stats_players"
```

### Executar leitura simples de tabela principal

Exemplo representativo:

```bash
sqlite3 /CAMINHO/DO/matchzy.db "SELECT COUNT(*) FROM matchzy_stats_players;"
```

### Inspecionar presença do SQL de ranking reconciliado

```bash
ls -l /opt/cs2-portal/sql/ranking_v2.sql
```

Regra importante:
- este documento não vira repositório de queries completas
- ele aponta a camada, seus papéis e suas verificações

---

## Problemas comuns

### 1. Query quebra após atualização do MatchZy

Causas comuns:

- tabela ou coluna mudou
- schema novo não foi reconciliado
- SQL antigo ficou incompatível

Impacto:
- um ou mais artefatos da v2 deixam de ser gerados corretamente

---

### 2. Ranking muda sem mudança visível no frontend

Causas comuns:

- lógica do ranking alterada dentro do SQL
- regra pública mudou sem atualização documental correspondente

Impacto:
- o portal continua “funcionando”, mas com semântica pública alterada

---

### 3. Agregado funciona, detalhe quebra

Causas comuns:

- query de coleção saudável
- query de detalhe quebrada
- drift localizado em parte do schema

Impacto:
- listagem pública funciona
- página de detalhe falha

---

### 4. Query executa, mas output público fica incoerente

Causas comuns:

- aliases ou shape intermediário mudaram
- ETL ou contrato JSON não foram ajustados
- estrutura pública esperada pelo frontend divergiu

Impacto:
- SQL parece saudável
- frontend ou contrato público quebram depois

---

### 5. Banco está acessível, mas JSON não atualiza

Causas comuns:

- query falhando silenciosamente ou não sendo usada como esperado
- statefile/ETL incremental mascarando o problema
- troubleshooting focado demais no Nginx e pouco na camada SQL

Impacto:
- stale data
- falsa percepção de que “o problema é só cache”

---

## Boas práticas de manutenção

As boas práticas desta camada incluem:

- manter SQL versionado em diretório próprio
- tratar mudança de query como mudança relevante
- validar impacto da query no JSON público
- reconciliar docs de SQL, ETL e contratos
- registrar mudanças estruturais importantes em impl-log
- evitar embutir lógica SQL complexa de forma invisível dentro de shell sem documentação correspondente

---

## Limites deste documento

Este documento não detalha:

- conteúdo completo de todas as queries
- dump linha a linha dos SQLs
- troubleshooting avançado de cada query
- shape completo dos artefatos intermediários
- governança completa de versionamento do repositório de scripts
- otimização de performance das queries em profundidade

Esses tópicos devem ser tratados em documentos complementares ou impl-logs quando necessário.

---

## Critério de pronto deste documento

Este documento pode ser considerado maduro quando:

- a árvore real de SQLs estiver confirmada no ambiente
- os grupos principais de query estiverem reconciliados com os artefatos públicos da v2
- a dependência de `ranking_v2.sql` estiver formalizada sem ambiguidade
- a relação entre SQL, schema e contratos estiver clara
- os riscos de drift de schema estiverem representados de forma útil
- ele puder ser lido como referência da camada SQL da v2 sem depender do master legado

---

## Última revisão

- Status: ativo
- Classificação: canônico
- Contexto: portal estático / SQL queries and views
- Última revisão: 2026-03-18