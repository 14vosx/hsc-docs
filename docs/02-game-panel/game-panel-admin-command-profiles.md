# Admin Command Profiles

## Navegação rápida

- [Home da documentação](../README.md)
- [Game Panel](./README.md)
- [Master Index](../00-governance/99-master-index.md)

---
## Objetivo

Documentar a superfície administrativa do lado jogo por categoria de comando, por perfil operacional e por contexto de uso após a equalização do HSC baseada no CounterStrikeSharp.

Este documento existe para registrar:

- quais superfícies de comando existem no ambiente atual
- como esses comandos se distribuem entre CounterStrikeSharp, CS2-SimpleAdmin e MatchZy
- quais perfis operacionais fazem sentido no baseline atual
- quando usar, quando evitar e como interpretar cada família de comandos

---

## Escopo

Este documento cobre:

- categorias de comandos do ecossistema administrativo atual
- perfis operacionais do HSC no lado jogo
- comandos disponíveis por camada funcional
- critérios de uso e de não uso
- diferenças entre comando disponível e comando canônico

Este documento não cobre em profundidade:

- todas as aliases raras ou de baixa frequência de MatchZy prática avançada
- todas as mensagens de erro possíveis de cada plugin
- tuning detalhado de presets competitivos
- modelagem futura de grupos intermediários ainda não existentes

---

## Estado atual

A superfície administrativa atual do lado jogo é composta por três blocos principais:

- **CounterStrikeSharp** como camada-base de permissões
- **CS2-SimpleAdmin** como administração do servidor
- **MatchZy** como administração da partida

Além disso:

- **Fun Commands** amplia o SimpleAdmin
- **Stealth Module** amplia o SimpleAdmin
- **PlayerSettings** e **MenuManager** são suporte estrutural, não superfície principal de governança

Leitura importante:

este documento fala da **superfície funcional disponível no ambiente**, mas sempre distinguindo entre:

- comando tecnicamente disponível
- comando canonicamente recomendado no HSC

---

## Source of truth / evidências

A classificação deste documento se apoia em:

- `Commands.json` do ecossistema CS2-SimpleAdmin
- config viva dos módulos do SimpleAdmin
- superfície confirmada em runtime do MatchZy
- strings/comandos observados na build instalada do MatchZy
- testes funcionais de `!css_admin` e `.whitelist`
- baseline administrativa centralizada no CounterStrikeSharp

---

## Relações com outros documentos

Este arquivo é complementar a:

- `docs/02-game-panel/game-panel-admin-authority-runtime.md`
- `docs/02-game-panel/game-panel-admin-operational-runbooks.md`
- `docs/02-game-panel/matchzy.md`
- `docs/02-game-panel/plugins-installed.md`
- `docs/02-game-panel/game-panel-operational-runbooks.md`

Este documento descreve a **superfície de comandos e perfis** do contexto administrativo do lado jogo.  
Ele não substitui a modelagem da autoridade nem os runbooks de manutenção.

---

## Perfis operacionais do baseline atual

### 1. Jogador comum

Perfil:

- não está em `configs/admins.json`
- não recebe grupo administrativo do CounterStrikeSharp
- não deve acessar a camada administrativa do servidor

Superfície esperada:

- comandos normais de jogador/time no MatchZy
- comandos dependentes do estado atual da partida
- nenhuma superfície administrativa de servidor

### 2. Admin pleno do HSC (`#hsc/root`)

Perfil:

- está em `configs/admins.json`
- recebe `#hsc/root`
- herda flags administrativas `@css/*`
- deve conseguir operar tanto o CS2-SimpleAdmin quanto o MatchZy administrativo

Superfície esperada:

- moderação do servidor
- mensagens administrativas
- map/cvar/rcon
- fun commands permitidos pelo grupo
- comandos administrativos de MatchZy

### 3. Perfis intermediários

Estado atual:

- ainda **não fazem parte** da baseline viva do HSC
- podem ser introduzidos depois, se houver necessidade real

Exemplos futuros possíveis:

- `#hsc/mod`
- `#hsc/match-admin`
- `#hsc/fun`

---

## Camada CounterStrikeSharp

### Papel operacional

A camada CounterStrikeSharp não é a superfície principal de comando do staff no dia a dia.  
Seu papel principal é:

- resolver autoridade
- carregar admins e grupos
- sustentar permissões usadas pelos plugins
- permitir overrides de comando quando necessário

### Quando pensar na camada CounterStrikeSharp

Usar essa camada conceitualmente quando a pergunta for:

- quem pode ou não pode usar um comando?
- qual grupo concede determinada capacidade?
- um plugin está lendo autoridade do lugar certo?
- precisamos introduzir permissão refinada por comando?

---

## Camada CS2-SimpleAdmin — categorias de comando

### 1. Informação e painel administrativo

Comandos típicos:

- `css_admin`
- `css_adminhelp`
- `css_who`
- `css_players`
- `css_disconnected`
- `css_penalties`
- `css_warns`

Quando usar:

- para abrir painel administrativo
- para inspecionar usuários conectados ou recentes
- para consultar situação disciplinar/comunicacional
- para orientar uma intervenção administrativa antes de agir

Quando evitar:

- não usar como substituto de governança de cadastro persistente

### 2. Governança local do ecossistema SimpleAdmin

Comandos disponíveis:

- `css_addadmin`
- `css_deladmin`
- `css_addgroup`
- `css_delgroup`
- `css_reloadadmins`
- `css_reloadbans`
- `css_pluginsmanager`

Leitura canônica do HSC:

esses comandos **existem**, mas não devem ser o workflow oficial de governança de admins do HSC.

Quando usar:

- `css_reloadbans` pode ser útil operacionalmente
- `css_pluginsmanager` pode ser útil para staff técnico em troubleshooting local

Quando evitar:

- evitar `css_addadmin`, `css_deladmin`, `css_addgroup` e `css_delgroup` como rotina persistente de governança
- no HSC, cadastro e grupos devem nascer no CounterStrikeSharp

### 3. Moderação disciplinar

Comandos típicos:

- `css_kick`
- `css_ban`
- `css_addban`
- `css_banip`
- `css_unban`
- `css_warn`
- `css_unwarn`
- `css_gag`
- `css_addgag`
- `css_ungag`
- `css_mute`
- `css_addmute`
- `css_unmute`
- `css_silence`
- `css_addsilence`
- `css_unsilence`

Quando usar:

- infração clara de conduta
- abuso de chat/voz
- necessidade de retirada do jogador da sessão
- sanção temporária ou permanente conforme regra operacional do servidor

Quando evitar:

- evitar sanção sem contexto mínimo
- evitar uso impulsivo em ambiente de scrim organizada sem staff alinhado

### 4. Comunicação administrativa

Comandos típicos:

- `css_asay`
- `css_say`
- `css_psay`
- `css_csay`
- `css_hsay`
- `css_cssay`
- `css_adminvoice`

Quando usar:

- comunicar decisão de staff
- orientar jogadores durante problema operacional
- coordenar staff em sessão viva
- transmitir aviso curto e claro

Quando evitar:

- evitar spam administrativo
- evitar poluir HUD/chat com mensagens desnecessárias

### 5. Controle do servidor

Comandos típicos:

- `css_map`
- `css_wsmap`
- `css_rr`
- `css_cvar`
- `css_rcon`

Quando usar:

- mudança de mapa
- restart de round/game
- ajuste de configuração técnica
- execução de operação administrativa legítima do servidor

Quando evitar:

- evitar `rcon` e `cvar` sem necessidade técnica clara
- evitar alteração de estado do servidor durante match sensível sem coordenação

### 6. Intervenção sobre jogador/entidade

Comandos típicos:

- `css_slay`
- `css_slap`
- `css_team`
- `css_rename`
- `css_prename`
- `css_tp`
- `css_bring`
- `css_resize`
- `css_hide`
- `css_hidecomms`

Quando usar:

- ajuste administrativo pontual
- correção de estado de jogador
- troubleshooting de sessão
- ocultação de staff quando a política do momento pedir

Quando evitar:

- evitar intervenção lúdica fora de contexto
- evitar uso invasivo sem necessidade operacional

---

## Camada Fun Commands — categorias de comando

### Papel operacional

O módulo Fun Commands amplia a intervenção do staff no runtime.

Comandos típicos confirmados no baseline:

- `css_noclip`
- `css_god`
- `css_freeze`
- `css_unfreeze`
- `css_respawn`
- `css_give`
- `css_strip`
- `css_hp`
- `css_speed`
- `css_gravity`
- `css_money`

### Quando usar

- troubleshooting local
- sessão casual controlada
- preparação ou ajuste em ambiente não competitivo estrito
- recuperação rápida de estado em testes internos

### Quando evitar

- evitar em match competitiva real sem consenso
- evitar como hábito em ambiente que deveria permanecer próximo do competitivo padrão

### Leitura canônica do HSC

Esses comandos existem e fazem parte da superfície do staff root, mas devem ser tratados como **ferramenta de intervenção**, não como baseline de operação competitiva.

---

## Camada Stealth Module

### Papel operacional

Permitir ocultação ou discrição de staff conforme a política do módulo e a permissão associada.

Comandos típicos / superfície associada:

- `css_hide`
- `css_stealth`
- `css_hidecomms`

### Quando usar

- observação discreta de sessão
- redução de ruído administrativo
- necessidade tática de staff acompanhar sem chamar atenção

### Quando evitar

- evitar como rotina sem motivo
- evitar uso que gere ambiguidade operacional para o próprio staff

---

## Camada MatchZy — perfis e categorias

O MatchZy precisa ser lido em duas metades.

### 1. Comandos de jogador/time

Comandos importantes do baseline:

- `.ready`
- `.unready`
- `.pause`
- `.unpause`
- `.stay`
- `.switch`
- `.stop`
- `.coach`
- `.uncoach`
- `.tech`

Quando usar:

- fluxo normal da sessão competitiva
- coordenação entre times
- pausa operacional
- ready flow
- coach quando o modo da sessão comportar isso

Observação importante sobre `.tech`:

na baseline atual, o MatchZy está configurado com `matchzy_tech_pause_flag ""`, então a leitura canônica é que o `.tech` está sem flag extra específica e tende a operar como comando aberto no contexto atual do plugin.

### 2. Comandos administrativos de partida

Comandos importantes do baseline:

- `.whitelist`
- `.roundknife`
- `.rk`
- `.playout`
- `.readyrequired`
- `.forceready`
- `.forcepause`
- `.forceunpause`
- `.match`
- `.loadmatch`
- `.start`
- `.restart`
- `.endmatch`
- `.prac`
- `.exitprac`
- `.rcon`
- `.skipveto`
- `.reload_admins`

Quando usar:

- iniciar ou forçar estado de partida
- ajustar fluxo competitivo quando a sessão exige intervenção administrativa
- mudar regra de ready flow
- controlar whitelist da sessão
- alternar knife round e playout
- entrar ou sair de prática

Quando evitar:

- evitar usar comandos administrativos de MatchZy sem necessidade clara
- evitar `forceready` como substituto de boa coordenação quando a sessão pode ser conduzida normalmente
- evitar `rcon` fora de necessidade técnica explícita

### Observação sobre dependência de estado

Nem todo comando do MatchZy falha por permissão quando “não faz nada”.  
Alguns dependem do estado atual da partida.

Exemplo operacional importante:

- `.forceready` pode não produzir o efeito esperado quando o contexto de ready flow ainda não está maduro ou quando a sessão não preenche o requisito mínimo de jogadores prontos

Regra prática:

**em MatchZy, ausência de efeito nem sempre significa ausência de autoridade.**

---

## Superfícies avançadas de prática do MatchZy

Além da camada competitiva principal, o MatchZy instalado também expõe uma superfície ampla de prática e utilitários, incluindo famílias como:

- nades (`loadnade`, `savenade`, `listnades`, `importnade`)
- rethrow (`rethrow`, `rethrowflash`, `rethrowsmoke`, etc.)
- posição (`savepos`, `loadpos`, `watchme`)
- bots e testes (`bot`, `nobots`)
- spawn visualization (`showspawns`, `hidespawns`)
- dry/practice utilities (`dry`, `dryrun`, `timer`, etc.)

Leitura canônica:

essa superfície existe, mas não deve ser confundida com a camada administrativa mínima que todo staff precisa dominar.

---

## Comando disponível vs comando canônico no HSC

Esta distinção é crítica.

### Tecnicamente disponível

O comando existe no plugin carregado e pode responder no runtime.

### Canonicamente recomendado

O comando faz parte do workflow sustentado pelo modelo administrativo do HSC.

Exemplos:

- `css_addadmin` → tecnicamente disponível, mas **não** canonicamente recomendado para governança persistente
- `.whitelist` → tecnicamente disponível e canonicamente recomendado para teste administrativo do MatchZy
- `!css_admin` → tecnicamente disponível e canonicamente recomendado como smoke test do SimpleAdmin
- `css_rcon` → disponível, mas de uso restrito e criterioso

---

## Perfil atual do HSC

No baseline de hoje, o HSC opera essencialmente com um perfil ativo forte:

- `#hsc/root`

Isso significa que, no estado atual:

- não há segmentação fina por papel intermediário
- o staff ativo tem uma superfície ampla de comando
- a sustentabilidade depende mais de disciplina operacional do que de compartimentalização de permissões

Leitura correta:

isso é aceitável enquanto o número de admins for pequeno e o contexto de operação continuar controlado.

---

## Recomendações de uso por contexto

### Sessão competitiva real

Priorizar:

- MatchZy de fluxo normal
- MatchZy administrativo quando necessário
- SimpleAdmin apenas para intervenção de staff realmente necessária

Evitar:

- Fun Commands desnecessários
- rcon/cvar sem necessidade
- mudanças bruscas no servidor durante a sessão

### Troubleshooting interno

Priorizar:

- `!css_admin`
- `css_players`
- `css_who`
- `css_cvar`
- `css_rcon`
- `.whitelist`
- `.prac`
- comandos utilitários pontuais

### Gestão disciplinar

Priorizar:

- warn, kick, gag, mute, silence, ban
- mensagens administrativas claras
- registro e consistência de critério

### Testes de baseline administrativa

Priorizar:

- `!css_admin`
- `.whitelist`
- validação de carga de `admins.json` e `admin_groups.json`

---

## Resumo executivo

A superfície de comando do HSC no lado jogo ficou organizada assim:

- CounterStrikeSharp → autoridade e política
- CS2-SimpleAdmin → administração do servidor
- Fun Commands / Stealth → extensões controladas da administração
- MatchZy → administração da partida e fluxo competitivo

O comando certo depende de:

- quem é o usuário
- em que camada o problema existe
- se o objetivo é governança, moderação, troubleshooting ou controle da partida

Esse é o mapa que deve orientar o uso prático dos comandos daqui para frente.
