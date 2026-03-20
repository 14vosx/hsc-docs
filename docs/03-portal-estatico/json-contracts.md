# JSON Contracts

## Objetivo

Documentar os contratos JSON publicados pela Static API v2 do contexto Portal Estático do ecossistema HSC.

Este documento existe para registrar, de forma estável e auditável:

- quais artefatos JSON públicos existem na v2
- qual é o papel de cada contrato na arquitetura pública do portal
- quais campos e estruturas são esperados em nível macro
- quais invariantes o frontend assume ao consumir esses recursos
- quais regras de compatibilidade precisam ser preservadas
- quais limites e cuidados existem ao evoluir os contratos públicos

---

## Navegação

### Entrada
- [Home da documentação](../README.md)
- [Portal Estático](./README.md)
- [Master Index](../00-governance/99-master-index.md)

### Origem e transformação dos dados
- [Data Sources — MatchZy SQLite](./data-sources-matchzy-sqlite.md)
- [SQL Queries and Views](./sql-queries-and-views.md)
- [ETL Bash Pipeline](./etl-bash-pipeline.md)
- [Static API v2](./static-api-v2.md)

### Consumo e publicação
- [Frontend Structure](./portal-estatico-frontend-structure.md)
- [Nginx Publishing and Cache](./nginx-publishing-cache.md)
- [Operational Runbooks](./portal-estatico-operational-runbooks.md)

### Infraestrutura e suporte
- [Systemd Automation](../01-infra-hostinger/systemd-automation.md)
- [Nginx Static Serving](../01-infra-hostinger/nginx-static-serving.md)
- [Observability and Troubleshooting](./portal-estatico-observability-troubleshooting.md)

---

## Escopo

Este documento cobre:

- os princípios gerais dos contratos JSON da v2
- os artefatos públicos principais da Static API v2
- os contratos macro de `health.json`, `ranking.json`, `matches.json`, `match/{id}.json`, `maps.json`, `map/{map}.json`, `player/{steamid64}.json` e `steam-cache/{steamid64}.json`
- os campos derivados e enriquecimentos conhecidos em alto nível
- as regras de compatibilidade entre ETL, publicação e frontend
- as validações mínimas esperadas dos contratos

Este documento não cobre em profundidade:

- queries SQL de origem
- detalhe do pipeline ETL script por script
- troubleshooting aprofundado
- a implementação do frontend
- o schema SQLite linha a linha
- o funcionamento do MatchZy como runtime de jogo

Esses tópicos vivem em documentos próprios do contexto ou em outros contextos canônicos.

---

## Estado atual

O estado operacional conhecido dos contratos JSON da v2 é:

- a camada pública de dados do portal é publicada como arquivos JSON estáticos
- cada recurso público possui papel específico dentro da navegação do portal
- o frontend depende desses contratos em tempo de consumo
- a geração dos JSONs é feita por ETL Bash
- a publicação final é servida por Nginx
- a compatibilidade dos contratos é crítica para o funcionamento do portal
- a v2 é a versão canônica atual da API estática

A integridade do portal depende não apenas da existência dos arquivos, mas também da estabilidade semântica desses contratos.

---

## Source of truth / evidências

As principais evidências deste documento, nesta fase de migração documental, são:

- documentação consolidada atual do ecossistema HSC
- blueprint técnico consolidado do HSC
- reconciliação documental da Static API v2
- inventário dos recursos públicos da v2
- reconciliação entre ETL, outputs públicos e consumo pelo frontend
- referências ao dossiê público de jogador, enriquecimento de Steam e artefatos agregados da v2

Enquanto a migração canônica do contexto não estiver concluída, essas fontes seguem sendo usadas como base de reconciliação do estado real.

---

## Relações com outros documentos

Este arquivo é complementar a:

- `docs/03-portal-estatico/README.md`
- `docs/03-portal-estatico/portal-estatico-architecture-runtime.md`
- `docs/03-portal-estatico/static-api-v2.md`
- `docs/03-portal-estatico/data-sources-matchzy-sqlite.md`
- `docs/03-portal-estatico/etl-bash-pipeline.md`
- `docs/03-portal-estatico/sql-queries-and-views.md`
- `docs/03-portal-estatico/portal-estatico-frontend-structure.md`
- `docs/03-portal-estatico/portal-estatico-operational-runbooks.md`
- `docs/03-portal-estatico/portal-estatico-observability-troubleshooting.md`

Este documento descreve os contratos públicos da v2 em nível canônico.  
Ele não substitui os documentos de ETL, origem dos dados, frontend ou troubleshooting.

---

## Princípios de contrato

Os contratos JSON da v2 seguem os seguintes princípios:

- são públicos e estáticos
- são publicados em paths previsíveis
- são consumidos diretamente pelo frontend
- devem ser compatíveis com a versão de frontend em produção
- não devem expor artefatos parciais
- devem refletir a saída final validada do ETL
- devem ser tratados como interface pública da camada de dados do portal

Regra de ouro:

**qualquer mudança relevante de shape, nome, path ou semântica em JSON público deve ser tratada como mudança de contrato**

---

## Estrutura geral dos contratos

Os contratos da v2 se dividem, conceitualmente, em quatro grupos:

### 1. Saúde operacional

- `health.json`

### 2. Coleções públicas agregadas

- `ranking.json`
- `matches.json`
- `maps.json`

### 3. Recursos públicos de detalhe

- `match/{id}.json`
- `map/{map}.json`
- `player/{steamid64}.json`

### 4. Recursos auxiliares de enriquecimento

- `steam-cache/{steamid64}.json`

Essa separação é importante porque o frontend consome cada grupo com expectativas diferentes.

---

## Contrato de `health.json`

### Papel

`health.json` é o artefato de saúde da camada pública da v2.

### Finalidade

Seu objetivo é:

- sinalizar o estado recente da geração/publicação
- apoiar troubleshooting da pipeline
- ajudar a distinguir problema de geração de problema de frontend
- servir como artefato técnico de verificação do contexto

### Estrutura macro esperada

Em nível macro, o contrato de `health.json` deve conter informações coerentes com:

- estado geral da geração
- data/hora de atualização
- checks essenciais do contexto
- indicadores de integridade da camada pública

Regra importante:
- `health.json` deve refletir a saúde operacional da publicação da v2
- ele não deve ser tratado como substituto da validação funcional completa dos outros recursos

---

## Contrato de `ranking.json`

### Papel

`ranking.json` é a coleção pública principal de ranking de jogadores.

### Finalidade

Seu objetivo é:

- expor ordenação pública de jogadores
- alimentar páginas agregadas de ranking
- servir como uma das fontes para cobertura de dossiês públicos de jogadores

### Estrutura macro esperada

Em nível macro, `ranking.json` deve conter:

- uma coleção ordenada de jogadores
- identificador consistente do jogador, especialmente `steamid64`
- campos agregados de performance coerentes com a regra do ranking v2
- estrutura compatível com renderização de tabela/lista no frontend

### Requisitos de consistência

`ranking.json` deve ser coerente com:

- `player/{steamid64}.json`
- regras de ranking publicadas pela v2
- universo de jogadores efetivamente expostos ao público

---

## Contrato de `matches.json`

### Papel

`matches.json` é a coleção pública principal de partidas.

### Finalidade

Seu objetivo é:

- expor a lista agregada de partidas públicas
- alimentar páginas de listagem e navegação por partidas
- servir como base para acesso ao detalhe incremental de cada partida

### Estrutura macro esperada

Em nível macro, `matches.json` deve conter:

- uma coleção de partidas
- identificadores de match utilizáveis para navegação a `match/{id}.json`
- metadados básicos de cada partida
- estrutura adequada para listagem pública

### Requisitos de consistência

`matches.json` deve ser coerente com:

- `match/{id}.json`
- a ordenação pública esperada da coleção
- a presença dos recursos de detalhe realmente publicados

---

## Contrato de `match/{id}.json`

### Papel

`match/{id}.json` representa o detalhe público incremental de uma partida específica.

### Finalidade

Seu objetivo é:

- fornecer detalhe público além da coleção de matches
- alimentar a página de detalhe de uma partida
- preservar granularidade pública controlada por match

### Estrutura macro esperada

Em nível macro, o contrato deve conter:

- identificação clara da partida
- metadados relevantes do match
- informações estruturais necessárias para a página de detalhe
- consistência com o item correspondente em `matches.json`

### Requisitos de consistência

O recurso de detalhe deve:

- corresponder a um match conhecido pela camada pública
- manter coerência mínima com a coleção
- não assumir dependência de backend dinâmico em tempo de renderização

---

## Contrato de `maps.json`

### Papel

`maps.json` é a coleção pública agregada por mapa.

### Finalidade

Seu objetivo é:

- expor visão agregada por mapa
- alimentar páginas/listas de mapas
- servir de base para recursos detalhados por mapa

### Estrutura macro esperada

Em nível macro, `maps.json` deve conter:

- uma coleção de mapas
- identificador ou slug consistente de cada mapa
- métricas agregadas públicas coerentes com a camada de origem
- estrutura apropriada para listagem e navegação

### Requisitos de consistência

`maps.json` deve ser coerente com:

- `map/{map}.json`
- a nomenclatura pública de mapas adotada pelo portal
- os dados efetivamente derivados da tabela de origem

---

## Contrato de `map/{map}.json`

### Papel

`map/{map}.json` representa o detalhe público incremental de um mapa específico.

### Finalidade

Seu objetivo é:

- expor detalhe público por mapa
- complementar a coleção agregada de mapas
- alimentar páginas específicas de mapa

### Estrutura macro esperada

Em nível macro, o contrato deve conter:

- identificação do mapa
- dados agregados ou detalhados coerentes com a página pública correspondente
- consistência com a entrada correspondente em `maps.json`

### Requisitos de consistência

O recurso de detalhe deve:

- usar identificador/slug estável
- corresponder ao universo de mapas publicado na coleção
- permanecer compatível com a navegação pública do frontend

---

## Contrato de `player/{steamid64}.json`

### Papel

`player/{steamid64}.json` é o dossiê público de jogador na v2.

### Finalidade

Seu objetivo é:

- representar a entidade pública de jogador
- alimentar a página de detalhe do jogador
- consolidar dados públicos individuais derivados do ranking, do banco e do enriquecimento disponível

### Estrutura macro esperada

Em nível macro, esse contrato deve conter:

- identificador principal `steamid64`
- informações públicas de identidade e/ou apresentação do jogador
- métricas públicas individuais
- dados coerentes com o ranking e com a origem operacional disponível
- estrutura apropriada para a renderização da página pública de jogador

### Requisitos de consistência

O dossiê de jogador deve ser coerente com:

- `ranking.json`
- `steam-cache/{steamid64}.json`, quando houver enriquecimento associado
- os dados estruturais disponíveis na camada de origem

---

## Contrato de `steam-cache/{steamid64}.json`

### Papel

`steam-cache/{steamid64}.json` é o recurso auxiliar de enriquecimento associado ao jogador.

### Finalidade

Seu objetivo é:

- complementar o dossiê público de jogador com dados auxiliares
- reduzir necessidade de busca repetida por enriquecimentos externos
- preservar uma camada pública estática de metadados auxiliares

### Estrutura macro esperada

Em nível macro, esse recurso pode conter campos como:

- identificador `steamid64`
- dados auxiliares de apresentação pública
- informações de avatar ou perfil, quando reconciliadas nesse fluxo

Regra importante:
- o cache auxiliar não deve substituir a identidade estrutural do jogador na v2
- ele é complementar ao `player/{steamid64}.json`

---

## Campos derivados e enriquecimento

A v2 já possui evidência reconciliada de enriquecimentos públicos associados à camada de jogador.

Exemplos conhecidos em alto nível incluem:

- `avatarMedium`
- `steamProfileUrl`

Esses campos existem para melhorar a experiência pública sem alterar a natureza estática da v2.

Regra editorial:
- enriquecimento deve continuar sendo tratado como complementar
- o portal não deve depender de fonte externa em tempo de renderização para compor o básico do contrato público

---

## Regras de compatibilidade

Os contratos JSON da v2 devem preservar compatibilidade com o frontend ativo.

Isso implica:

- não remover campos críticos sem coordenação
- não renomear recursos públicos sem coordenação
- não mudar o shape de listas/objetos sem coordenação
- não alterar semântica central de um artefato sem atualização documental e alinhamento com o frontend

Regra canônica:
- se o frontend já consome um campo como estrutural, sua mudança deve ser tratada como breaking change

---

## Compatibilidade entre agregados e detalhes

A v2 precisa preservar coerência entre:

- coleções e seus recursos detalhados
- ranking e player
- maps e map detail
- matches e match detail

Exemplos de coerência esperada:

- item em `ranking.json` deve poder levar a `player/{steamid64}.json`
- item em `matches.json` deve poder levar a `match/{id}.json`
- item em `maps.json` deve poder levar a `map/{map}.json`

Quebras nessa coerência afetam diretamente a navegabilidade do portal.

---

## Regras de validação

A validação mínima dos contratos deve incluir:

- existência do arquivo esperado
- JSON válido
- shape macro compatível com a expectativa do frontend
- identificadores coerentes entre recursos relacionados
- artefatos públicos sem truncamento
- coerência mínima entre coleção e detalhe

Ferramentas de validação típicas incluem:

- `jq`
- inspeção de paths públicos
- fetch real do frontend
- smoke checks manuais

Exemplos representativos:

```bash
jq . /var/www/api/cs2/v2/health.json
jq . /var/www/api/cs2/v2/ranking.json
jq . /var/www/api/cs2/v2/matches.json
```

Substitua paths conforme necessário para recursos detalhados.

---

## Riscos de quebra de contrato

Os principais riscos desta camada incluem:

- mudança de campo sem coordenação com o frontend
- mudança de path público sem coordenação
- mudança de identificador de detalhe
- alteração de semântica entre coleção e detalhe
- enriquecimento desaparecendo sem fallback adequado
- publicação parcial de arquivo JSON
- mudança em query ou ETL sem atualização documental

Esses riscos devem ser mitigados por:

- disciplina de mudança
- validação pós-geração
- escrita atômica
- impl-log quando a mudança for relevante
- alinhamento entre ETL, contratos e frontend

---

## Sinais de saúde dos contratos

Os sinais de saúde esperados incluem:

- todos os recursos principais presentes
- JSONs válidos
- recursos de detalhe coerentes com suas coleções
- ranking coerente com páginas de player
- recursos de mapa coerentes com páginas de mapa
- frontend consumindo a v2 sem erro estrutural
- ausência de truncamento ou shape inesperado

Saúde de contrato não é apenas “arquivo existe”.  
É necessário que o shape continue útil para o portal.

---

## Problemas comuns

### 1. JSON válido, mas shape incompatível

Causas comuns:

- campo removido
- campo renomeado
- mudança de nesting
- array virou objeto ou vice-versa

Impacto:
- frontend quebra mesmo com JSON sintaticamente válido

---

### 2. Coleção publica item sem detalhe correspondente

Causas comuns:

- falha no incremental
- drift entre etapas da pipeline
- problema de path de output

Impacto:
- navegação quebra ao sair da lista para a página de detalhe

---

### 3. Dossiê de jogador existe, mas ranking não referencia corretamente

Causas comuns:

- mudança nas regras do ranking
- geração parcial
- inconsistência de `steamid64`

Impacto:
- incoerência entre listagem agregada e recurso de detalhe

---

### 4. Recurso auxiliar de Steam está ausente ou stale

Causas comuns:

- enriquecimento não atualizado
- falha em cache auxiliar
- pipeline parcial

Impacto:
- experiência visual degradada em páginas de player
- sem necessariamente quebrar a camada estatística principal

---

### 5. Health indica normalidade, mas contrato específico quebrou

Causas comuns:

- `health.json` cobre apenas checks básicos
- regressão localizada em um recurso
- mudança parcial de ETL

Impacto:
- falsa percepção de integridade completa da v2

---

## Limites deste documento

Este documento não detalha:

- cada campo individual de cada JSON
- regras SQL que alimentam cada campo
- implementação do parsing no frontend
- troubleshooting aprofundado de geração
- a origem exata de cada dado no schema SQLite

Esses detalhes pertencem a documentos específicos do contexto.

---

## Critério de pronto deste documento

Este documento pode ser considerado maduro quando:

- os contratos macro estiverem reconciliados com os arquivos realmente publicados
- a relação entre coleção e detalhe estiver clara
- os identificadores estruturais estiverem estáveis
- os enriquecimentos conhecidos estiverem formalizados em nível adequado
- as regras de compatibilidade estiverem explícitas
- ele puder ser lido como definição canônica dos contratos públicos da v2 sem depender do master legado

---

## Última revisão

- Status: ativo
- Classificação: canônico
- Contexto: portal estático / contratos JSON da Static API v2
- Última revisão: 2026-03-18