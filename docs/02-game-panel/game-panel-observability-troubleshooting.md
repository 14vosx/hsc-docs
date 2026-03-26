# Observability and Troubleshooting

## Navegação rápida

- [Home da documentação](../README.md)
- [Game Panel](./README.md)
- [Master Index](../00-governance/99-master-index.md)

---
## Objetivo

Documentar os sinais de saúde, os pontos de observabilidade e a sequência padrão de diagnóstico do contexto Game Panel do ecossistema HSC.

Este documento existe para registrar, de forma estável e auditável:

- como validar a saúde do lado jogo
- como separar falhas de AMP, instância, runtime do CS2, MatchZy, plugins e persistência
- quais comandos operacionais são usados na triagem inicial
- quais incidentes são mais prováveis nessa camada
- como reduzir o tempo de diagnóstico em produção
- como evitar confundir problema do lado jogo com problema da Infra Hostinger ou do Portal Estático

---

## Escopo

Este documento cobre:

- sinais de saúde do Game Panel
- observabilidade da instância `MixHAXIXE01`
- observabilidade do runtime do servidor CS2
- observabilidade da camada MatchZy
- observabilidade da camada de plugins
- observabilidade da persistência estatística local
- sequência recomendada de diagnóstico
- interpretação de sintomas recorrentes
- falhas comuns do lado jogo

Este documento não cobre em profundidade:

- troubleshooting detalhado da Infra Hostinger
- troubleshooting aprofundado do ETL do Portal Estático
- contratos JSON da Static API v2
- troubleshooting da Auth API no Lightsail
- debugging profundo de código de plugin ou framework
- recuperação avançada multi-camada fora do escopo do lado jogo

Esses tópicos vivem em documentos próprios dos contextos `01-infra-hostinger`, `03-portal-estatico` e `04-infra-aws-lightsail`.

---

## Estado atual

O contexto `02-game-panel` possui, no estado operacional conhecido, as seguintes camadas críticas de observabilidade:

- AMP Instance Manager
- instância oficial `MixHAXIXE01`
- runtime do servidor Counter-Strike 2
- CounterStrikeSharp
- MatchZy `0.8.15`
- plugins auxiliares
- persistência estatística local via `matchzy.db`
- relação estrutural com o Portal Estático, que consome os dados produzidos pelo lado jogo

A observabilidade dessa camada é orientada principalmente por:

- presença estrutural da instância
- estado do runtime e da camada de plugins
- comportamento real da sessão competitiva
- uso de `css_plugins list`
- inspeção do `matchzy.db`
- validação indireta de que novas partidas geram novos dados

O lado jogo precisa ser analisado como sistema operacional completo.  
Um sintoma “no portal” pode nascer aqui.  
Um sintoma “no MatchZy” pode, na verdade, nascer em AMP, na instância ou na camada base.

---

## Source of truth / evidências

As principais evidências deste documento, nesta fase de migração documental, são:

- documentação consolidada atual do ecossistema HSC
- blueprint técnico consolidado do HSC
- documentação reconciliada da camada AMP/CS2
- documentação reconciliada do MatchZy e dos plugins
- histórico operacional do servidor ligado a scrim, BO1, BO3, veto, coach e warmup
- reconciliação da base estatística local usada pelo Portal Estático

Enquanto a migração canônica do contexto não estiver concluída, essas fontes seguem sendo usadas como base de reconciliação do estado real.

---

## Relações com outros documentos

Este arquivo é complementar a:

- `docs/02-game-panel/README.md`
- `docs/02-game-panel/game-panel-architecture-runtime.md`
- `docs/02-game-panel/amp-instance-manager.md`
- `docs/02-game-panel/instance-mixhaxixe01.md`
- `docs/02-game-panel/cs2-server-configuration.md`
- `docs/02-game-panel/matchzy.md`
- `docs/02-game-panel/plugins-installed.md`
- `docs/02-game-panel/game-panel-operational-runbooks.md`
- `docs/03-portal-estatico/data-sources-matchzy-sqlite.md`
- `docs/03-portal-estatico/portal-estatico-observability-troubleshooting.md`
- `docs/01-infra-hostinger/infra-hostinger-observability-troubleshooting.md`

Este documento descreve triagem, observabilidade e troubleshooting do lado jogo.  
Ele não substitui os documentos de runbook, MatchZy, plugins, configuração do servidor ou troubleshooting da infraestrutura base.

---

## Sinais de saúde do contexto

Os sinais de saúde esperados do Game Panel incluem:

- instância oficial `MixHAXIXE01` presente e coerente
- runtime do servidor CS2 operacional
- camada de plugins carregando
- MatchZy visível e funcional no runtime
- comandos e fluxos competitivos utilizáveis
- sessão real executável em scrim, BO1 ou BO3 sem improviso estrutural
- persistência estatística local sendo produzida
- tabelas principais do `matchzy.db` evoluindo com novas partidas
- ausência de falha estrutural recorrente na camada de administração da partida
- ausência de drift evidente entre o que o staff acha que está rodando e o que o servidor realmente está executando

A saúde do contexto deve ser avaliada em camadas, e não por um único indicador.

---

## Camadas de observabilidade

### Camada 1 — Substrate e instância

Objetivo:
- validar se o lado jogo possui base estrutural íntegra para existir

Sinais típicos:
- árvore da instância presente
- identidade `MixHAXIXE01` coerente
- container/workload relevante presente no host

---

### Camada 2 — Runtime do servidor

Objetivo:
- validar se o servidor CS2 realmente está operacional

Sinais típicos:
- servidor sobe
- sessão prática pode ser iniciada
- ambiente do jogo responde conforme o esperado

---

### Camada 3 — Camada de plugins

Objetivo:
- validar se a malha de plugins está carregada

Sinais típicos:
- `css_plugins list` responde coerentemente
- MatchZy aparece
- plugins auxiliares esperados aparecem

---

### Camada 4 — Fluxo competitivo

Objetivo:
- validar se o servidor não apenas sobe, mas opera competitivamente

Sinais típicos:
- scrim possível
- BO1 possível
- BO3 possível
- warmup previsível
- veto/coach/fallback admin operáveis quando o cenário exige

---

### Camada 5 — Persistência estatística

Objetivo:
- validar se o lado jogo continua produzindo a base de dados do ecossistema

Sinais típicos:
- `matchzy.db` existe
- tabelas principais existem
- contagem de registros cresce após partidas reais

---

### Camada 6 — Impacto cruzado no Portal Estático

Objetivo:
- entender se um problema do lado jogo está contaminando a camada pública

Sinais típicos:
- partidas acontecem, mas o portal fica stale
- MatchZy aparentemente roda, mas stats não evoluem
- schema/localização do banco divergiu

Essa camada existe para evitar troubleshooting isolado demais.

---

## Logs, artefatos e pontos principais de observação

Os pontos de observação mais relevantes do contexto incluem:

### Árvore da instância oficial

```bash
ls -lah /home/amp/.ampdata/instances/
ls -lah /home/amp/.ampdata/instances/MixHAXIXE01/
```

Uso principal:
- confirmar presença estrutural da instância
- detectar drift, ausência ou incoerência de árvore

---

### Runtime de container associado

```bash
docker ps --format "table {{.Names}}\t{{.Image}}\t{{.Ports}}"
docker ps -a --format "table {{.Names}}\t{{.Status}}\t{{.Image}}"
```

Uso principal:
- confirmar presença do workload relevante
- detectar container ausente ou em estado incorreto

---

### Camada de plugins

```bash
css_plugins list
```

Uso principal:
- inventário real do runtime
- confirmar presença de MatchZy
- confirmar presença de plugins auxiliares reconciliados

---

### Persistência estatística local

Substitua pelo path real reconciliado do banco.

```bash
ls -l /CAMINHO/DO/matchzy.db
sqlite3 /CAMINHO/DO/matchzy.db ".tables"
```

Uso principal:
- validar base estatística
- detectar ausência do banco
- detectar ausência das tabelas esperadas

---

### Volume básico das tabelas principais

Substitua pelo path real reconciliado do banco.

```bash
sqlite3 /CAMINHO/DO/matchzy.db "SELECT COUNT(*) FROM matchzy_stats_matches;"
sqlite3 /CAMINHO/DO/matchzy.db "SELECT COUNT(*) FROM matchzy_stats_players;"
sqlite3 /CAMINHO/DO/matchzy.db "SELECT COUNT(*) FROM matchzy_stats_maps;"
```

Uso principal:
- verificar se a persistência evolui
- correlacionar partidas reais com crescimento estatístico

---

## Smoke checks

Os smoke checks mínimos do contexto devem incluir:

### Validar a presença estrutural da instância

```bash
ls -lah /home/amp/.ampdata/instances/MixHAXIXE01/
```

### Validar presença do workload do lado jogo

```bash
docker ps --format "table {{.Names}}\t{{.Image}}\t{{.Ports}}"
```

### Validar plugins carregados

```bash
css_plugins list
```

### Validar a presença do `matchzy.db`

Substitua pelo path real reconciliado.

```bash
ls -l /CAMINHO/DO/matchzy.db
```

### Validar tabelas principais

```bash
sqlite3 /CAMINHO/DO/matchzy.db ".tables"
```

### Validar volume estatístico mínimo

```bash
sqlite3 /CAMINHO/DO/matchzy.db "SELECT COUNT(*) FROM matchzy_stats_matches;"
```

### Validar operação prática mínima

Em cenário real:
- confirmar que o servidor pode entrar em warmup e seguir para fluxo competitivo esperado
- confirmar que o staff não está bloqueado por ausência de MatchZy, plugin ou admin funcional

---

## Sequência de diagnóstico

Quando houver incidente no lado jogo, usar a seguinte ordem.

### Etapa 1 — Confirmar base estrutural

1. validar Infra Hostinger
2. validar Docker host
3. validar presença da instância oficial

Comandos típicos:

```bash
sudo systemctl status docker
docker ps --format "table {{.Names}}\t{{.Image}}\t{{.Ports}}"
ls -lah /home/amp/.ampdata/instances/MixHAXIXE01/
```

---

### Etapa 2 — Confirmar runtime do lado jogo

1. validar que o workload real associado à instância está presente
2. confirmar que o servidor não está em estado estruturalmente ausente ou morto
3. só depois escalar para MatchZy ou plugins

Comandos típicos:

```bash
docker ps -a --format "table {{.Names}}\t{{.Status}}\t{{.Image}}"
```

Observação:
- o diagnóstico do runtime real não deve depender só do painel
- o host continua sendo fonte importante de validação

---

### Etapa 3 — Confirmar camada de plugins

1. executar `css_plugins list`
2. confirmar MatchZy
3. confirmar plugins auxiliares relevantes
4. detectar ausência, incompatibilidade ou inventário inesperado

Comandos típicos:

```bash
css_plugins list
```

---

### Etapa 4 — Confirmar fluxo competitivo

1. verificar se a sessão real consegue seguir o flow esperado
2. diferenciar falha técnica de falha operacional humana
3. validar warmup, início de scrim, BO1 ou BO3 conforme o cenário

Essa etapa depende de observação prática do comportamento do servidor e do staff.

---

### Etapa 5 — Confirmar persistência estatística

1. localizar o banco
2. validar existência do banco
3. validar tabelas principais
4. validar se o volume de dados evolui

Comandos típicos:

```bash
ls -l /CAMINHO/DO/matchzy.db
sqlite3 /CAMINHO/DO/matchzy.db ".tables"
sqlite3 /CAMINHO/DO/matchzy.db "SELECT COUNT(*) FROM matchzy_stats_matches;"
```

---

### Etapa 6 — Confirmar impacto cruzado no portal

Se o jogo parece saudável, mas o portal está stale:

1. validar se o banco realmente evoluiu
2. validar se o path do banco continua o esperado
3. só depois escalar para ETL e Portal Estático

Essa etapa evita atribuir ao portal um problema cuja raiz nasceu no lado jogo.

---

## Falhas comuns

### AMP/instância parecem saudáveis, mas o jogo não opera corretamente

Sintomas:

- árvore da instância existe
- substrate parece saudável
- sessão prática continua ruim

Causas comuns:

- problema no runtime do CS2
- MatchZy degradado
- plugin em conflito
- configuração divergente do modo esperado

Impacto:
- troubleshooting precisa sair da infraestrutura e entrar no runtime do jogo

---

### `css_plugins list` não mostra MatchZy

Sintomas:

- MatchZy não aparece no inventário do runtime
- fluxo competitivo não opera como esperado

Causas comuns:

- plugin não carregou
- problema na base CounterStrikeSharp
- incompatibilidade do runtime
- erro estrutural do lado plugins

Impacto:
- sessão competitiva degrada imediatamente
- persistência estatística pode deixar de evoluir

---

### MatchZy aparece, mas a sessão continua ruim

Sintomas:

- plugin listado
- flow real continua inadequado
- admins não conseguem operar como esperado

Causas comuns:

- configuração errada do servidor
- warmup/transição ruins
- modo operacional divergente
- problema de permissão/admin
- runbook não seguido

Impacto:
- falsa sensação de que “o plugin está ok”
- problema real continua no fluxo prático

---

### Partida acontece, mas `matchzy.db` não evolui

Sintomas:

- jogo ocorre normalmente
- contagem de registros não cresce
- portal depois fica stale

Causas comuns:

- problema de escrita
- problema de persistência do MatchZy
- path incorreto do banco
- drift de instância

Impacto:
- camada pública do ecossistema para de refletir a operação real

---

### Instância foi alterada e o portal quebra depois

Sintomas:

- lado jogo parece ter mudado pouco
- ETL ou portal deixam de consumir dados corretamente

Causas comuns:

- renomeação de instância
- path do banco mudou
- árvore da instância mudou sem reconciliação

Impacto:
- incidente multi-contexto
- a raiz está no lado jogo/instância, mas a falha emerge no portal

---

### Plugins carregam, mas inventário real diverge da documentação

Sintomas:

- `css_plugins list` mostra algo diferente do esperado
- runbooks ou expectativas do time não batem com o ambiente

Causas comuns:

- plugin foi adicionado ou removido sem atualização documental
- documentação ficou stale
- ambiente real mudou silenciosamente

Impacto:
- troubleshooting mais lento
- operação mais dependente de memória informal

---

### Sessão trava por ausência de admin funcional

Sintomas:

- servidor sobe
- MatchZy aparece
- operação da sessão trava mesmo assim

Causas comuns:

- ausência de admin com permissão adequada
- fallback admin não preparado
- dependência excessiva de uma pessoa específica
- runbook não institucionalizado

Impacto:
- problema não é apenas técnico
- a saúde operacional do servidor degrada do mesmo jeito

---

## Interpretação de sintomas

Alguns padrões úteis de interpretação:

### Docker/instância falham + plugins nem chegam a ser avaliados
Provável problema abaixo do MatchZy:
- substrate
- instância
- runtime base

### Instância existe + MatchZy não aparece
Provável problema de camada de plugins:
- CounterStrikeSharp
- carregamento
- compatibilidade

### MatchZy aparece + sessão continua ruim
Provável problema de:
- configuração
- runbook
- permissão/admin
- fluxo real de partida

### Partida ocorre + banco não evolui
Provável problema de:
- persistência
- MatchZy parcialmente degradado
- árvore/path da instância

### Banco evolui + portal não atualiza
Provável problema fora do Game Panel:
- ETL
- publishing
- Portal Estático

### Portal stale + jogo também sem gerar novas stats
Provável raiz no lado jogo:
- MatchZy
- instância
- persistência local

---

## Lacunas atuais de observabilidade

As lacunas conhecidas ou prováveis desta camada incluem:

- observabilidade ainda fortemente manual
- dependência alta de `css_plugins list`, inspeção de banco e comportamento real da sessão
- ausência, até onde reconciliado, de stack formal de métricas e alertas dedicada ao lado jogo
- parte importante da validação ainda depende de interpretação humana do flow competitivo
- risco de confundir falha operacional humana com falha técnica do servidor

Essas lacunas explicam por que a sequência de diagnóstico precisa ser explícita.

---

## Regras de ouro de troubleshooting

As regras de ouro deste contexto são:

1. nunca assumir que a sessão está pronta só porque o servidor sobe
2. nunca assumir que MatchZy carregado basta por si só
3. sempre validar a instância `MixHAXIXE01` antes de aprofundar em plugin
4. sempre usar `css_plugins list` como fotografia do runtime real
5. sempre validar persistência estatística quando o portal parecer stale
6. sempre separar problema de plugin de problema de runbook/admin
7. nunca culpar o portal antes de confirmar se o lado jogo continua produzindo dados
8. nunca tratar renomeação ou drift da instância como detalhe cosmético

---

## Limites deste documento

Este documento não detalha:

- debugging profundo do MatchZy
- todos os comandos exatos de cada cenário de BO1, BO3, veto ou coach
- configuração linha a linha do servidor
- toda a árvore interna da instância
- troubleshooting aprofundado da Infra Hostinger
- troubleshooting aprofundado do Portal Estático

Esses tópicos pertencem a documentos complementares.

---

## Critério de pronto deste documento

Este documento pode ser considerado maduro quando:

- os sinais de saúde do lado jogo estiverem alinhados com a prática real
- a sequência de diagnóstico refletir o uso operacional real do servidor
- as falhas comuns cobrirem os incidentes mais prováveis do contexto
- os comandos aqui descritos forem suficientes para triagem inicial sem depender do master legado
- a distinção entre falha estrutural, falha de plugin, falha de fluxo competitivo e falha de persistência estiver clara
- ele puder ser usado como runbook real de triagem do Game Panel

---

## Última revisão

- Status: ativo
- Classificação: canônico
- Contexto: game panel / observability and troubleshooting
- Última revisão: 2026-03-18