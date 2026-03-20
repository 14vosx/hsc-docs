# Infra AWS Lightsail — README

## Objetivo

Documentar o contexto canônico da infraestrutura AWS Lightsail que hospeda a Auth API do ecossistema HSC.

Este contexto existe para registrar, de forma estável e auditável:

- o runtime real da Auth API em produção
- a topologia local do host
- a relação entre Nginx, Node.js via systemd e MariaDB local
- o fluxo operacional de deploy, rollback e troubleshooting
- as responsabilidades deste contexto dentro da arquitetura híbrida do HSC

---

## Navegação rápida

### Entrada
- [Home da documentação](../README.md)
- [Governance](../00-governance/README.md)
- [Master Index](../00-governance/99-master-index.md)

### Documentos deste contexto
- [Architecture Runtime](./infra-aws-lightsail-architecture-runtime.md)
- [Nginx Reverse Proxy](./nginx-reverse-proxy.md)
- [Node Systemd](./node-systemd.md)
- [MariaDB Local](./mariadb-local.md)
- [Auth API Operations](./auth-api-operations.md)
- [Deploy / Release / Rollback](./deploy-release-rollback.md)
- [Backup / Restore](./backup-restore.md)
- [Operational Runbooks](./infra-aws-lightsail-operational-runbooks.md)
- [Observability and Troubleshooting](./infra-aws-lightsail-observability-troubleshooting.md)
- [References and Inventory](./infra-aws-lightsail-references-inventory.md)

### Relações com outros contextos
- [Infra Hostinger](../01-infra-hostinger/README.md)
- [Portal Estático](../03-portal-estatico/README.md)
- [Backoffice Admin](../05-backoffice-admin/README.md)

### Trilhas operacionais relevantes
- [Admin API Contracts](../05-backoffice-admin/admin-api-contracts.md)
- [Auth, RBAC and Guards](../05-backoffice-admin/auth-rbac-and-guards.md)
- [Frontend Structure — Backoffice](../05-backoffice-admin/backoffice-admin-frontend-structure.md)
- [Nginx Static Serving](../01-infra-hostinger/nginx-static-serving.md)
- [Documentation System](../00-governance/documentation-system.md)

---

## Escopo

Este contexto cobre exclusivamente a camada AWS Lightsail ligada à Auth API.

Estão dentro do escopo deste contexto:

- host Ubuntu 22.04 LTS
- IP estático
- Nginx com TLS e reverse proxy
- serviço Node.js da Auth API executado via systemd
- MariaDB InnoDB local bound em `127.0.0.1`
- deploy determinístico por TAG
- rollback operacional
- superfícies públicas e administrativas da Auth API
- observabilidade e troubleshooting deste runtime

Este contexto não cobre, em profundidade:

- a VPS Hostinger
- o Game Panel / AMP / servidor CS2
- o ETL da Static API v2
- o frontend do portal estático
- credenciais e arquivos sensíveis
- backlog e roadmap de produto

---

## Estado atual

O estado atual conhecido deste contexto é:

- provedor: AWS Lightsail
- sistema operacional: Ubuntu 22.04 LTS
- exposição pública: Nginx com TLS e reverse proxy
- runtime da aplicação: Node.js via systemd
- banco de dados: MariaDB InnoDB local em `127.0.0.1`
- modelo de deploy: determinístico por TAG
- modelo operacional de administração: session-first com caminho break-glass administrativo
- papel arquitetural: camada de identidade, conteúdo e superfícies administrativas do ecossistema HSC

A Auth API está posicionada como backend central das superfícies dinâmicas do HSC, separada da camada Hostinger, que continua responsável pelo jogo, ETL e portal estático.

---

## O que roda aqui

Este contexto hospeda os componentes centrais da Auth API, incluindo:

- Nginx como edge HTTP/HTTPS do contexto
- serviço `hsc-auth-api` via systemd
- aplicação Node.js da Auth API
- MariaDB local
- fluxos de deploy e rollback
- superfícies HTTP públicas e administrativas da Auth API

Em termos funcionais, este contexto concentra:

- health endpoint
- conteúdo de News
- conteúdo de Seasons
- superfícies administrativas ligadas a conteúdo
- base estrutural para Events
- autenticação administrativa e controles de acesso operacionais

---

## O que não roda aqui

Este contexto não é responsável por:

- operar a VPS Hostinger
- publicar o portal estático
- gerar a Static API v2
- operar timers e ETL do portal
- gerenciar AMP ou a instância `MixHAXIXE01`
- operar MatchZy, plugins ou servidor CS2
- servir `matchzy.db` ou artefatos da camada de jogo

Esses papéis pertencem a outros contextos canônicos do repositório.

---

## Papel da Auth API no ecossistema HSC

A Auth API representa a camada dinâmica de identidade, conteúdo e administração do HSC.

Seu papel no ecossistema é:

- centralizar autenticação e superfícies administrativas
- expor conteúdo consumível externamente por rotas públicas
- sustentar fluxos futuros de backoffice e account
- manter a separação entre portal estático e backend dinâmico
- preservar a arquitetura híbrida do HSC, onde:
  - Hostinger sustenta jogo, ETL e portal
  - AWS Lightsail sustenta identidade, conteúdo e administração

Esta separação reduz acoplamento entre o runtime do jogo e o runtime da aplicação dinâmica.

---

## Topologia resumida

A topologia resumida deste contexto é:

- internet
- Nginx com TLS e reverse proxy
- Auth API Node.js via systemd
- MariaDB local bound em loopback

Em termos operacionais:

- o tráfego externo entra pelo Nginx
- o Nginx encaminha para o serviço local da Auth API
- a Auth API acessa o MariaDB local
- o deploy da aplicação ocorre por TAG, com `npm ci` e restart controlado do serviço
- o rollback é tratado por fluxo determinístico próprio

---

## Source of truth / evidências

As principais evidências que sustentam este contexto, nesta fase de migração documental, são:

- documentação consolidada atual do ecossistema HSC
- blueprint técnico consolidado do HSC
- documentos específicos de operação da Auth API em AWS Lightsail
- documentos de deploy, release e rollback da Auth API
- impl-logs ligados a CORS, sessões admin, schema snapshot, fail-closed e workflow de release

Enquanto a migração do contexto não estiver concluída, os documentos antigos seguem como fonte de extração e checagem cruzada, mas não como canônico final.

---

## Documentos canônicos deste contexto

Os documentos canônicos previstos para este contexto são:

- `docs/04-infra-aws-lightsail/README.md`
- `docs/04-infra-aws-lightsail/infra-aws-lightsail-architecture-runtime.md`
- `docs/04-infra-aws-lightsail/infra-aws-lightsail-network-dns-tls.md`
- `docs/04-infra-aws-lightsail/nginx-reverse-proxy.md`
- `docs/04-infra-aws-lightsail/node-systemd.md`
- `docs/04-infra-aws-lightsail/mariadb-local.md`
- `docs/04-infra-aws-lightsail/deploy-release-rollback.md`
- `docs/04-infra-aws-lightsail/backup-restore.md`
- `docs/04-infra-aws-lightsail/auth-api-operations.md`
- `docs/04-infra-aws-lightsail/infra-aws-lightsail-observability-troubleshooting.md`
- `docs/04-infra-aws-lightsail/infra-aws-lightsail-references-inventory.md`

Este `README.md` funciona como porta de entrada e índice local do contexto.

---

## Dependências externas

Este contexto depende de componentes externos e integrações relevantes, incluindo:

- DNS e resolução pública dos subdomínios do contexto
- emissão e renovação de certificado TLS
- repositório Git da Auth API
- fluxo de release por TAG
- consumidores externos das rotas públicas da Auth API
- frontends atuais e futuros que dependem de CORS e superfícies administrativas

Também existe dependência arquitetural do restante do ecossistema, já que este contexto não existe isoladamente: ele compõe a metade dinâmica da arquitetura híbrida do HSC.

---

## Relação com outros contextos

Este contexto se relaciona diretamente com:

### `docs/01-infra-hostinger/`
Relacionamento indireto de arquitetura global do ecossistema.  
A Hostinger sustenta o lado game + portal, enquanto o Lightsail sustenta identidade + conteúdo + admin.

### `docs/02-game-panel/`
Sem acoplamento operacional direto de runtime, mas com relação indireta dentro do ecossistema HSC.

### `docs/03-portal-estatico/`
Relação importante para consumo de conteúdo público, especialmente quando o portal ou seus espelhos consomem superfícies derivadas da Auth API.

### `docs/90-adr/`
Deve registrar decisões arquiteturais relevantes deste contexto, como a fixação da Auth API em AWS Lightsail.

### `docs/95-impl-log/`
Deve registrar mudanças incrementais relevantes deste contexto.

### `docs/97-audit/`
Pode conter análises, gaps e verificações de conformidade ligadas a esta infraestrutura.

### `docs/98-legacy/`
Preserva manuais e documentos históricos usados como base de migração.

---

## Relações com outros documentos

Este arquivo é complementar a:

- `docs/99-master-index.md`
- `docs/00-governance/README.md`
- `docs/00-governance/documentation-system.md`

Dentro deste próprio contexto, ele é a porta de entrada para todos os documentos canônicos específicos da camada AWS Lightsail.

---

## Critério de pronto deste contexto

Este contexto poderá ser considerado formalmente migrado quando:

- o runtime da Auth API estiver documentado sem depender do master antigo
- a topologia do host estiver clara
- o fluxo de deploy e rollback estiver formalizado
- MariaDB local estiver documentado em nível operacional
- as superfícies operacionais da Auth API estiverem registradas
- troubleshooting e observabilidade estiverem centralizados
- os documentos antigos passarem a servir apenas como referência cruzada

---

## Última revisão

- Status: ativo
- Classificação: canônico
- Contexto: infraestrutura AWS Lightsail / Auth API
- Última revisão: 2026-03-18