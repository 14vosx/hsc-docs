# Legacy Master Documents Migration Map

## Objetivo

Registrar como os antigos documentos mestres do ecossistema HSC devem ser preservados dentro de `docs/98-legacy/` sem continuar governando a navegação principal do sistema documental novo.

Este documento existe para:

- identificar os dois masters históricos principais
- explicar qual é o novo papel deles no repositório
- mapear, em alto nível, quais contextos canônicos atuais substituem quais partes desses arquivos
- reduzir a chance de voltar a usar os masters antigos como índice vivo do ecossistema

---

## Documentos históricos cobertos

Os documentos históricos principais cobertos por este mapa são:

- `HSC_Master_Blueprint.md`
- `HSC_MASTER_DOCUMENTATION.md`

---

## Regra canônica

A partir da migração documental por contexto:

- esses arquivos passam a ser **material legado preservado**
- eles continuam úteis para reconciliação histórica
- eles **não** continuam como navegação principal
- eles **não** continuam como índice oficial do ecossistema
- eles **não** substituem os documentos canônicos por contexto

Regra importante:

**quando houver conflito entre um master legado e o sistema novo, prevalece o ambiente real validado e o documento canônico atual do contexto correto**

---

## Destino recomendado no repositório

Os dois arquivos históricos devem ser preservados em `docs/98-legacy/` mantendo nome reconhecível.

Destino recomendado:

- `docs/98-legacy/HSC_Master_Blueprint.md`
- `docs/98-legacy/HSC_MASTER_DOCUMENTATION.md`

Regra editorial:

- preservar nome original ajuda rastreabilidade
- o conteúdo histórico não precisa ser reescrito agora
- o que muda, nesta fase, é o **papel documental** desses arquivos

---

## Novo papel dos masters antigos

## `HSC_Master_Blueprint.md`

Novo papel:

- blueprint histórico do ecossistema
- memória de arquitetura ampla
- apoio de reconciliação
- apoio para entender decisões e fases anteriores do projeto

Não deve mais ser tratado como:

- índice mestre vivo
- fonte principal de operação diária
- substituto dos contextos canônicos

---

## `HSC_MASTER_DOCUMENTATION.md`

Novo papel:

- consolidação histórica ampla do ecossistema
- apoio para extração de detalhes ainda não migrados
- referência histórica de estrutura, naming, fluxos e fases anteriores

Não deve mais ser tratado como:

- master vivo principal
- fonte automática da verdade atual
- documento que “manda” no sistema novo

---

## Mapa de substituição por contexto

Este mapa mostra, em alto nível, qual contexto novo substitui qual tipo de assunto antes concentrado nos antigos masters.

## 1. Infraestrutura Hostinger

Tudo o que nos antigos masters tratava de:

- VPS Hostinger
- Debian
- Nginx do lado game + portal
- Docker host
- Certbot
- systemd e timers do host
- filesystem público e operacional
- paths e permissões do lado Hostinger

agora deve ser lido prioritariamente em:

- `docs/01-infra-hostinger/`

Documentos centrais de substituição:

- `docs/01-infra-hostinger/README.md`
- `docs/01-infra-hostinger/infra-hostinger-architecture-runtime.md`
- `docs/01-infra-hostinger/network-dns-tls.md`
- `docs/01-infra-hostinger/nginx-static-serving.md`
- `docs/01-infra-hostinger/docker-host.md`
- `docs/01-infra-hostinger/certbot.md`
- `docs/01-infra-hostinger/systemd-automation.md`
- `docs/01-infra-hostinger/filesystem-paths-permissions.md`
- `docs/01-infra-hostinger/observability-troubleshooting.md`
- `docs/01-infra-hostinger/infra-hostinger-references-inventory.md`

---

## 2. Game Panel / servidor CS2

Tudo o que nos antigos masters tratava de:

- AMP
- instância `MixHAXIXE01`
- servidor CS2
- MatchZy
- plugins
- presets e aliases
- runbooks de scrim, BO1, BO3, veto, coach, warmup
- persistência estatística do lado jogo

agora deve ser lido prioritariamente em:

- `docs/02-game-panel/`

Documentos centrais de substituição:

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

---

## 3. Portal Estático / Static API v2

Tudo o que nos antigos masters tratava de:

- portal público
- API estática
- ETL Bash
- `matchzy.db`
- ranking, matches, maps e players públicos
- contratos JSON
- frontend estático
- publishing do portal e da v2
- observabilidade do portal

agora deve ser lido prioritariamente em:

- `docs/03-portal-estatico/`

Documentos centrais de substituição:

- `docs/03-portal-estatico/README.md`
- `docs/03-portal-estatico/portal-estatico-architecture-runtime.md`
- `docs/03-portal-estatico/static-api-v2.md`
- `docs/03-portal-estatico/data-sources-matchzy-sqlite.md`
- `docs/03-portal-estatico/etl-bash-pipeline.md`
- `docs/03-portal-estatico/sql-queries-and-views.md`
- `docs/03-portal-estatico/json-contracts.md`
- `docs/03-portal-estatico/frontend-structure.md`
- `docs/03-portal-estatico/nginx-publishing-cache.md`
- `docs/03-portal-estatico/operational-runbooks.md`
- `docs/03-portal-estatico/observability-troubleshooting.md`
- `docs/03-portal-estatico/portal-estatico-references-inventory.md`

---

## 4. Auth API / AWS Lightsail

Tudo o que nos antigos masters tratava de:

- Auth API
- backend dinâmico
- Lightsail
- Ubuntu 22.04 LTS
- Nginx reverse proxy da API
- Node.js via systemd
- MariaDB local
- deploy
- rollback
- backup e restore
- observabilidade da API

agora deve ser lido prioritariamente em:

- `docs/04-infra-aws-lightsail/`

Documentos centrais de substituição:

- `docs/04-infra-aws-lightsail/README.md`
- `docs/04-infra-aws-lightsail/infra-aws-lightsail-architecture-runtime.md`
- `docs/04-infra-aws-lightsail/node-systemd.md`
- `docs/04-infra-aws-lightsail/deploy-release-rollback.md`
- `docs/04-infra-aws-lightsail/mariadb-local.md`
- `docs/04-infra-aws-lightsail/auth-api-operations.md`
- `docs/04-infra-aws-lightsail/network-dns-tls.md`
- `docs/04-infra-aws-lightsail/nginx-reverse-proxy.md`
- `docs/04-infra-aws-lightsail/observability-troubleshooting.md`
- `docs/04-infra-aws-lightsail/backup-restore.md`
- `docs/04-infra-aws-lightsail/infra-aws-lightsail-references-inventory.md`

---

## 5. Regras do sistema documental

Tudo o que nos antigos masters tratava, mesmo que indiretamente, de:

- “como a documentação deveria funcionar”
- “qual é o índice oficial”
- “qual é a navegação principal”
- “como separar verdade atual de material histórico”

agora deve ser lido prioritariamente em:

- `docs/00-governance/README.md`
- `docs/00-governance/documentation-system.md`
- `docs/00-governance/99-master-index.md`

---

## Como consultar os antigos masters corretamente

Ao consultar um dos dois masters históricos, a leitura correta deve ser:

1. usar o master legado para encontrar contexto histórico
2. identificar a qual contexto canônico atual aquele trecho pertence
3. consultar o documento novo correspondente
4. validar contra o runtime real, se necessário
5. só então decidir se há algo a reconciliar no canônico atual

Regra importante:

- o master legado pode apontar uma pista
- o contexto canônico atual é quem deve fechar a interpretação oficial

---

## Como lidar com informação ainda não migrada

Se algum trecho importante ainda existir apenas em um dos masters históricos, o fluxo correto é:

1. identificar o contexto canônico dono do assunto
2. validar a informação no ambiente real, quando necessário
3. migrar a parte útil para o documento canônico correto
4. registrar a mudança em impl-log, se fizer sentido
5. manter o master antigo preservado como histórico

Regra importante:

- o objetivo não é editar indefinidamente o master antigo
- o objetivo é migrar valor real para o sistema novo

---

## Sinais de regressão documental

Os sinais mais claros de regressão após esta migração seriam:

- voltar a usar `HSC_Master_Blueprint.md` como índice principal
- voltar a usar `HSC_MASTER_DOCUMENTATION.md` como “verdade viva” sem reconciliação
- atualizar primeiro o master antigo e só depois o canônico novo
- usar os masters antigos para decidir o estado atual sem olhar o runtime e o contexto correto
- deixar de atualizar o índice mestre e os contextos novos, mas continuar mexendo apenas nos consolidados antigos

Se isso acontecer, o sistema documental começa a regredir para o modelo anterior.

---

## Ação prática desta fase

A ação prática recomendada nesta fase é:

- preservar os dois arquivos dentro de `docs/98-legacy/`
- manter este mapa como ponte entre o modelo antigo e o modelo novo
- consultar os contexts canônicos como navegação principal daqui em diante

---

## Critério de pronto deste mapa

Este documento pode ser considerado adequado quando:

- os dois masters históricos estiverem claramente posicionados como legado
- ficar explícito qual contexto novo substitui qual tipo de assunto antigo
- qualquer pessoa consiga entender como consultar os antigos masters sem voltar a depender deles como navegação viva

---

## Última revisão

- Status: ativo
- Classificação: canônico
- Contexto: legado / mapa de migração dos masters antigos
- Última revisão: 2026-03-18e