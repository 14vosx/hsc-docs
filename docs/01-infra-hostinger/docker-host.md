# Docker Host

## Objetivo

Documentar o papel do Docker como componente da camada-base da Infra Hostinger no ecossistema HSC.

Este documento existe para registrar, de forma estável e auditável:

- como o Docker participa da topologia do lado Hostinger
- qual é o papel do Docker como substrate do host
- como o runtime do jogo depende dessa camada
- quais limites existem entre Docker host e operação do Game Panel
- quais validações mínimas ajudam a manter essa camada íntegra
- quais falhas comuns podem surgir quando o Docker host sai do estado esperado

---

## Escopo

Este documento cobre:

- o papel do Docker na Infra Hostinger
- a relação entre Docker e workloads do lado game
- a relação entre Docker e o host Debian
- a relação entre Docker e AMP em nível estrutural
- comandos básicos de validação do runtime de containers
- sintomas e problemas comuns dessa camada

Este documento não cobre em profundidade:

- operação detalhada do AMP Instance Manager
- operação detalhada da instância `MixHAXIXE01`
- configuração do servidor CS2
- plugins, MatchZy, warmup, veto ou coach
- pipeline ETL da Static API v2
- Auth API no AWS Lightsail

Esses tópicos vivem em documentos próprios dos contextos `02-game-panel`, `03-portal-estatico` e `04-infra-aws-lightsail`.

---

## Estado atual

O estado operacional conhecido da camada Docker do lado Hostinger é:

- o host Debian utiliza Docker como substrate de containers
- o lado game do ecossistema depende dessa camada
- há evidência reconciliada de container associado à instância oficial:
  - `AMP_MixHAXIXE01`
- o lado portal e publishing não depende de Docker para seu serving público direto, mas convive sobre o mesmo host
- o Docker host é uma peça estrutural do lado game, e indiretamente influencia a saúde da cadeia de dados públicos

Isso significa que:

- Docker pertence à Infra Hostinger
- mas a semântica operacional do que roda dentro do container pertence ao contexto `02-game-panel`

---

## Source of truth / evidências

As principais evidências deste documento, nesta fase de migração documental, são:

- documentação consolidada atual do ecossistema HSC
- blueprint técnico consolidado do HSC
- documentação reconciliada da Infra Hostinger
- documentação reconciliada da camada AMP/CS2
- inventário operacional já observado do lado Hostinger com container ligado à instância oficial

Enquanto a migração canônica do contexto não estiver concluída, essas fontes seguem sendo usadas como base de reconciliação do estado real.

---

## Relações com outros documentos

Este arquivo é complementar a:

- `docs/01-infra-hostinger/README.md`
- `docs/01-infra-hostinger/infra-hostinger-architecture-runtime.md`
- `docs/01-infra-hostinger/filesystem-paths-permissions.md`
- `docs/01-infra-hostinger/infra-hostinger-observability-troubleshooting.md`
- `docs/02-game-panel/game-panel-architecture-runtime.md`
- `docs/02-game-panel/amp-instance-manager.md`
- `docs/02-game-panel/instance-mixhaxixe01.md`

Este documento descreve o Docker como camada-base do host.  
Ele não substitui os documentos de operação do Game Panel ou do servidor CS2.

---

## Papel do Docker no host

No lado Hostinger, o Docker é o substrate de containers do host.

Seu papel inclui:

- sustentar workloads containerizados do lado game
- fornecer isolamento operacional para a instância de jogo
- coexistir com Nginx, ETL, filesystem e automações do host
- integrar-se à topologia geral sustentada pelo Debian

Em termos arquiteturais:

- Docker é parte da infraestrutura base
- o container não é o contexto final
- o contexto final do jogo existe sobre essa base, via AMP/Game Panel

---

## Relação entre Debian e Docker

O Docker depende diretamente da saúde do host Debian.

Essa relação inclui:

- disponibilidade do daemon Docker
- integridade dos binários e serviços do host
- suporte de rede do host
- suporte de filesystem do host
- disponibilidade de recursos como CPU, RAM e disco
- convivência com outras camadas do host, como Nginx, ETL e systemd

Regra importante:

- problemas no host Debian podem se refletir como problemas nos containers
- nem todo problema “do jogo” nasce no container; a raiz pode estar no substrate

---

## Relação entre Docker e AMP

A relação estrutural conhecida entre Docker e AMP é:

- o Docker oferece o substrate de execução containerizada
- o AMP gerencia a instância que roda sobre essa base
- a instância oficial `MixHAXIXE01` está associada a esse runtime
- o lado Game Panel usa a infraestrutura Docker do host para materializar o workload do servidor CS2

Regra arquitetural:

- Docker host é assunto de Infra Hostinger
- o que o AMP faz sobre ele é assunto de Game Panel

Essa separação evita misturar substrate com operação da aplicação do jogo.

---

## Workloads relevantes

O workload relevante já reconciliado desta camada é o lado game do ecossistema HSC.

Em termos práticos, o Docker host participa da sustentação de:

- ambiente do servidor CS2
- instância oficial `MixHAXIXE01`
- runtime operacional do lado Game Panel

Impactos indiretos dessa camada no restante do ecossistema:

- se o lado game do container falhar, a geração de dados do MatchZy pode parar
- se a geração de dados parar, a Static API v2 tende a ficar stale
- portanto, a saúde do Docker host pode impactar o Portal Estático de forma indireta

---

## Container relevante conhecido

O container operacional já reconciliado neste contexto é:

- `AMP_MixHAXIXE01`

Esse nome deve ser tratado, até revisão explícita, como referência operacional importante da camada Docker do lado Hostinger.

Regra importante:

- o nome do container é uma evidência útil para troubleshooting e validação
- a semântica completa do workload continua pertencendo ao contexto `02-game-panel`

---

## Responsabilidades da camada Docker host

As responsabilidades da camada Docker host incluem:

- manter o daemon Docker funcional
- manter o runtime de containers íntegro
- permitir que o workload do jogo exista de forma previsível
- não interferir negativamente nos paths operacionais do host
- coexistir com Nginx, systemd, ETL e demais serviços da VPS
- preservar a base para o funcionamento do lado game

O Docker host não é responsável por:

- servir o portal público diretamente
- publicar a Static API v2
- gerir CORS, sessão ou conteúdo dinâmico da Auth API
- executar lógica pública do frontend

---

## Limites desta camada

Os limites canônicos desta camada são:

- ela documenta o Docker como substrate
- ela não documenta a lógica interna do workload CS2
- ela não documenta MatchZy em profundidade
- ela não documenta plugins
- ela não documenta runbooks de partida
- ela não documenta a pipeline do portal

Isso significa que:

- troubleshooting do daemon Docker pertence aqui
- troubleshooting do comportamento do servidor CS2 pertence ao contexto `02-game-panel`

---

## Dependências operacionais

A camada Docker host depende, no mínimo, de:

- daemon Docker funcional
- host Debian íntegro
- recursos de CPU e memória adequados
- espaço em disco suficiente
- filesystem do host coerente com a árvore operacional do lado game
- convivência saudável com o restante da infraestrutura do host

Dependências cruzadas importantes:

- a saúde do lado game depende do Docker host
- a saúde do portal pode ser afetada indiretamente se o Docker host comprometer a geração da fonte de dados
- mudanças na camada Docker precisam respeitar dependências do AMP e da instância oficial

---

## Invariantes operacionais

Os invariantes conhecidos desta camada incluem:

- o Docker é parte estrutural da Infra Hostinger
- o lado game depende dele como substrate
- o nome da instância/container oficial é parte relevante da topologia
- o Docker precisa coexistir com Nginx, ETL e automações do host
- o Docker host não deve ser tratado como camada autônoma desligada do restante da VPS
- troubleshooting do lado game precisa sempre considerar a saúde do Docker host

Esses invariantes ajudam a manter a separação correta entre infraestrutura base e workload operacional.

---

## Comandos de validação

Os comandos abaixo representam verificações típicas da camada Docker host.

### Ver status do serviço Docker

```bash
sudo systemctl status docker
```

### Listar containers ativos

```bash
docker ps --format "table {{.Names}}\t{{.Image}}\t{{.Ports}}"
```

### Listar todos os containers

```bash
docker ps -a --format "table {{.Names}}\t{{.Status}}\t{{.Image}}"
```

### Inspecionar container relevante conhecido

```bash
docker inspect AMP_MixHAXIXE01
```

### Ver logs do daemon Docker

```bash
sudo journalctl -u docker -n 100 --no-pager
```

### Ver uso geral do Docker

```bash
docker system df
```

Regra importante:

- os comandos acima ajudam a validar a saúde da camada Docker
- eles não substituem a validação funcional do Game Panel ou do servidor CS2

---

## Sinais de saúde da camada Docker

Os sinais de saúde esperados incluem:

- serviço Docker ativo
- container relevante presente e coerente com o runtime esperado
- ausência de erro crítico recorrente no `journalctl` do Docker
- `docker ps` compatível com o estado esperado do lado game
- ausência de falhas sistêmicas de runtime de container
- ausência de saturação óbvia de storage ou estado quebrado do daemon

A saúde dessa camada deve ser lida como saúde do substrate, não como prova completa de que o servidor CS2 está operacionalmente correto.

---

## Problemas comuns

### 1. Serviço Docker parado

Causas comuns:

- falha do daemon
- problema no host Debian
- reinicialização incompleta
- erro estrutural do runtime de container

Impacto:
- o lado game pode ficar totalmente indisponível
- a cadeia de dados públicos pode parar de evoluir

---

### 2. Container esperado não aparece

Causas comuns:

- instância não está em execução
- container foi recriado com nome divergente
- problema de inicialização do workload
- falha do lado AMP

Impacto:
- o servidor CS2 pode estar indisponível
- a origem do `matchzy.db` pode deixar de evoluir

---

### 3. Container existe, mas está em estado incorreto

Causas comuns:

- crash loop
- problema de imagem
- falha de runtime interno
- dependência quebrada do lado Game Panel

Impacto:
- o substrate está de pé
- mas o workload útil não está saudável

---

### 4. Docker saudável, mas o jogo continua quebrado

Causas comuns:

- o problema está no workload interno
- problema do AMP
- problema de plugin
- problema de configuração do servidor
- problema da árvore da instância

Impacto:
- troubleshooting precisa sair da Infra Hostinger e ir para `02-game-panel`

---

### 5. Saturação de disco ou runtime degradado

Causas comuns:

- acúmulo de artefatos Docker
- imagens antigas
- storage do host degradado
- uso de disco em conflito com publishing e outras camadas do host

Impacto:
- degradação do container
- degradação do ETL
- degradação geral do lado Hostinger

---

## Sequência básica de diagnóstico

Quando houver suspeita de problema na camada Docker host, a sequência recomendada é:

1. validar status do serviço Docker
2. listar containers ativos
3. verificar presença e estado do container relevante
4. consultar logs do daemon Docker
5. validar saúde geral do host (disco, serviços, estabilidade)
6. só depois aprofundar em AMP, MatchZy, plugins ou configuração do servidor

Essa sequência ajuda a responder rapidamente:

- o substrate de container está de pé?
- o workload relevante existe?
- o problema é do Docker host ou do Game Panel?

---

## Riscos e cuidados

Os principais riscos desta camada incluem:

- tratar problema do workload como se fosse problema do Docker host
- ignorar a dependência do lado portal em relação à evolução da fonte de dados do jogo
- mudar o runtime de container sem considerar dependências do AMP
- acumular estado do Docker e degradar o host
- perder rastreabilidade do container relevante da instância oficial

Cuidados permanentes:

- validar periodicamente o estado do daemon Docker
- validar presença do container oficial esperado
- correlacionar falhas do lado game com o substrate antes de concluir causa raiz
- manter este documento focado no host, não na semântica operacional do jogo

---

## Limites deste documento

Este documento não detalha:

- conteúdo interno da imagem/container
- comandos do AMP
- configuração do servidor CS2
- MatchZy
- plugins
- runbooks de operação do jogo
- troubleshooting aprofundado do workload da instância

Esses tópicos pertencem ao contexto `02-game-panel`.

---

## Critério de pronto deste documento

Este documento pode ser considerado maduro quando:

- o papel do Docker como substrate estiver reconciliado sem ambiguidade
- o container relevante do ambiente real estiver confirmado
- a separação entre problema de Docker host e problema de workload estiver clara
- os comandos de validação refletirem a prática operacional real
- ele puder ser usado como referência da camada Docker da Infra Hostinger sem depender do master legado

---

## Última revisão

- Status: ativo
- Classificação: canônico
- Contexto: infraestrutura Hostinger / Docker host
- Última revisão: 2026-03-18