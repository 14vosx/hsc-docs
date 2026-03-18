# Infra Hostinger — README

## Objetivo

Documentar o contexto canônico da Infra Hostinger que sustenta a base operacional do ecossistema HSC.

Este contexto existe para registrar, de forma estável e auditável:

- o papel da VPS Hostinger como camada-base do ecossistema
- a infraestrutura Debian que hospeda os workloads públicos do lado Hostinger
- a relação entre Nginx, Docker, Certbot, systemd e filesystem do host
- os limites entre infraestrutura base, Game Panel e Portal Estático
- as dependências operacionais que precisam permanecer estáveis para o funcionamento do lado Hostinger do HSC

---

## Escopo

Este contexto cobre exclusivamente a camada-base da VPS Hostinger.

Estão dentro do escopo deste contexto:

- host Debian
- Nginx como edge do lado Hostinger
- TLS e Certbot
- Docker como substrate do host
- systemd e automações do host
- filesystem, paths, ownership e permissões da camada-base
- relação estrutural entre host, publishing público e workloads sustentados sobre ele

Este contexto não cobre, em profundidade:

- operação do AMP Instance Manager
- operação da instância `MixHAXIXE01`
- MatchZy e plugins do servidor CS2
- runbooks de partida, scrim, veto, coach ou warmup
- pipeline ETL Bash da Static API v2
- contratos JSON do portal
- Auth API no AWS Lightsail
- credenciais e arquivos sensíveis

---

## Estado atual

O estado operacional conhecido deste contexto é:

- a camada-base do lado Hostinger roda sobre Debian
- o Nginx atua como edge de publicação pública para portal e API estática
- o host utiliza Docker como parte do substrate operacional
- há uso de Certbot para TLS
- automações do host usam systemd e timers
- workloads do ecossistema rodam sobre essa base, especialmente:
  - Game Panel / AMP / servidor CS2
  - Portal Estático
  - Static API v2
- o host depende de paths e permissões estáveis para manter publishing, automação e consumo público corretos

A Infra Hostinger é a base técnica da metade “game + portal” do ecossistema HSC.

---

## O que roda aqui

Este contexto sustenta, em nível de base, componentes como:

- Debian
- Nginx
- Certbot
- Docker host
- automações e timers do host
- filesystem público e operacional
- publishing do portal e da Static API v2
- workloads superiores que dependem dessa base, como AMP e o runtime público do portal

Do ponto de vista arquitetural, esta camada não é “o produto final”, mas sim a infraestrutura que torna possível a existência dos outros contextos hospedados do lado Hostinger.

---

## O que não roda aqui

Este contexto não deve ser confundido com:

### Game Panel

Este contexto não documenta, em profundidade:

- o AMP Instance Manager
- a operação da instância `MixHAXIXE01`
- MatchZy
- plugins
- runbooks de partidas

Esses itens pertencem a `docs/02-game-panel/`.

### Portal Estático

Este contexto não documenta, em profundidade:

- Static API v2
- pipeline ETL Bash
- contratos JSON
- estrutura do frontend
- troubleshooting específico do portal

Esses itens pertencem a `docs/03-portal-estatico/`.

### Auth API

Este contexto não documenta:

- Nginx reverse proxy do Lightsail
- Node.js via systemd da Auth API
- MariaDB local do backend dinâmico
- deploy/release/rollback da Auth API

Esses itens pertencem a `docs/04-infra-aws-lightsail/`.

---

## Papel da VPS Hostinger no ecossistema HSC

A VPS Hostinger é a camada-base do lado público e operacional do jogo dentro do HSC.

Seu papel no ecossistema é:

- sustentar o ambiente do servidor CS2
- sustentar a camada de publishing do portal
- sustentar a camada de publicação da Static API v2
- fornecer substrate de rede, filesystem e automação para workloads superiores
- separar o lado “game + portal” do lado “backend dinâmico” hospedado no Lightsail

Em termos arquiteturais:

- Hostinger = jogo, portal e publicação estática
- AWS Lightsail = Auth API, identidade, conteúdo dinâmico e superfícies administrativas

Essa separação reduz acoplamento e ajuda a organizar o ecossistema por responsabilidade.

---

## Contextos dependentes hospedados sobre esta base

Os principais contextos que dependem da Infra Hostinger são:

### `docs/02-game-panel/`

Depende da base Hostinger para:

- host Linux
- Docker host
- runtime subjacente do AMP
- persistência de arquivos da instância
- paths operacionais do lado CS2

### `docs/03-portal-estatico/`

Depende da base Hostinger para:

- publishing via Nginx
- filesystem público
- automações do host
- execução do ETL
- leitura/escrita nos diretórios públicos e operacionais

Esses contextos são independentes em responsabilidade documental, mas dependem diretamente da estabilidade da camada-base documentada aqui.

---

## Topologia resumida

A topologia resumida deste contexto é:

- internet
- Nginx no host Hostinger
- filesystem público e operacional
- Docker host
- systemd e timers
- workloads superiores:
  - Game Panel / AMP / CS2
  - Portal Estático / Static API v2

Em termos operacionais:

- o tráfego externo chega ao Nginx
- o Nginx serve o portal e a API estática
- o host mantém automações e paths públicos
- o host sustenta workloads que produzem dados ou servem assets
- a saúde dessa camada afeta diretamente disponibilidade pública do lado Hostinger

---

## Documentos canônicos deste contexto

Os documentos canônicos previstos para este contexto são:

- `docs/01-infra-hostinger/README.md`
- `docs/01-infra-hostinger/architecture-runtime.md`
- `docs/01-infra-hostinger/network-dns-tls.md`
- `docs/01-infra-hostinger/nginx-static-serving.md`
- `docs/01-infra-hostinger/docker-host.md`
- `docs/01-infra-hostinger/certbot.md`
- `docs/01-infra-hostinger/systemd-automation.md`
- `docs/01-infra-hostinger/filesystem-paths-permissions.md`
- `docs/01-infra-hostinger/observability-troubleshooting.md`
- `docs/01-infra-hostinger/references-inventory.md`

Este `README.md` funciona como porta de entrada e índice local do contexto.

---

## Dependências externas

Este contexto depende de componentes e condições externas relevantes, incluindo:

- DNS e resolução pública do lado Hostinger
- integridade do host Debian
- Nginx funcional
- Certbot funcional e TLS válido
- Docker funcional
- paths públicos e operacionais íntegros
- permissões corretas para escrita pelo ETL e leitura pelo Nginx
- automações do host executando como esperado
- integridade do runtime subjacente que sustenta o Game Panel e o Portal Estático

Também há dependência de disciplina operacional para evitar:

- drift de path
- drift de permissões
- publicação pública inconsistente
- quebra de automação do host

---

## Source of truth / evidências

As principais evidências que sustentam este contexto, nesta fase de migração documental, são:

- documentação consolidada atual do ecossistema HSC
- blueprint técnico consolidado do HSC
- documentação específica da infraestrutura Hostinger e da camada AMP/CS2
- reconciliação documental dos paths públicos e operacionais
- reconciliação dos componentes Debian, Nginx, Docker, Certbot e systemd no lado Hostinger

Enquanto a migração do contexto não estiver concluída, os documentos antigos seguem como fonte de extração e checagem cruzada, mas não como canônico final.

---

## Relação com outros contextos

Este contexto se relaciona diretamente com:

### `docs/02-game-panel/`

Relação estrutural forte.  
O Game Panel existe sobre a base operacional desta VPS.

### `docs/03-portal-estatico/`

Relação estrutural forte.  
O Portal Estático e a Static API v2 são publicados e sustentados sobre esta infraestrutura.

### `docs/04-infra-aws-lightsail/`

Relação indireta de arquitetura global.  
O Lightsail sustenta a metade dinâmica do ecossistema, enquanto a Hostinger sustenta a metade game + portal.

### `docs/90-adr/`

Deve registrar decisões arquiteturais relevantes sobre separação de contextos, hosting e papel desta infraestrutura no ecossistema.

### `docs/95-impl-log/`

Deve registrar mudanças incrementais relevantes desta camada, como mudanças de paths, automações, publishing, TLS ou substrate do host.

### `docs/97-audit/`

Pode conter análises, gaps, inconsistências ou verificações ligadas à infraestrutura Hostinger.

### `docs/98-legacy/`

Preserva documentação histórica usada como base de migração desta camada.

---

## Relações com outros documentos

Este arquivo é complementar a:

- `docs/99-master-index.md`
- `docs/00-governance/README.md`
- `docs/00-governance/documentation-system.md`

Dentro deste próprio contexto, ele é a porta de entrada para todos os documentos canônicos específicos da Infra Hostinger.

---

## Critério de pronto deste contexto

Este contexto poderá ser considerado formalmente migrado quando:

- a VPS Hostinger estiver documentada como camada-base sem depender do master antigo
- a separação entre host base, Game Panel e Portal Estático estiver clara
- os componentes Debian, Nginx, Docker, Certbot e systemd estiverem corretamente posicionados
- os paths, permissões e dependências do host estiverem formalizados
- troubleshooting e observabilidade da camada-base estiverem centralizados
- os documentos antigos passarem a servir apenas como referência cruzada

---

## Última revisão

- Status: ativo
- Classificação: canônico
- Contexto: infraestrutura Hostinger / porta de entrada do contexto
- Última revisão: 2026-03-18