# Instance `MixHAXIXE01`

## Objetivo

Documentar a instância oficial `MixHAXIXE01` do ecossistema HSC, registrando seu papel estrutural no lado jogo, sua posição na topologia operacional e sua relevância para a cadeia de dados públicos do projeto.

Este documento existe para registrar, de forma estável e auditável:

- por que `MixHAXIXE01` é a instância canônica do ecossistema
- como ela se relaciona com AMP, CS2, MatchZy e plugins
- quais dependências operacionais e de path passam por essa instância
- por que o nome e a árvore da instância não são detalhes cosméticos
- quais riscos surgem quando a instância diverge do estado esperado
- quais validações mínimas ajudam a manter essa camada íntegra

---

## Escopo

Este documento cobre:

- o papel estrutural da instância `MixHAXIXE01`
- a relação entre a instância e o AMP
- a relação entre a instância e o runtime do servidor CS2
- a relação entre a instância e a persistência local do lado game
- a relação entre a instância e a cadeia pública de dados do portal
- a árvore operacional da instância em alto nível
- riscos e problemas comuns ligados a drift de instância

Este documento não cobre em profundidade:

- a Infra Hostinger como substrate
- Docker host em detalhe
- configuração linha a linha do servidor CS2
- MatchZy em profundidade
- inventário completo de plugins
- pipeline ETL da Static API v2
- Auth API no AWS Lightsail

Esses tópicos vivem em documentos próprios dos contextos `01-infra-hostinger`, `02-game-panel` e `03-portal-estatico`.

---

## Estado atual

O estado operacional conhecido desta camada é:

- a instância oficial do lado game é `MixHAXIXE01`
- ela é a unidade operacional principal do servidor CS2 no ecossistema HSC
- ela existe sob a árvore `amp` do host
- ela é gerida pelo AMP Instance Manager
- ela sustenta o runtime do servidor CS2
- ela sustenta a presença estrutural do `matchzy.db`
- o Portal Estático depende indiretamente dessa instância porque a fonte estatística pública nasce em sua árvore operacional

Isso significa que:

- `MixHAXIXE01` é parte da verdade estrutural do ecossistema
- seu nome, path e árvore operacional têm impacto além do próprio lado jogo

---

## Source of truth / evidências

As principais evidências deste documento, nesta fase de migração documental, são:

- documentação consolidada atual do ecossistema HSC
- blueprint técnico consolidado do HSC
- documentação reconciliada da camada AMP/CS2
- reconciliação estrutural da instância oficial `MixHAXIXE01`
- reconciliação do path da árvore AMP
- reconciliação da dependência do `matchzy.db` em relação à instância

Enquanto a migração canônica do contexto não estiver concluída, essas fontes seguem sendo usadas como base de reconciliação do estado real.

---

## Relações com outros documentos

Este arquivo é complementar a:

- `docs/02-game-panel/README.md`
- `docs/02-game-panel/game-panel-architecture-runtime.md`
- `docs/02-game-panel/amp-instance-manager.md`
- `docs/02-game-panel/cs2-server-configuration.md`
- `docs/02-game-panel/matchzy.md`
- `docs/02-game-panel/plugins-installed.md`
- `docs/02-game-panel/operational-runbooks.md`
- `docs/02-game-panel/observability-troubleshooting.md`
- `docs/03-portal-estatico/data-sources-matchzy-sqlite.md`

Este documento descreve a instância oficial do lado jogo.  
Ele não substitui os documentos de AMP, MatchZy, configuração do servidor ou Portal Estático.

---

## Papel da instância no ecossistema

A instância `MixHAXIXE01` é a unidade concreta do lado jogo no HSC.

Seu papel inclui:

- materializar o workload principal do servidor CS2
- concentrar a árvore operacional do servidor
- servir como referência estrutural do lado game
- sustentar a persistência local ligada ao runtime
- servir como origem indireta da base de dados estatística pública do ecossistema

Em termos arquiteturais:

- Infra Hostinger sustenta o substrate
- AMP gerencia a instância
- `MixHAXIXE01` é a unidade operacional concreta
- sobre ela roda o CS2 com MatchZy e plugins

---

## Por que `MixHAXIXE01` é estrutural

O nome `MixHAXIXE01` não é apenas um rótulo visual.

Ele é estrutural porque:

- aparece na árvore de diretórios do lado `amp`
- serve como âncora para localizar artefatos do runtime do jogo
- participa do path onde vive o `matchzy.db`
- é usado como identidade operacional da instância principal do ecossistema
- seu drift pode quebrar dependências indiretas fora do lado jogo

Regra canônica:

- renomear, mover ou recriar a instância sem reconciliação documental e operacional pode gerar impacto multi-contexto

---

## Relação com o AMP

A relação com o AMP é direta e central.

O AMP existe, na prática, para administrar a instância `MixHAXIXE01`.

Essa relação inclui:

- gestão da instância
- preservação da identidade operacional do lado game
- manutenção do contexto onde o servidor CS2 opera
- ligação entre painel administrativo e árvore real da instância no host

Regra importante:

- AMP e instância não são sinônimos
- o AMP é a camada de gestão
- `MixHAXIXE01` é a unidade operacional gerida

---

## Relação com o runtime do CS2

A instância `MixHAXIXE01` sustenta o runtime real do servidor CS2 do ecossistema.

Essa relação inclui:

- arquivos e estrutura do servidor
- configs do runtime
- integração com CounterStrikeSharp
- integração com MatchZy
- integração com plugins instalados
- persistência local produzida durante a operação do jogo

Se a instância estiver incoerente, o servidor pode:

- não subir corretamente
- subir com configuração divergente
- produzir dados incorretos
- ou operar de forma inconsistente com os runbooks esperados

---

## Relação com MatchZy

O MatchZy depende estruturalmente da instância porque seu runtime e sua persistência vivem dentro do contexto materializado por `MixHAXIXE01`.

Essa relação inclui:

- presença do plugin no runtime correto
- gravação do banco SQLite do lado game
- aderência ao fluxo competitivo do servidor
- produção de estatísticas que depois alimentam o portal

Regra importante:

- MatchZy não existe “no abstrato” do host
- ele existe no contexto real da instância

---

## Relação com plugins

Os plugins carregados em produção dependem da integridade da instância para operar corretamente.

Essa relação inclui:

- carregamento do plugin no runtime correto
- acesso aos paths e assets corretos
- convivência com CounterStrikeSharp e MatchZy
- persistência auxiliar quando aplicável

Se a instância estiver em drift, plugins podem:

- deixar de carregar
- carregar em contexto incorreto
- perder acesso a configs ou paths
- ou produzir comportamento inconsistente

---

## Relação com a persistência local

A instância é o contexto estrutural que sustenta a persistência local do lado game.

O artefato mais importante, do ponto de vista do ecossistema mais amplo, é:

- `matchzy.db`

Esse banco é importante porque:

- reflete as estatísticas produzidas pelo lado jogo
- é lido pelo ETL do Portal Estático
- conecta o runtime do servidor à camada pública de dados

Regra canônica:

- a persistência estatística pública depende da integridade estrutural da instância

---

## Relação com o Portal Estático

O Portal Estático depende indiretamente de `MixHAXIXE01`.

Essa dependência existe porque:

- o ETL do portal precisa localizar o `matchzy.db`
- o path do banco nasce dentro da árvore da instância
- o nome da instância participa do path estrutural
- qualquer drift dessa árvore pode quebrar a geração da v2

Isso significa que a instância tem impacto fora do contexto `02-game-panel`.

---

## Árvore operacional em alto nível

A instância vive sob a base estrutural:

- `/home/amp/.ampdata/instances/`

A referência estrutural reconciliada é:

- `/home/amp/.ampdata/instances/MixHAXIXE01/`

Regra importante:

- este documento reconhece a árvore estrutural da instância como canônica em alto nível
- paths internos mais detalhados do runtime podem ser documentados em arquivos específicos do contexto, conforme validação adicional do host

---

## Dependências ocultas de path

As dependências ocultas mais importantes da instância incluem:

- localização do `matchzy.db`
- localização de configs do lado game
- localização de arquivos do servidor
- pressupostos operacionais de scripts, plugins e troubleshooting
- expectativas humanas dos operadores sobre onde os artefatos vivem

Essas dependências costumam passar despercebidas até ocorrer:

- renomeação
- migração
- rebuild
- ajuste manual de árvore
- ou intervenção operacional não documentada

---

## Dependências operacionais

A instância `MixHAXIXE01` depende, no mínimo, de:

- Infra Hostinger saudável
- Docker host funcional
- AMP funcional
- árvore `amp` íntegra
- runtime CS2 funcional
- CounterStrikeSharp funcional
- MatchZy funcional
- plugins coerentes com o ambiente
- paths e permissões corretos
- operators/admins usando runbooks alinhados ao estado real

Essas dependências explicam por que a instância precisa de documentação própria e explícita.

---

## Invariantes operacionais

Os invariantes conhecidos desta camada incluem:

- a instância oficial é `MixHAXIXE01`
- a instância vive sob a árvore `amp`
- o runtime do servidor CS2 do ecossistema depende dela
- o `matchzy.db` depende estruturalmente dessa árvore
- o Portal Estático depende indiretamente dela
- renomeação ou drift da instância é mudança estrutural real, não detalhe administrativo
- troubleshooting do lado jogo deve sempre considerar se a instância real corresponde ao que os operadores acreditam que existe

Esses invariantes ajudam a preservar estabilidade operacional e documental.

---

## Sinais de saúde da instância

Os sinais de saúde esperados incluem:

- existência da árvore da instância no path esperado
- coerência entre AMP e a instância real no filesystem
- runtime do servidor associado funcionando
- MatchZy operando no contexto correto
- persistência local sendo produzida
- ausência de drift óbvio entre o nome da instância e o uso real que o ecossistema faz dela

A saúde da instância não deve ser inferida apenas por ela “aparecer” no painel.  
Ela precisa estar alinhada ao runtime real e à sua árvore operacional.

---

## Comandos de validação

Os comandos abaixo representam verificações típicas da instância.

### Verificar existência da base estrutural das instâncias

```bash
ls -lah /home/amp/.ampdata/instances/
```

### Verificar existência da instância oficial

```bash
ls -lah /home/amp/.ampdata/instances/MixHAXIXE01/
```

### Verificar presença do container relevante no host

```bash
docker ps --format "table {{.Names}}\t{{.Image}}\t{{.Ports}}"
```

### Verificar se a identidade da instância continua aparecendo no runtime do host

Exemplo representativo:

```bash
docker ps -a --format "table {{.Names}}\t{{.Status}}\t{{.Image}}" | grep MixHAXIXE01
```

### Verificar a árvore da instância em nível estrutural

Exemplo representativo:

```bash
find /home/amp/.ampdata/instances/MixHAXIXE01 -maxdepth 2 -type d | sort
```

Regra importante:

- esses comandos validam a existência e a coerência estrutural da instância
- a validação funcional do jogo continua pertencendo aos runbooks e ao troubleshooting do contexto

---

## Problemas comuns

### 1. Instância existe, mas o runtime não corresponde

Causas comuns:

- drift entre AMP e filesystem
- reconstrução incompleta
- configuração divergente
- runtime apontando para estado diferente do esperado

Impacto:
- o servidor pode parecer “presente”, mas operar de forma errada

---

### 2. Instância foi renomeada ou movida

Causas comuns:

- reorganização manual
- rebuild não reconciliado
- intervenção operacional não documentada

Impacto:
- quebra do lado game
- quebra da localização do `matchzy.db`
- impacto indireto no Portal Estático

---

### 3. Instância íntegra, mas persistência do MatchZy não evolui

Causas comuns:

- problema do plugin
- problema do runtime do jogo
- problema de gravação na árvore da instância

Impacto:
- o servidor pode continuar rodando
- mas a camada pública de dados deixa de refletir novas partidas

---

### 4. AMP mostra a instância, mas o host não reflete a expectativa

Causas comuns:

- painel e filesystem em drift
- estado parcialmente reconstruído
- troubleshooting baseado só na UI do painel

Impacto:
- diagnóstico errado
- correções no lugar errado
- aumento do risco operacional

---

### 5. Dependência indireta do portal quebra silenciosamente

Causas comuns:

- mudança de árvore da instância
- mudança de path do banco
- renomeação não reconciliada

Impacto:
- o jogo pode continuar operando
- o ETL do portal quebra depois

---

## Sequência básica de diagnóstico

Quando houver suspeita de problema ligado à instância, a sequência recomendada é:

1. validar a saúde da Infra Hostinger
2. validar a saúde do Docker host
3. validar a presença da instância oficial no filesystem
4. validar coerência entre o nome da instância e o runtime efetivo
5. validar a presença estrutural do `matchzy.db` no contexto da instância
6. só depois aprofundar em MatchZy, plugins ou configuração do jogo

Essa sequência ajuda a responder rapidamente:

- a instância realmente existe como o ecossistema espera?
- o runtime está alinhado a ela?
- o problema é da instância ou do que roda sobre ela?

---

## Limites deste documento

Este documento não detalha:

- toda a árvore interna da instância
- cada arquivo de configuração do servidor
- runbooks de partida
- lista completa de plugins e versões
- troubleshooting aprofundado do MatchZy
- troubleshooting do ETL do portal

Esses tópicos pertencem a documentos específicos do contexto.

---

## Critério de pronto deste documento

Este documento pode ser considerado maduro quando:

- a posição estrutural da instância estiver reconciliada sem ambiguidade
- a dependência do nome `MixHAXIXE01` estiver formalizada claramente
- a relação entre instância, runtime do jogo e `matchzy.db` estiver clara
- os riscos de drift e renomeação estiverem bem representados
- ele puder ser usado como referência da instância oficial do ecossistema sem depender do master legado

---

## Última revisão

- Status: ativo
- Classificação: canônico
- Contexto: game panel / instância `MixHAXIXE01`
- Última revisão: 2026-03-18