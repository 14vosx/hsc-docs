# Audit — Post-Migration Open Items

## Objetivo

Registrar, de forma curta e explícita, as pendências reais identificadas logo após a migração documental inicial dos 4 contextos canônicos do ecossistema HSC.

Este documento existe para:

- transformar lacunas já conhecidas em backlog documental visível
- separar claramente o que já foi estruturado do que ainda precisa de reconciliação
- orientar as próximas revisões sem voltar à ambiguidade dos antigos masters
- servir como checklist inicial da fase de pós-migração

---

## Escopo

Esta auditoria cobre apenas pendências reais já identificadas durante a migração inicial.

Ela não tenta:

- reauditar todo o ecossistema
- reescrever documentos canônicos já criados
- abrir backlog funcional de produto
- substituir impl-logs ou runbooks

---

## Estado geral

A migração documental inicial foi concluída com sucesso em nível estrutural.

Os 4 contextos canônicos iniciais já estão criados e navegáveis:

1. Infra Hostinger
2. Game Panel
3. Portal Estático
4. Infra AWS Lightsail

O estado atual é:

- **estrutura canônica = pronta**
- **reconciliação fina de runtime = parcialmente concluída**
- **restam alguns itens de cleanup e ajuste documental pontual**

---

## Itens já resolvidos

## 1. Hostname público final do portal no lado Hostinger

### Status
- Resolvido

### Estado consolidado
Os hostnames públicos canônicos do lado portal são:

- `haxixesmokeclub.com`
- `www.haxixesmokeclub.com`

Os seguintes hostnames não devem ser tratados como ativos no estado atual:

- `portal.haxixesmokeclub.com`
- `api.haxixesmokeclub.com`

### Impacto documental
Este ponto já pode ser refletido em:

- `docs/01-infra-hostinger/network-dns-tls.md`
- `docs/01-infra-hostinger/references-inventory.md`
- `docs/03-portal-estatico/nginx-publishing-cache.md`

---

## 2. Path HTTP público final da Static API v2

### Status
- Resolvido

### Estado consolidado
A Static API v2 é servida publicamente em:

- `/api/cs2/v2/`

Também ficou reconciliado que:

- `/api/cs2/v1/` retorna `404`
- existem redirects de compat para endpoints antigos:
  - `/api/ranking.json`
  - `/api/matches.json`
  - `/api/health.json`

### Impacto documental
Este ponto já pode ser refletido em:

- `docs/03-portal-estatico/static-api-v2.md`
- `docs/03-portal-estatico/nginx-publishing-cache.md`
- `docs/03-portal-estatico/references-inventory.md`
- `docs/01-infra-hostinger/nginx-static-serving.md`

---

## 3. Hostname final da Auth API

### Status
- Resolvido

### Estado consolidado
A Auth API está canonicamente no Lightsail com:

- hostname: `auth-api.haxixesmokeclub.com`
- Nginx reverse proxy no próprio Lightsail
- `proxy_pass http://127.0.0.1:3000`

### Impacto documental
Este ponto já pode ser refletido em:

- `docs/04-infra-aws-lightsail/network-dns-tls.md`
- `docs/04-infra-aws-lightsail/nginx-reverse-proxy.md`
- `docs/04-infra-aws-lightsail/references-inventory.md`

---

## 4. Path final completo do `matchzy.db`

### Status
- Resolvido

### Estado consolidado
O path canônico vivo do banco é:

```bash
/home/amp/.ampdata/instances/MixHAXIXE01/counter-strike2/730/game/csgo/addons/counterstrikesharp/plugins/MatchZy/matchzy.db
```

Também foi encontrado um path de backup, que não deve ser tratado como fonte ativa:

```bash
/home/amp/.ampdata/instances/MixHAXIXE01/counter-strike2/730/game/csgo/backup-hsc/MatchZy.live-dir.pre-upstream.20260317-205843/matchzy.db
```

### Impacto documental
Este ponto já pode ser refletido em:

- `docs/02-game-panel/instance-mixhaxixe01.md`
- `docs/02-game-panel/matchzy.md`
- `docs/02-game-panel/references-inventory.md`
- `docs/03-portal-estatico/data-sources-matchzy-sqlite.md`
- `docs/03-portal-estatico/references-inventory.md`

---

## Pendências reais ainda abertas

## 1. Inventário completo de plugins em produção ainda não foi consolidado

### Contexto afetado

- `02-game-panel`

### Situação

Já está reconciliado que:

- MatchZy é central
- WeaponPaints já apareceu em produção
- `css_plugins list` é a fonte prática de verdade do runtime

O que ainda falta:

- capturar a lista completa atual de plugins carregados
- confirmar versões além do MatchZy
- distinguir quais plugins são:
  - competitivos
  - administrativos
  - cosméticos
  - auxiliares de persistência

### Ação recomendada

Capturar `css_plugins list` no runtime atual e revisar:

- `docs/02-game-panel/plugins-installed.md`
- `docs/02-game-panel/references-inventory.md`

---

## 2. Inventário real de presets e aliases do lado jogo ainda está incompleto

### Contexto afetado

- `02-game-panel`

### Situação

Já existe evidência suficiente de que:

- presets e aliases fazem parte da operação real
- `load_1v1.cfg` é exemplo reconciliado no histórico do ecossistema

O que ainda falta:

- consolidar quais presets estão realmente em uso hoje
- separar baseline do servidor de configs auxiliares
- reduzir dependência de memória informal do staff

### Ação recomendada

Revisar ambiente real e atualizar:

- `docs/02-game-panel/cs2-server-configuration.md`
- `docs/02-game-panel/operational-runbooks.md`
- `docs/02-game-panel/references-inventory.md`

---

## 3. Nomes exatos de services/timers ainda precisam de validação direta em host

### Contextos afetados

- `01-infra-hostinger`
- `03-portal-estatico`

### Situação

A intenção operacional já está correta e reconciliada:

- geração da v2
- geração de `health.json`
- automação via `systemd`

O que ainda falta:

- confirmar os nomes exatos das units e timers ativos em produção
- amarrar isso com precisão nos documentos

### Ação recomendada

Validar com `systemctl list-timers --all` e revisar:

- `docs/01-infra-hostinger/systemd-automation.md`
- `docs/01-infra-hostinger/references-inventory.md`
- `docs/03-portal-estatico/operational-runbooks.md`
- `docs/03-portal-estatico/references-inventory.md`

---

## 4. Comando agregador real de regeneração da v2 ainda não foi congelado

### Contexto afetado

- `03-portal-estatico`

### Situação

A ordem normativa da pipeline já foi documentada corretamente, mas ainda não foi congelado, como verdade canônica final:

- o comando agregador real usado no ambiente atual
- ou a unit/timer exata que materializa essa execução

### Ação recomendada

Validar no host e revisar:

- `docs/03-portal-estatico/etl-bash-pipeline.md`
- `docs/03-portal-estatico/operational-runbooks.md`
- `docs/03-portal-estatico/references-inventory.md`

---

## 5. Root/alias final dos vhosts públicos ainda precisa de confirmação fina

### Contextos afetados

- `01-infra-hostinger`
- `03-portal-estatico`

### Situação

Os paths de filesystem e os principais endpoints já estão reconciliados.  
O que ainda falta confirmar sem ambiguidade é:

- a forma final completa dos blocos de vhost relevantes no runtime real
- se existe algum detalhe residual de root/alias além do já validado

### Ação recomendada

Validar o host e revisar:

- `docs/01-infra-hostinger/nginx-static-serving.md`
- `docs/01-infra-hostinger/references-inventory.md`
- `docs/03-portal-estatico/nginx-publishing-cache.md`

---

## 6. Diretório final dos dumps do backup da Auth API ainda não está fixado

### Contexto afetado

- `04-infra-aws-lightsail`

### Situação

Já está reconciliado que:

- existe script de backup
- existe rotina diária
- existe retenção conhecida

O que ainda falta formalizar sem ambiguidade:

- diretório final dos dumps
- procedimento completo de restore validado contra o host atual

### Ação recomendada

Validar no host e revisar:

- `docs/04-infra-aws-lightsail/backup-restore.md`
- `docs/04-infra-aws-lightsail/references-inventory.md`

---

## 7. Vhost legado da Auth API ainda existe na Hostinger

### Contexto afetado

- `01-infra-hostinger`

### Situação

O runtime atual confirmou que a Auth API canônica está no Lightsail.

Mesmo assim, ainda existe configuração Nginx na Hostinger contendo:

- `server_name auth-api.haxixesmokeclub.com`

Isso deve ser tratado como:

- configuração legada / stale
- item de cleanup técnico
- não como prova de que a Auth API ainda roda canonicamente na Hostinger

### Ação recomendada

Registrar esse drift em:

- `docs/01-infra-hostinger/references-inventory.md`

E tratar o cleanup técnico em momento próprio do host.

---

## 8. Material legado ainda não foi formalmente reorganizado além do núcleo inicial

### Contexto afetado

- governança geral do repositório

### Situação

A nova base canônica já existe e os dois masters históricos principais já foram preservados em `98-legacy/`.

O que ainda pode evoluir:

- classificar outros documentos históricos relevantes
- expandir o uso disciplinado de `98-legacy/`

### Ação recomendada

Executar nova onda controlada de legado apenas quando necessário.

---

## 9. Diretórios `95-impl-log/` e `97-audit/` ainda precisam virar rotina viva

### Contexto afetado

- governança geral do repositório

### Situação

Já existe:

- primeiro impl-log
- primeira auditoria

O que ainda falta:

- transformar essas pastas em parte real do fluxo de manutenção
- evitar voltar ao padrão antigo de só revisar documentação em grandes ondas

### Ação recomendada

Adotar como rotina:

1. mudança real no runtime
2. impl-log curto
3. atualização do documento canônico correspondente
4. auditoria apenas quando surgir gap relevante

---

## Priorização recomendada

### Prioridade alta

1. inventário atual de plugins
2. units/timers reais do host
3. comando agregador real da v2

### Prioridade média

4. inventário de presets e aliases
5. root/alias final dos vhosts
6. diretório final dos dumps do backup

### Prioridade estrutural contínua

7. cleanup do vhost legado da Auth API na Hostinger
8. institucionalização de impl-log + audit como rotina
9. expansão controlada de `98-legacy/`

---

## Leitura canônica desta auditoria

A leitura correta deste documento é:

- a migração inicial foi bem-sucedida
- os itens mais críticos de hostname e path principal já foram reconciliados
- o backlog restante agora está menor, mais técnico e mais localizado
- a próxima fase é manutenção disciplinada, não reconstrução estrutural

---

## Critério de encerramento desta auditoria

Esta auditoria pode ser considerada encerrada quando:

- os itens de prioridade alta estiverem resolvidos ou redistribuídos em auditorias específicas
- os documentos canônicos afetados forem atualizados
- o backlog residual estiver menor e mais localizado por contexto

---

## Última revisão

- Status: ativo
- Classificação: audit
- Contexto: pendências pós-migração inicial
- Última revisão: 2026-03-18