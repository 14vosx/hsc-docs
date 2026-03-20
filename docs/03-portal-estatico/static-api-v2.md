# Static API v2

## Objetivo

Documentar a Static API v2 como a camada pública de dados do contexto Portal Estático do ecossistema HSC.

Este documento existe para registrar, de forma estável e auditável:

- o papel da Static API v2 dentro da arquitetura do HSC
- o base path oficial da API estática
- os recursos públicos publicados na v2
- os invariantes operacionais e de compatibilidade dessa camada
- a relação entre ETL, contratos JSON e consumo pelo frontend
- os limites conhecidos e cuidados de evolução da v2

---

## Navegação

### Entrada
- [Home da documentação](../README.md)
- [Portal Estático](./README.md)
- [Master Index](../00-governance/99-master-index.md)

### Origem dos dados
- [MatchZy](../02-game-panel/matchzy.md)
- [Data Sources — MatchZy SQLite](./data-sources-matchzy-sqlite.md)
- [SQL Queries and Views](./sql-queries-and-views.md)
- [ETL Bash Pipeline](./etl-bash-pipeline.md)

### Contratos e consumo
- [JSON Contracts](./json-contracts.md)
- [Frontend Structure](./portal-estatico-frontend-structure.md)
- [Nginx Publishing and Cache](./nginx-publishing-cache.md)

### Operação e suporte
- [Systemd Automation](../01-infra-hostinger/systemd-automation.md)
- [Nginx Static Serving](../01-infra-hostinger/nginx-static-serving.md)
- [Operational Runbooks](./portal-estatico-operational-runbooks.md)
- [Observability and Troubleshooting](./portal-estatico-observability-troubleshooting.md)

---

## Escopo

Este documento cobre:

- o papel arquitetural da Static API v2
- o base path oficial
- os recursos públicos conhecidos da v2
- a lógica de versionamento da camada
- os invariantes operacionais da API estática
- a relação entre v2, ETL e frontend
- limites e riscos de compatibilidade

Este documento não cobre em profundidade:

- estrutura campo a campo de cada JSON
- queries SQL detalhadas
- pipeline ETL script por script
- troubleshooting aprofundado
- operação do runtime do jogo
- Auth API no Lightsail

Esses tópicos vivem em documentos específicos do contexto.

---

## Estado atual

O estado operacional conhecido da Static API v2 é:

- a v2 é a camada pública de dados do Portal Estático
- a v2 é composta por arquivos JSON estáticos
- a geração da v2 é feita por ETL Bash
- a fonte primária de dados é o `matchzy.db`
- a publicação final ocorre via Nginx
- a v2 é consumida pelo frontend público por `fetch`
- a v2 substitui o modelo anterior como versão oficial em uso
- a arquitetura do portal assume v2 como contrato público estável da camada de dados

A v2 existe para entregar dados públicos previsíveis com custo operacional baixo e baixa complexidade de serving.

---

## Source of truth / evidências

As principais evidências deste documento, nesta fase de migração documental, são:

- documentação consolidada atual do ecossistema HSC
- blueprint técnico consolidado do HSC
- documentação reconciliada da Static API v2
- inventário de recursos públicos já documentados
- reconciliação entre ETL, paths de publicação e consumo pelo frontend

Enquanto a migração canônica do contexto não estiver concluída, essas fontes seguem sendo usadas como base de reconciliação.

---

## Relações com outros documentos

Este arquivo é complementar a:

- `docs/03-portal-estatico/README.md`
- `docs/03-portal-estatico/portal-estatico-architecture-runtime.md`
- `docs/03-portal-estatico/data-sources-matchzy-sqlite.md`
- `docs/03-portal-estatico/etl-bash-pipeline.md`
- `docs/03-portal-estatico/sql-queries-and-views.md`
- `docs/03-portal-estatico/json-contracts.md`
- `docs/03-portal-estatico/portal-estatico-frontend-structure.md`
- `docs/03-portal-estatico/nginx-publishing-cache.md`
- `docs/03-portal-estatico/portal-estatico-operational-runbooks.md`
- `docs/03-portal-estatico/portal-estatico-observability-troubleshooting.md`

Este documento descreve a camada v2 como contrato público de alto nível.  
Ele não substitui os documentos de contrato detalhado, pipeline ou troubleshooting.

---

## Papel da Static API v2 no ecossistema

A Static API v2 é a camada pública de dados que conecta:

- o runtime operacional do jogo
- o pipeline ETL
- a publicação estática via Nginx
- o consumo público pelo Portal Estático

Seu papel é:

- transformar dados operacionais em artefatos públicos previsíveis
- expor contratos estáticos simples para o frontend
- reduzir a necessidade de backend dinâmico no portal
- permitir evolução controlada do formato de dados
- desacoplar a consulta pública da geração dos dados

Em termos arquiteturais, a v2 é uma “static file API”.

---

## Base path oficial

O base path oficial conhecido da Static API v2 é:

- `/var/www/api/cs2/v2/` no filesystem
- caminho público correspondente sob a camada servida pelo Nginx

Regra canônica:

- a v2 é organizada como conjunto de arquivos estáticos dentro da pasta `v2/`
- o versionamento da API é expresso pelo path
- o frontend deve consumir explicitamente a versão ativa esperada

Qualquer mudança desse path deve ser tratada como alteração relevante de contexto.

---

## Recursos publicados

Os recursos públicos conhecidos da Static API v2 incluem, no mínimo:

### `health.json`

Função:
- expor saúde operacional da cadeia de geração/publicação
- refletir checks básicos do contexto

---

### `ranking.json`

Função:
- expor ranking consolidado dos jogadores dentro das regras da v2

---

### `matches.json`

Função:
- expor lista pública de partidas ou recorte público de matches

---

### `match/{id}.json`

Função:
- expor detalhe público incremental de uma partida específica

---

### `maps.json`

Função:
- expor visão agregada por mapa

---

### `map/{map}.json`

Função:
- expor detalhe público por mapa

---

### `player/{steamid64}.json`

Função:
- expor dossiê público de jogador baseado no `steamid64`

---

### `steam-cache/{steamid64}.json`

Função:
- expor enriquecimento/cache ligado à camada pública do jogador

---

## Papel de cada recurso na arquitetura

Os recursos da v2 se distribuem, conceitualmente, em três grupos:

### 1. Saúde operacional

- `health.json`

Usado para:
- validar geração e publicação
- ajudar troubleshooting
- separar erro de pipeline de erro de frontend

---

### 2. Coleções públicas

- `ranking.json`
- `matches.json`
- `maps.json`

Usadas para:
- carregar listas públicas
- alimentar páginas agregadas do portal
- reduzir necessidade de montagem de agregados no browser

---

### 3. Recursos incrementais/detalhados

- `match/{id}.json`
- `map/{map}.json`
- `player/{steamid64}.json`
- `steam-cache/{steamid64}.json`

Usados para:
- páginas de detalhe
- enriquecimento específico
- publicação incremental controlada

---

## Regras de versionamento

A Static API v2 adota versionamento por pasta.

Isso significa:

- a versão faz parte do caminho da API estática
- mudanças relevantes de contrato devem ser tratadas com cuidado
- o frontend consome explicitamente a versão publicada que espera

Objetivos desse modelo:

- preservar compatibilidade
- permitir evolução controlada
- evitar quebra silenciosa do portal
- tornar o contrato público mais previsível

Regra importante:
- a v2 é a versão canônica atual
- a documentação nova não deve reintroduzir v1 como camada ativa

---

## Invariantes da v2

Os invariantes operacionais conhecidos da v2 incluem:

- a v2 é publicada como arquivos estáticos
- a geração da v2 depende de ETL determinístico
- a v2 não deve ser atualizada com escrita parcial visível ao público
- a v2 depende de lock global para evitar concorrência indevida
- a v2 depende de compatibilidade entre schema do SQLite, queries e contratos publicados
- a v2 depende de Nginx servindo apenas artefatos públicos finais
- a v2 deve permanecer compatível com o frontend em produção

Esses invariantes são parte central da confiabilidade da camada.

---

## Compatibilidade com o frontend

O frontend do Portal Estático depende da v2 como contrato público.

Isso implica:

- o browser espera arquivos em paths previsíveis
- mudanças de shape JSON podem quebrar páginas públicas
- mudanças de nome de recurso, path ou ausência de campo crítico precisam ser tratadas com disciplina
- a evolução do ETL não pode ignorar a compatibilidade com o portal

Regra prática:
- mudança na v2 deve ser tratada como mudança de contrato
- se a mudança impacta o frontend, ela precisa ser coordenada com `json-contracts.md` e `frontend-structure.md`

---

## Relação com o ETL

A Static API v2 não é escrita manualmente.

Ela é resultado do ETL Bash, que:

- consulta a fonte SQLite
- aplica queries e transformações
- escreve artefatos intermediários
- publica JSONs finais nos paths públicos da v2

Isso significa:

- falha no ETL afeta diretamente a v2
- drift entre SQL e schema afeta diretamente a v2
- erro de permissionamento pode impedir a publicação da v2 mesmo com ETL logicamente correto

A v2 é, portanto, inseparável da disciplina operacional do pipeline.

---

## Relação com a fonte de dados

A fonte primária conhecida da v2 é o `matchzy.db`, derivado do contexto operacional do jogo.

Implicações:

- a qualidade da v2 depende da qualidade e compatibilidade dos dados de origem
- limitações estruturais do MatchZy propagam limites para a v2
- mudança de instância/path do banco pode quebrar a geração inteira da camada

A v2 não “cria” verdade nova; ela publica, transforma e organiza a verdade operacional disponível na fonte.

---

## Relação com o Nginx

O Nginx é a camada de entrega da v2 ao público.

Responsabilidades do Nginx em relação à v2:

- servir os JSONs finais
- expor o base path público esperado
- aplicar cache curto quando apropriado
- bloquear artefatos que não devem ser públicos
- manter a separação entre paths internos e publicação pública

O Nginx não é o autor da v2; ele apenas publica seus outputs.

---

## Limites conhecidos

Os limites conhecidos ou estruturais da v2 incluem:

- dependência do schema e das capacidades do MatchZy
- ausência de backend dinâmico para composição em tempo real
- necessidade de regeneração para refletir novos dados
- risco de stale data quando o ETL não roda corretamente
- risco de quebra do portal se contratos mudarem sem coordenação

Também existem limites herdados do lado de dados operacionais, como:

- ausência de certos eventos granulares
- ausência de algumas visões analíticas detalhadas
- dependência de JSON1 no SQLite para parte da lógica de geração

Regra editorial:
- limites estruturais devem ser explicitados, não mascarados como se a v2 fosse uma API dinâmica completa

---

## Riscos de evolução

Os principais riscos ao evoluir a v2 incluem:

- mudar shape de JSON sem coordenação com o frontend
- alterar path público sem atualizar consumo
- alterar regras de geração sem revisar contratos
- publicar arquivo parcial por quebra de atomicidade
- introduzir concorrência na geração
- depender de query incompatível com o schema real

Mitigações recomendadas:

- tratar mudança na v2 como mudança de contrato
- manter `json-contracts.md` alinhado
- manter o frontend alinhado
- preservar lock global e escrita atômica
- registrar mudanças relevantes em `95-impl-log/`

---

## Sinais de saúde da v2

Os sinais de saúde esperados desta camada incluem:

- `health.json` presente e coerente
- `ranking.json` presente e consumível
- `matches.json` presente e consumível
- recursos de detalhe acessíveis conforme esperado
- frontend consumindo a v2 sem erro estrutural
- ausência de artefatos temporários ou parciais na publicação
- geração recente e consistente com o estado esperado do ETL

A saúde da v2 deve ser avaliada tanto pelo filesystem quanto pelo consumo real no portal.

---

## Limites deste documento

Este documento não detalha:

- campos individuais de cada JSON
- regras SQL de cada recurso
- detalhes incrementais de geração
- troubleshooting aprofundado da publicação
- detalhe do schema SQLite
- detalhe do frontend que consome a v2

Esses tópicos pertencem a documentos próprios.

---

## Critério de pronto deste documento

Este documento pode ser considerado maduro quando:

- a lista de recursos publicados estiver confirmada contra a publicação real
- o base path estiver validado no ambiente
- a lógica de versionamento por pasta estiver formalizada sem ambiguidade
- os invariantes da v2 estiverem claros
- a relação entre ETL, Nginx e frontend estiver corretamente representada
- ele puder ser lido como a definição macro da camada de dados públicos sem depender do master legado

---

## Última revisão

- Status: ativo
- Classificação: canônico
- Contexto: portal estático / Static API v2
- Última revisão: 2026-03-18