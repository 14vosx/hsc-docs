# MatchZy

## Objetivo

Documentar o papel do MatchZy no contexto Game Panel do ecossistema HSC, registrando sua função operacional no servidor CS2, sua relação com o fluxo competitivo da instância e sua importância estrutural para a cadeia pública de dados do projeto.

Este documento existe para registrar, de forma estável e auditável:

- o que o MatchZy representa no runtime do servidor
- como ele se relaciona com a instância `MixHAXIXE01`
- como ele participa de scrim, BO1, BO3, veto, coach e administração da partida
- como ele participa da persistência estatística local
- por que o MatchZy impacta tanto a operação do jogo quanto o Portal Estático
- quais sinais de saúde, limites e falhas comuns existem nessa camada

---

## Navegação

### Entrada
- [Home da documentação](../README.md)
- [Game Panel](./README.md)
- [Master Index](../00-governance/99-master-index.md)

### Contexto operacional imediato
- [CS2 Server Configuration](./cs2-server-configuration.md)
- [Plugins Installed](./plugins-installed.md)
- [Instance MixHAXIXE01](./instance-mixhaxixe01.md)
- [Operational Runbooks](./operational-runbooks.md)

### Integração com dados e portal
- [Data Sources — MatchZy SQLite](../03-portal-estatico/data-sources-matchzy-sqlite.md)
- [SQL Queries and Views](../03-portal-estatico/sql-queries-and-views.md)
- [ETL Bash Pipeline](../03-portal-estatico/etl-bash-pipeline.md)
- [Static API v2](../03-portal-estatico/static-api-v2.md)

### Infraestrutura adjacente
- [Docker Host](../01-infra-hostinger/docker-host.md)
- [Filesystem Paths and Permissions](../01-infra-hostinger/filesystem-paths-permissions.md)
- [Observability and Troubleshooting](./game-panel-observability-troubleshooting.md)

---

## Escopo

Este documento cobre:

- o papel do MatchZy no lado jogo
- a relação entre MatchZy e o runtime do CS2
- a relação entre MatchZy e CounterStrikeSharp
- a relação entre MatchZy e a persistência estatística local
- a participação do MatchZy nos fluxos competitivos do servidor
- a relevância estrutural do MatchZy para a cadeia pública de dados
- sinais de saúde, riscos e falhas comuns dessa camada

Este documento não cobre em profundidade:

- configuração linha a linha de todos os arquivos do servidor
- inventário completo de todos os plugins instalados
- troubleshooting aprofundado do AMP
- pipeline ETL do Portal Estático
- contratos JSON da Static API v2
- Auth API no AWS Lightsail

Esses tópicos vivem em documentos próprios dos contextos `01-infra-hostinger`, `02-game-panel` e `03-portal-estatico`.

---

## Estado atual

O estado operacional conhecido do MatchZy no ecossistema HSC é:

- o MatchZy é peça central da operação competitiva do servidor
- ele roda no contexto do servidor CS2 da instância oficial `MixHAXIXE01`
- ele depende do runtime do CounterStrikeSharp
- ele organiza parte importante do fluxo de partida administrada
- ele participa de modos como scrim, BO1 e BO3
- ele influencia fluxos como veto, coach, warmup e administração da partida
- ele produz persistência estatística local usada fora do contexto do jogo
- sua base estatística alimenta, indiretamente, o Portal Estático por meio do `matchzy.db`

A versão reconciliada no ecossistema atual é:

- MatchZy `0.8.15`

Isso significa que o MatchZy não é apenas um plugin de conveniência; ele é um componente estrutural do HSC.

---

## Source of truth / evidências

As principais evidências deste documento, nesta fase de migração documental, são:

- documentação consolidada atual do ecossistema HSC
- blueprint técnico consolidado do HSC
- documentação reconciliada do lado AMP/CS2
- documentação reconciliada da Static API v2
- reconciliação das tabelas estatísticas derivadas do MatchZy
- histórico operacional de uso do MatchZy para fluxo competitivo e geração de stats

Enquanto a migração canônica do contexto não estiver concluída, essas fontes seguem sendo usadas como base de reconciliação do estado real.

---

## Relações com outros documentos

Este arquivo é complementar a:

- `docs/02-game-panel/README.md`
- `docs/02-game-panel/game-panel-architecture-runtime.md`
- `docs/02-game-panel/instance-mixhaxixe01.md`
- `docs/02-game-panel/cs2-server-configuration.md`
- `docs/02-game-panel/plugins-installed.md`
- `docs/02-game-panel/operational-runbooks.md`
- `docs/02-game-panel/game-panel-observability-troubleshooting.md`
- `docs/03-portal-estatico/data-sources-matchzy-sqlite.md`
- `docs/03-portal-estatico/static-api-v2.md`

Este documento descreve o MatchZy como componente central do runtime do lado jogo.  
Ele não substitui os documentos de runbook, plugins, ETL ou contratos públicos.

---

## Papel do MatchZy no HSC

No ecossistema HSC, o MatchZy cumpre dois papéis ao mesmo tempo.

### 1. Papel operacional de jogo

O MatchZy existe para sustentar o fluxo competitivo do servidor, incluindo:

- organização de partida administrada
- modos competitivos como BO1 e BO3
- suporte aos fluxos ligados a veto e match flow
- suporte à administração prática de scrim
- suporte a cenários operacionais como coach, warmup e transições de estado da partida

### 2. Papel estrutural de dados

O MatchZy também existe como produtor da base estatística do ecossistema.

Isso inclui:

- persistência local das estatísticas de partida
- base para ranking público
- base para páginas públicas de player
- base para agregados por mapa
- base para listagem e detalhe público de matches

Regra canônica:

- no HSC, MatchZy é simultaneamente componente de gameplay e componente de dados

---

## Relação com o runtime do CS2

O MatchZy existe dentro do runtime do servidor CS2.

Essa relação inclui:

- carregamento no contexto correto do servidor
- convivência com CounterStrikeSharp
- interação com outros plugins
- influência direta no comportamento do servidor durante o fluxo competitivo
- produção de artefatos estatísticos coerentes com a operação real da partida

O MatchZy não roda “ao lado” do jogo.  
Ele é parte do runtime efetivo do lado competitivo do servidor.

---

## Relação com CounterStrikeSharp

A saúde do MatchZy depende estruturalmente da camada CounterStrikeSharp.

Essa relação inclui:

- carregamento do plugin no runtime
- ciclo de vida do plugin dentro do servidor
- integração com outros plugins do mesmo ecossistema
- estabilidade da malha de plugins usada pela instância

Regra importante:

- se o CounterStrikeSharp ou a camada de plugins estiver degradada, o MatchZy pode falhar ou ficar parcialmente funcional
- nem toda falha “de MatchZy” nasce no próprio MatchZy

---

## Relação com a instância `MixHAXIXE01`

O MatchZy depende diretamente da instância oficial do ecossistema.

Essa relação inclui:

- runtime do servidor associado à instância
- árvore operacional onde vivem os artefatos do lado game
- persistência estatística local
- dependências de path que depois serão consumidas pela camada pública do portal

Regra importante:

- o MatchZy não é componente abstrato do host
- ele opera no contexto real da instância `MixHAXIXE01`

---

## Fluxos competitivos suportados

O MatchZy participa dos principais fluxos competitivos do servidor.

### Scrim

Papel:
- permitir o uso do servidor como ambiente de treino competitivo estruturado

### BO1

Papel:
- sustentar fluxo simples de partida única
- operar regras competitivas coerentes com o servidor

### BO3

Papel:
- sustentar fluxo de série
- influenciar escolha de mapas e progressão competitiva mais estruturada

### Veto

Papel:
- participar do fluxo de picks e bans
- organizar fase de mapas antes da partida ou da série, conforme o modo configurado

### Coach

Papel:
- participar do contexto operacional em que modo coach é relevante e permitido

### Warmup e match flow

Papel:
- influenciar ou sustentar transições relevantes do estado da partida

Regra editorial:

- este documento reconhece a participação do MatchZy nesses fluxos
- o passo a passo operacional desses cenários vive em `operational-runbooks.md`

---

## Relação com administração da partida

O MatchZy participa diretamente da experiência administrativa do servidor.

Essa relação inclui:

- comandos e ações ligadas ao flow de partida
- dependência de permissões/admins adequados
- necessidade de fallback administrativo quando não há admins operando do jeito ideal
- necessidade de runbooks claros para reduzir improviso operacional

Esse ponto é importante porque o MatchZy é, na prática, uma ferramenta operacional do staff do servidor, não apenas uma peça de backend estatístico.

---

## Relação com permissões e fallback admin

A operação real do MatchZy depende de como o servidor lida com permissões administrativas.

Isso implica:

- ausência de admins adequados pode degradar a operação prática do fluxo competitivo
- o ecossistema já precisou pensar em fallback administrativo quando ninguém está operando com o perfil esperado
- regras de administração e fallback não são detalhe secundário; elas afetam diretamente a usabilidade do MatchZy em produção

Regra importante:

- MatchZy saudável sem modelo operacional de admin saudável ainda gera degradação prática do servidor

---

## Persistência estatística

O MatchZy participa diretamente da produção da persistência estatística local.

O artefato estrutural mais importante associado a essa camada é:

- `matchzy.db`

Esse banco é importante porque:

- registra estatísticas operacionais do servidor
- fornece a matéria-prima da Static API v2
- conecta a operação do jogo à camada pública do portal

O MatchZy, portanto, não só comanda o flow competitivo; ele também alimenta o lado público do ecossistema.

---

## Tabelas estatísticas principais

As tabelas principais já reconciliadas como derivadas do MatchZy e relevantes para o ecossistema são:

### `matchzy_stats_matches`

Papel:
- base da listagem e do detalhe de partidas

### `matchzy_stats_maps`

Papel:
- base dos agregados e detalhes por mapa

### `matchzy_stats_players`

Papel:
- base do ranking e do dossiê público de jogador

Essas tabelas mostram de forma concreta como o MatchZy extrapola o contexto do jogo e passa a influenciar o Portal Estático.

---

## Relação com o Portal Estático

O Portal Estático depende indiretamente do MatchZy porque a camada pública de dados nasce da sua persistência estatística.

Essa relação inclui:

- geração do `matchzy.db`
- schema compatível com o ETL
- continuidade da produção de stats
- estabilidade de identificadores como match id, map e `steamid64`

Regra arquitetural:

- se o MatchZy parar de produzir ou mudar estruturalmente sem reconciliação, o Portal Estático degrada depois
- isso faz do MatchZy um componente crítico além do contexto `02-game-panel`

---

## Dependências operacionais

O MatchZy depende, no mínimo, de:

- Infra Hostinger saudável
- Docker host funcional
- AMP funcional
- instância `MixHAXIXE01` íntegra
- runtime CS2 funcional
- CounterStrikeSharp funcional
- permissões administrativas coerentes
- configs do servidor compatíveis com o fluxo operacional esperado
- persistência local íntegra
- ausência de conflito destrutivo com plugins auxiliares

Dependências cruzadas importantes:

- o fluxo competitivo do servidor depende do MatchZy
- a cadeia pública de dados do portal depende da sua persistência estatística
- troubleshooting do MatchZy precisa considerar tanto gameplay quanto dados

---

## Invariantes operacionais

Os invariantes conhecidos desta camada incluem:

- o MatchZy é componente central do lado jogo
- a versão reconciliada do ecossistema atual é `0.8.15`
- o MatchZy participa tanto da operação da partida quanto da produção estatística
- o `matchzy.db` é artefato estrutural do ecossistema
- tabelas `matches`, `maps` e `players` derivadas do MatchZy são base da v2 do portal
- o MatchZy depende do contexto correto da instância `MixHAXIXE01`
- troubleshooting do MatchZy não pode ignorar permissões/admins e nem a camada de persistência

Esses invariantes ajudam a preservar a leitura correta da sua importância no HSC.

---

## Sinais de saúde do MatchZy

Os sinais de saúde esperados incluem:

- MatchZy carregado no runtime do servidor
- fluxo competitivo executável em cenário real
- transições básicas da partida funcionando como esperado
- comandos operacionais coerentes com o runbook vigente
- persistência local sendo produzida
- tabelas estatísticas evoluindo após partidas reais
- ausência de falha estrutural recorrente no lado competitivo

A saúde do MatchZy deve ser inferida tanto por comportamento in-game quanto por persistência estatística.

---

## Comandos de validação

Os comandos abaixo representam verificações típicas da camada MatchZy.

### Listar plugins carregados

Exemplo representativo já usado operacionalmente no ecossistema:

```bash
css_plugins list
```

### Inspecionar comandos disponíveis no runtime

Exemplo representativo:

```bash
css_plugins list
```

Observação:
- a validação real da presença do MatchZy no runtime costuma depender da camada de plugins carregados e dos comandos expostos pelo servidor

### Validar presença estrutural do banco do MatchZy

Substitua pelo path real reconciliado no ambiente.

```bash
ls -l /CAMINHO/DO/matchzy.db
```

### Validar tabelas principais do banco

```bash
sqlite3 /CAMINHO/DO/matchzy.db ".tables"
```

### Validar volume básico de registros

Exemplo representativo:

```bash
sqlite3 /CAMINHO/DO/matchzy.db "SELECT COUNT(*) FROM matchzy_stats_matches;"
sqlite3 /CAMINHO/DO/matchzy.db "SELECT COUNT(*) FROM matchzy_stats_players;"
sqlite3 /CAMINHO/DO/matchzy.db "SELECT COUNT(*) FROM matchzy_stats_maps;"
```

Regra importante:

- esses comandos validam a presença estrutural e estatística
- a validação completa do fluxo competitivo continua dependendo do runtime real da partida

---

## Problemas comuns

### 1. MatchZy não carrega

Causas comuns:

- problema na camada CounterStrikeSharp
- plugin ausente ou incompatível
- problema no runtime da instância
- conflito com ambiente do servidor

Impacto:
- fluxo competitivo degrada imediatamente
- produção de stats pode deixar de evoluir

---

### 2. MatchZy carrega, mas o fluxo de partida falha

Causas comuns:

- configuração divergente
- runbook operacional não alinhado ao estado real
- problema de admin/permissão
- conflito funcional com outro componente do runtime

Impacto:
- o servidor parece “de pé”
- mas o valor prático competitivo fica degradado

---

### 3. Fluxo de partida funciona, mas o banco não evolui

Causas comuns:

- problema de persistência local
- problema de escrita do MatchZy
- path incorreto
- falha de gravação na árvore da instância

Impacto:
- jogo continua operando
- Portal Estático passa a ficar stale depois

---

### 4. Stats existem, mas estão incoerentes com o portal

Causas comuns:

- drift de schema
- persistência parcial
- mudança estrutural no MatchZy sem reconciliação da cadeia pública
- ETL lendo path ou schema inesperado

Impacto:
- o problema aparece no portal, mas a raiz pode estar no lado game

---

### 5. MatchZy saudável, mas operação humana continua ruim

Causas comuns:

- falta de admins adequados
- ausência de fallback operacional
- runbooks insuficientes
- dependência de memória informal

Impacto:
- a tecnologia está carregada
- a operação prática do servidor continua degradada

---

## Sequência básica de diagnóstico

Quando houver suspeita de problema na camada MatchZy, a sequência recomendada é:

1. validar a saúde da Infra Hostinger
2. validar a saúde do Docker host
3. validar a saúde do AMP e da instância `MixHAXIXE01`
4. validar a presença do MatchZy na camada de plugins
5. validar o comportamento real do fluxo competitivo
6. validar a persistência estatística local
7. só depois aprofundar em ETL do portal ou comportamento de frontend

Essa sequência ajuda a responder rapidamente:

- o MatchZy está realmente carregado?
- ele está operando funcionalmente?
- ele está persistindo corretamente?
- o problema é do MatchZy ou da camada pública que o consome?

---

## Limites deste documento

Este documento não detalha:

- todos os comandos operacionais do MatchZy
- cada cenário de BO1, BO3, veto ou coach em passo a passo
- toda a configuração linha a linha do plugin
- inventário completo dos plugins auxiliares
- troubleshooting aprofundado da camada CounterStrikeSharp
- troubleshooting aprofundado do ETL que consome o banco

Esses tópicos pertencem a documentos específicos do contexto.

---

## Critério de pronto deste documento

Este documento pode ser considerado maduro quando:

- o papel do MatchZy como componente de gameplay e de dados estiver reconciliado sem ambiguidade
- a relação com `MixHAXIXE01`, CS2 e `matchzy.db` estiver clara
- a versão operacional `0.8.15` estiver formalizada no contexto correto
- os sinais de saúde e failure modes mais relevantes estiverem representados
- ele puder ser usado como referência canônica do MatchZy no HSC sem depender do master legado

---

## Última revisão

- Status: ativo
- Classificação: canônico
- Contexto: game panel / MatchZy
- Última revisão: 2026-03-18