# AMP Instance Manager

## Objetivo

Documentar o papel do AMP Instance Manager no contexto Game Panel do ecossistema HSC, registrando sua função operacional, sua relação com a instância oficial `MixHAXIXE01` e seus limites dentro da arquitetura do lado jogo.

Este documento existe para registrar, de forma estável e auditável:

- o que o AMP faz no ecossistema HSC
- como ele se posiciona entre a Infra Hostinger e o runtime do servidor CS2
- quais responsabilidades operacionais pertencem ao AMP
- quais responsabilidades não pertencem ao AMP
- como o AMP se relaciona com Docker, filesystem, instância e MatchZy
- quais sintomas e falhas comuns surgem quando o AMP sai do estado esperado

---

## Escopo

Este documento cobre:

- o papel do AMP Instance Manager no lado jogo
- a relação entre AMP e a instância `MixHAXIXE01`
- a relação entre AMP e o runtime do servidor CS2
- a relação entre AMP e a árvore operacional do host
- a relação entre AMP e Docker em nível estrutural
- operações recorrentes de administração da instância em alto nível
- limites e problemas comuns dessa camada

Este documento não cobre em profundidade:

- Docker host como substrate da Infra Hostinger
- configuração detalhada do servidor CS2
- MatchZy em profundidade
- inventário detalhado de plugins
- runbooks detalhados de scrim, BO1, BO3, veto, coach e warmup
- pipeline ETL do Portal Estático
- Auth API no AWS Lightsail

Esses tópicos vivem em documentos próprios dos contextos `01-infra-hostinger`, `02-game-panel` e `03-portal-estatico`.

---

## Estado atual

O estado operacional conhecido do AMP no ecossistema HSC é:

- o AMP Instance Manager é a camada de gestão da instância do lado jogo
- a instância oficial conhecida do ecossistema é `MixHAXIXE01`
- o AMP existe sobre a Infra Hostinger
- o AMP depende do substrate do host e da árvore operacional sob `amp`
- o runtime do servidor CS2 é administrado por meio dessa camada
- o `matchzy.db` e outros artefatos do lado jogo dependem estruturalmente da instância gerida pelo AMP
- o ecossistema já possui dependências ocultas em paths que passam pela árvore da instância

Isso significa que o AMP não é apenas “o painel”; ele é uma peça estrutural da topologia operacional do HSC.

---

## Source of truth / evidências

As principais evidências deste documento, nesta fase de migração documental, são:

- documentação consolidada atual do ecossistema HSC
- blueprint técnico consolidado do HSC
- documentação específica da camada AMP/CS2
- reconciliação da instância oficial `MixHAXIXE01`
- reconciliação dos paths do lado `amp`
- reconciliação da dependência estrutural do `matchzy.db` em relação à instância

Enquanto a migração canônica do contexto não estiver concluída, essas fontes seguem sendo usadas como base de reconciliação do estado real.

---

## Relações com outros documentos

Este arquivo é complementar a:

- `docs/02-game-panel/README.md`
- `docs/02-game-panel/game-panel-architecture-runtime.md`
- `docs/02-game-panel/instance-mixhaxixe01.md`
- `docs/02-game-panel/cs2-server-configuration.md`
- `docs/02-game-panel/matchzy.md`
- `docs/02-game-panel/operational-runbooks.md`
- `docs/02-game-panel/observability-troubleshooting.md`
- `docs/01-infra-hostinger/docker-host.md`
- `docs/01-infra-hostinger/filesystem-paths-permissions.md`

Este documento descreve o papel do AMP como camada de gestão da instância.  
Ele não substitui os documentos de configuração do servidor, MatchZy ou runbooks de operação.

---

## Papel do AMP no HSC

No ecossistema HSC, o AMP é a camada de administração operacional do servidor de jogo.

Seu papel inclui:

- manter a existência operacional da instância do lado game
- organizar a administração da instância oficial
- servir como superfície prática de gestão do servidor CS2
- conectar o substrate da Infra Hostinger ao runtime do jogo
- concentrar a identidade operacional da instância dentro da árvore `amp`

Em termos arquiteturais, o AMP ocupa a posição intermediária entre:

- Infra Hostinger
- instância `MixHAXIXE01`
- runtime do servidor CS2

---

## Responsabilidades operacionais do AMP

As responsabilidades principais do AMP neste contexto incluem:

- gerenciar a instância do servidor
- preservar a identidade operacional da instância oficial
- sustentar a administração prática do lado jogo
- organizar a estrutura operacional ligada ao runtime do servidor
- coexistir com a árvore de arquivos que sustenta configs, persistência e dados do lado game
- oferecer uma camada previsível de gestão sobre o workload do CS2

Regra importante:

- o AMP administra o runtime
- ele não substitui o substrate da Infra Hostinger
- ele não substitui MatchZy
- ele não substitui os runbooks humanos de operação

---

## O que o AMP não é

O AMP não deve ser confundido com:

### Infraestrutura base

O AMP não é:

- Debian
- Nginx
- Certbot
- Docker host como substrate
- filesystem público do portal

Esses componentes pertencem à Infra Hostinger.

### Runtime competitivo em si

O AMP também não é, por si só:

- MatchZy
- plugin administrativo
- config específica de BO3
- contrato estatístico do portal
- pipeline ETL

Esses componentes existem sobre ou ao redor da instância gerida pelo AMP, mas têm documentação própria.

---

## Relação com Docker

A relação estrutural entre AMP e Docker é:

- Docker pertence à camada-base do host
- o AMP se apoia nesse substrate para materializar a instância
- o runtime real do servidor passa pela combinação:
  - Infra Hostinger
  - Docker host
  - AMP
  - instância
  - CS2

Regra arquitetural:

- quando o daemon Docker falha, o AMP pode perder sua base de operação
- quando o Docker está saudável, isso ainda não prova que o AMP ou a instância estão saudáveis

Essa separação é importante para troubleshooting correto.

---

## Relação com a instância `MixHAXIXE01`

A relação entre AMP e a instância oficial é central no ecossistema.

A instância oficial conhecida é:

- `MixHAXIXE01`

O AMP existe, na prática, para gerir essa instância e seu runtime associado.

Implicações operacionais:

- a instância é a unidade concreta do lado jogo
- o nome da instância aparece em paths críticos
- renomear a instância ou alterar sua árvore pode quebrar dependências fora do próprio lado game
- o Portal Estático depende indiretamente da estabilidade dessa identidade por causa do path do `matchzy.db`

Regra canônica:

- `MixHAXIXE01` não é apenas um nome visual do painel
- ela é uma dependência estrutural do ecossistema

---

## Relação com filesystem do host

O AMP se apoia em uma árvore estrutural do host.

Base reconciliada em alto nível:

- `/home/amp/.ampdata/instances/`

Essa árvore é importante porque:

- concentra as instâncias
- sustenta a organização operacional do lado game
- abriga artefatos críticos do runtime do servidor
- serve de caminho para a persistência que depois alimenta o portal

Regra importante:

- a árvore `~amp/.ampdata` é operacional, não pública
- ajustes de permissões ou paths nessa árvore precisam respeitar o contexto do AMP
- mudanças mal coordenadas nessa árvore podem quebrar tanto o runtime do jogo quanto a cadeia de dados pública

---

## Relação com o servidor CS2

O AMP administra o contexto no qual o servidor CS2 existe operacionalmente.

Essa relação inclui:

- a instância do servidor
- o runtime carregado para o jogo
- a interação com configs do servidor
- a manutenção da operação prática do ambiente

Em termos práticos:

- se o AMP falha, a operação do servidor pode se tornar indisponível ou impraticável
- se o servidor falha, pode ser necessário investigar se a causa nasce no runtime do jogo ou na camada de gestão da instância

---

## Relação com MatchZy e plugins

O AMP não substitui MatchZy nem plugins, mas organiza o contexto no qual eles existem.

Essa relação inclui:

- instância íntegra para que o runtime carregue
- estrutura de arquivos e paths coerentes
- base operacional para que plugins funcionem
- sustentação indireta da persistência local do MatchZy

Regra operacional:

- MatchZy e plugins são componentes do runtime do servidor
- AMP é a camada que mantém o contexto administrável em que esses componentes operam

---

## Operações recorrentes em alto nível

As operações recorrentes ligadas ao AMP, em alto nível, incluem:

- verificar se a instância oficial existe e está íntegra
- validar o estado operacional do servidor associado
- validar a relação entre árvore da instância e runtime real
- identificar rapidamente se a falha está:
  - no substrate
  - no AMP
  - na instância
  - no runtime do jogo

Regra importante:

- este documento não congela workflows de clique ou UI do painel
- ele formaliza o papel operacional da camada AMP dentro do ecossistema

---

## Dependências operacionais

O AMP depende, no mínimo, de:

- Infra Hostinger saudável
- Docker host funcional
- árvore `amp` íntegra
- instância oficial íntegra
- paths consistentes sob `.ampdata`
- runtime do servidor coerente com a estrutura da instância
- ausência de drift destrutivo entre painel, filesystem e workload real

Dependências cruzadas importantes:

- o lado jogo depende do AMP para gestão prática da instância
- o Portal Estático depende indiretamente da estabilidade dessa camada porque a fonte estatística nasce sob a árvore da instância
- mudanças no AMP precisam ser tratadas com cuidado porque podem afetar o ecossistema além do próprio jogo

---

## Invariantes operacionais

Os invariantes conhecidos desta camada incluem:

- o AMP é a camada de gestão da instância do lado game
- a instância oficial é `MixHAXIXE01`
- a árvore operacional sob `amp` é parte estrutural do contexto
- o AMP depende do substrate da Infra Hostinger
- o AMP não substitui MatchZy, plugins ou runbooks humanos
- a saúde do AMP precisa ser distinguida da saúde do Docker host e da saúde do servidor CS2

Esses invariantes ajudam a manter a separação correta entre host, gestão da instância e runtime do jogo.

---

## Sinais de saúde do AMP

Os sinais de saúde esperados desta camada incluem:

- existência íntegra da instância oficial
- coerência entre o que o AMP representa e o que existe no filesystem
- continuidade operacional do runtime do servidor gerido pela instância
- ausência de drift evidente entre árvore da instância e workload real
- possibilidade prática de operar o servidor por meio da camada de gestão da instância

A saúde do AMP não deve ser inferida apenas pelo fato de o host estar de pé.  
Ela precisa ser lida em conjunto com a instância e com o runtime real do jogo.

---

## Problemas comuns

### 1. AMP saudável, mas o jogo quebrado

Causas comuns:

- problema no runtime do CS2
- problema em MatchZy
- problema em plugin
- problema em config do servidor

Impacto:
- o substrate de gestão está presente
- mas o workload útil não está saudável

Esse caso deve escalar para documentos do próprio Game Panel.

---

### 2. Docker saudável, mas a instância não opera como esperado

Causas comuns:

- problema na camada AMP
- drift entre instância e runtime
- problema na árvore da instância
- incoerência do workload associado

Impacto:
- troubleshooting não deve parar no Docker host
- a causa provável está acima do substrate

---

### 3. Instância renomeada ou path alterado

Causas comuns:

- reorganização manual não reconciliada
- mudança estrutural no ambiente
- intervenção operacional sem atualização documental

Impacto:
- quebra do runtime do lado game
- quebra indireta da referência do `matchzy.db`
- impacto potencial no Portal Estático

---

### 4. Árvore da instância íntegra, mas persistência local inconsistente

Causas comuns:

- problema do MatchZy
- problema de plugin
- problema de gravação no runtime do jogo

Impacto:
- o AMP parece saudável
- mas a produção de dados do ecossistema degrada

---

### 5. Troubleshooting misturando AMP com Infra Hostinger

Causas comuns:

- confusão entre substrate e gestão da instância
- falta de separação documental
- leitura imprecisa da topologia

Impacto:
- diagnóstico lento
- correções no lugar errado
- aumento de risco operacional

---

## Sequência básica de diagnóstico

Quando houver suspeita de falha na camada AMP, a sequência recomendada é:

1. validar a saúde da Infra Hostinger
2. validar a saúde do Docker host
3. validar a existência e coerência da instância oficial
4. validar a árvore da instância no filesystem
5. verificar se o runtime do servidor está de fato refletindo essa instância
6. só depois aprofundar em MatchZy, plugins ou configuração do jogo

Essa sequência ajuda a responder rapidamente:

- o problema está na base?
- está na camada AMP?
- está na instância?
- ou está no runtime do jogo?

---

## Limites deste documento

Este documento não detalha:

- fluxos específicos da interface do AMP
- comandos específicos de operação do painel
- configuração linha a linha do servidor
- MatchZy em detalhe
- plugins em detalhe
- runbooks detalhados de partida
- troubleshooting aprofundado do CS2

Esses tópicos pertencem a documentos específicos do contexto `02-game-panel`.

---

## Critério de pronto deste documento

Este documento pode ser considerado maduro quando:

- o papel do AMP no ecossistema estiver reconciliado sem ambiguidade
- a relação entre AMP, instância e árvore `amp` estiver clara
- a dependência estrutural de `MixHAXIXE01` estiver formalizada
- a distinção entre problema de AMP, problema de substrate e problema de runtime do jogo estiver clara
- ele puder ser usado como referência da camada de gestão da instância sem depender do master legado

---

## Última revisão

- Status: ativo
- Classificação: canônico
- Contexto: game panel / AMP Instance Manager
- Última revisão: 2026-03-18