# Architecture Runtime

## Objetivo

Documentar a topologia real de runtime da Infra Hostinger que sustenta a metade game + portal do ecossistema HSC.

Este documento existe para registrar, de forma estável e auditável:

- o inventário essencial do host Debian
- os componentes reais que compõem a camada-base do lado Hostinger
- a relação entre Nginx, Docker, Certbot, systemd e filesystem do host
- a posição de AMP, Portal Estático e Static API v2 como workloads sobre essa base
- os fluxos principais que atravessam a infraestrutura
- os failure domains centrais desta camada

---

## Escopo

Este documento cobre:

- provedor e host operacional do contexto
- sistema operacional Debian
- Nginx como edge do lado Hostinger
- Docker como substrate do host
- Certbot e TLS na borda pública
- systemd e timers do host
- publishing público do portal e da Static API v2
- relação estrutural com workloads superiores como AMP e Portal Estático
- dependências operacionais e pontos de falha da camada-base

Este documento não cobre em profundidade:

- operação do AMP Instance Manager
- operação da instância `MixHAXIXE01`
- plugins, MatchZy e runbooks do servidor CS2
- pipeline ETL script por script
- contratos JSON da Static API v2
- Auth API no AWS Lightsail

Esses tópicos vivem em documentos próprios dos contextos `02-game-panel`, `03-portal-estatico` e `04-infra-aws-lightsail`.

---

## Estado atual

O runtime atual conhecido do contexto `01-infra-hostinger` é:

- provedor: Hostinger
- sistema operacional: Debian
- edge público: Nginx
- TLS: Certbot / Let's Encrypt
- substrate de containers: Docker
- automação do host: systemd e timers
- publishing público do portal: `/var/www/portal/cs2/`
- publishing público da Static API v2: `/var/www/api/cs2/v2/`
- base operacional do ETL: `/opt/cs2-portal/`
- workload operacional do lado jogo sobre essa base:
  - AMP Instance Manager
  - instância `MixHAXIXE01`
  - servidor CS2
- workload operacional do lado publicação sobre essa base:
  - Portal Estático
  - Static API v2
  - automações de geração e publishing

A Infra Hostinger opera como substrate compartilhado do lado público e operacional do jogo no HSC.

---

## Source of truth / evidências

As evidências principais deste runtime, nesta fase de migração, são:

- documentação consolidada atual do ecossistema HSC
- blueprint técnico consolidado do HSC
- documentação específica da camada Hostinger / AMP / CS2
- reconciliação dos paths públicos e operacionais do host
- reconciliação do uso de Nginx, Docker, Certbot e systemd timers no lado Hostinger

Os nomes de componentes, paths e relações descritos aqui foram reconciliados a partir desses documentos.

---

## Relações com outros documentos

Este arquivo é complementar a:

- `docs/01-infra-hostinger/README.md`
- `docs/01-infra-hostinger/network-dns-tls.md`
- `docs/01-infra-hostinger/nginx-static-serving.md`
- `docs/01-infra-hostinger/docker-host.md`
- `docs/01-infra-hostinger/certbot.md`
- `docs/01-infra-hostinger/systemd-automation.md`
- `docs/01-infra-hostinger/filesystem-paths-permissions.md`
- `docs/01-infra-hostinger/infra-hostinger-observability-troubleshooting.md`
- `docs/02-game-panel/README.md`
- `docs/03-portal-estatico/README.md`

Este documento descreve a topologia da camada-base Hostinger.  
Ele não substitui os documentos especializados de rede, edge, automação, Game Panel ou Portal Estático.

---

## Visão geral do runtime

A topologia lógica do lado Hostinger é:

- internet
- Nginx no host Debian
- filesystem público e operacional do host
- Docker no host
- Certbot e gestão de TLS
- systemd e timers
- workloads superiores sobre o host:
  - AMP / servidor CS2
  - Portal Estático / Static API v2
  - ETL e publishing público

Representação resumida:

- tráfego externo chega ao Nginx
- o Nginx serve portal e API estática
- o host mantém automações e paths operacionais
- o Docker sustenta workloads do lado game
- systemd organiza timers e serviços do host
- o filesystem conecta geração, publicação e leitura pública

Este desenho faz da VPS Hostinger a fundação técnica do lado público game + portal do HSC.

---

## Inventário do host

### Provedor

- Hostinger

### Sistema operacional

- Debian

### Papel do host

O host existe para sustentar:

- edge público estático do lado Hostinger
- substrate para o runtime do jogo
- substrate para o Portal Estático
- ETL e publicação da Static API v2
- automações recorrentes do host
- filesystem público e operacional compartilhado

Esse host não é apenas uma máquina do jogo; ele também sustenta a camada pública de consulta do ecossistema.

---

## Componentes do contexto

### Debian

O Debian é a base do host.

Responsabilidades nesta camada:

- execução do Nginx
- execução do Docker
- execução do Certbot
- execução de systemd e timers
- gestão de filesystem, permissões e ownership
- sustentação dos diretórios públicos e operacionais
- suporte à execução dos workloads superiores

O sistema operacional é parte central do runtime porque define o comportamento de serviços, paths, automações e superfície pública.

---

### Nginx

O Nginx é o edge do lado Hostinger.

Papel no runtime:

- receber tráfego público HTTP/HTTPS
- publicar o portal público
- publicar a Static API v2
- servir conteúdo same-origin complementar, quando aplicável
- bloquear exposição de artefatos que não devem ser públicos
- participar da camada de cache e entrega do lado Hostinger

No desenho atual, o Nginx é a principal superfície pública da metade game + portal do ecossistema.

---

### Docker host

O Docker opera como substrate de containers no host Hostinger.

Papel no runtime:

- sustentar workloads containerizados do lado jogo
- fornecer base de execução para o ambiente do servidor CS2
- coexistir com o publishing estático e com as automações do host

Regra importante:
- o Docker pertence à camada-base
- a operação detalhada da instância, do AMP e do servidor CS2 vive no contexto `02-game-panel`

---

### Certbot

O Certbot integra a camada de segurança de transporte do host.

Papel no runtime:

- emissão e renovação de certificados TLS
- sustentação da exposição HTTPS do lado Hostinger
- integração operacional com o Nginx

A saúde do Certbot afeta diretamente a disponibilidade pública do portal e da API estática em modo seguro.

---

### systemd

O systemd é o mecanismo de automação e gerenciamento de serviços do host.

Papel no runtime:

- orquestrar serviços do host
- sustentar timers recorrentes
- permitir execução previsível de automações
- apoiar geração de artefatos públicos e checks operacionais
- servir como base de observabilidade via status e logs

No lado Hostinger, systemd é parte essencial da confiabilidade do publishing e da manutenção do host.

---

### AMP como workload sobre o host

O AMP não é a camada-base, mas é sustentado por ela.

Relação com a Infra Hostinger:

- depende do host Debian
- depende do Docker host
- depende do filesystem e dos paths do host
- depende da disponibilidade operacional do substrate
- gera, indiretamente, a fonte de dados que alimenta a Static API v2

Regra arquitetural:
- AMP vive sobre esta infraestrutura
- a documentação operacional do AMP vive em `docs/02-game-panel/`

---

### Portal e Static API como workloads publicados pelo host

O Portal Estático e a Static API v2 também vivem sobre esta infraestrutura.

Relação com a Infra Hostinger:

- dependem do filesystem público do host
- dependem do Nginx para publishing
- dependem de automações do host para geração e atualização
- dependem da integridade de ownership e permissões
- dependem da estabilidade desta camada para continuar acessíveis ao público

Regra arquitetural:
- o host publica
- o contexto `03-portal-estatico` descreve o que é publicado e como é gerado

---

## Dependências entre camadas

A Infra Hostinger sustenta várias camadas que se apoiam umas nas outras.

### Dependência 1 — edge e filesystem

- Nginx depende de paths públicos corretos
- os paths públicos dependem do filesystem íntegro
- as permissões precisam permitir escrita pelo fluxo de geração e leitura pelo edge público

### Dependência 2 — automação e publicação

- o ETL e a publicação da v2 dependem de systemd/timers, scripts e paths corretos
- o publishing saudável depende da saúde da automação do host

### Dependência 3 — runtime do jogo e dados públicos

- o contexto de jogo depende do substrate Docker/AMP
- o Portal Estático depende indiretamente dos dados produzidos nesse lado do ecossistema

### Dependência 4 — TLS e disponibilidade pública

- a saúde do Certbot e do Nginx afeta diretamente a experiência pública do portal e da v2

---

## Fluxos principais

### Tráfego externo → Nginx

Fluxo esperado:

1. o cliente acessa o domínio público do lado Hostinger
2. o tráfego chega ao host
3. o Nginx recebe a conexão
4. o Nginx serve portal, Static API v2 e recursos públicos auxiliares

Esse fluxo representa a borda pública do lado Hostinger.

---

### Nginx → arquivos estáticos

Fluxo esperado:

1. o Nginx resolve o path público solicitado
2. lê o arquivo correspondente na árvore pública
3. serve o arquivo ao cliente

Esse fluxo depende de:

- paths corretos
- ownership e permissões corretos
- ausência de artefato parcial
- publishing íntegro da camada pública

---

### Host → Docker / AMP

Fluxo esperado:

1. o host sustenta Docker
2. o Docker sustenta o ambiente do lado jogo
3. o AMP e a instância CS2 operam sobre essa base
4. o runtime do jogo produz a fonte de dados que alimenta o portal

Esse fluxo mostra por que a Infra Hostinger e o Game Panel precisam ser separados, mas ainda assim fortemente relacionados.

---

### Host → timers / automação

Fluxo esperado:

1. systemd e timers executam rotinas do host
2. essas rotinas sustentam geração, checks e publishing
3. os artefatos gerados chegam aos diretórios públicos
4. o Nginx passa a servir o estado mais recente da camada pública

Esse fluxo é central para o Portal Estático e para a Static API v2.

---

## Paths oficiais

Os paths oficiais conhecidos desta camada incluem:

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

### SQLs do portal

- `/opt/cs2-portal/sql/`

### Lock global conhecido

- `/opt/cs2-portal/locks/gen-all-v2.lock`

### Scripts operacionais

- `/usr/local/bin/`

### Base estrutural da instância AMP

- árvore operacional sob `/home/amp/.ampdata/instances/`

Esses paths devem ser tratados como referência operacional canônica até revisão explícita.

---

## Failure domains do host

Os principais failure domains desta camada incluem:

### 1. Nginx

Se o Nginx falhar:

- o portal e a Static API v2 ficam indisponíveis publicamente
- os arquivos podem continuar íntegros no disco, mas deixam de ser acessíveis ao usuário

---

### 2. Certbot / TLS

Se o TLS falhar:

- a superfície pública pode ficar degradada ou operacionalmente indisponível
- o serviço pode parecer “de pé”, mas com erro de confiança ou transporte

---

### 3. Filesystem / permissões

Se paths, ownership ou permissões divergirem:

- o ETL pode não conseguir escrever
- o Nginx pode não conseguir ler
- workflows do lado AMP podem falhar
- a borda pública pode servir estado incompleto ou indisponível

---

### 4. Docker host

Se o Docker falhar:

- o lado jogo fica comprometido
- a origem dos dados públicos pode parar de evoluir
- impactos podem aparecer primeiro no Game Panel e depois no Portal Estático

---

### 5. systemd / timers

Se timers ou serviços do host falharem:

- o ETL pode deixar de rodar
- a v2 pode ficar stale
- checks e automações deixam de refletir o estado real

---

### 6. Drift entre host e workloads

Se o substrate do host divergir do que os workloads esperam:

- paths deixam de bater
- o portal quebra mesmo com ETL “saudável”
- o Game Panel quebra mesmo com Docker “de pé”
- troubleshooting fica mais difícil

---

## Limites deste documento

Este documento não detalha:

- vhosts e configuração textual do Nginx
- política detalhada de DNS e TLS
- comandos operacionais do AMP
- pipeline ETL em detalhe
- estrutura de JSON da v2
- plugins, MatchZy ou operação da partida
- runtime da Auth API

Esses detalhes vivem em documentos próprios.

---

## Critério de pronto deste documento

Este documento pode ser considerado maduro quando:

- o inventário da camada-base estiver validado contra o host real
- os paths oficiais estiverem confirmados
- a relação entre host, Nginx, Docker, Certbot, systemd e workloads estiver clara
- os failure domains estiverem corretamente refletidos
- ele puder ser lido como visão topológica da Infra Hostinger sem depender do master legado

---

## Última revisão

- Status: ativo
- Classificação: canônico
- Contexto: infraestrutura Hostinger / arquitetura de runtime
- Última revisão: 2026-03-18