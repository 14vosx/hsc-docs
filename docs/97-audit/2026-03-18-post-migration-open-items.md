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

O estado atual, porém, ainda inclui pendências de reconciliação fina com o ambiente real.

Leitura correta do momento atual:

- **estrutura canônica = pronta**
- **reconciliação fina com runtime = ainda em andamento**

---

## Pendências reais abertas

## 1. Hostnames públicos finais ainda precisam de fixação explícita

### Contextos afetados

- `01-infra-hostinger`
- `03-portal-estatico`
- `04-infra-aws-lightsail`

### Situação

A topologia geral está correta, mas ainda faltam confirmações explícitas no ambiente real para congelar sem ambiguidade:

- hostname público oficial do lado Hostinger
- path HTTP público final da Static API v2
- hostname público final da Auth API no Lightsail

### Ação recomendada

Validar no ambiente real e atualizar:

- `network-dns-tls.md`
- `references-inventory.md`

---

## 2. Path final completo do `matchzy.db` ainda precisa ser fixado

### Contextos afetados

- `02-game-panel`
- `03-portal-estatico`

### Situação

Já está reconciliado que:

- o banco estrutural principal é o `matchzy.db`
- ele vive sob a árvore da instância `MixHAXIXE01`

O que ainda falta congelar com precisão canônica é:

- o path completo final do arquivo no ambiente real

### Ação recomendada

Validar no host e atualizar:

- `docs/02-game-panel/references-inventory.md`
- `docs/03-portal-estatico/data-sources-matchzy-sqlite.md`
- `docs/03-portal-estatico/references-inventory.md`

---

## 3. Inventário completo de plugins em produção ainda não foi consolidado

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

## 4. Inventário real de presets e aliases do lado jogo ainda está incompleto

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

## 5. Nomes exatos de services/timers ainda precisam de validação direta em host

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

## 6. Comando agregador real de regeneração da v2 ainda não foi congelado

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

## 7. Root/alias final dos vhosts públicos ainda precisa de confirmação fina

### Contextos afetados

- `01-infra-hostinger`
- `03-portal-estatico`

### Situação

Os paths de filesystem já estão reconciliados.  
O que ainda falta confirmar sem ambiguidade é:

- como o Nginx aponta exatamente para esses paths no runtime real
- se o path público final da v2 está coerente com essa configuração

### Ação recomendada

Validar o host e revisar:

- `docs/01-infra-hostinger/nginx-static-serving.md`
- `docs/01-infra-hostinger/references-inventory.md`
- `docs/03-portal-estatico/nginx-publishing-cache.md`

---

## 8. Diretório final dos dumps do backup da Auth API ainda não está fixado

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

## 9. Material legado ainda não foi formalmente reorganizado em `98-legacy/`

### Contexto afetado

- governança geral do repositório

### Situação

A nova base canônica já existe, mas os antigos masters e documentos históricos ainda não passaram por uma etapa formal de:

- seleção
- classificação
- realocação para `98-legacy/`

### Ação recomendada

Executar uma fase própria de legado para:

- preservar histórico útil
- reduzir ambiguidade de navegação
- evitar que arquivos antigos continuem sendo tratados como canônico vivo

---

## 10. Diretórios `95-impl-log/` e `97-audit/` acabaram de ser inaugurados e ainda precisam virar rotina

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

1. hostnames públicos finais
2. path final do `matchzy.db`
3. inventário atual de plugins
4. units/timers reais do host
5. comando agregador real da v2

### Prioridade média

6. inventário de presets e aliases
7. root/alias final dos vhosts
8. diretório final dos dumps do backup

### Prioridade estrutural contínua

9. reorganização do legado
10. institucionalização de impl-log + audit como rotina

---

## Leitura canônica desta auditoria

A leitura correta deste documento é:

- a migração inicial foi bem-sucedida
- as pendências agora estão visíveis
- nenhuma dessas pendências invalida a nova base canônica
- a próxima fase é reconciliação fina, não recomeço estrutural

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