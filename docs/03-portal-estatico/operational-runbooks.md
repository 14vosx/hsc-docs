# Operational Runbooks

## Objetivo

Documentar os procedimentos operacionais recorrentes do contexto Portal Estático do ecossistema HSC, com foco na geração da Static API v2, na publicação pública via Nginx, no mirror same-origin de conteúdo e nas validações mínimas que mantêm o portal íntegro no runtime real.

Este documento existe para registrar, de forma estável e auditável:

- como validar rapidamente se o portal e a v2 estão saudáveis
- como executar a pipeline agregada principal da v2
- como executar a geração do `health.json`
- como validar o mirror de `content/news`
- como diferenciar problema de geração, problema de publicação e problema de borda pública
- como reduzir dependência de memória informal na operação da camada pública

---

## Escopo

Este documento cobre:

- runbook de validação inicial da camada pública
- runbook da pipeline agregada da v2
- runbook de geração do `health.json`
- runbook de validação do portal público
- runbook de validação do mirror same-origin de News
- runbook de verificação de timers/services do host
- runbook de validação da fonte SQLite do portal
- regras de ouro operacionais da camada pública

Este documento não cobre em profundidade:

- configuração completa do Nginx linha a linha
- troubleshooting aprofundado do AMP
- troubleshooting aprofundado do servidor CS2
- configuração detalhada dos plugins
- Auth API no Lightsail
- contratos JSON campo a campo

Esses tópicos vivem em documentos próprios dos contextos `01-infra-hostinger`, `02-game-panel`, `03-portal-estatico` e `04-infra-aws-lightsail`.

---

## Estado atual

O estado operacional conhecido do Portal Estático exige que a operação prática seja pensada em camadas:

- Infra Hostinger saudável
- Nginx saudável
- base operacional `/opt/cs2-portal/` íntegra
- scripts `gen-*` íntegros
- timers e services principais do host saudáveis
- `matchzy.db` acessível no path canônico
- árvore pública `/var/www/api/cs2/v2/` íntegra
- portal público disponível em `/portal/cs2/`
- Static API v2 disponível em `/api/cs2/v2/`
- mirror same-origin disponível em `/content/news/`

Também já existe evidência reconciliada de que o heartbeat principal da camada é:

- `gen-all-v2.timer` / `gen-all-v2.service`
- `gen-health.timer` / `gen-health.service`

E que o comando agregador real da v2 é:

- `/usr/local/bin/gen-all-v2.sh`

Este documento assume que o portal deve ser operado de forma previsível e repetível.

---

## Source of truth / evidências

As principais evidências deste documento, nesta fase de migração documental, são:

- documentação consolidada atual do ecossistema HSC
- blueprint técnico consolidado do HSC
- documentação reconciliada da Infra Hostinger
- documentação reconciliada do Portal Estático
- runtime real da Hostinger
- timers e services reais do host
- paths públicos e internos reais reconciliados
- inventário real da base `/opt/cs2-portal/`
- path canônico vivo do `matchzy.db`

Enquanto a migração canônica do contexto não estiver concluída em nível de refinamento total, essas fontes seguem sendo usadas como base de reconciliação do estado real.

---

## Relações com outros documentos

Este arquivo é complementar a:

- `docs/03-portal-estatico/README.md`
- `docs/03-portal-estatico/portal-estatico-architecture-runtime.md`
- `docs/03-portal-estatico/static-api-v2.md`
- `docs/03-portal-estatico/data-sources-matchzy-sqlite.md`
- `docs/03-portal-estatico/etl-bash-pipeline.md`
- `docs/03-portal-estatico/nginx-publishing-cache.md`
- `docs/03-portal-estatico/observability-troubleshooting.md`
- `docs/03-portal-estatico/portal-estatico-references-inventory.md`
- `docs/01-infra-hostinger/systemd-automation.md`
- `docs/01-infra-hostinger/infra-hostinger-references-inventory.md`

Este documento descreve procedimentos recorrentes de operação da camada pública.  
Ele não substitui os documentos de topologia, ETL, publicação, inventário ou troubleshooting aprofundado.

---

## Princípios operacionais do contexto

Os runbooks deste contexto seguem estes princípios:

- nunca assumir que arquivo presente em disco significa publicação pública saudável
- nunca assumir que Nginx saudável significa pipeline saudável
- sempre validar a fonte antes de culpar a publicação
- sempre distinguir geração, publicação e consumo
- sempre preferir sequência explícita a improviso operacional
- sempre tratar `gen-all-v2` e `gen-health` como heartbeat principal da camada
- sempre validar hostnames e paths públicos reais do runtime

Regra de ouro:

**o Portal Estático só está saudável quando a fonte existe, a geração conclui, a árvore pública é atualizada e a borda responde nos hostnames canônicos**

---

## Runbook de validação inicial da camada pública

Este runbook deve ser usado:

- antes de mexer na v2
- após mudança em scripts `gen-*`
- após mudança em unit/timer do host
- após intervenção manual no Nginx
- quando o portal parecer stale
- quando houver dúvida sobre o estado real da camada pública

### Objetivo

Confirmar rapidamente que a camada pública do portal/v2 está operacional.

### Sequência lógica

1. validar Infra Hostinger
2. validar Nginx
3. validar timers/services principais
4. validar fonte SQLite
5. validar artefatos públicos
6. validar borda pública em `haxixesmokeclub.com`

### Validar hostnames públicos principais

```bash
curl -I https://haxixesmokeclub.com/
curl -I https://www.haxixesmokeclub.com/
```

### Validar portal público

```bash
curl -I https://haxixesmokeclub.com/portal/cs2/
curl -I https://www.haxixesmokeclub.com/portal/cs2/
```

### Validar health da v2

```bash
curl -I https://haxixesmokeclub.com/api/cs2/v2/health.json
curl -sS https://haxixesmokeclub.com/api/cs2/v2/health.json
```

### Critério de sucesso

A validação inicial só deve ser tratada como satisfatória quando:

- o host responde nos domínios canônicos
- o portal responde
- `health.json` responde
- não há sinal imediato de drift entre fonte, geração e publicação

---

## Runbook da pipeline agregada da v2

Este runbook deve ser usado:

- quando a v2 estiver stale
- após mudança relevante em scripts
- após incidentes de geração
- quando for necessário forçar atualização imediata da v2
- quando houver suspeita de falha em geração por etapas

### Objetivo

Executar e validar a pipeline agregada principal da Static API v2.

### Comando agregador canônico

```bash
/usr/local/bin/gen-all-v2.sh
```

### Contexto operacional reconciliado

A unit principal do host usa:

- `gen-all-v2.service`
- `User=amp`
- `Group=www-data`
- `WorkingDirectory=/var/www`

Lock principal reconciliado:

```bash
/opt/cs2-portal/locks/gen-all-v2.lock
```

### Etapas reconciliadas da pipeline

A execução agregada chama, em ordem reconciliada:

1. `gen-matches.sh`
2. `gen-ranking.sh`
3. `gen-players-incremental.sh`
4. `gen-players-from-ranking.sh`
5. `gen-maps.sh`

### Execução manual

Quando for necessário executar manualmente:

```bash
sudo -u amp /usr/local/bin/gen-all-v2.sh
```

Observação operacional:
- a unit usa `amp` com `Group=www-data`
- quando possível, operar respeitando esse contexto reduz risco de drift de permissão

### Critério de sucesso

A pipeline agregada é considerada bem-sucedida quando:

- conclui sem falha estrutural
- os artefatos públicos principais são atualizados
- `ranking.json`, `matches.json` e derivados continuam coerentes
- `health.json` deixa de acusar aging indevido logo após a geração

---

## Runbook de geração do `health.json`

Este runbook deve ser usado:

- quando o endpoint de health parecer stale
- após geração agregada
- após incidentes na v2
- quando houver suspeita de falha só na camada de observabilidade pública

### Objetivo

Gerar e validar o `health.json` da v2.

### Comando reconciliado

```bash
/usr/local/bin/gen-health.sh
```

### Unit reconciliada

- `gen-health.service`
- `User=amp`
- `Group=www-data`

### Execução manual

```bash
sudo -u amp /usr/local/bin/gen-health.sh
```

### Path público esperado

```text
/var/www/api/cs2/v2/health.json
```

### Validação pública

```bash
curl -I https://haxixesmokeclub.com/api/cs2/v2/health.json
curl -sS https://haxixesmokeclub.com/api/cs2/v2/health.json
```

### Critério de sucesso

A geração do `health.json` é considerada saudável quando:

- o arquivo existe na árvore pública
- o endpoint responde publicamente
- os checks internos de aging dos artefatos não indicam anomalia incompatível com o momento da geração

---

## Runbook de validação da fonte SQLite

Este runbook deve ser usado:

- antes de culpar ETL
- quando ranking, matches ou players estiverem stale
- após update do MatchZy
- após incidente no lado jogo
- quando houver suspeita de path errado da fonte

### Objetivo

Confirmar que a fonte primária da v2 continua íntegra.

### Path canônico vivo da fonte

```bash
/home/amp/.ampdata/instances/MixHAXIXE01/counter-strike2/730/game/csgo/addons/counterstrikesharp/plugins/MatchZy/matchzy.db
```

### Validar presença do banco

```bash
ls -l /home/amp/.ampdata/instances/MixHAXIXE01/counter-strike2/730/game/csgo/addons/counterstrikesharp/plugins/MatchZy/matchzy.db
realpath /home/amp/.ampdata/instances/MixHAXIXE01/counter-strike2/730/game/csgo/addons/counterstrikesharp/plugins/MatchZy/matchzy.db
```

### Validar tabelas principais

```bash
sqlite3 /home/amp/.ampdata/instances/MixHAXIXE01/counter-strike2/730/game/csgo/addons/counterstrikesharp/plugins/MatchZy/matchzy.db ".tables"
```

### Validar volume básico

```bash
sqlite3 /home/amp/.ampdata/instances/MixHAXIXE01/counter-strike2/730/game/csgo/addons/counterstrikesharp/plugins/MatchZy/matchzy.db "SELECT COUNT(*) FROM matchzy_stats_matches;"
sqlite3 /home/amp/.ampdata/instances/MixHAXIXE01/counter-strike2/730/game/csgo/addons/counterstrikesharp/plugins/MatchZy/matchzy.db "SELECT COUNT(*) FROM matchzy_stats_players;"
sqlite3 /home/amp/.ampdata/instances/MixHAXIXE01/counter-strike2/730/game/csgo/addons/counterstrikesharp/plugins/MatchZy/matchzy.db "SELECT COUNT(*) FROM matchzy_stats_maps;"
```

### Critério de sucesso

A fonte é considerada saudável quando:

- o banco existe no path canônico
- as tabelas principais existem
- o volume de dados evolui de forma coerente com a atividade do servidor

---

## Runbook de validação do portal público

Este runbook deve ser usado quando a suspeita principal recair sobre o frontend público.

### Objetivo

Confirmar que o portal está publicado corretamente em `/portal/cs2/`.

### Validação pública

```bash
curl -I https://haxixesmokeclub.com/portal/cs2/
curl -I https://www.haxixesmokeclub.com/portal/cs2/
```

### Validar assets estáveis

```bash
curl -I https://haxixesmokeclub.com/portal/cs2/assets/
```

### Validar rota de player

```bash
curl -I https://haxixesmokeclub.com/portal/cs2/player/
```

### Critério de sucesso

O portal é considerado saudável quando:

- responde no path canônico
- os assets são resolvidos pela borda
- a navegação SPA continua coerente com os fallbacks do Nginx

---

## Runbook de validação da Static API v2 pública

Este runbook deve ser usado quando a suspeita principal recair sobre a camada JSON pública.

### Objetivo

Confirmar que a v2 pública está publicada corretamente em `/api/cs2/v2/`.

### Validar endpoints principais

```bash
curl -I https://haxixesmokeclub.com/api/cs2/v2/health.json
curl -I https://haxixesmokeclub.com/api/cs2/v2/ranking.json
curl -I https://haxixesmokeclub.com/api/cs2/v2/matches.json
```

### Ler endpoints principais

```bash
curl -sS https://haxixesmokeclub.com/api/cs2/v2/health.json
curl -sS https://haxixesmokeclub.com/api/cs2/v2/ranking.json
curl -sS https://haxixesmokeclub.com/api/cs2/v2/matches.json
```

### Validar desativação da v1

```bash
curl -I https://haxixesmokeclub.com/api/cs2/v1/
```

### Validar redirects de compatibilidade

```bash
curl -I https://haxixesmokeclub.com/api/health.json
curl -I https://haxixesmokeclub.com/api/ranking.json
curl -I https://haxixesmokeclub.com/api/matches.json
```

### Critério de sucesso

A v2 pública é considerada saudável quando:

- os JSONs principais respondem
- a v1 continua desativada
- os endpoints antigos redirecionam conforme esperado
- a política pública da borda continua coerente com a arquitetura v2-only

---

## Runbook de validação do mirror same-origin de News

Este runbook deve ser usado quando houver suspeita específica em `content/news`.

### Objetivo

Confirmar que o mirror same-origin de conteúdo está íntegro.

### Validar lista de News

```bash
curl -I https://haxixesmokeclub.com/content/news/
curl -sS https://haxixesmokeclub.com/content/news/
```

### Validar redirects sem barra

O comportamento esperado é redirecionar:

- `/content/news` → `/content/news/`
- `/content/news/<slug>` → `/content/news/<slug>/`

Exemplo genérico:

```bash
curl -I https://haxixesmokeclub.com/content/news
```

### Critério de sucesso

O mirror é considerado saudável quando:

- a lista responde
- a normalização com barra funciona
- o comportamento do mirror não depende da Auth API online no momento da leitura pública, e sim do cache estático já gerado

---

## Runbook de verificação de timers e services do host

Este runbook deve ser usado quando houver suspeita de falha na automação.

### Objetivo

Confirmar que o heartbeat principal da camada pública continua ativo.

### Validar timers relevantes

```bash
systemctl list-timers --all --no-pager | grep -Ei 'gen|v2|health|portal|hsc'
```

### Validar units relevantes

```bash
systemctl list-unit-files --type=service --no-pager | grep -Ei 'gen|v2|health|portal|hsc'
systemctl list-unit-files --type=timer --no-pager | grep -Ei 'gen|v2|health|portal|hsc'
```

### Validar metadados principais das units

```bash
grep -RInE 'Description=|ExecStart=|WorkingDirectory=|User=|Group=|OnCalendar=|WantedBy=' /etc/systemd /lib/systemd/system 2>/dev/null | grep -Ei 'gen|v2|health|portal|hsc'
```

### Critério de sucesso

A automação é considerada saudável quando:

- `gen-health.timer` está ativo
- `gen-all-v2.timer` está ativo
- `gen-health.service` e `gen-all-v2.service` continuam apontando para os scripts corretos
- não há drift entre unit file e pipeline real do host

---

## Runbook de verificação da base operacional do portal

Este runbook deve ser usado quando houver suspeita de problema estrutural fora do Nginx.

### Objetivo

Confirmar que a base `/opt/cs2-portal/` continua íntegra.

### Validar estrutura base

```bash
ls -lah /opt/cs2-portal
find /opt/cs2-portal -maxdepth 2 -type f | sort
```

### Validar scripts auxiliares de content/news

```bash
ls -lah /opt/cs2-portal/bin
```

### Validar locks

```bash
ls -lah /opt/cs2-portal/locks
```

### Validar statefiles

```bash
ls -lah /opt/cs2-portal/state
```

### Critério de sucesso

A base operacional é considerada saudável quando:

- a árvore base existe
- locks e statefiles existem nos paths esperados
- a camada operacional do portal continua consistente com a automação reconciliada do host

---

## Regras de ouro

As regras de ouro operacionais deste contexto são:

1. nunca culpar Nginx antes de validar geração
2. nunca culpar ETL antes de validar o `matchzy.db`
3. sempre tratar `gen-all-v2.sh` como orquestrador principal reconciliado
4. sempre tratar `gen-health.sh` como heartbeat público mínimo da v2
5. sempre validar o domínio canônico real, não hostnames históricos inativos
6. nunca usar `/api/cs2/v1/` como referência viva
7. sempre distinguir portal, v2 e mirror de News como superfícies relacionadas, mas não idênticas
8. nunca tratar timer instalado como timer ativo sem validar `list-timers`

---

## Problemas comuns

### 1. Portal responde, mas JSON da v2 está stale

Causas comuns:

- falha em `gen-all-v2`
- fonte SQLite stale
- timer ativo, mas script com problema
- lock impedindo execução útil

---

### 2. `health.json` stale, mas resto parece saudável

Causas comuns:

- falha isolada em `gen-health`
- arquivo público não regenerado
- observabilidade da v2 degradada antes da v2 inteira

---

### 3. v2 parece saudável, mas `content/news` falha

Causas comuns:

- falha nas units de content/news
- ausência do cache gerado de News
- problema isolado do mirror, não da v2 inteira

---

### 4. JSON existe em disco, mas não responde publicamente

Causas comuns:

- problema de Nginx
- alias/root divergente
- teste feito em path antigo
- publicação em árvore diferente da esperada

---

### 5. Operador usa unit granular como se fosse heartbeat principal

Causas comuns:

- leitura antiga do host
- confusão entre units instaladas e timers ativos
- documentação stale

---

## Limites deste documento

Este documento não detalha:

- o conteúdo completo de cada script Bash
- troubleshooting profundo de SQL
- troubleshooting profundo do Nginx
- recuperação avançada de arquivos públicos corrompidos
- troubleshooting aprofundado do lado jogo
- operação da Auth API no Lightsail

Esses tópicos pertencem a documentos complementares.

---

## Critério de pronto deste documento

Este documento pode ser considerado maduro quando:

- os procedimentos principais da camada pública estiverem claros
- `gen-all-v2` e `gen-health` estiverem corretamente posicionados como heartbeat real do host
- a validação do portal, da v2 e do mirror same-origin estiver clara
- a separação entre fonte, geração e publicação estiver compreensível
- ele puder ser usado como guia prático de operação do Portal Estático sem depender do master legado

---

## Última revisão

- Status: ativo
- Classificação: canônico
- Contexto: portal estático / operational runbooks
- Última revisão: 2026-03-18