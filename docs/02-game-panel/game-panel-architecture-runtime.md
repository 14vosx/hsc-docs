# Architecture Runtime

## Objetivo

Documentar a topologia real de runtime do contexto Game Panel do ecossistema HSC, registrando como AMP, instância, servidor CS2, MatchZy, plugins e persistências operacionais se relacionam.

Este documento existe para registrar, de forma estável e auditável:

- a visão geral do runtime do lado jogo
- a posição do AMP na topologia operacional
- o papel da instância oficial `MixHAXIXE01`
- a relação entre Counter-Strike 2, MatchZy e plugins
- a relação entre o runtime do jogo e a produção de dados estatísticos
- as dependências operacionais e os failure domains centrais desta camada

---

## Escopo

Este documento cobre:

- a topologia operacional do Game Panel
- o papel estrutural do AMP
- a instância oficial `MixHAXIXE01`
- o runtime do servidor CS2
- o papel do MatchZy
- o papel dos plugins instalados
- a produção de dados operacionais locais
- a relação com a Infra Hostinger e com o Portal Estático
- os principais failure domains do lado jogo

Este documento não cobre em profundidade:

- Docker host como substrate da Infra Hostinger
- Nginx, Certbot, DNS e TLS da Hostinger
- pipeline ETL Bash da Static API v2
- contratos JSON do portal
- Auth API no AWS Lightsail
- runbooks detalhados de operação da partida

Esses tópicos vivem em documentos próprios dos contextos `01-infra-hostinger`, `03-portal-estatico` e `04-infra-aws-lightsail`.

---

## Estado atual

O runtime atual conhecido do contexto `02-game-panel` é:

- o lado jogo roda sobre a base Hostinger
- o AMP Instance Manager gerencia a instância oficial do ecossistema
- a instância oficial conhecida é `MixHAXIXE01`
- o servidor Counter-Strike 2 roda como workload dessa instância
- o MatchZy é componente central da operação competitiva
- plugins complementam administração, gameplay e persistência auxiliar
- o runtime do jogo produz estatísticas locais persistidas
- essas estatísticas alimentam, indiretamente, o Portal Estático por meio do `matchzy.db`

O Game Panel é a camada onde o servidor CS2 deixa de ser apenas infraestrutura e passa a operar como sistema competitivo real.

---

## Source of truth / evidências

As principais evidências deste documento, nesta fase de migração documental, são:

- documentação consolidada atual do ecossistema HSC
- blueprint técnico consolidado do HSC
- documentação específica da camada AMP/CS2
- documentação reconciliada do MatchZy, plugins e runbooks do servidor
- reconciliação estrutural da instância `MixHAXIXE01`
- reconciliação do papel do `matchzy.db` como fonte estatística do ecossistema público

Enquanto a migração canônica do contexto não estiver concluída, essas fontes seguem sendo usadas como base de reconciliação do estado real.

---

## Relações com outros documentos

Este arquivo é complementar a:

- `docs/02-game-panel/README.md`
- `docs/02-game-panel/amp-instance-manager.md`
- `docs/02-game-panel/instance-mixhaxixe01.md`
- `docs/02-game-panel/cs2-server-configuration.md`
- `docs/02-game-panel/matchzy.md`
- `docs/02-game-panel/plugins-installed.md`
- `docs/02-game-panel/mariadb-runtime.md`
- `docs/02-game-panel/game-panel-operational-runbooks.md`
- `docs/02-game-panel/game-panel-observability-troubleshooting.md`
- `docs/03-portal-estatico/data-sources-matchzy-sqlite.md`

Este documento descreve a topologia do runtime do lado jogo.  
Ele não substitui os documentos especializados de configuração, MatchZy, plugins ou runbooks.

---

## Visão geral do runtime

A topologia lógica do lado jogo é:

- Infra Hostinger
- Docker host
- AMP Instance Manager
- instância `MixHAXIXE01`
- servidor Counter-Strike 2
- CounterStrikeSharp
- MatchZy
- plugins auxiliares
- persistência operacional local
- geração indireta da base estatística usada pelo Portal Estático

Representação resumida:

- o host sustenta o runtime
- o AMP gerencia a instância
- a instância materializa o servidor CS2
- o servidor roda com CounterStrikeSharp e plugins
- o MatchZy controla parte central da lógica competitiva
- o runtime produz dados persistidos localmente
- esses dados alimentam a cadeia pública do portal

Esse desenho mostra que o Game Panel não é apenas “o servidor”, mas a malha operacional inteira do lado jogo.

---

## Componentes do contexto

### AMP Instance Manager

O AMP é a camada de gerenciamento da instância do servidor.

Papel no runtime:

- administrar a instância do lado game
- fornecer camada operacional acima do substrate
- manter a existência operacional da instância oficial
- conectar administração prática e runtime do jogo

No desenho atual, o AMP é o principal orquestrador do workload CS2.

---

### Instância `MixHAXIXE01`

A instância oficial do ecossistema é:

- `MixHAXIXE01`

Papel no runtime:

- representar a unidade operacional principal do lado jogo
- concentrar a árvore de dados, configs e artefatos ligados ao servidor
- sustentar o `matchzy.db` e demais elementos estruturais do lado game
- funcionar como dependência escondida de outras camadas do ecossistema

Regra arquitetural:

- mudança de nome ou estrutura dessa instância pode quebrar não apenas o jogo, mas também integrações com a camada pública de dados

---

### Counter-Strike 2

O Counter-Strike 2 é o workload principal da camada.

Papel no runtime:

- operar a experiência real do servidor
- receber players, admins e coaches
- executar regras de partida, warmup e match flow
- sustentar a experiência competitiva do HSC

O CS2, porém, não opera sozinho; ele depende da camada de plugins e da administração da instância.

---

### CounterStrikeSharp

O CounterStrikeSharp é parte estrutural do runtime do servidor.

Papel no runtime:

- fornecer a base de plugins do lado CS2
- permitir a execução de MatchZy e demais plugins relevantes
- servir como camada intermediária entre o servidor e as extensões operacionais

No desenho atual, a saúde do runtime de plugins depende dessa camada permanecer íntegra.

---

### MatchZy

O MatchZy é componente central do contexto.

Papel no runtime:

- sustentar lógica competitiva do servidor
- organizar partidas, fluxos de match e comportamento operacional do lado competitivo
- participar da produção de estatísticas persistidas
- atuar como ponte entre operação do jogo e publicação pública de dados

O MatchZy possui dupla natureza:

- componente operacional de partida
- componente estrutural da cadeia de dados públicos

---

### Plugins auxiliares

Os plugins auxiliares complementam a operação do servidor.

Papel no runtime:

- oferecer capacidades administrativas
- expandir a experiência do servidor
- sustentar funções de visual, permissões, persistência ou gameplay complementar
- coexistir com MatchZy dentro da mesma malha operacional

Regra importante:

- plugin carregado em produção é parte da topologia real do contexto
- ele não deve ser tratado como detalhe informal

---

## Fluxos principais

### Fluxo de operação do servidor

1. a infraestrutura Hostinger sustenta o substrate do lado jogo
2. o AMP gerencia a instância `MixHAXIXE01`
3. a instância executa o servidor CS2
4. o CounterStrikeSharp sustenta a camada de plugins
5. MatchZy e plugins operam sobre o runtime
6. admins e players interagem com o servidor em fluxo real de jogo

Esse é o fluxo operacional primário do contexto.

---

### Fluxo de partida competitiva

1. o servidor sobe com a instância e configs vigentes
2. MatchZy e plugins carregam
3. os operadores aplicam o modo de partida adequado
4. o fluxo competitivo ocorre em BO1, BO3, scrim, treino ou modo equivalente
5. o runtime registra eventos e estatísticas pertinentes
6. a persistência local passa a refletir o estado das partidas jogadas

Esse fluxo é o núcleo do valor prático do contexto.

---

### Fluxo de geração de estatísticas

1. o runtime do jogo produz atividade operacional
2. o MatchZy persiste estatísticas localmente
3. o `matchzy.db` passa a refletir o estado acumulado do servidor
4. o Portal Estático lê essa base por ETL em contexto separado
5. a camada pública do ecossistema passa a refletir a operação do lado jogo

Esse fluxo mostra por que o Game Panel e o Portal Estático são separados, mas estruturalmente conectados.

---

## Persistência local

A persistência operacional conhecida do lado jogo inclui, no mínimo:

- base SQLite do MatchZy
- artefatos da árvore da instância AMP
- possíveis persistências auxiliares de plugins
- configurações versionadas e arquivos do runtime

O elemento mais importante, do ponto de vista do ecossistema mais amplo, é:

- `matchzy.db`

Esse banco é parte do runtime do jogo, mas tem impacto direto fora dele por alimentar a v2 do portal.

---

## Relação com a Infra Hostinger

O Game Panel depende da Infra Hostinger para:

- host Debian
- Docker host
- árvore estrutural sob `amp`
- disponibilidade de recursos da VPS
- integridade do filesystem base
- substrate de runtime do lado game

Regra importante:

- se a Infra Hostinger falhar, o Game Panel pode deixar de existir operacionalmente
- mas o inverso não é verdadeiro em todos os casos: a base pode estar saudável e o runtime do jogo ainda assim estar degradado

---

## Relação com o Portal Estático

O Portal Estático depende do Game Panel como origem indireta dos dados estatísticos públicos.

Essa relação inclui:

- existência do `matchzy.db`
- estabilidade da árvore da instância
- compatibilidade do schema
- continuidade da produção de estatísticas
- saúde do MatchZy e do runtime do jogo

Regra arquitetural:

- o Game Panel produz a fonte
- o Portal Estático transforma e publica a fonte

---

## Dependências operacionais

O runtime do Game Panel depende, no mínimo, de:

- substrate da Infra Hostinger saudável
- Docker host funcional
- AMP funcional
- instância `MixHAXIXE01` íntegra
- servidor CS2 operacional
- CounterStrikeSharp funcional
- MatchZy carregando corretamente
- plugins carregando corretamente
- configs coerentes com o modo de operação esperado
- persistência local íntegra
- disciplina operacional de admins e runbooks claros

Essas dependências explicam por que o contexto precisa de documentação própria, separada da infraestrutura base.

---

## Invariantes operacionais

Os invariantes conhecidos do runtime incluem:

- o lado jogo depende da instância oficial `MixHAXIXE01`
- o MatchZy é componente central do runtime
- plugins em produção fazem parte da verdade do contexto
- a persistência local do MatchZy é estrutural para o ecossistema
- o Game Panel não deve ser confundido com o substrate Docker
- o runtime do jogo precisa ser analisado como sistema operacional completo, e não apenas como container ou apenas como plugin

Esses invariantes ajudam a manter a identidade técnica do contexto.

---

## Failure domains

Os principais failure domains desta camada incluem:

### 1. AMP

Se o AMP falhar:

- a administração da instância fica comprometida
- o workload do jogo pode deixar de ser operável corretamente
- o troubleshooting do lado game se torna mais difícil

---

### 2. Instância `MixHAXIXE01`

Se a instância falhar, for renomeada ou ficar incoerente:

- o servidor pode deixar de operar corretamente
- a árvore de dados do lado game pode se tornar inacessível
- o Portal Estático pode perder a referência da sua fonte de dados

---

### 3. Runtime CS2

Se o servidor CS2 falhar:

- a experiência do jogo quebra
- não há operação competitiva real
- a produção de dados futuros é interrompida

---

### 4. MatchZy

Se o MatchZy falhar:

- fluxos competitivos ficam degradados
- comportamentos centrais da partida deixam de operar como esperado
- a base estatística local pode parar de ser produzida corretamente

---

### 5. Plugins auxiliares

Se um plugin falhar:

- a administração do servidor pode degradar
- recursos auxiliares podem desaparecer
- parte da experiência do runtime pode ficar inconsistente

---

### 6. Persistência local

Se a persistência local falhar:

- estatísticas podem deixar de evoluir
- a camada pública do portal fica stale ou inconsistente
- o troubleshooting se espalha para fora do Game Panel

---

### 7. Drift entre runtime e runbook

Se a configuração real divergir do que os operadores acreditam estar rodando:

- scrim, BO3, veto, coach e fallback admin podem se comportar de forma inesperada
- troubleshooting fica muito mais caro
- a operação volta a depender de memória informal

---

## Sinais de saúde do contexto

Os sinais de saúde esperados incluem:

- instância oficial reconhecível e íntegra
- runtime do servidor CS2 funcional
- MatchZy carregado e operando
- plugins principais carregados
- configurações coerentes com o modo de operação esperado
- persistência local atualizando
- operação prática de scrim/partida executável
- ausência de erro estrutural recorrente do lado game

A saúde do contexto deve ser avaliada tanto por componentes técnicos quanto por comportamento operacional real.

---

## Limites deste documento

Este documento não detalha:

- lista exata de plugins e suas versões
- configurações linha a linha do servidor
- fluxos operacionais passo a passo de BO1, BO3, veto ou coach
- detalhes do banco auxiliar MariaDB do lado game, quando existir
- troubleshooting aprofundado de cada sintoma do servidor
- detalhes da Infra Hostinger ou do Portal Estático

Esses tópicos pertencem a documentos específicos do contexto.

---

## Critério de pronto deste documento

Este documento pode ser considerado maduro quando:

- a topologia AMP → instância → CS2 → MatchZy → persistência estiver reconciliada sem ambiguidade
- a centralidade da instância `MixHAXIXE01` estiver clara
- a relação entre runtime do jogo e produção de dados públicos estiver formalizada
- os failure domains estiverem corretamente representados
- ele puder ser lido como visão topológica do Game Panel sem depender do master legado

---

## Última revisão

- Status: ativo
- Classificação: canônico
- Contexto: game panel / arquitetura de runtime
- Última revisão: 2026-03-18