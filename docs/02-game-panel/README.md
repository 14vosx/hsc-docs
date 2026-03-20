# Game Panel — README

## Objetivo

Documentar o contexto canônico do Game Panel do ecossistema HSC, incluindo a camada operacional do servidor CS2, o AMP Instance Manager, a instância oficial `MixHAXIXE01`, o MatchZy, os plugins instalados e os runbooks de operação do lado jogo.

Este contexto existe para registrar, de forma estável e auditável:

- o papel do Game Panel dentro do ecossistema HSC
- a relação entre AMP, instância, servidor CS2 e plugins
- os limites entre infraestrutura base, operação do jogo e publicação pública de dados
- as dependências operacionais que precisam permanecer estáveis para o funcionamento do servidor
- os procedimentos recorrentes de operação e troubleshooting do lado game

---

## Navegação rápida

### Entrada
- [Home da documentação](../README.md)
- [Governance](../00-governance/README.md)
- [Master Index](../00-governance/99-master-index.md)

### Documentos deste contexto
- [AMP Instance Manager](./amp-instance-manager.md)
- [Architecture Runtime](./game-panel-architecture-runtime.md)
- [CS2 Server Configuration](./cs2-server-configuration.md)
- [Instance MixHAXIXE01](./instance-mixhaxixe01.md)
- [MatchZy](./matchzy.md)
- [Plugins Installed](./plugins-installed.md)
- [Operational Runbooks](./operational-runbooks.md)
- [Observability and Troubleshooting](./observability-troubleshooting.md)
- [References and Inventory](./game-panel-references-inventory.md)

### Relações com outros contextos
- [Infra Hostinger](../01-infra-hostinger/README.md)
- [Portal Estático](../03-portal-estatico/README.md)
- [Infra AWS Lightsail](../04-infra-aws-lightsail/README.md)

### Trilhas operacionais relevantes
- [Data Sources — MatchZy SQLite](../03-portal-estatico/data-sources-matchzy-sqlite.md)
- [SQL Queries and Views](../03-portal-estatico/sql-queries-and-views.md)
- [ETL Bash Pipeline](../03-portal-estatico/etl-bash-pipeline.md)
- [Static API v2](../03-portal-estatico/static-api-v2.md)
- [Docker Host](../01-infra-hostinger/docker-host.md)

---

## Escopo

Este contexto cobre exclusivamente a camada operacional do servidor CS2 sobre a base Hostinger.

Estão dentro do escopo deste contexto:

- AMP Instance Manager
- instância oficial `MixHAXIXE01`
- runtime do servidor Counter-Strike 2
- MatchZy
- plugins instalados
- configurações operacionais do servidor
- runbooks de scrim, BO1, BO3, veto, coach, warmup e fallback admin
- troubleshooting funcional do lado jogo
- MariaDB do lado game, se e quando fizer parte do runtime real

Este contexto não cobre, em profundidade:

- Debian, Nginx, Docker host, Certbot e systemd como camada-base
- pipeline ETL Bash da Static API v2
- contratos JSON do portal
- frontend público do portal
- Auth API no AWS Lightsail
- credenciais e arquivos sensíveis

---

## Estado atual

O estado operacional conhecido deste contexto é:

- o lado jogo do HSC opera sobre a infraestrutura Hostinger
- o AMP Instance Manager gerencia a instância oficial do ecossistema
- a instância oficial conhecida é `MixHAXIXE01`
- o servidor CS2 roda como workload operacional dessa instância
- o MatchZy é peça central do runtime do jogo
- há plugins instalados e relevantes para a operação do servidor
- o lado jogo produz a base de dados estatística que depois alimenta o Portal Estático
- a operação do Game Panel exige runbooks claros para evitar dependência de memória informal

O Game Panel é a camada operacional viva do lado CS2 do HSC.

---

## O que é o Game Panel no HSC

O Game Panel é a camada responsável por operar o servidor de jogo do ecossistema.

Seu papel é:

- sustentar a operação do CS2 em runtime
- permitir gestão prática da instância via AMP
- manter MatchZy e plugins funcionando
- sustentar fluxos de scrim e partida organizada
- produzir a base estatística operacional que alimenta a cadeia pública de dados do portal

Em termos arquiteturais, o Game Panel é o contexto que transforma a Infra Hostinger em um servidor de jogo efetivamente utilizável.

---

## O que roda aqui

Este contexto sustenta, em nível operacional:

- AMP Instance Manager
- instância `MixHAXIXE01`
- servidor Counter-Strike 2
- MatchZy
- plugins auxiliares
- arquivos de configuração do runtime do jogo
- banco(s) ou artefatos operacionais ligados ao lado game
- rotinas práticas de administração da instância

Do ponto de vista arquitetural, esta camada não é a borda pública do portal e não é o backend dinâmico da Auth API; ela é o runtime de jogo do ecossistema.

---

## O que não roda aqui

Este contexto não deve ser confundido com:

### Infra Hostinger

Este contexto não documenta, em profundidade:

- Debian
- Nginx edge
- Certbot
- Docker host como substrate
- systemd e timers do host
- filesystem base e permissões do host

Esses itens pertencem a `docs/01-infra-hostinger/`.

### Portal Estático

Este contexto não documenta, em profundidade:

- Static API v2
- pipeline ETL Bash
- contratos JSON
- frontend público
- publishing via Nginx

Esses itens pertencem a `docs/03-portal-estatico/`.

### Auth API

Este contexto não documenta:

- runtime do Lightsail
- Nginx reverse proxy da Auth API
- Node.js via systemd da camada dinâmica
- MariaDB local do backend dinâmico
- deploy/release/rollback da Auth API

Esses itens pertencem a `docs/04-infra-aws-lightsail/`.

---

## Papel do Game Panel no ecossistema HSC

O Game Panel é a camada que opera o jogo e produz o runtime competitivo do HSC.

Seu papel no ecossistema é:

- manter a instância do servidor CS2 utilizável
- sustentar a experiência de scrim e partidas organizadas
- centralizar MatchZy e plugins do lado game
- preservar a estrutura da instância oficial
- gerar a base operacional de stats que alimenta o Portal Estático
- separar a lógica do jogo da infraestrutura base e da camada pública de publicação

Em termos arquiteturais:

- Infra Hostinger = substrate
- Game Panel = operação do jogo
- Portal Estático = publicação pública dos dados
- AWS Lightsail = backend dinâmico da Auth API

Essa separação reduz confusão entre “host”, “jogo”, “portal” e “backend dinâmico”.

---

## Componentes centrais do contexto

Os componentes centrais já reconciliados deste contexto incluem:

### AMP Instance Manager

Papel:
- gerenciar a instância do lado game
- oferecer camada de administração da operação do servidor

### Instância `MixHAXIXE01`

Papel:
- representar a instância oficial do ecossistema
- sustentar a árvore operacional do runtime do jogo

### Counter-Strike 2

Papel:
- workload principal do contexto
- servidor efetivamente utilizado pelos players e admins

### MatchZy

Papel:
- administrar lógica competitiva e estatística do runtime de partida
- atuar como peça central da operação do servidor e da geração de dados

### Plugins

Papel:
- complementar a operação do servidor
- oferecer capacidades administrativas, visuais ou funcionais adicionais

---

## Relação com a Infra Hostinger

Este contexto depende diretamente da Infra Hostinger para:

- host Debian
- Docker host
- árvore operacional da instância
- filesystem do lado `amp`
- disponibilidade do substrate que sustenta o runtime do jogo

Regra arquitetural:

- o Game Panel vive sobre a Hostinger
- mas sua semântica operacional precisa ser documentada separadamente para evitar mistura entre infraestrutura e workload

---

## Relação com o Portal Estático

Este contexto se relaciona diretamente com o Portal Estático porque o lado jogo produz a origem de dados que depois será transformada na Static API v2.

Essa relação inclui:

- MatchZy produzindo estatísticas
- persistência operacional em SQLite
- dependência do portal em relação ao `matchzy.db`
- dependência do ETL em relação ao schema e ao path da instância

Regra importante:

- o Game Panel produz a fonte
- o Portal Estático consome a fonte e publica os artefatos derivados

---

## Relação com o MatchZy

O MatchZy é peça central deste contexto por duas razões:

- ele participa diretamente da operação das partidas
- ele participa da produção da base estatística do ecossistema

Isso significa que o MatchZy tem dupla natureza:

- componente operacional de jogo
- componente estrutural da cadeia de dados públicos

Por isso, ele precisa de documentação própria dentro deste contexto.

---

## Relação com plugins

Os plugins instalados fazem parte do runtime real do servidor.

Eles podem influenciar:

- administração
- permissões
- experiência dos players
- persistência de dados auxiliares
- operação de MatchZy e rotinas da partida

Regra importante:

- plugin instalado em produção é parte da verdade do contexto
- ele não deve ficar apenas em memória informal ou em prints de console

---

## Documentos canônicos deste contexto

Os documentos canônicos previstos para este contexto são:

- `docs/02-game-panel/README.md`
- `docs/02-game-panel/game-panel-architecture-runtime.md`
- `docs/02-game-panel/amp-instance-manager.md`
- `docs/02-game-panel/instance-mixhaxixe01.md`
- `docs/02-game-panel/cs2-server-configuration.md`
- `docs/02-game-panel/matchzy.md`
- `docs/02-game-panel/plugins-installed.md`
- `docs/02-game-panel/mariadb-runtime.md`
- `docs/02-game-panel/operational-runbooks.md`
- `docs/02-game-panel/observability-troubleshooting.md`
- `docs/02-game-panel/game-panel-references-inventory.md`

Este `README.md` funciona como porta de entrada e índice local do contexto.

---

## Dependências externas

Este contexto depende de componentes e condições externas relevantes, incluindo:

- substrate íntegro da Infra Hostinger
- Docker host funcional
- árvore operacional da instância AMP íntegra
- MatchZy funcional
- plugins carregando corretamente
- configurações do servidor coerentes
- permissões e admins operando como esperado
- integridade da produção local de dados estatísticos

Também há dependência de disciplina operacional para evitar:

- drift de configuração
- perda de permissões/admins
- inconsistência entre modo de jogo e runbook
- regressão após update de plugin ou ajuste manual no runtime

---

## Source of truth / evidências

As principais evidências que sustentam este contexto, nesta fase de migração documental, são:

- documentação consolidada atual do ecossistema HSC
- blueprint técnico consolidado do HSC
- documentação específica da camada AMP/CS2
- documentação reconciliada de MatchZy, plugins e runbooks do servidor
- reconciliação do papel da instância `MixHAXIXE01` no ecossistema

Enquanto a migração do contexto não estiver concluída, os documentos antigos seguem como fonte de extração e checagem cruzada, mas não como canônico final.

---

## Relação com outros contextos

Este contexto se relaciona diretamente com:

### `docs/01-infra-hostinger/`

Relação estrutural forte.  
A Infra Hostinger sustenta o substrate técnico sobre o qual o Game Panel existe.

### `docs/03-portal-estatico/`

Relação estrutural forte.  
O Portal Estático depende dos dados produzidos a partir do runtime do jogo operado neste contexto.

### `docs/04-infra-aws-lightsail/`

Relação indireta de arquitetura global.  
A camada dinâmica da Auth API vive fora do lado game, em Lightsail.

### `docs/90-adr/`

Deve registrar decisões arquiteturais relevantes sobre a operação do lado jogo, adoção de MatchZy, estrutura de instância ou regras estruturais do runtime.

### `docs/95-impl-log/`

Deve registrar mudanças incrementais relevantes deste contexto, como updates de plugin, ajustes de configuração, mudanças de fluxo operacional ou endurecimentos administrativos.

### `docs/97-audit/`

Pode conter análises, gaps, inconsistências ou verificações ligadas ao Game Panel.

### `docs/98-legacy/`

Preserva documentação histórica usada como base de migração desta camada.

---

## Relações com outros documentos

Este arquivo é complementar a:

- `docs/99-master-index.md`
- `docs/00-governance/README.md`
- `docs/00-governance/documentation-system.md`

Dentro deste próprio contexto, ele é a porta de entrada para todos os documentos canônicos específicos do Game Panel.

---

## Critério de pronto deste contexto

Este contexto poderá ser considerado formalmente migrado quando:

- a camada operacional do servidor CS2 estiver documentada sem depender do master antigo
- o papel do AMP estiver claro
- a instância `MixHAXIXE01` estiver formalizada como dependência estrutural do ecossistema
- o MatchZy estiver documentado como componente operacional e produtor de dados
- os plugins instalados estiverem inventariados
- os runbooks operacionais estiverem centralizados
- troubleshooting e observabilidade do lado jogo estiverem centralizados
- os documentos antigos passarem a servir apenas como referência cruzada

---

## Última revisão

- Status: ativo
- Classificação: canônico
- Contexto: game panel / porta de entrada do contexto
- Última revisão: 2026-03-18