# Architecture Runtime

## Objetivo

Documentar a arquitetura real de runtime do contexto Portal Estático do ecossistema HSC, registrando a cadeia de dados, os componentes envolvidos e as dependências operacionais que sustentam a publicação pública do portal e da Static API v2.

Este documento existe para registrar, de forma estável e auditável:

- a topologia do contexto do portal
- os componentes reais que compõem a cadeia de publicação
- a relação entre SQLite, ETL Bash, JSON estático, Nginx e browser
- os paths oficiais mais relevantes do contexto
- as dependências operacionais centrais dessa camada
- os principais failure domains do runtime do portal

---

## Escopo

Este documento cobre:

- a visão geral da arquitetura do Portal Estático
- os componentes da cadeia de publicação
- o fluxo browser → Nginx → JSON
- o fluxo MatchZy → SQLite → ETL → JSON
- o fluxo de espelho same-origin de conteúdo, quando aplicável
- os paths principais do contexto
- as dependências operacionais e os failure domains

Este documento não cobre em profundidade:

- o detalhe completo do ETL script por script
- contratos JSON campo a campo
- queries SQL detalhadas
- troubleshooting aprofundado
- configuração completa do Nginx
- operação do AMP e da instância CS2
- Auth API no Lightsail

Esses tópicos vivem em documentos próprios deste contexto ou em outros contextos canônicos.

---

## Estado atual

O runtime conhecido do contexto `03-portal-estatico` é:

- portal público 100% estático
- frontend servido por Nginx
- dados públicos expostos por Static API v2
- artefatos JSON gerados por ETL Bash
- fonte de dados baseada em SQLite (`matchzy.db`)
- pipeline determinístico com lock global
- escrita atômica de JSONs
- estrutura de publicação baseada em paths públicos dedicados
- possibilidade de espelho same-origin para conteúdo News

A topologia deste contexto existe para entregar experiência pública estável, barata e desacoplada do backend dinâmico principal.

---

## Source of truth / evidências

As principais evidências deste documento, nesta fase de migração documental, são:

- documentação consolidada atual do ecossistema HSC
- blueprint técnico consolidado do HSC
- documentação específica do portal e da Static API v2
- documentação reconciliada do pipeline ETL Bash
- inventário de paths e invariantes operacionais já consolidados

Enquanto a migração canônica do contexto não estiver concluída, essas fontes seguem sendo usadas como base de reconciliação do estado real.

---

## Relações com outros documentos

Este arquivo é complementar a:

- `docs/03-portal-estatico/README.md`
- `docs/03-portal-estatico/static-api-v2.md`
- `docs/03-portal-estatico/data-sources-matchzy-sqlite.md`
- `docs/03-portal-estatico/etl-bash-pipeline.md`
- `docs/03-portal-estatico/sql-queries-and-views.md`
- `docs/03-portal-estatico/json-contracts.md`
- `docs/03-portal-estatico/frontend-structure.md`
- `docs/03-portal-estatico/nginx-publishing-cache.md`
- `docs/03-portal-estatico/portal-estatico-operational-runbooks.md`
- `docs/03-portal-estatico/portal-estatico-observability-troubleshooting.md`

Este documento descreve a topologia e os fluxos do runtime do portal.  
Ele não substitui os documentos especializados do contexto.

---

## Visão geral da arquitetura

A arquitetura do Portal Estático é composta por uma cadeia simples e deliberadamente desacoplada:

- MatchZy produz dados operacionais no contexto do jogo
- esses dados são persistidos em SQLite
- o ETL Bash lê o banco e gera JSONs públicos
- os JSONs são publicados em diretório estático
- o Nginx serve os arquivos públicos
- o frontend consome esses arquivos via browser

Representação lógica resumida:

- runtime do jogo
- `matchzy.db`
- ETL Bash
- Static API v2 em arquivos JSON
- Nginx
- browser

Esse desenho evita depender de backend dinâmico na camada do portal.

---

## Componentes do contexto

### Portal estático

O portal é a interface pública do contexto.

Responsabilidades:

- apresentar navegação pública
- consumir JSONs da Static API v2
- renderizar páginas de ranking, partidas, jogadores e mapas
- operar sem framework frontend como base da camada pública atual

O portal não é responsável por gerar os dados que consome.

---

### Static API v2

A Static API v2 é a camada pública de dados do contexto.

Responsabilidades:

- expor arquivos JSON estáveis
- refletir o estado recente do ETL
- servir de contrato entre pipeline de geração e frontend

A Static API v2 não é um backend dinâmico; ela é um conjunto de artefatos estáticos publicados.

---

### ETL Bash

O ETL Bash é a ponte entre a fonte de dados e a publicação pública.

Responsabilidades:

- ler o SQLite do MatchZy
- aplicar queries e transformações
- gerar JSONs da v2
- manter ordem normativa de geração
- aplicar lock global
- preservar escrita atômica
- gerar health do contexto

Esse componente é central para a confiabilidade do portal.

---

### SQLite MatchZy

O SQLite do MatchZy é a fonte primária de dados operacionais deste contexto.

Responsabilidades:

- armazenar estatísticas geradas pelo runtime do jogo
- servir como input do ETL
- sustentar ranking, matches, players e maps publicados na v2

O banco SQLite não é parte da superfície pública do portal.

---

### Nginx

O Nginx é a camada de publicação pública do contexto.

Responsabilidades:

- servir o frontend estático
- servir os arquivos da Static API v2
- servir espelho same-origin de conteúdo, quando aplicável
- proteger arquivos que não devem ser expostos publicamente
- atuar como edge de entrega do portal e da API estática

O Nginx não gera os dados; ele entrega o resultado gerado previamente.

---

### News same-origin cache

O contexto pode operar com espelho same-origin de conteúdo de News.

Responsabilidades:

- oferecer consumo previsível pelo frontend
- reduzir acoplamento direto do browser com camada dinâmica externa, quando aplicável
- manter conteúdo público acessível sob o mesmo domínio público do portal

Esse componente é complementar.  
Ele não transforma a arquitetura do portal em backend dinâmico.

---

## Fluxo browser → Nginx → JSON

O fluxo público esperado deste contexto é:

1. o usuário acessa o portal ou um recurso público da API estática
2. o browser chama o domínio/caminho público do portal
3. o Nginx recebe a requisição
4. o Nginx serve arquivos HTML/CSS/JS do portal ou arquivos JSON da Static API v2
5. o browser processa o frontend
6. o frontend faz fetch dos JSONs públicos necessários
7. a interface é preenchida com dados estáticos recentes

Esse fluxo pressupõe que a geração dos dados já ocorreu antes do acesso do usuário.

---

## Fluxo MatchZy → SQLite → ETL → JSON

O fluxo de geração de dados do contexto é:

1. o runtime do jogo produz dados operacionais
2. o MatchZy persiste esses dados em `matchzy.db`
3. o ETL Bash lê o banco SQLite
4. o ETL gera artefatos temporários
5. os artefatos são validados e movidos atomicamente para os paths públicos
6. a Static API v2 passa a refletir o novo estado gerado

Esse desenho depende de:

- queries compatíveis com o schema
- lock global para evitar corrida
- statefiles coerentes para incremental
- escrita atômica para evitar JSON parcial

---

## Fluxo Auth API → espelho local de News

Quando o espelho same-origin de News está ativo, o fluxo conceitual é:

1. a origem dinâmica do conteúdo existe em outra camada do ecossistema
2. o conteúdo é reconciliado ou espelhado para consumo público same-origin
3. o Nginx publica esse conteúdo no contexto do portal
4. o browser consome o recurso sob o mesmo domínio público do portal

Esse fluxo é importante porque mostra que o portal pode entregar conteúdo público integrado sem se tornar dependente de backend dinâmico próprio.

---

## Paths oficiais

Os paths principais conhecidos deste contexto incluem:

### Publicação do portal

- `/var/www/portal/cs2/`

### Publicação da Static API v2

- `/var/www/api/cs2/v2/`

### Base operacional do ETL

- `/opt/cs2-portal/`

### Locks do ETL

- `/opt/cs2-portal/locks/`

### Statefiles do ETL

- `/opt/cs2-portal/state/`

### Lock global conhecido

- `/opt/cs2-portal/locks/gen-all-v2.lock`

### Scripts operacionais

- `/usr/local/bin/*.sh`

Esses paths são fundamentais para entender o runtime deste contexto e sua relação com a Infra Hostinger.

---

## Dependências operacionais

Este contexto depende, em nível de runtime, de:

- Nginx funcional
- filesystem com permissões corretas
- ETL Bash funcional
- SQLite acessível no path real do contexto do jogo
- lock global funcionando
- queries compatíveis com o schema do banco
- statefiles íntegros
- escrita atômica funcionando corretamente
- JSONs publicados em diretórios acessíveis ao Nginx
- frontend compatível com os contratos publicados

Dependências cruzadas importantes:

- o contexto depende de dados produzidos fora dele, no runtime do jogo
- o contexto depende da Infra Hostinger para serving, filesystem e automação
- o contexto depende de disciplina operacional para não publicar artefatos parciais

---

## Invariantes operacionais

Os invariantes já reconciliados deste contexto incluem:

- o portal é 100% estático
- o portal não executa backend dinâmico próprio
- o ETL não deve rodar concorrentemente
- a escrita dos JSONs deve ser atômica
- a Static API v2 deve ser servida como conjunto de arquivos
- a fonte de dados deve continuar sendo o SQLite do MatchZy enquanto este desenho permanecer oficial
- o frontend deve permanecer compatível com a v2 publicada
- Nginx deve servir apenas arquivos públicos da camada

Esses invariantes são centrais para manter a arquitetura previsível.

---

## Failure domains

Os principais failure domains deste contexto são:

### 1. Fonte de dados

Se o `matchzy.db` estiver indisponível, inconsistente ou incompatível:

- o ETL falha
- a v2 deixa de atualizar
- o portal passa a servir dados antigos ou incompletos

---

### 2. ETL

Se o ETL falhar:

- novos JSONs deixam de ser gerados
- health do contexto pode indicar degradação
- o frontend continua de pé, mas consumindo dados stale ou incompletos

---

### 3. Lock e concorrência

Se o lock global falhar ou for ignorado:

- podem surgir arquivos inconsistentes
- a v2 pode entrar em estado parcialmente atualizado
- troubleshooting fica mais difícil

---

### 4. Filesystem e permissões

Se paths, ownership ou permissões estiverem incorretos:

- o ETL pode não conseguir escrever
- o Nginx pode não conseguir ler
- a publicação pública quebra mesmo com pipeline teoricamente correto

---

### 5. Nginx

Se o Nginx falhar:

- o portal e a Static API v2 ficam indisponíveis publicamente
- os arquivos podem continuar existindo no disco, mas sem exposição útil

---

### 6. Compatibilidade entre ETL e frontend

Se o contrato JSON mudar sem coordenação:

- o portal pode quebrar mesmo com JSONs gerados corretamente
- erros aparecem no browser e não necessariamente no ETL

---

### 7. Espelho same-origin

Se o fluxo de espelho de conteúdo falhar:

- o portal pode perder uma superfície pública específica
- o restante da v2 pode continuar saudável
- o problema pode parecer de frontend, mas estar na camada de integração de conteúdo

---

## Sinais de saúde do contexto

Os sinais de saúde esperados deste contexto incluem:

- JSONs recentes publicados em `/var/www/api/cs2/v2/`
- `health.json` coerente
- ranking, matches, players e maps acessíveis publicamente
- frontend carregando e consumindo os JSONs esperados
- ETL executando sem concorrência indevida
- ausência de artefatos temporários órfãos na publicação
- Nginx servindo corretamente portal e API estática

A saúde deste contexto deve ser avaliada tanto pelo ponto de vista do pipeline quanto pelo ponto de vista do consumo público.

---

## Limites deste documento

Este documento não detalha:

- ordem completa do pipeline script por script
- estrutura campo a campo dos JSONs
- queries SQL específicas
- configuração textual do Nginx
- troubleshooting detalhado do portal
- operação detalhada do runtime do jogo
- origem exata do `matchzy.db` dentro do contexto AMP

Esses detalhes pertencem a documentos específicos.

---

## Critério de pronto deste documento

Este documento pode ser considerado maduro quando:

- a cadeia completa SQLite → ETL → JSON → Nginx → browser estiver reconciliada com o ambiente real
- os paths oficiais estiverem confirmados
- os failure domains estiverem corretamente representados
- as dependências cruzadas estiverem claras
- ele puder ser lido como visão topológica do contexto sem depender do master legado

---

## Última revisão

- Status: ativo
- Classificação: canônico
- Contexto: portal estático / arquitetura de runtime
- Última revisão: 2026-03-18