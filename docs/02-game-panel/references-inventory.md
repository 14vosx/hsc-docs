# References and Inventory

## Objetivo

Consolidar o inventário de referências, evidências, artefatos operacionais, componentes críticos e itens pendentes de validação do contexto Game Panel do ecossistema HSC.

Este documento existe para registrar, de forma estável e auditável:

- quais documentos alimentam este contexto
- quais artefatos reais do ambiente são source of truth operacional
- quais componentes e paths são críticos para o lado jogo
- quais comandos ajudam a validar rapidamente o contexto
- quais pontos ainda dependem de confirmação direta no ambiente real
- quais são os limites documentais deste contexto

---

## Escopo

Este documento cobre:

- documentos de origem do contexto
- artefatos reais do runtime do lado jogo
- componentes operacionais centrais
- paths críticos conhecidos
- plugins e persistências já reconciliados
- comandos de validação
- itens pendentes de confirmação
- limites documentais do contexto

Este documento não cobre em profundidade:

- infraestrutura base da Hostinger
- pipeline ETL detalhado da Static API v2
- contratos JSON do portal
- Auth API no Lightsail
- credenciais e arquivos sensíveis
- histórico completo de mudanças

Esses assuntos vivem nos documentos especializados dos contextos `01-infra-hostinger`, `03-portal-estatico` e `04-infra-aws-lightsail`, bem como em impl-logs e material legado.

---

## Estado atual

O contexto `02-game-panel` já possui estrutura canônica definida para documentar:

- arquitetura de runtime do lado jogo
- AMP Instance Manager
- instância oficial `MixHAXIXE01`
- configuração do servidor CS2
- MatchZy
- plugins instalados
- runbooks operacionais
- observabilidade e troubleshooting do lado game

Neste estágio da reconciliação, o contexto já possui confirmação operacional suficiente para fixar sem ambiguidade:

- a instância oficial `MixHAXIXE01`
- o container relevante `AMP_MixHAXIXE01`
- a versão reconciliada do MatchZy `0.8.15`
- o path vivo do `matchzy.db`
- o path de backup identificado do `matchzy.db`
- o inventário real de plugins atualmente carregados no runtime
- a presença de CS2-SimpleAdmin como camada administrativa ativa do servidor

Ainda restam pendências menores, principalmente ligadas a:

- natureza exata de persistências auxiliares administrativas
- inventário final dos presets e aliases realmente usados
- baseline final entre config principal e configs auxiliares
- eventual auditoria de artefatos de plugin instalados no disco, mas não carregados

---

## Source of truth / evidências

As evidências que sustentam este contexto se dividem em quatro grupos principais.

### 1. Documentação consolidada do ecossistema

Usada para:

- visão macro do papel do Game Panel no HSC
- separação entre Infra Hostinger, Game Panel, Portal Estático e Lightsail
- topologia geral do lado jogo

### 2. Documentação específica da camada AMP/CS2

Usada para:

- estrutura operacional da instância
- MatchZy
- fluxo competitivo do servidor
- runbooks do lado jogo
- persistência local e relação com o portal

### 3. Impl-logs e registros incrementais

Usados para:

- rastrear mudanças em plugins, presets, permissões e fluxo operacional
- registrar endurecimentos administrativos
- preservar rastreabilidade de ajustes relevantes do lado jogo

### 4. Ambiente real do runtime

Usado para:

- validar inventário real de plugins
- validar a existência e coerência da instância `MixHAXIXE01`
- validar presença do workload no host
- validar a presença e evolução do `matchzy.db`
- confirmar o estado real do servidor e dos componentes carregados

Regra principal:

- quando houver conflito entre documento histórico e ambiente real validado, prevalece o ambiente real validado

---

## Relações com outros documentos

Este arquivo é complementar a:

- `docs/02-game-panel/README.md`
- `docs/02-game-panel/architecture-runtime.md`
- `docs/02-game-panel/amp-instance-manager.md`
- `docs/02-game-panel/instance-mixhaxixe01.md`
- `docs/02-game-panel/cs2-server-configuration.md`
- `docs/02-game-panel/matchzy.md`
- `docs/02-game-panel/plugins-installed.md`
- `docs/02-game-panel/operational-runbooks.md`
- `docs/02-game-panel/observability-troubleshooting.md`

Este documento não substitui nenhum dos arquivos acima.  
Ele funciona como fechamento de inventário e referência do contexto.

---

## Documentos de origem

Os documentos abaixo são as principais fontes de extração e reconciliação deste contexto.

### Fontes primárias

- documentação consolidada atual do ecossistema HSC
- blueprint técnico consolidado do HSC
- documentação específica da camada AMP/CS2
- documentação específica do MatchZy e dos fluxos operacionais do servidor

### Fontes secundárias

- documentação reconciliada da Infra Hostinger
- documentação reconciliada do Portal Estático
- impl-logs ligados a plugins, presets, permissões administrativas e operação do servidor

### Fontes de apoio histórico

- master antigo
- blueprint histórico
- documentação legada do lado CS2
- materiais `_old` úteis para reconciliar runbooks, paths, plugins e decisões operacionais

Regra documental:

- fontes históricas ajudam a reconciliar
- mas não governam o canônico sozinhas

---

## Artefatos reais do contexto

Os artefatos reais conhecidos ou esperados do contexto Game Panel incluem:

### Instância oficial

- `MixHAXIXE01`

### Base estrutural das instâncias

- `/home/amp/.ampdata/instances/`

### Árvore estrutural da instância oficial

- `/home/amp/.ampdata/instances/MixHAXIXE01/`

### Runtime de container associado

- container operacional conhecido: `AMP_MixHAXIXE01`

### Camada de plugins

- runtime baseado em CounterStrikeSharp
- inventário real verificável por `css_plugins list`

### Plugin competitivo principal

- MatchZy `0.8.15`

### Plugins core reconciliados

- `PlayerSettings [Core]` `0.9.3`
- `MenuManager [Core]` `1.4.1`

### Camada administrativa reconciliada

- `CS2-SimpleAdmin (RELEASE)` `1.7.8-beta-10b`
- `[CS2-SimpleAdmin] Stealth Module` `v1.0.2`
- `CS2-SimpleAdmin Fun Commands` `1.0.0`

### Plugin auxiliar já reconciliado

- `WeaponPaints` `3.2b`

### Persistência estatística estrutural ativa

- `matchzy.db`

### Tabelas estatísticas principais

- `matchzy_stats_matches`
- `matchzy_stats_maps`
- `matchzy_stats_players`

Esses artefatos devem ser tratados como inventário-base do contexto até revisão explícita.

---

## Componentes operacionais centrais

Os componentes principais conhecidos deste contexto são:

### AMP Instance Manager

Papel:
- camada de gestão da instância do lado jogo

### Instância `MixHAXIXE01`

Papel:
- unidade operacional principal do servidor CS2 no ecossistema

### Runtime do servidor CS2

Papel:
- workload real do lado jogo

### CounterStrikeSharp

Papel:
- base da camada de plugins

### MatchZy

Papel:
- plugin competitivo central
- componente operacional e estrutural de dados

### CS2-SimpleAdmin

Papel:
- camada administrativa principal do servidor
- base das capacidades operacionais de admin

### Plugins core auxiliares

Papel:
- storage de settings de player
- infraestrutura de menus
- apoio funcional a outros plugins

### Plugins auxiliares

Papel:
- complementar administração, experiência do servidor e persistências auxiliares

### Persistência local

Papel:
- registrar o estado estatístico do lado jogo
- alimentar indiretamente o Portal Estático

---

## Paths críticos

Os paths críticos conhecidos deste contexto incluem:

### Base estrutural das instâncias AMP

- `/home/amp/.ampdata/instances/`

### Árvore da instância oficial

- `/home/amp/.ampdata/instances/MixHAXIXE01/`

### Árvore base de plugins

- `/home/amp/.ampdata/instances/MixHAXIXE01/counter-strike2/730/game/csgo/addons/counterstrikesharp/plugins/`

### Path canônico vivo do `matchzy.db`

- `/home/amp/.ampdata/instances/MixHAXIXE01/counter-strike2/730/game/csgo/addons/counterstrikesharp/plugins/MatchZy/matchzy.db`

### Path de backup identificado do `matchzy.db`

- `/home/amp/.ampdata/instances/MixHAXIXE01/counter-strike2/730/game/csgo/backup-hsc/MatchZy.live-dir.pre-upstream.20260317-205843/matchzy.db`

### Diretórios reconciliados de plugins

- `/home/amp/.ampdata/instances/MixHAXIXE01/counter-strike2/730/game/csgo/addons/counterstrikesharp/plugins/CS2-SimpleAdmin/`
- `/home/amp/.ampdata/instances/MixHAXIXE01/counter-strike2/730/game/csgo/addons/counterstrikesharp/plugins/CS2-SimpleAdmin_FunCommands/`
- `/home/amp/.ampdata/instances/MixHAXIXE01/counter-strike2/730/game/csgo/addons/counterstrikesharp/plugins/CS2-SimpleAdmin_StealthModule/`
- `/home/amp/.ampdata/instances/MixHAXIXE01/counter-strike2/730/game/csgo/addons/counterstrikesharp/plugins/MatchZy/`
- `/home/amp/.ampdata/instances/MixHAXIXE01/counter-strike2/730/game/csgo/addons/counterstrikesharp/plugins/MenuManagerCore/`
- `/home/amp/.ampdata/instances/MixHAXIXE01/counter-strike2/730/game/csgo/addons/counterstrikesharp/plugins/PlayerSettings/`
- `/home/amp/.ampdata/instances/MixHAXIXE01/counter-strike2/730/game/csgo/addons/counterstrikesharp/plugins/WeaponPaints/`

Regra prática:

- o path em `addons/counterstrikesharp/plugins/MatchZy/` é a fonte viva canônica do banco principal
- o path em `backup-hsc/` é backup histórico e não deve ser tratado como fonte ativa
- o inventário vivo de plugins deve sempre ser lido em conjunto com `css_plugins list`
- qualquer mudança nesses paths deve ser tratada como alteração relevante e refletida no canônico e, quando necessário, no impl-log

---

## Componentes já reconciliados

Os itens abaixo já possuem relevância reconciliada suficiente no contexto atual.

### Instância oficial

- `MixHAXIXE01`

### Plugin competitivo principal

- MatchZy `0.8.15`

### Plugin auxiliar já visto e validado em produção

- WeaponPaints `3.2b`

### Plugins core reconciliados

- `PlayerSettings [Core]` `0.9.3`
- `MenuManager [Core]` `1.4.1`

### Camada administrativa reconciliada

- `CS2-SimpleAdmin (RELEASE)` `1.7.8-beta-10b`
- `[CS2-SimpleAdmin] Stealth Module` `v1.0.2`
- `CS2-SimpleAdmin Fun Commands` `1.0.0`

### Persistência estatística principal ativa

- `matchzy.db`

### Tabelas estruturais da persistência

- `matchzy_stats_matches`
- `matchzy_stats_maps`
- `matchzy_stats_players`

### Comando operacional de inventário de plugins

- `css_plugins list`

### Container relevante do host

- `AMP_MixHAXIXE01`

Esses itens já devem ser tratados como parte da verdade operacional do contexto.

---

## Inventário vivo de plugins carregados

O runtime atual reconciliado possui **7 plugins carregados**:

1. `WeaponPaints` `3.2b`
2. `PlayerSettings [Core]` `0.9.3`
3. `CS2-SimpleAdmin Fun Commands` `1.0.0`
4. `MatchZy` `0.8.15`
5. `MenuManager [Core]` `1.4.1`
6. `[CS2-SimpleAdmin] Stealth Module` `v1.0.2`
7. `CS2-SimpleAdmin (RELEASE)` `1.7.8-beta-10b`

Leitura canônica:

- este é o inventário vivo principal do contexto
- qualquer divergência futura deve ser tratada como mudança real de runtime
- o documento `plugins-installed.md` é o detalhamento semântico desse inventário
- este arquivo registra o inventário como parte do fechamento operacional do contexto

---

## Persistências auxiliares já observadas

Além do `matchzy.db`, o runtime em disco já mostrou evidência suficiente de persistências/configurações auxiliares em:

### CS2-SimpleAdmin

Subárvores observadas:
- `CS2-SimpleAdmin/data`
- `CS2-SimpleAdmin/Database`

### WeaponPaints

Subárvores observadas:
- `WeaponPaints/data`
- `WeaponPaints/gamedata`

### PlayerSettings

Leitura reconciliada:
- storage central de settings dos jogadores

Regra importante:

- a existência dessas persistências auxiliares não muda o fato de que a persistência estrutural principal da cadeia pública continua sendo o `matchzy.db`
- mas o contexto agora já deve reconhecer explicitamente que a camada de administração e personalização possui storage auxiliar real

---

## Workloads e dependências cruzadas

Os principais workloads e dependências cruzadas deste contexto incluem:

### Dependência da Infra Hostinger

O Game Panel depende de:

- Debian
- Docker host
- árvore operacional sob `amp`
- substrate íntegro do host

### Dependência do lado portal

O Portal Estático depende do Game Panel para:

- origem do `matchzy.db`
- continuidade da produção de stats
- estabilidade do schema e do path da persistência local

### Dependência operacional humana

O lado jogo depende de:

- runbooks claros
- staff/admins capazes de operar scrim, BO1, BO3, veto, coach e fallback
- alinhamento entre runtime real e expectativa operacional

### Dependência da camada administrativa

A operação prática do servidor depende hoje, de forma explícita, do ecossistema:

- `CS2-SimpleAdmin`
- `CS2-SimpleAdmin Stealth Module`
- `CS2-SimpleAdmin Fun Commands`

---

## Comandos de validação

Os comandos abaixo formam um kit mínimo de validação do contexto.

### Validar presença estrutural das instâncias

```bash
ls -lah /home/amp/.ampdata/instances/
```

### Validar presença da instância oficial

```bash
ls -lah /home/amp/.ampdata/instances/MixHAXIXE01/
```

### Validar workload/container relevante

```bash
docker ps --format "table {{.Names}}\t{{.Image}}\t{{.Ports}}"
docker ps -a --format "table {{.Names}}\t{{.Status}}\t{{.Image}}"
```

### Validar associação da instância ao runtime conhecido

```bash
docker ps -a --format "table {{.Names}}\t{{.Status}}\t{{.Image}}" | grep MixHAXIXE01
```

### Validar camada de plugins no runtime

```bash
css_plugins list
```

### Validar diretórios principais de plugins no host

```bash
find /home/amp/.ampdata/instances/MixHAXIXE01/counter-strike2/730/game/csgo/addons/counterstrikesharp/plugins -maxdepth 2 -type d | sort
```

### Validar presença estrutural do banco ativo do MatchZy

```bash
ls -l /home/amp/.ampdata/instances/MixHAXIXE01/counter-strike2/730/game/csgo/addons/counterstrikesharp/plugins/MatchZy/matchzy.db
realpath /home/amp/.ampdata/instances/MixHAXIXE01/counter-strike2/730/game/csgo/addons/counterstrikesharp/plugins/MatchZy/matchzy.db
```

### Validar presença do backup identificado

```bash
ls -l /home/amp/.ampdata/instances/MixHAXIXE01/counter-strike2/730/game/csgo/backup-hsc/MatchZy.live-dir.pre-upstream.20260317-205843/matchzy.db
```

### Validar tabelas principais do banco ativo

```bash
sqlite3 /home/amp/.ampdata/instances/MixHAXIXE01/counter-strike2/730/game/csgo/addons/counterstrikesharp/plugins/MatchZy/matchzy.db ".tables"
```

### Validar volume básico das tabelas principais

```bash
sqlite3 /home/amp/.ampdata/instances/MixHAXIXE01/counter-strike2/730/game/csgo/addons/counterstrikesharp/plugins/MatchZy/matchzy.db "SELECT COUNT(*) FROM matchzy_stats_matches;"
sqlite3 /home/amp/.ampdata/instances/MixHAXIXE01/counter-strike2/730/game/csgo/addons/counterstrikesharp/plugins/MatchZy/matchzy.db "SELECT COUNT(*) FROM matchzy_stats_players;"
sqlite3 /home/amp/.ampdata/instances/MixHAXIXE01/counter-strike2/730/game/csgo/addons/counterstrikesharp/plugins/MatchZy/matchzy.db "SELECT COUNT(*) FROM matchzy_stats_maps;"
```

---

## Itens pendentes de confirmação

Os itens abaixo ainda devem ser confirmados diretamente no ambiente real para elevar o grau de confiança do contexto.

### 1. Natureza exata da subárvore `CS2-SimpleAdmin/Database`

Já há evidência suficiente de persistência auxiliar administrativa, mas o papel exato dessa camada ainda pode ser detalhado melhor no futuro.

### 2. Inventário final dos presets e aliases realmente usados

Já há evidência de presets e aliases úteis, mas a lista atual em produção ainda precisa ser consolidada a partir do ambiente real.

### 3. Baseline exato entre config principal e configs auxiliares

É desejável fechar explicitamente quais arquivos e presets representam:
- baseline do servidor
- modo scrim
- modo BO1
- modo BO3
- treino e presets auxiliares

### 4. Eventuais artefatos de plugin instalados no disco, mas não carregados

O inventário vivo já está reconciliado, mas uma auditoria futura ainda pode mapear artefatos órfãos ou stale no disco.

### 5. Eventuais persistências auxiliares adicionais além das já identificadas

É útil confirmar se outros plugins em produção mantêm bases ou artefatos adicionais que devam ser citados formalmente neste contexto.

---

## Itens fora do escopo deste contexto

Os itens abaixo não pertencem ao inventário canônico do Game Panel:

- DNS, TLS e Certbot da Hostinger
- serving público do Nginx do portal
- pipeline ETL detalhado da v2
- contratos JSON do portal
- frontend público do portal
- runtime da Auth API no Lightsail
- credenciais reais
- arquivos de acesso sensíveis
- documentação histórica não reconciliada

Esses itens pertencem a outros contextos ou devem permanecer fora do fluxo normal do repositório documental.

---

## Limites documentais do contexto

Os limites documentais deste contexto são:

- ele documenta a operação do lado jogo
- ele não documenta sozinho o substrate da Infra Hostinger
- ele não documenta sozinho a publicação pública do portal
- ele não substitui o índice mestre
- ele não substitui impl-logs históricos
- ele depende de confirmação periódica contra o ambiente real para permanecer confiável

---

## Critério de manutenção

Este documento deve ser atualizado quando houver:

- mudança de instância oficial
- mudança relevante no inventário de plugins carregados
- mudança de path estrutural do banco do MatchZy
- mudança importante na organização do runtime do lado jogo
- atualização relevante de versão do MatchZy
- mudança relevante na camada administrativa do servidor
- confirmação ou resolução de item pendente listado aqui

Mudanças pequenas de comportamento funcional devem ser refletidas primeiro no documento especializado correspondente, e não necessariamente aqui.

---

## Critério de pronto deste documento

Este documento pode ser considerado maduro quando:

- os artefatos reais do lado jogo estiverem confirmados sem ambiguidade
- o inventário de plugins carregados estiver reconciliado com o runtime atual
- o path final do `matchzy.db` estiver fixado
- a camada administrativa baseada em CS2-SimpleAdmin estiver explicitamente refletida
- os itens pendentes estiverem resolvidos ou explicitamente mantidos como pendência consciente
- ele puder ser usado como inventário confiável do Game Panel sem depender do master legado

---

## Última revisão

- Status: ativo
- Classificação: canônico
- Contexto: game panel / references and inventory
- Última revisão: 2026-03-18