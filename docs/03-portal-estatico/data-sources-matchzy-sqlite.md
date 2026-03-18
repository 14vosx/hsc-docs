# Data Sources — MatchZy SQLite

## Objetivo

Documentar a fonte primária de dados do contexto Portal Estático, baseada no banco SQLite gerado pelo MatchZy, registrando seu papel arquitetural, seus paths críticos, suas tabelas relevantes e os limites estruturais que impactam a Static API v2.

Este documento existe para registrar, de forma estável e auditável:

- qual é a fonte de verdade de stats do portal
- onde o `matchzy.db` se posiciona na arquitetura
- quais tabelas principais sustentam a v2
- quais dependências ocultas existem no path e na estrutura da instância
- quais limitações do banco impactam os artefatos públicos
- quais riscos de drift de schema precisam ser observados

---

## Escopo

Este documento cobre:

- a fonte de verdade do lado estatístico para o portal
- o papel do `matchzy.db`
- o path operacional conhecido do banco
- as tabelas principais consumidas pelo ETL
- a dependência de SQLite com suporte a JSON1
- a dependência estrutural da instância AMP / Game Panel
- limitações estruturais dos dados disponíveis
- riscos de drift de schema

Este documento não cobre em profundidade:

- o funcionamento do MatchZy como ferramenta operacional de jogo
- as queries SQL detalhadas do ETL
- o pipeline completo script por script
- os contratos JSON finais da v2
- troubleshooting aprofundado do portal
- Auth API e seus bancos

Esses tópicos vivem em documentos próprios deste contexto ou em outros contextos canônicos.

---

## Estado atual

A fonte primária conhecida de dados estatísticos do Portal Estático é o banco SQLite do MatchZy.

O estado operacional conhecido dessa camada é:

- a origem de dados públicos da Static API v2 vem do `matchzy.db`
- esse banco é produzido no contexto operacional do servidor CS2
- o ETL Bash lê diretamente esse SQLite
- as tabelas principais já reconciliadas sustentam ranking, matches, maps e players
- a arquitetura atual depende da estabilidade do path da instância e da compatibilidade do schema do banco
- o portal não consulta o banco diretamente; ele consome apenas os JSONs gerados a partir dele

Esse desenho torna o `matchzy.db` a principal fonte estatística do lado público do HSC.

---

## Source of truth / evidências

As principais evidências deste documento, nesta fase de migração documental, são:

- documentação consolidada atual do ecossistema HSC
- blueprint técnico consolidado do HSC
- documentação reconciliada do MatchZy e da Static API v2
- reconciliação dos paths do `matchzy.db` dentro da instância AMP
- reconciliação das tabelas principais consumidas pelo ETL

Enquanto a migração canônica do contexto não estiver concluída, essas fontes seguem sendo usadas como base de reconciliação do estado real.

---

## Relações com outros documentos

Este arquivo é complementar a:

- `docs/03-portal-estatico/README.md`
- `docs/03-portal-estatico/architecture-runtime.md`
- `docs/03-portal-estatico/static-api-v2.md`
- `docs/03-portal-estatico/etl-bash-pipeline.md`
- `docs/03-portal-estatico/sql-queries-and-views.md`
- `docs/03-portal-estatico/json-contracts.md`
- `docs/03-portal-estatico/observability-troubleshooting.md`
- `docs/02-game-panel/instance-mixhaxixe01.md`
- `docs/02-game-panel/matchzy.md`

Este documento descreve a fonte de dados estatística usada pelo portal.  
Ele não substitui os documentos de pipeline, queries, contratos ou operação do jogo.

---

## Fonte de verdade de stats

A fonte de verdade estatística do Portal Estático, no desenho atual do ecossistema, é o SQLite produzido pelo MatchZy.

Isso significa:

- ranking público deriva do banco SQLite
- listas e detalhes de matches derivam do banco SQLite
- páginas de player derivam do banco SQLite
- agregados por mapa derivam do banco SQLite
- a v2 só é tão boa quanto a integridade e a compatibilidade do banco de origem

Regra canônica:

- para a camada pública de stats, o `matchzy.db` é a origem primária
- o portal não deve inventar ou inferir dados fora do que foi produzido e transformado a partir dessa fonte

---

## Papel do `matchzy.db` na arquitetura

O `matchzy.db` ocupa o papel de base operacional intermediária entre:

- runtime do jogo
- pipeline ETL
- publicação pública da Static API v2

Fluxo resumido:

1. o runtime do servidor CS2 produz estatísticas
2. o MatchZy persiste essas estatísticas em SQLite
3. o ETL lê esse banco
4. o ETL gera JSONs estáticos
5. o portal consome esses JSONs

Implicações:

- se o banco não atualizar, a v2 fica stale
- se o schema do banco mudar, o ETL pode quebrar
- se o path do banco mudar, o pipeline deixa de funcionar
- se os dados no banco estiverem inconsistentes, isso pode se propagar para o público

---

## Path oficial do `matchzy.db`

O path reconciliado historicamente para o banco depende da estrutura da instância AMP e do runtime do servidor CS2.

A referência estrutural conhecida inclui a instância oficial:

- `MixHAXIXE01`

E a base de paths observada no ecossistema segue a árvore ligada ao AMP, dentro de algo como:

- `/home/amp/.ampdata/instances/MixHAXIXE01/...`

Regra importante:

- o nome da instância é parte da dependência estrutural do path
- qualquer renomeação da instância ou mudança de árvore no runtime do jogo pode quebrar a localização do banco
- o path exato do `matchzy.db` deve ser mantido reconciliado com o ambiente real

Enquanto este contexto não receber a validação final do host em sua nova documentação, o canônico deve assumir:

- o `matchzy.db` vive sob a árvore operacional da instância `MixHAXIXE01`
- a dependência do nome da instância é real e deve ser explicitada

---

## Dependência da instância AMP

O banco de dados não vive em abstração.

Ele depende diretamente da estrutura operacional da instância gerida pelo AMP / Game Panel.

Isso implica:

- o portal depende indiretamente do Game Panel para localizar sua fonte de dados
- o ETL depende de um path que nasce fora do contexto do portal
- o nome da instância e a árvore de diretórios não são detalhes cosméticos; são parte da topologia

Regra arquitetural importante:

- o contexto `03-portal-estatico` consome o banco
- mas a origem do banco nasce no contexto `02-game-panel`

Por isso, qualquer alteração de instância, path ou layout precisa ser tratada de forma coordenada entre os dois contextos.

---

## Tabelas principais

As tabelas principais já reconciliadas como base da Static API v2 são:

### `matchzy_stats_matches`

Papel:
- sustentar listagem de partidas
- sustentar metadados principais de matches
- alimentar detalhe incremental por match

Uso típico na cadeia pública:
- `matches.json`
- `match/{id}.json`

---

### `matchzy_stats_maps`

Papel:
- sustentar agregados por mapa
- sustentar visão pública de performance e volume por mapa
- alimentar listas e detalhes ligados a mapas

Uso típico na cadeia pública:
- `maps.json`
- `map/{map}.json`

---

### `matchzy_stats_players`

Papel:
- sustentar ranking e dossiê público de jogador
- alimentar páginas de player
- sustentar agregações de performance individual

Uso típico na cadeia pública:
- `ranking.json`
- `player/{steamid64}.json`

---

## Papel das tabelas na v2

As tabelas não são publicadas diretamente; elas são transformadas pelo ETL.

A relação conceitual é:

- `matchzy_stats_matches` → coleções e detalhes de partidas
- `matchzy_stats_maps` → coleções e detalhes por mapa
- `matchzy_stats_players` → ranking e dossiês públicos de jogador

Essa separação é importante porque:

- ajuda a entender de onde cada recurso público deriva
- facilita troubleshooting quando um recurso específico quebra
- ajuda a localizar drift de schema ou erro em query

---

## Dependência de SQLite JSON1

A cadeia atual possui dependência conhecida de SQLite com suporte a JSON1 para parte das consultas e transformações usadas na geração da v2.

Isso implica:

- o ambiente que executa as queries precisa ser compatível com esse recurso
- parte da lógica do ETL pressupõe essa capacidade
- quebra ou incompatibilidade de suporte pode afetar geração de artefatos públicos

Regra operacional:

- compatibilidade do SQLite com JSON1 é parte do funcionamento do ETL
- não deve ser tratada como detalhe incidental

---

## Dependência de schema

O ETL depende de schema compatível com as queries versionadas.

Isso significa:

- nomes de tabelas precisam permanecer coerentes
- colunas esperadas pelas queries precisam existir
- mudanças no MatchZy ou no banco precisam ser reconciliadas antes de tratar a v2 como saudável
- snapshots ou auditorias de schema podem ser necessários em marcos importantes

O portal não falha “por causa do frontend” quando a quebra está na base estrutural do banco.  
Muitas falhas públicas têm origem no schema ou na compatibilidade do banco de origem.

---

## Limitações estruturais dos dados

A camada pública está limitada ao que o `matchzy.db` efetivamente oferece.

Isso implica limites como:

- ausência de backend analítico dinâmico para compor dados além do que foi persistido
- dependência das entidades e agregações expostas pelo MatchZy
- eventuais ausências de granularidade mais profunda em determinados eventos de jogo
- dependência da forma como o plugin escreve estatísticas
- limitação do que pode ser reconstruído apenas a partir das tabelas existentes

Regra editorial:

- o portal deve assumir o banco como fonte real, não como fonte idealizada
- qualquer lacuna estrutural do banco deve ser reconhecida como limite da camada pública

---

## Limitações específicas que podem impactar a v2

Sem congelar detalhes não validados além do necessário, os limites típicos desta fonte incluem:

- granularidade analítica menor do que uma plataforma dedicada de telemetria
- dependência de agregados já produzidos ou facilmente extraíveis
- dificuldade de reconstruir certos contextos se o dado não foi persistido pelo MatchZy
- dependência alta da integridade do `steamid64` e de joins lógicos sobre player
- possíveis lacunas em eventos muito específicos não registrados no banco

Esses limites não inviabilizam a v2, mas definem seu teto atual.

---

## Riscos de drift de schema

Os principais riscos de drift nesta camada incluem:

- tabela renomeada ou alterada sem ajuste do ETL
- coluna esperada pela query deixando de existir
- alteração de shape do dado sem atualização documental
- mudança de versão do MatchZy com impacto estrutural não documentado
- path do banco mudando silenciosamente junto com a instância

Sintomas típicos de drift:

- ETL falha
- JSON deixa de ser gerado
- JSON é gerado com lacunas
- recursos específicos do portal quebram
- health do pipeline acusa inconsistência

Mitigações recomendadas:

- versionar queries
- manter snapshots e auditorias estruturais quando necessário
- registrar mudança relevante em impl-log
- reconciliar docs do portal e do Game Panel quando a origem mudar

---

## Sinais de saúde da fonte de dados

Os sinais de saúde esperados desta camada incluem:

- `matchzy.db` presente no path esperado
- SQLite legível pelo ETL
- tabelas principais existentes
- schema compatível com as queries versionadas
- ETL conseguindo gerar artefatos públicos
- ausência de erro estrutural recorrente nas execuções da pipeline

A saúde da fonte de dados deve ser inferida pela combinação entre:

- presença do banco
- compatibilidade do schema
- sucesso do pipeline
- coerência do output público

---

## Comandos de validação

Os comandos abaixo representam verificações típicas da fonte de dados.

### Verificar existência do banco

Substitua pelo path real validado no ambiente.

```bash
ls -l /CAMINHO/DO/matchzy.db
```

### Inspecionar tabelas do banco

Substitua pelo path real validado no ambiente.

```bash
sqlite3 /CAMINHO/DO/matchzy.db ".tables"
```

### Inspecionar schema de tabela principal

Exemplo representativo:

```bash
sqlite3 /CAMINHO/DO/matchzy.db ".schema matchzy_stats_players"
```

### Testar leitura simples

Exemplo representativo:

```bash
sqlite3 /CAMINHO/DO/matchzy.db "SELECT COUNT(*) FROM matchzy_stats_matches;"
```

Regra importante:
- não transformar este documento em dump de schema completo
- usar validação pontual para confirmar integridade e compatibilidade

---

## Problemas comuns

### 1. `matchzy.db` não encontrado

Causas comuns:

- path mudou
- instância foi renomeada
- árvore operacional do AMP mudou
- permissão impede acesso

Impacto:
- ETL não roda
- a v2 não atualiza

---

### 2. Banco existe, mas schema divergiu

Causas comuns:

- atualização do MatchZy
- alteração estrutural não documentada
- query versionada desatualizada

Impacto:
- falha de geração
- recursos específicos da v2 quebram
- troubleshooting pode parecer “erro do portal”, mas a raiz está no banco

---

### 3. Banco está íntegro, mas dados estão stale

Causas comuns:

- runtime do jogo não está atualizando stats
- ETL não está consumindo o banco novo
- statefiles ou incremental ficaram incoerentes

Impacto:
- portal continua de pé, mas com dados antigos

---

### 4. Uma tabela principal funciona, outra não

Causas comuns:

- drift parcial de schema
- query específica quebrada
- corrupção localizada
- inconsistência do pipeline incremental

Impacto:
- parte da v2 funciona
- parte da v2 quebra ou degrada

---

## Limites deste documento

Este documento não detalha:

- queries SQL completas
- mapeamento campo a campo de cada tabela
- pipeline completo de geração
- troubleshooting do Game Panel
- troubleshooting aprofundado do ETL
- shape completo dos JSONs publicados

Esses tópicos pertencem a documentos específicos.

---

## Critério de pronto deste documento

Este documento pode ser considerado maduro quando:

- o path real do `matchzy.db` estiver validado sem ambiguidade
- a dependência da instância `MixHAXIXE01` estiver formalizada e reconciliada
- as tabelas principais estiverem confirmadas contra o ambiente real
- os limites estruturais da fonte estiverem claros
- os riscos de drift de schema estiverem representados de forma útil
- ele puder ser lido como referência da fonte de dados do portal sem depender do master legado

---

## Última revisão

- Status: ativo
- Classificação: canônico
- Contexto: portal estático / fonte de dados MatchZy SQLite
- Última revisão: 2026-03-18