# HSC Docs — Master Index

## Objetivo

Este arquivo é o índice mestre canônico da documentação do ecossistema HSC.

Ele existe para:

- orientar a navegação do repositório documental
- apontar os contextos canônicos oficiais
- mostrar o estado da migração documental
- reduzir dependência dos antigos masters consolidados
- servir como ponto único de entrada para leitura e manutenção da documentação

---

## Regra canônica

A documentação do HSC passa a ser organizada por contexto.

Os contextos canônicos iniciais são:

1. Infra Hostinger
2. Game Panel
3. Portal Estático
4. Infra AWS Lightsail

Os antigos documentos consolidados amplos deixam de ser a fonte principal de navegação e passam a servir como apoio histórico durante a migração.

---

## Como navegar esta documentação

A ordem recomendada de leitura é:

1. `docs/00-governance/README.md`
2. `docs/00-governance/documentation-system.md`
3. este arquivo
4. o contexto específico que você precisa operar ou revisar

Sugestão de leitura por necessidade:

- infraestrutura base Hostinger → `docs/01-infra-hostinger/`
- operação do servidor CS2 → `docs/02-game-panel/`
- portal público e Static API v2 → `docs/03-portal-estatico/`
- Auth API no Lightsail → `docs/04-infra-aws-lightsail/`
- mudanças incrementais do sistema documental → `docs/95-impl-log/`
- pendências e gaps documentais → `docs/97-audit/`
- material histórico e masters antigos → `docs/98-legacy/`

---

## Estado atual da migração

Status da migração canônica inicial:

- `00-governance` → ativo
- `01-infra-hostinger` → estruturado
- `02-game-panel` → estruturado
- `03-portal-estatico` → estruturado
- `04-infra-aws-lightsail` → estruturado
- `90-adr` → reservado
- `95-impl-log` → ativo
- `97-audit` → ativo
- `98-legacy` → ativo

Interpretação do status:

- **ativo** → diretório já em uso real no sistema documental
- **estruturado** → documentos canônicos iniciais já criados
- **reservado** → diretório mantido para fase posterior

---

## Estrutura mestre do repositório

```text
docs/
├── 00-governance/
│   ├── README.md
│   ├── documentation-system.md
│   └── 99-master-index.md
│
├── 01-infra-hostinger/
│   ├── README.md
│   ├── architecture-runtime.md
│   ├── network-dns-tls.md
│   ├── nginx-static-serving.md
│   ├── docker-host.md
│   ├── certbot.md
│   ├── systemd-automation.md
│   ├── filesystem-paths-permissions.md
│   ├── observability-troubleshooting.md
│   └── references-inventory.md
│
├── 02-game-panel/
│   ├── README.md
│   ├── architecture-runtime.md
│   ├── amp-instance-manager.md
│   ├── instance-mixhaxixe01.md
│   ├── cs2-server-configuration.md
│   ├── matchzy.md
│   ├── plugins-installed.md
│   ├── mariadb-runtime.md
│   ├── operational-runbooks.md
│   ├── observability-troubleshooting.md
│   └── references-inventory.md
│
├── 03-portal-estatico/
│   ├── README.md
│   ├── architecture-runtime.md
│   ├── static-api-v2.md
│   ├── data-sources-matchzy-sqlite.md
│   ├── etl-bash-pipeline.md
│   ├── sql-queries-and-views.md
│   ├── json-contracts.md
│   ├── frontend-structure.md
│   ├── nginx-publishing-cache.md
│   ├── operational-runbooks.md
│   ├── observability-troubleshooting.md
│   └── references-inventory.md
│
├── 04-infra-aws-lightsail/
│   ├── README.md
│   ├── architecture-runtime.md
│   ├── node-systemd.md
│   ├── deploy-release-rollback.md
│   ├── mariadb-local.md
│   ├── auth-api-operations.md
│   ├── network-dns-tls.md
│   ├── nginx-reverse-proxy.md
│   ├── observability-troubleshooting.md
│   ├── backup-restore.md
│   └── references-inventory.md
│
├── 90-adr/
│
├── 95-impl-log/
│   └── 2026-03-18-initial-canonical-context-migration.md
│
├── 97-audit/
│   └── 2026-03-18-post-migration-open-items.md
│
└── 98-legacy/
    ├── README.md
    ├── master-documents-migration-map.md
    ├── HSC_Master_Blueprint.md
    └── HSC_MASTER_DOCUMENTATION.md
```

---

## Índice por contexto

## 00 — Governance

### `docs/00-governance/README.md`
Porta de entrada da governança documental do repositório.

### `docs/00-governance/documentation-system.md`
Define regras, convenções e funcionamento do sistema documental do HSC.

### `docs/00-governance/99-master-index.md`
Índice mestre canônico da documentação.

---

## 01 — Infra Hostinger

### `docs/01-infra-hostinger/README.md`
Porta de entrada do contexto da infraestrutura base Hostinger.

### `docs/01-infra-hostinger/architecture-runtime.md`
Topologia geral da VPS Hostinger como substrate do lado game + portal.

### `docs/01-infra-hostinger/network-dns-tls.md`
Camada de rede, DNS e TLS do lado Hostinger.

### `docs/01-infra-hostinger/nginx-static-serving.md`
Papel do Nginx como edge e camada de serving estático do portal e da v2.

### `docs/01-infra-hostinger/docker-host.md`
Docker como substrate do lado game na Hostinger.

### `docs/01-infra-hostinger/certbot.md`
Papel do Certbot na sustentação da camada HTTPS do lado Hostinger.

### `docs/01-infra-hostinger/systemd-automation.md`
Timers, services e automação recorrente do host.

### `docs/01-infra-hostinger/filesystem-paths-permissions.md`
Paths, ownership, grupos e permissões críticos da camada base.

### `docs/01-infra-hostinger/observability-troubleshooting.md`
Runbook de triagem da infraestrutura Hostinger.

### `docs/01-infra-hostinger/references-inventory.md`
Inventário e referências do contexto Hostinger.

---

## 02 — Game Panel

### `docs/02-game-panel/README.md`
Porta de entrada do contexto operacional do servidor CS2.

### `docs/02-game-panel/architecture-runtime.md`
Topologia do lado jogo: AMP, instância, CS2, MatchZy, plugins e persistência.

### `docs/02-game-panel/amp-instance-manager.md`
Papel do AMP como camada de gestão da instância.

### `docs/02-game-panel/instance-mixhaxixe01.md`
Documento canônico da instância oficial `MixHAXIXE01`.

### `docs/02-game-panel/cs2-server-configuration.md`
Camadas de configuração do servidor CS2.

### `docs/02-game-panel/matchzy.md`
Documento canônico do MatchZy no HSC.

### `docs/02-game-panel/plugins-installed.md`
Inventário e papel da camada de plugins do servidor.

### `docs/02-game-panel/mariadb-runtime.md`
Estado atual da camada MariaDB no lado game, distinguindo confirmado, parcialmente confirmado e não confirmado.

### `docs/02-game-panel/operational-runbooks.md`
Runbooks de scrim, BO1, BO3, veto, coach, warmup, fallback admin e checagens do lado jogo.

### `docs/02-game-panel/observability-troubleshooting.md`
Runbook de triagem do Game Panel.

### `docs/02-game-panel/references-inventory.md`
Inventário e referências do contexto Game Panel.

---

## 03 — Portal Estático

### `docs/03-portal-estatico/README.md`
Porta de entrada do contexto do portal público e da Static API v2.

### `docs/03-portal-estatico/architecture-runtime.md`
Topologia do portal: SQLite → ETL → JSON → Nginx → browser.

### `docs/03-portal-estatico/static-api-v2.md`
Documento canônico da Static API v2.

### `docs/03-portal-estatico/data-sources-matchzy-sqlite.md`
Fonte de dados primária do portal baseada no `matchzy.db`.

### `docs/03-portal-estatico/etl-bash-pipeline.md`
Pipeline ETL Bash da geração da v2.

### `docs/03-portal-estatico/sql-queries-and-views.md`
Papel da camada SQL versionada na geração da v2.

### `docs/03-portal-estatico/json-contracts.md`
Contratos públicos JSON da v2.

### `docs/03-portal-estatico/frontend-structure.md`
Estrutura do frontend estático do portal.

### `docs/03-portal-estatico/nginx-publishing-cache.md`
Publishing e cache da borda pública do portal/v2.

### `docs/03-portal-estatico/operational-runbooks.md`
Runbooks operacionais do portal e da API estática.

### `docs/03-portal-estatico/observability-troubleshooting.md`
Runbook de triagem do Portal Estático.

### `docs/03-portal-estatico/references-inventory.md`
Inventário e referências do contexto Portal Estático.

---

## 04 — Infra AWS Lightsail

### `docs/04-infra-aws-lightsail/README.md`
Porta de entrada do contexto da Auth API no Lightsail.

### `docs/04-infra-aws-lightsail/architecture-runtime.md`
Topologia da instância Lightsail e da Auth API.

### `docs/04-infra-aws-lightsail/node-systemd.md`
Execução da Auth API em Node.js via systemd.

### `docs/04-infra-aws-lightsail/deploy-release-rollback.md`
Fluxo canônico de deploy, release e rollback da Auth API.

### `docs/04-infra-aws-lightsail/mariadb-local.md`
Papel do MariaDB local do backend dinâmico.

### `docs/04-infra-aws-lightsail/auth-api-operations.md`
Superfícies operacionais e invariantes da Auth API.

### `docs/04-infra-aws-lightsail/network-dns-tls.md`
Camada de rede, DNS e TLS do contexto Lightsail.

### `docs/04-infra-aws-lightsail/nginx-reverse-proxy.md`
Nginx como reverse proxy da Auth API.

### `docs/04-infra-aws-lightsail/observability-troubleshooting.md`
Runbook de triagem do contexto Lightsail.

### `docs/04-infra-aws-lightsail/backup-restore.md`
Backup e restore do MariaDB/local runtime da Auth API.

### `docs/04-infra-aws-lightsail/references-inventory.md`
Inventário e referências do contexto Lightsail.

---

## 90 — ADR

Diretório reservado para Architecture Decision Records do ecossistema.

Objetivo:

- registrar decisões arquiteturais relevantes
- preservar contexto de decisão
- reduzir ambiguidades futuras

Estado atual:

- reservado para próxima fase

---

## 95 — Impl Log

Diretório ativo para impl-logs e histórico incremental de mudanças relevantes.

Objetivo:

- registrar mudanças pequenas e médias sem poluir documentos canônicos
- sustentar rastreabilidade
- facilitar reconciliação futura

### `docs/95-impl-log/2026-03-18-initial-canonical-context-migration.md`
Registro formal da migração documental inicial para o modelo canônico por contexto.

---

## 97 — Audit

Diretório ativo para auditorias, gap analysis, reconciliações e verificações documentais.

Objetivo:

- listar pendências
- registrar inconsistências entre docs e runtime
- concentrar checks de qualidade documental

### `docs/97-audit/2026-03-18-post-migration-open-items.md`
Auditoria curta das pendências reais abertas logo após a migração inicial.

---

## 98 — Legacy

Diretório ativo para material legado e apoio histórico.

Objetivo:

- preservar documentação antiga relevante
- evitar perda de contexto histórico
- permitir migração progressiva sem destruir fontes antigas
- impedir que o legado continue governando o sistema novo

### `docs/98-legacy/README.md`
Porta de entrada da camada de legado e regra oficial de uso do material histórico.

### `docs/98-legacy/master-documents-migration-map.md`
Mapa de migração dos antigos masters para os contextos canônicos atuais.

### `docs/98-legacy/HSC_Master_Blueprint.md`
Blueprint histórico preservado como material legado.

### `docs/98-legacy/HSC_MASTER_DOCUMENTATION.md`
Master histórico preservado como material legado.

---

## Regras de manutenção do índice mestre

Este arquivo deve ser atualizado quando houver:

- criação de novo documento canônico
- remoção ou renomeação de documento canônico
- abertura formal de novo contexto
- mudança de status de um diretório reservado
- ativação de diretórios antes reservados
- conclusão de uma nova fase da migração documental

Regra importante:

- o índice mestre não deve virar documentação técnica detalhada de contexto
- ele deve continuar sendo índice, mapa e referência de navegação

---

## Critério de pronto do índice mestre

Este índice pode ser considerado saudável quando:

- todos os documentos canônicos existentes aparecem aqui
- os 4 contextos iniciais canônicos estão listados corretamente
- os diretórios ativos de impl-log, audit e legacy estão refletidos
- a separação entre contexto vivo e material histórico está clara
- um novo colaborador consegue entender onde procurar cada assunto sem depender dos antigos masters

---

## Última revisão

- Status: ativo
- Classificação: canônico
- Contexto: governança / índice mestre
- Última revisão: 2026-03-18