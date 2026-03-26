# Admin References Inventory

## Objetivo

Inventariar os arquivos, paths, componentes, baselines e checkpoints operacionais relevantes para a camada administrativa do contexto Game Panel do HSC após a equalização baseada no CounterStrikeSharp.

Este documento existe para registrar, de forma explícita:

- onde vive cada artefato administrativo relevante
- quais arquivos devem ser tratados como fonte canônica
- quais arquivos devem permanecer vazios para evitar drift
- qual é a baseline confirmada atual
- quais checkpoints devem ser revisitados em troubleshooting futuro

---

## Escopo

Este documento cobre:

- paths vivos da camada administrativa do lado jogo
- baseline atual de arquivos e grupos
- componentes/plugins envolvidos na administração
- checkpoints de validação
- guardrails de manutenção

Este documento não cobre em profundidade:

- runbook detalhado de alteração
- explicação conceitual de cada comando
- troubleshooting completo de todos os plugins

Esses tópicos vivem nos documentos próprios desta trilha administrativa.

---

## Estado atual

A baseline administrativa atual do HSC no Game Panel é:

- autoridade centralizada em CounterStrikeSharp
- grupo vivo `#hsc/root`
- dois admins vivos no baseline atual
- MatchZy sem lista paralela de admins
- SimpleAdmin sem cadastro paralelo vivo de admins e grupos
- config do SimpleAdmin saneada para SQLite local

---

## Source of truth / evidências

Este inventário se apoia em baseline já validada no ambiente real, incluindo:

- leitura de `configs/admins.json`
- leitura de `configs/admin_groups.json`
- leitura de `cfg/MatchZy/admins.json`
- leitura de `CS2-SimpleAdmin/data/admins.json`
- leitura de `CS2-SimpleAdmin/data/groups.json`
- leitura de `CS2-SimpleAdmin.json`
- validação de boot do CounterStrikeSharp
- validação de `!css_admin`
- validação de `.whitelist`

---

## Relações com outros documentos

Este arquivo é complementar a:

- `docs/02-game-panel/game-panel-admin-authority-runtime.md`
- `docs/02-game-panel/game-panel-admin-operational-runbooks.md`
- `docs/02-game-panel/game-panel-admin-command-profiles.md`
- `docs/02-game-panel/game-panel-references-inventory.md`
- `docs/02-game-panel/plugins-installed.md`

Este documento descreve o **inventário vivo** da camada administrativa do lado jogo.  
Ele não substitui o runbook nem o modelo canônico de autoridade.

---

## Paths vivos canônicos

### CounterStrikeSharp

Raiz:

- `/home/amp/.ampdata/instances/MixHAXIXE01/counter-strike2/730/game/csgo/addons/counterstrikesharp/`

Configs administrativos vivos:

- `/home/amp/.ampdata/instances/MixHAXIXE01/counter-strike2/730/game/csgo/addons/counterstrikesharp/configs/admins.json`
- `/home/amp/.ampdata/instances/MixHAXIXE01/counter-strike2/730/game/csgo/addons/counterstrikesharp/configs/admin_groups.json`
- `/home/amp/.ampdata/instances/MixHAXIXE01/counter-strike2/730/game/csgo/addons/counterstrikesharp/configs/admin_overrides.json`

Logs relevantes:

- `/home/amp/.ampdata/instances/MixHAXIXE01/counter-strike2/730/game/csgo/addons/counterstrikesharp/logs/`

### CS2-SimpleAdmin

Configs vivos:

- `/home/amp/.ampdata/instances/MixHAXIXE01/counter-strike2/730/game/csgo/addons/counterstrikesharp/configs/plugins/CS2-SimpleAdmin/CS2-SimpleAdmin.json`
- `/home/amp/.ampdata/instances/MixHAXIXE01/counter-strike2/730/game/csgo/addons/counterstrikesharp/configs/plugins/CS2-SimpleAdmin/Commands.json`
- `/home/amp/.ampdata/instances/MixHAXIXE01/counter-strike2/730/game/csgo/addons/counterstrikesharp/configs/plugins/CS2-SimpleAdmin_FunCommands/CS2-SimpleAdmin_FunCommands.json`
- `/home/amp/.ampdata/instances/MixHAXIXE01/counter-strike2/730/game/csgo/addons/counterstrikesharp/configs/plugins/CS2-SimpleAdmin_StealthModule/CS2-SimpleAdmin_StealthModule.json`

Persistência viva do plugin:

- `/home/amp/.ampdata/instances/MixHAXIXE01/counter-strike2/730/game/csgo/addons/counterstrikesharp/plugins/CS2-SimpleAdmin/cs2-simpleadmin.sqlite`

Artefatos que devem permanecer vazios:

- `/home/amp/.ampdata/instances/MixHAXIXE01/counter-strike2/730/game/csgo/addons/counterstrikesharp/plugins/CS2-SimpleAdmin/data/admins.json`
- `/home/amp/.ampdata/instances/MixHAXIXE01/counter-strike2/730/game/csgo/addons/counterstrikesharp/plugins/CS2-SimpleAdmin/data/groups.json`

### MatchZy

Configs vivos:

- `/home/amp/.ampdata/instances/MixHAXIXE01/counter-strike2/730/game/csgo/cfg/MatchZy/config.cfg`
- `/home/amp/.ampdata/instances/MixHAXIXE01/counter-strike2/730/game/csgo/cfg/MatchZy/admins.json`

Artefato que deve permanecer vazio:

- `/home/amp/.ampdata/instances/MixHAXIXE01/counter-strike2/730/game/csgo/cfg/MatchZy/admins.json`

---

## Baseline viva atual

### `admins.json`

Estado esperado:

- apenas os admins vivos do HSC
- modelagem por grupos
- sem flags soltas desnecessárias por usuário

Baseline atual:

- `cu preto games` → `76561199089555261` → `#hsc/root`
- `DJ-RASTAMAN` → `76561198389181649` → `#hsc/root`

### `admin_groups.json`

Estado esperado:

- grupo `#hsc/root`
- flags administrativas centrais do HSC
- immunity `100`

### `admin_overrides.json`

Estado esperado:

- ausente ou vazio enquanto não houver necessidade real de override

### `cfg/MatchZy/admins.json`

Estado esperado:

```json
{}
```

### `CS2-SimpleAdmin/data/admins.json`

Estado esperado:

```json
{}
```

### `CS2-SimpleAdmin/data/groups.json`

Estado esperado:

```json
{}
```

---

## Componentes envolvidos na autoridade

### CounterStrikeSharp

Responsabilidade:

- fonte de verdade
- carga de admins e grupos
- resolução inicial de permissões

### CS2-SimpleAdmin

Responsabilidade:

- administração do servidor
- moderação
- painel de staff
- comandos administrativos

### CS2-SimpleAdmin Fun Commands

Responsabilidade:

- superfície adicional de intervenção utilitária

### CS2-SimpleAdmin Stealth Module

Responsabilidade:

- ocultação/discrição de staff conforme política do plugin

### MatchZy

Responsabilidade:

- administração da partida
- ready flow e controles competitivos

### PlayerSettings [Core]

Responsabilidade:

- persistência auxiliar por jogador

### MenuManager [Core]

Responsabilidade:

- infraestrutura de menus

---

## Checkpoints operacionais relevantes

### Checkpoint 1 — Boot do CounterStrikeSharp

Sinais esperados no log:

- `Loading Admins from .../configs/admins.json`
- `Loading Admin Groups from .../configs/admin_groups.json`
- carga subsequente de MatchZy e CS2-SimpleAdmin sem erro

### Checkpoint 2 — Baseline do SimpleAdmin

Sinais esperados:

- `!css_admin` abre corretamente
- `DatabaseType` em `CS2-SimpleAdmin.json` continua `SQLite`
- não há resíduo ativo de Railway

### Checkpoint 3 — Baseline do MatchZy

Sinais esperados:

- `.whitelist` responde para admin válido
- `cfg/MatchZy/admins.json` permanece vazio
- `matchzy_everyone_is_admin` continua refletindo a política desejada

### Checkpoint 4 — Ausência de drift

Sinais esperados:

- `data/admins.json` e `data/groups.json` do SimpleAdmin continuam vazios
- ninguém foi adicionado diretamente em `cfg/MatchZy/admins.json`
- os admins vivos continuam legíveis em `configs/admins.json`

---

## Configs relevantes do baseline atual

### SimpleAdmin

Pontos relevantes já confirmados:

- `DatabaseType: SQLite`
- `SqliteFilePath: cs2-simpleadmin.sqlite`
- `ReloadAdminsEveryMapChange: false`
- `MultiServerMode: true`
- catálogo de flags administrativas `@css/*`

### MatchZy

Pontos relevantes já confirmados:

- `matchzy_everyone_is_admin false`
- `matchzy_allow_force_ready true`
- `matchzy_minimum_ready_required 2`
- `matchzy_tech_pause_flag ""`

Leitura operacional:

- o staff deve interpretar alguns comandos à luz do estado da partida
- `.forceready`, por exemplo, pode depender do contexto de ready flow
- `.whitelist` é um bom teste administrativo imediato

---

## Guardrails de manutenção

### Guardrail 1

Se precisar mudar staff, mexer em `admins.json`.

### Guardrail 2

Se precisar mudar papel, mexer em `admin_groups.json`.

### Guardrail 3

Se precisar refinar comando específico, considerar `admin_overrides.json`.

### Guardrail 4

Nunca usar listas locais de plugin como atalho permanente de governança.

### Guardrail 5

Após mudança administrativa, sempre reiniciar a instância e validar o runtime.

---

## Itens úteis para troubleshooting futuro

Quando houver dúvida sobre autoridade, revisar na seguinte ordem:

1. `configs/admins.json`
2. `configs/admin_groups.json`
3. log mais recente do CounterStrikeSharp
4. `cfg/MatchZy/admins.json`
5. `CS2-SimpleAdmin/data/admins.json`
6. `CS2-SimpleAdmin/data/groups.json`
7. smoke tests `!css_admin` e `.whitelist`

Essa ordem reduz investigação desnecessária e mantém o troubleshooting alinhado ao modelo canônico.

---

## Resumo executivo

O inventário administrativo do HSC ficou pequeno e explícito:

- poucos arquivos importam
- a origem canônica está clara
- os artefatos que não devem carregar autoridade também estão claros
- o troubleshooting passou a ter ordem lógica objetiva

Esse inventário deve ser a referência rápida de paths e checkpoints para toda manutenção futura.
