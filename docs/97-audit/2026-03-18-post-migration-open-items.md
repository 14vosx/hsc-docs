# Post-Migration Open Items Audit

## Objetivo

Registrar, de forma curta e honesta, as pendências reais que permaneceram após a migração inicial dos quatro contextos canônicos do sistema documental HSC.

Este documento separa:

- pendência operacional real
- refinamento documental opcional
- cleanup técnico ainda não executado

Ele não existe para reabrir a fase de migração estrutural.  
Ele existe para mostrar o que **ainda falta de verdade**.

---

## Estado do checkpoint

Após a reconciliação do runtime real da Hostinger, do Game Panel, do Portal Estático e do AWS Lightsail, o sistema documental já pode ser tratado como:

- canônico
- auditável
- operacionalmente confiável

Além disso, a pendência de maior risco operacional já foi encerrada:

- **restore prático do backup do MariaDB no Lightsail validado com sucesso em base temporária**

Isso reduz substancialmente o risco documental e operacional do contexto da Auth API.

---

## Itens que já foram fechados

Os itens abaixo, que antes apareciam como pendência, já podem ser tratados como **resolvidos**:

### 1. Hostnames públicos finais da Hostinger

Resolvido.

### 2. Path vivo real do `matchzy.db`

Resolvido.

### 3. Inventário real de plugins carregados no Game Panel

Resolvido.

### 4. Pipeline real da Static API v2

Resolvido.

### 5. Timers/services reais do host Hostinger

Resolvido.

### 6. Runtime real da Auth API no Lightsail via `systemd`

Resolvido.

### 7. Diretório real dos dumps do MariaDB no Lightsail

Resolvido.

### 8. Mecanismo exato de agendamento do backup no Lightsail

Resolvido.

### 9. Retenção real do backup no Lightsail

Resolvido.

### 10. Restore testado na prática no Lightsail

Resolvido.

Evidência resumida:
- dump restaurado com sucesso em `hsc_auth_restore_test`
- estrutura coerente com a produção
- comparação básica de volume coerente
- base temporária removida ao final

---

## Pendências reais ainda abertas

Neste checkpoint, restam poucas pendências reais.

### 1. Cleanup do drift residual da Auth API antiga na Hostinger

Ainda existe material residual no host Hostinger relacionado à Auth API, incluindo pelo menos:

- configuração/vhost antigo da borda `auth-api`
- unit residual `hsc-auth-api.service`

Status:
- identificado
- não crítico para a arquitetura atual
- ainda não limpo formalmente

Prioridade:
- média

---

### 2. Estratégia externa/off-host de backup da Auth API

O backup local do MariaDB no Lightsail já está reconciliado e validado.  
Ainda não foi confirmado neste ciclo se existe:

- cópia externa dos dumps
- retenção fora do host
- snapshot complementar com governança definida

Status:
- não confirmado
- não bloqueia o checkpoint
- continua sendo melhoria relevante de resiliência

Prioridade:
- média

---

### 3. Fotografia final completa do vhost público da Hostinger

Os blocos críticos do Nginx da Hostinger já foram reconciliados e a operação está confiável.  
Ainda pode haver valor em congelar uma fotografia final completa do vhost para auditoria futura.

Status:
- opcional
- refino documental
- não bloqueia operação

Prioridade:
- baixa

---

## Itens que não devem mais voltar como pendência

Os itens abaixo **não devem mais reaparecer** como “abertos”, salvo mudança real de runtime:

- hostnames reais do lado Hostinger
- runtime real do Node no Lightsail
- unit `hsc-auth-api.service`
- path `/opt/hsc/hsc-auth-api`
- binding observado em `0.0.0.0:3000`
- script real `/opt/hsc/backup-mariadb.sh`
- cron real `15 3 * * * /opt/hsc/backup-mariadb.sh`
- retenção real `14 dias`
- path vivo do `matchzy.db`
- inventário vivo de plugins
- pipeline real `gen-all-v2.sh`
- heartbeat `gen-all-v2.timer` + `gen-health.timer`

Esses pontos já devem ser tratados como parte da base reconciliada do sistema.

---

## Leitura correta do momento atual

A fase de migração documental inicial terminou.

O que resta agora é:

- cleanup técnico
- endurecimento de resiliência
- refino opcional de auditoria

Leitura canônica deste checkpoint:

**não há mais pendência estrutural grande aberta.**

---

## Critério de encerramento deste audit

Este audit pode ser tratado como suficientemente estável quando:

- os itens resolvidos continuarem resolvidos no runtime
- o drift residual da Hostinger for limpo ou conscientemente aceito por mais um ciclo
- a política externa/off-host de backup for confirmada ou explicitamente descartada

Até lá, este documento continua válido como retrato honesto do “restante real”.

---

## Última revisão

- Status: ativo
- Classificação: canônico
- Contexto: audit / open items
- Última revisão: 2026-03-18