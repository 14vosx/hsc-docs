# Game Panel Admin Authority Runtime

## Navegação rápida

- [Home da documentação](../README.md)
- [Game Panel](./README.md)
- [Master Index](../00-governance/99-master-index.md)

---
## Objetivo

Documentar, de forma canônica, a arquitetura de autoridade administrativa do contexto Game Panel do ecossistema HSC após a equalização concluída com base no CounterStrikeSharp.

Este documento existe para registrar, de forma estável, auditável e operacionalmente útil:

- qual é a fonte de verdade administrativa do lado jogo
- como a autoridade se propaga para MatchZy e CS2-SimpleAdmin
- quais arquivos compõem a baseline viva de administração do servidor
- quais papéis cada plugin exerce dentro do modelo final
- quais regras evitam o retorno do drift de permissões

---

## Escopo

Este documento cobre:

- o modelo de autoridade administrativa do lado CS2
- a relação entre CounterStrikeSharp, CS2-SimpleAdmin e MatchZy
- a baseline viva de arquivos e paths administrativos
- o estado confirmado após a equalização
- os princípios de manutenção sustentável dessa arquitetura

Este documento não cobre em profundidade:

- warmup, BO1, BO3, veto e coach como fluxo competitivo
- troubleshooting amplo de AMP e da instância
- ETL do portal estático
- Auth API e backoffice
- personalização de skins, knives ou agents

Esses tópicos vivem em documentos próprios do contexto `02-game-panel` e dos demais contextos canônicos.

---

## Estado atual

O estado canônico atual do lado jogo é:

- o **CounterStrikeSharp** é a fonte única de verdade para administração
- a autoridade administrativa viva é modelada por `admins.json` + `admin_groups.json`
- o grupo operacional ativo do HSC é `#hsc/root`
- o `MatchZy` deixou de manter lista paralela de admins como fonte primária
- o `CS2-SimpleAdmin` deixou de manter admins e grupos paralelos como fonte primária
- o `CS2-SimpleAdmin` continua como camada administrativa do servidor
- o `MatchZy` continua como camada administrativa da partida
- o bootstrap do runtime continua carregando `admins.json` e `admin_groups.json` antes dos plugins administrativos

Leitura correta do estado:

- a autoridade foi centralizada
- as camadas auxiliares continuam existindo
- o drift anterior foi removido
- a manutenção futura deve preservar essa topologia

---

## Source of truth / evidências

A baseline documental deste arquivo se apoia em evidência direta reconciliada no ambiente real:

- carga de `admins.json` pelo CounterStrikeSharp no boot
- carga de `admin_groups.json` pelo CounterStrikeSharp no boot
- grupo vivo `#hsc/root` com flags `@css/*`
- `cfg/MatchZy/admins.json` zerado para evitar autoridade paralela
- `plugins/CS2-SimpleAdmin/data/admins.json` zerado
- `plugins/CS2-SimpleAdmin/data/groups.json` zerado
- `CS2-SimpleAdmin` funcional via `!css_admin`
- `MatchZy` funcional via `.whitelist`
- `CS2-SimpleAdmin.json` operando com `DatabaseType: SQLite`
- limpeza do resíduo legado do Railway no config do SimpleAdmin

Regra importante:

este documento descreve o **estado canônico desejado e validado**, não um histórico passo a passo da implementação.

---

## Relações com outros documentos

Este arquivo é complementar a:

- `docs/02-game-panel/README.md`
- `docs/02-game-panel/game-panel-architecture-runtime.md`
- `docs/02-game-panel/plugins-installed.md`
- `docs/02-game-panel/matchzy.md`
- `docs/02-game-panel/game-panel-operational-runbooks.md`
- `docs/02-game-panel/game-panel-observability-troubleshooting.md`
- `docs/02-game-panel/game-panel-references-inventory.md`
- `docs/00-governance/documentation-system.md`
- `docs/00-governance/99-master-index.md`

Este documento descreve o modelo canônico de autoridade administrativa do lado jogo.  
Ele não substitui o inventário geral de plugins nem os runbooks de partida.

---

## Princípio canônico da autoridade do lado jogo

No HSC, a autoridade administrativa do contexto Game Panel deve ser tratada em camadas distintas.

### Camada 1 — CounterStrikeSharp

O CounterStrikeSharp é a **camada-base de autoridade**.

No modelo atual, isso significa:

- o bootstrap administrativo começa no CounterStrikeSharp
- `admins.json` é a fonte viva de usuários administrativos
- `admin_groups.json` é a fonte viva de papéis administrativos
- `admin_overrides.json`, quando usado, deve refinar permissões de comando de forma transversal

Regra canônica:

**nenhuma camada auxiliar deve competir com o CounterStrikeSharp como fonte de verdade para admins.**

### Camada 2 — CS2-SimpleAdmin

O CS2-SimpleAdmin deve ser tratado como **consumidor da autoridade** vinda do CounterStrikeSharp para fins de administração do servidor.

Seu papel canônico é:

- moderação
- intervenção administrativa sobre jogadores
- comandos utilitários de staff
- menus administrativos
- administração operacional do servidor em runtime

Ele não deve ser tratado como base canônica de cadastro persistente de admins.

### Camada 3 — MatchZy

O MatchZy deve ser tratado como **consumidor da autoridade** para fins de administração da partida.

Seu papel canônico é:

- controle do fluxo competitivo
- ready flow
- whitelist
- knife round
- playout
- partida organizada
- prática e comandos ligados ao contexto do jogo

Ele não deve ser tratado como fonte primária de cadastro de admins.

---

## Modelo final de autoridade

O modelo final do HSC no contexto Game Panel é:

```text
CounterStrikeSharp
├── admins.json
├── admin_groups.json
└── admin_overrides.json (opcional)
        ↓
        ├── CS2-SimpleAdmin
        │   ├── core admin
        │   ├── Fun Commands
        │   └── Stealth Module
        ↓
        └── MatchZy
            ├── comandos de jogador/time
            └── comandos de admin de partida
```

Leitura correta:

- a autoridade nasce no CounterStrikeSharp
- os plugins administrativos e competitivos consomem essa autoridade
- listas paralelas só podem existir como detalhe técnico auxiliar e nunca como fonte primária de governança

---

## Baseline viva confirmada

### Arquivos vivos de autoridade

#### CounterStrikeSharp

- `addons/counterstrikesharp/configs/admins.json`
- `addons/counterstrikesharp/configs/admin_groups.json`
- `addons/counterstrikesharp/configs/admin_overrides.json` (atualmente não usado)

#### CS2-SimpleAdmin

- `addons/counterstrikesharp/configs/plugins/CS2-SimpleAdmin/CS2-SimpleAdmin.json`
- `addons/counterstrikesharp/configs/plugins/CS2-SimpleAdmin/Commands.json`
- `addons/counterstrikesharp/configs/plugins/CS2-SimpleAdmin_FunCommands/CS2-SimpleAdmin_FunCommands.json`
- `addons/counterstrikesharp/configs/plugins/CS2-SimpleAdmin_StealthModule/CS2-SimpleAdmin_StealthModule.json`
- `addons/counterstrikesharp/plugins/CS2-SimpleAdmin/cs2-simpleadmin.sqlite`

#### MatchZy

- `cfg/MatchZy/config.cfg`
- `cfg/MatchZy/admins.json`

### Arquivos que devem permanecer sem autoridade paralela

No modelo atual, estes arquivos devem permanecer vazios ou sem cadastro administrativo vivo próprio:

- `cfg/MatchZy/admins.json` → deve permanecer `{}`
- `plugins/CS2-SimpleAdmin/data/admins.json` → deve permanecer `{}`
- `plugins/CS2-SimpleAdmin/data/groups.json` → deve permanecer `{}`

Regra importante:

se esses artefatos voltarem a ser usados como cadastro primário de admin, o drift reaparece.

---

## Baseline atual de grupos e admins

### Grupo canônico ativo

Grupo vivo:

- `#hsc/root`

Flags atuais do grupo:

- `@css/generic`
- `@css/chat`
- `@css/changemap`
- `@css/slay`
- `@css/kick`
- `@css/ban`
- `@css/permban`
- `@css/unban`
- `@css/showip`
- `@css/cvar`
- `@css/rcon`
- `@css/cheats`
- `@css/config`
- `@css/root`

Immunity atual:

- `100`

### Admins vivos atuais

Admins atualmente ligados ao grupo `#hsc/root`:

- `cu preto games` → `76561199089555261`
- `DJ-RASTAMAN` → `76561198389181649`

Leitura correta:

- a lista viva de admins está somente em `configs/admins.json`
- o grupo vivo está somente em `configs/admin_groups.json`
- o servidor deve ser mantido a partir desses dois arquivos

---

## Papel de cada plugin no modelo atual

### CounterStrikeSharp

Papel canônico:

- framework base do runtime administrativo
- fonte única de verdade para admins
- resolução inicial de grupos e flags
- sustentação da camada de plugins do lado jogo

### CS2-SimpleAdmin

Papel canônico:

- moderação do servidor
- administração prática em runtime
- menus administrativos
- comandos de informação, moderação, comunicação e controle do servidor

Papel que **não** deve assumir:

- cadastro canônico persistente de admins do HSC

### CS2-SimpleAdmin Fun Commands

Papel canônico:

- ampliar a superfície administrativa com comandos de intervenção utilitária
- operar como módulo do ecossistema SimpleAdmin

### CS2-SimpleAdmin Stealth Module

Papel canônico:

- permitir ocultação/stealth do staff conforme a política do plugin
- operar como módulo do ecossistema SimpleAdmin

### MatchZy

Papel canônico:

- administração do fluxo de partida
- ready flow
- whitelist
- knife round
- playout
- práticas e controles competitivos

Papel que **não** deve assumir:

- cadastro canônico persistente de admins do HSC

### PlayerSettings [Core]

Papel canônico:

- persistência de preferências/configurações por jogador
- suporte estrutural para ecossistema de plugins

### MenuManager [Core]

Papel canônico:

- infraestrutura de menus para plugins que dependem de interface administrativa em runtime

---

## Por que este modelo é sustentável

O modelo atual é sustentável porque reduz o número de pontos onde a verdade administrativa pode divergir.

Antes da equalização, existiam três superfícies capazes de carregar ou aparentar carregar autoridade:

- CounterStrikeSharp
- CS2-SimpleAdmin
- MatchZy

Após a equalização:

- a verdade foi centralizada
- as camadas auxiliares foram mantidas como consumidoras
- o troubleshooting ficou mais simples
- o onboarding de staff ficou mais previsível
- a documentação ficou alinhada ao runtime real

### Benefícios operacionais diretos

- menos ambiguidade sobre quem é admin
- menos risco de comando funcionar num plugin e falhar em outro por drift de cadastro
- menor acoplamento entre governança de staff e dados locais do plugin
- rollback conceitualmente mais simples
- manutenção futura baseada em arquivos pequenos e previsíveis

---

## Regras de ouro do HSC para autoridade administrativa

### Regra 1

**Adicionar, editar e remover admins só pelo CounterStrikeSharp.**

### Regra 2

**Não usar `cfg/MatchZy/admins.json` como fonte de verdade.**

### Regra 3

**Não usar `plugins/CS2-SimpleAdmin/data/admins.json` ou `data/groups.json` como fonte de verdade.**

### Regra 4

**Não usar os comandos persistentes de cadastro do SimpleAdmin como workflow canônico de governança.**

Isso inclui, no fluxo normal do HSC:

- `css_addadmin`
- `css_deladmin`
- `css_addgroup`
- `css_delgroup`

Esses comandos podem existir no plugin, mas o uso deles como rotina canônica reintroduz drift.

### Regra 5

**Toda mudança de autoridade deve terminar com validação explícita de runtime.**

No mínimo:

- validar boot carregando `admins.json`
- validar boot carregando `admin_groups.json`
- validar `!css_admin`
- validar um comando administrativo de MatchZy, como `.whitelist`

### Regra 6

**Configs legadas não devem ficar carregando credenciais ou rotas mortas.**

Por isso, o `CS2-SimpleAdmin.json` foi saneado para `DatabaseType: SQLite` sem resíduo ativo de Railway.

---

## O que ainda não faz parte da baseline

O modelo atual já está estável, mas algumas expansões ainda não fazem parte da baseline viva:

- grupos intermediários como `#hsc/mod`, `#hsc/match-admin` ou `#hsc/fun`
- uso de `admin_overrides.json` para refino fino de comandos
- políticas formais de segregação entre moderação, operação de partida e manutenção técnica

Leitura correta:

essas evoluções são possíveis, mas ainda não devem ser assumidas como parte da baseline atual.

---

## Critério de sucesso do contexto administrativo

A camada administrativa do lado jogo só deve ser tratada como saudável quando:

- `admins.json` contém apenas os admins vivos do HSC
- `admin_groups.json` contém os grupos vivos do HSC
- o bootstrap carrega `admins.json` e `admin_groups.json`
- `!css_admin` funciona para quem deve funcionar
- um comando administrativo do MatchZy funciona para quem deve funcionar
- não existem listas paralelas voltando a divergir silenciosamente

---

## Resumo executivo

O HSC passou a operar com um modelo administrativo simples e canônico:

- autoridade centralizada no CounterStrikeSharp
- consumo dessa autoridade por CS2-SimpleAdmin e MatchZy
- remoção das fontes paralelas de cadastro de admin
- baseline viva pequena, explícita e sustentável

Esse é o modelo que deve permanecer como referência para toda manutenção futura do contexto Game Panel.
