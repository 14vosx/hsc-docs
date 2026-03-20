# Frontend Structure

## Objetivo

Documentar a estrutura do frontend do Portal Estático do ecossistema HSC, registrando sua stack, seus princípios de implementação, sua relação com a Static API v2 e os limites operacionais da camada pública.

Este documento existe para registrar, de forma estável e auditável:

- a stack real do frontend público
- os princípios arquiteturais da camada de interface
- a organização macro de rotas e assets
- a relação entre browser, Nginx e Static API v2
- as responsabilidades do frontend e o que explicitamente não pertence a ele
- os riscos e cuidados de evolução dessa camada

---

## Escopo

Este documento cobre:

- a stack do frontend público
- os princípios de implementação da camada
- a organização macro das rotas públicas
- a organização dos assets do portal
- a relação entre frontend e Static API v2
- o modelo de carregamento de dados no browser
- os limites funcionais e técnicos do frontend atual

Este documento não cobre em profundidade:

- contratos JSON campo a campo
- pipeline ETL Bash
- queries SQL
- configuração completa do Nginx
- troubleshooting detalhado do portal
- Auth API no Lightsail
- detalhes operacionais do Game Panel

Esses tópicos vivem em documentos próprios deste contexto ou em outros contextos canônicos.

---

## Estado atual

O estado operacional conhecido do frontend do Portal Estático é:

- o portal público é entregue como frontend estático
- a stack base é HTML, CSS modular e JavaScript com ESModules
- não há framework frontend como base da camada pública atual
- o browser consome dados públicos via Static API v2
- a renderização depende de assets estáticos e fetch de JSON
- a navegação pública do portal é orientada a páginas e views leves
- o frontend não executa backend dinâmico próprio
- a experiência pública depende da compatibilidade entre código estático e contratos JSON publicados

Esse desenho existe para manter a camada pública simples, previsível e com baixo custo operacional.

---

## Source of truth / evidências

As principais evidências deste documento, nesta fase de migração documental, são:

- documentação consolidada atual do ecossistema HSC
- blueprint técnico consolidado do HSC
- documentação reconciliada do portal e da Static API v2
- inventário das rotas públicas conhecidas
- reconciliação da stack frontend sem framework
- relação documentada entre frontend, Nginx e arquivos JSON públicos

Enquanto a migração canônica do contexto não estiver concluída, essas fontes seguem sendo usadas como base de reconciliação do estado real.

---

## Relações com outros documentos

Este arquivo é complementar a:

- `docs/03-portal-estatico/README.md`
- `docs/03-portal-estatico/portal-estatico-architecture-runtime.md`
- `docs/03-portal-estatico/static-api-v2.md`
- `docs/03-portal-estatico/json-contracts.md`
- `docs/03-portal-estatico/nginx-publishing-cache.md`
- `docs/03-portal-estatico/portal-estatico-operational-runbooks.md`
- `docs/03-portal-estatico/portal-estatico-observability-troubleshooting.md`

Este documento descreve a camada de frontend do portal.  
Ele não substitui os documentos de contrato, pipeline, publicação ou troubleshooting.

---

## Stack do frontend

A stack pública conhecida do portal é:

- HTML
- CSS modular
- JavaScript com ESModules

Características dessa stack:

- entrega totalmente estática
- sem framework frontend como base obrigatória
- sem necessidade de runtime SPA complexo com backend dedicado
- dependência direta de fetch de JSON público
- simplicidade deliberada de publicação e manutenção

Essa escolha arquitetural reforça o princípio do HSC de manter o portal público leve e desacoplado.

---

## Princípios de implementação

Os princípios conhecidos do frontend incluem:

- simplicidade estrutural
- baixo acoplamento com backend dinâmico
- consumo de arquivos JSON públicos previsíveis
- separação entre interface e geração de dados
- assets estáticos servidos por Nginx
- ausência de dependência obrigatória de framework
- previsibilidade de deploy/publicação
- foco em páginas públicas de consulta

Regra canônica:

- o frontend não deve reimplementar lógica que pertence ao ETL
- o frontend deve consumir contratos já consolidados pela v2
- a camada pública deve continuar simples de servir e de depurar

---

## Modelo de execução no browser

O modelo atual de execução do frontend é:

1. o browser recebe HTML, CSS e JavaScript do portal
2. o JavaScript inicializa a página pública correspondente
3. o frontend faz fetch da Static API v2
4. o frontend interpreta os JSONs publicados
5. a interface renderiza listas, detalhes e estados públicos
6. o usuário navega entre páginas e views públicas

Esse desenho implica que:

- o portal depende fortemente da disponibilidade e compatibilidade da v2
- falhas de frontend podem surgir mesmo com assets estáticos íntegros, se os contratos JSON quebrarem
- a experiência pública depende tanto do browser quanto da publicação dos dados

---

## Rotas públicas

As rotas públicas conhecidas da camada de portal incluem, em nível macro:

### `/ranking`

Função:
- exibir o ranking público dos jogadores
- consumir principalmente `ranking.json`

---

### `/matches`

Função:
- exibir a listagem pública de partidas
- consumir principalmente `matches.json`

---

### `/match/:id`

Função:
- exibir o detalhe público de uma partida específica
- consumir principalmente `match/{id}.json`

---

### `/player/:steamid`

Função:
- exibir o dossiê público de um jogador
- consumir principalmente `player/{steamid64}.json`
- pode usar enriquecimento auxiliar de `steam-cache/{steamid64}.json`

---

### `/maps`

Função:
- exibir visão pública agregada de mapas
- consumir principalmente `maps.json`

Regra editorial:

- a lista acima representa as superfícies públicas já reconciliadas
- qualquer expansão de navegação pública deve ser tratada de forma coordenada com contratos JSON e publicação estática

---

## Relação entre rotas e recursos públicos

A relação estrutural conhecida entre frontend e v2 é:

- `/ranking` → `ranking.json`
- `/matches` → `matches.json`
- `/match/:id` → `match/{id}.json`
- `/player/:steamid` → `player/{steamid64}.json`
- `/maps` → `maps.json`
- páginas de mapa específicas → `map/{map}.json`

Essa relação é importante porque:

- mostra dependência direta entre navegação e contratos da v2
- facilita troubleshooting quando uma página quebra
- ajuda a identificar se o problema está no frontend, no JSON ou no publishing

---

## Organização dos assets

A organização dos assets do portal deve permanecer orientada a entrega estática.

Em nível macro, os assets incluem:

- arquivos HTML
- folhas de estilo CSS
- módulos JavaScript
- imagens e assets visuais
- arquivos públicos auxiliares necessários à navegação e à identidade visual

Princípios esperados:

- estrutura previsível
- paths públicos estáveis
- separação clara entre assets de interface e artefatos de dados
- publicação controlada pela camada de Nginx

O frontend não deve misturar seus assets com a árvore pública da Static API v2 além do ponto de consumo.

---

## HTML, CSS modular e JS ESModules

### HTML

Papel:
- fornecer estrutura base das páginas públicas
- servir como entrada estática do portal

### CSS modular

Papel:
- organizar estilos de forma previsível
- reduzir acoplamento visual desnecessário
- manter manutenção relativamente simples da camada pública

### JavaScript com ESModules

Papel:
- organizar lógica de páginas e views
- separar responsabilidades por módulo
- consumir os contratos públicos da v2
- montar a interface no browser

Regra canônica:
- a camada pública atual não deve ser documentada como se dependesse de React, Angular, Vue ou outro framework sem que isso exista no runtime real

---

## Regras de carregamento de dados

O carregamento de dados do frontend segue o modelo:

- assets estáticos primeiro
- dados públicos depois, via fetch
- renderização final no browser com base nos JSONs recebidos

Implicações:

- o frontend precisa lidar com ausência, erro ou shape inesperado de JSON
- o portal depende de paths públicos previsíveis
- qualquer alteração de contrato precisa ser coordenada
- JSON inválido ou ausente afeta diretamente a renderização da página

Regra importante:
- o frontend não deve assumir que “se a página carregou, os dados estão corretos”
- a renderização depende da saúde dos recursos públicos consumidos

---

## Responsabilidades do frontend

As responsabilidades principais do frontend incluem:

- renderizar páginas públicas
- solicitar recursos corretos da v2
- mapear JSON para interface pública
- lidar com estados básicos de carregamento e erro
- preservar consistência de navegação pública
- manter compatibilidade com a versão ativa da v2

O frontend não é responsável por:

- gerar ranking
- calcular agregados primários do ETL
- persistir estado administrativo do sistema
- operar o runtime do jogo
- manter sessão administrativa
- substituir a Auth API

---

## O que não existe no frontend

O frontend atual do portal não deve ser tratado como se tivesse:

- backend dinâmico próprio
- SSR
- autenticação administrativa central
- escrita direta em banco
- camada de API dinâmica local
- framework SPA complexo obrigatório
- geração de dados estatísticos em tempo real
- dependência de renderização no servidor para funcionamento público básico

Esses pontos são importantes para evitar drift documental e expectativas erradas sobre a camada pública.

---

## Dependências operacionais da camada

O frontend depende, em nível operacional, de:

- Nginx servindo os assets estáticos
- Static API v2 publicada corretamente
- contratos JSON compatíveis com a implementação atual
- paths públicos estáveis
- assets íntegros
- browser executando ESModules corretamente
- integridade do cache e dos cabeçalhos públicos
- ausência de JSON truncado ou stale além do aceitável

Também há dependência indireta de:

- ETL funcional
- `matchzy.db` íntegro
- filesystem com permissões corretas
- publishing consistente

---

## Invariantes da camada

Os invariantes conhecidos do frontend incluem:

- o portal é público e estático
- a navegação depende de JSONs públicos já gerados
- o frontend não deve depender de backend dinâmico da própria camada
- os módulos JS devem permanecer compatíveis com os contratos da v2
- as rotas públicas devem permanecer coerentes com os recursos publicados
- a publicação deve continuar simples e previsível
- o consumo de dados deve permanecer same-origin ou coerente com a borda configurada

Esses invariantes ajudam a preservar a identidade técnica do portal.

---

## Riscos de evolução

Os principais riscos de evolução do frontend incluem:

- alteração de contrato JSON sem coordenação
- mudança de path público sem atualização do código
- quebra de navegação por recurso de detalhe ausente
- crescimento desordenado da camada sem documentação correspondente
- introdução de dependências complexas sem necessidade arquitetural real
- mistura entre responsabilidades de frontend e ETL
- dependência excessiva de enriquecimentos opcionais sem fallback

Mitigações recomendadas:

- tratar a v2 como contrato público
- documentar qualquer mudança relevante em impl-log
- manter a estrutura modular previsível
- validar rotas e recursos após publicação
- evitar “empurrar” lógica do backend para o browser sem necessidade

---

## Sinais de saúde do frontend

Os sinais de saúde esperados da camada incluem:

- assets estáticos sendo servidos corretamente
- páginas públicas carregando sem erro estrutural
- fetch de JSONs funcionando
- tabelas/listas/detalhes renderizando corretamente
- navegação entre coleções e detalhes funcionando
- ausência de erro estrutural recorrente no console por quebra de contrato
- comportamento coerente com a versão ativa da v2

A saúde do frontend depende de duas coisas ao mesmo tempo:

- integridade dos assets
- integridade dos dados publicados

---

## Problemas comuns

### 1. Página carrega, mas não mostra dados

Causas comuns:

- JSON ausente
- fetch falhando
- contrato incompatível
- recurso publicado em path diferente
- erro no módulo da página

Impacto:
- usuário vê interface vazia ou incompleta

---

### 2. Lista funciona, mas detalhe quebra

Causas comuns:

- recurso incremental ausente
- mismatch entre identificador da coleção e path do detalhe
- mudança de contrato não coordenada
- falha parcial do ETL

Impacto:
- navegação pública fica incoerente

---

### 3. Frontend quebra após mudança na v2

Causas comuns:

- campo removido
- shape alterado
- semântica de JSON mudou
- enriquecimento esperado não veio

Impacto:
- erro no browser mesmo com publishing aparentemente correto

---

### 4. Assets servem corretamente, mas JSON está stale

Causas comuns:

- ETL não executou
- cache servindo artefato antigo
- statefile ou incremental inconsistente
- problema de publishing

Impacto:
- frontend parece normal, mas mostra informação desatualizada

---

### 5. Console do browser mostra erro estrutural

Causas comuns:

- módulo JS esperando shape diferente
- recurso não encontrado
- JSON inválido
- navegação para rota não suportada pelo estado publicado

Impacto:
- quebra parcial ou total da experiência da página

---

## Limites conhecidos

Os limites conhecidos do frontend atual incluem:

- dependência total da v2 para dados públicos
- ausência de backend dinâmico local para composição sob demanda
- necessidade de coordenação explícita com ETL e contratos
- arquitetura voltada para simplicidade, não para alta abstração de UI
- dependência de paths e naming estáveis
- possibilidade de degradação visual quando recursos auxiliares, como enriquecimentos, não estão disponíveis

Esses limites não são defeitos; eles são parte do desenho da camada.

---

## Limites deste documento

Este documento não detalha:

- código-fonte módulo a módulo
- mapeamento exato de cada componente visual
- contratos JSON campo a campo
- troubleshooting aprofundado do browser
- headers de cache do Nginx
- estratégia visual e design system em profundidade

Esses tópicos devem ser tratados em documentos complementares, se necessário.

---

## Critério de pronto deste documento

Este documento pode ser considerado maduro quando:

- a stack real do frontend estiver reconciliada sem ambiguidade
- as rotas públicas principais estiverem corretamente representadas
- a relação entre frontend e v2 estiver clara
- os limites e responsabilidades da camada estiverem explícitos
- os riscos de evolução estiverem bem demarcados
- ele puder ser lido como referência da estrutura da camada pública sem depender do master legado

---

## Última revisão

- Status: ativo
- Classificação: canônico
- Contexto: portal estático / estrutura do frontend
- Última revisão: 2026-03-18