# Impl Log — Initial Canonical Context Migration

## Objetivo

Registrar formalmente a migração documental inicial do ecossistema HSC para o novo modelo canônico baseado em contextos.

Este impl-log existe para registrar, de forma estável e auditável:

- quando a nova base documental deixou de ser apenas planejamento e passou a existir de fato
- quais contextos canônicos iniciais foram estruturados
- quais documentos-base foram criados
- qual foi o impacto dessa mudança na governança do repositório
- quais pendências continuam abertas após essa primeira migração

---

## Tipo de mudança

- Classificação: estrutural documental
- Natureza: criação da base canônica inicial
- Escopo: governança + 4 contextos canônicos iniciais
- Impacto esperado: alto, porque muda a navegação oficial da documentação do ecossistema

---

## Data de referência

- Data da migração: 2026-03-18

---

## Contexto

Antes desta mudança, o ecossistema HSC dependia fortemente de documentos amplos e consolidados, tratados como “verdade do ecossistema”, mas já com sinais claros de drift documental.

Problemas observados no modelo anterior:

- acúmulo de informação muito heterogênea em poucos arquivos amplos
- dificuldade de manter o master alinhado ao runtime real
- perda de informação por dependência excessiva de agentes e reconstruções posteriores
- mistura entre infraestrutura, operação do jogo, portal público e backend dinâmico
- dificuldade de diferenciar o que estava confirmado, parcialmente confirmado ou apenas herdado de documentação antiga

Exemplo explícito já reconhecido no processo:

- a Auth API não usa mais Hostinger como runtime principal
- o runtime atual dessa camada já é AWS Lightsail
- esse tipo de mudança já demonstrava que o modelo anterior não estava sendo mantido com a granularidade necessária

Diante disso, foi adotado o modelo documental por contexto.

---

## Decisão aplicada

A decisão aplicada nesta migração foi:

**reestruturar a documentação do HSC por contexto canônico, com governança explícita, índice mestre e documentos especializados por camada operacional**

Os 4 contextos canônicos iniciais formalizados foram:

1. Infra Hostinger
2. Game Panel
3. Portal Estático
4. Infra AWS Lightsail

Além disso, foi formalizada a camada de governança do sistema documental.

---

## Resultado desta migração

Ao final desta primeira migração, o repositório documental passou a ter base canônica explícita para:

- governança da documentação
- índice mestre
- infraestrutura base do lado Hostinger
- operação do servidor CS2
- portal público e Static API v2
- Auth API no AWS Lightsail

Em termos práticos, esta mudança marca o momento em que:

- a documentação por contexto deixa de ser ideia
- e passa a ser a navegação oficial do ecossistema HSC

---

## Estrutura criada nesta migração

A estrutura canônica inicial criada/estruturada nesta fase foi:

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
├── 95-impl-log/
├── 97-audit/
└── 98-legacy/
```

---

## Documentos de governança criados/revisados

Durante esta migração, a camada de governança foi formalizada com estes documentos:

### `docs/00-governance/README.md`

Função:
- porta de entrada da governança
- explicitar que os 4 contextos canônicos iniciais já estão estruturados

### `docs/00-governance/documentation-system.md`

Função:
- formalizar o sistema documental
- registrar convenções editoriais
- registrar a regra de tratamento de fatos parcialmente confirmados

### `docs/00-governance/99-master-index.md`

Função:
- servir como índice mestre canônico
- consolidar a navegação oficial do repositório

---

## Contexto 01 estruturado: Infra Hostinger

O contexto `01-infra-hostinger/` foi estruturado para separar a camada-base do lado Hostinger dos demais contextos.

Objetivos alcançados nesta migração:

- explicitar Debian, Nginx, Docker host, Certbot e systemd como substrate
- separar infraestrutura base de Game Panel
- separar infraestrutura base de Portal Estático
- formalizar filesystem, paths e permissões como tema próprio
- formalizar observabilidade e troubleshooting da base Hostinger

Resultado prático:

- a Hostinger deixa de ser descrita de forma difusa dentro de masters amplos
- e passa a ter documentação canônica como camada própria

---

## Contexto 02 estruturado: Game Panel

O contexto `02-game-panel/` foi estruturado para separar a operação do servidor CS2 do substrate da Hostinger e da publicação do portal.

Objetivos alcançados nesta migração:

- formalizar o papel do AMP
- formalizar a instância oficial `MixHAXIXE01`
- formalizar o MatchZy como componente central
- formalizar plugins instalados como camada própria
- formalizar runbooks do lado jogo
- formalizar troubleshooting do Game Panel
- registrar, de forma honesta, o estado atual do `mariadb-runtime.md`

Resultado prático:

- o lado jogo passa a ter documentação canônica própria
- sem depender de memória informal ou mistura com infraestrutura base

---

## Contexto 03 estruturado: Portal Estático

O contexto `03-portal-estatico/` foi estruturado para separar a camada pública estática do resto do ecossistema.

Objetivos alcançados nesta migração:

- formalizar a Static API v2 como camada oficial
- formalizar o pipeline ETL Bash
- formalizar o `matchzy.db` como fonte estatística pública
- formalizar os contratos JSON da v2
- formalizar a estrutura do frontend estático
- formalizar publishing e cache via Nginx
- formalizar observabilidade e runbooks da camada pública

Resultado prático:

- o portal deixa de existir apenas como pedaços espalhados em documentação maior
- e passa a ter um contexto canônico navegável e auditável

---

## Contexto 04 estruturado: Infra AWS Lightsail

O contexto `04-infra-aws-lightsail/` foi estruturado para consolidar a Auth API em seu runtime atual real.

Objetivos alcançados nesta migração:

- substituir a leitura antiga que ainda deixava resquícios de Hostinger para Auth API
- formalizar Ubuntu 22.04 LTS, Nginx reverse proxy, Node via systemd e MariaDB local
- formalizar deploy/release/rollback
- formalizar backup/restore
- formalizar observabilidade do contexto
- formalizar inventário e referências do runtime real atual

Resultado prático:

- o runtime atual da Auth API passa a estar refletido corretamente na estrutura canônica
- reduzindo um dos drifts mais evidentes do modelo anterior

---

## Padrão editorial formalizado nesta migração

Além da estrutura por contexto, esta migração também consolidou um padrão editorial importante:

### tratamento explícito de fatos parcialmente confirmados

Esse padrão foi formalizado em:

- `docs/00-governance/documentation-system.md`

E aplicado de forma representativa em:

- `docs/02-game-panel/mariadb-runtime.md`

Resultado prático:

- o repositório passa a ter regra explícita para lidar com evidência parcial
- reduzindo o risco de inventar arquitetura “para completar documento”

---

## Impacto na navegação oficial

Depois desta migração, a navegação oficial da documentação do HSC passa a ser:

1. `docs/00-governance/README.md`
2. `docs/00-governance/documentation-system.md`
3. `docs/00-governance/99-master-index.md`
4. contexto específico necessário

Isso implica que:

- antigos masters amplos deixam de ser o ponto principal de navegação
- o índice mestre passa a ser a referência global do repositório
- cada camada do ecossistema passa a ser consultada no seu contexto correto

---

## O que esta migração não fez

Esta migração **não** significa que toda a documentação histórica do HSC já foi reconciliada por completo.

Ela não fez, ainda:

- ativação formal de ADRs
- ativação formal de auditorias
- migração organizada do legado para `98-legacy/`
- consolidação de todos os impl-logs históricos passados
- validação fina no ambiente real de todos os paths, hostnames, unit names e inventários ainda pendentes
- publicação web da documentação

Regra importante:

- esta mudança cria a base canônica
- mas não encerra a reconciliação fina do ecossistema

---

## Ganhos obtidos

Os principais ganhos desta migração foram:

- separação clara entre infraestrutura, jogo, portal e backend dinâmico
- redução da dependência dos masters amplos
- criação de fronteiras documentais explícitas
- criação de runbooks e inventários por contexto
- redução de ambiguidade entre o que é Hostinger e o que é Lightsail
- formalização da Static API v2 como camada própria
- formalização da instância `MixHAXIXE01` como dependência estrutural
- formalização do MatchZy como componente de gameplay e de dados
- formalização da regra editorial de fatos parcialmente confirmados

---

## Riscos residuais após esta migração

Os riscos residuais ainda existentes incluem:

- alguns paths ainda precisam de confirmação direta no ambiente real
- hostname(s) públicos finais ainda precisam de reconciliação explícita em alguns pontos
- inventário completo de plugins ainda precisa de captura no runtime atual
- inventário final de presets/aliases do lado jogo ainda precisa de consolidação
- detalhes finos de units/timers ainda dependem de validação direta em host
- material legado ainda não foi oficialmente reorganizado em `98-legacy/`

Esses riscos não invalidam a migração.  
Eles apenas definem a próxima fase de manutenção e reconciliação.

---

## Próxima fase recomendada

A próxima fase natural após esta migração é:

### 1. ativar `95-impl-log/` como trilha viva do repositório

Este documento já inaugura essa fase.

### 2. ativar `97-audit/`

Para registrar:
- pendências reais
- gaps entre docs e runtime
- reconciliações ainda abertas

### 3. reorganizar material histórico em `98-legacy/`

Para:
- preservar os antigos masters e documentos amplos
- manter utilidade histórica
- remover o papel deles como navegação oficial viva

### 4. revisar contextos novos a partir do runtime real

Especialmente:
- hostnames finais
- nomes exatos de units/timers
- path final do `matchzy.db`
- inventário completo de plugins
- baseline/presets do servidor

---

## Leitura canônica deste impl-log

A leitura canônica deste registro deve ser:

- 2026-03-18 marca a formalização da base documental por contexto do HSC
- os 4 contextos canônicos iniciais passam a existir como estrutura navegável
- a governança documental deixa de ser implícita e passa a ser explícita
- o repositório entra em uma nova fase: manutenção incremental disciplinada

---

## Critério de pronto deste registro

Este impl-log pode ser considerado adequado quando:

- registrar claramente o que foi migrado
- deixar explícito o impacto estrutural da mudança
- servir como marco zero da camada `95-impl-log/`
- ajudar futuras revisões a entenderem quando o modelo por contexto passou a ser oficial

---

## Última revisão

- Status: ativo
- Classificação: impl-log
- Contexto: migração documental inicial
- Última revisão: 2026-03-18