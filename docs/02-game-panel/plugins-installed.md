# Plugins Installed

## Objetivo

Documentar a camada de plugins instalada no contexto Game Panel do ecossistema HSC, registrando o inventário real reconciliado do runtime do servidor CS2, os plugins atualmente carregados em produção, seus diretórios no host e os pontos ainda pendentes de validação fina.

Este documento existe para registrar, de forma estável e auditável:

- quais plugins estão realmente carregados hoje no servidor
- quais nomes, versões, autores e descrições foram observados diretamente no runtime
- como o inventário carregado se relaciona com os diretórios presentes no host
- quais plugins são centrais para operação competitiva, administração e persistência auxiliar
- quais riscos existem quando inventário no disco e inventário carregado divergem
- como validar rapidamente essa camada no ambiente real

---

## Escopo

Este documento cobre:

- o inventário real de plugins carregados no runtime
- a distinção entre CounterStrikeSharp e plugins efetivamente carregados
- diretórios reconciliados de plugins no host
- plugins centrais, auxiliares e de administração
- dependências operacionais da camada de plugins
- comandos de validação
- riscos e problemas comuns ligados a carregamento, compatibilidade e drift de inventário

Este documento não cobre em profundidade:

- configuração linha a linha de cada plugin
- runbooks detalhados de BO1, BO3, veto, coach e warmup
- troubleshooting profundo do AMP
- pipeline ETL do Portal Estático
- contratos JSON da Static API v2
- Auth API no AWS Lightsail

Esses tópicos vivem em documentos próprios dos contextos `01-infra-hostinger`, `02-game-panel` e `03-portal-estatico`.

---

## Estado atual

O estado operacional conhecido e reconciliado da camada de plugins do servidor HSC é:

- o runtime do servidor usa CounterStrikeSharp como base de plugins
- o inventário real carregado em produção foi validado diretamente via `css_plugins list`
- o servidor possui atualmente **7 plugins carregados**
- o MatchZy segue sendo o plugin central do fluxo competitivo
- o WeaponPaints segue ativo como plugin de personalização e persistência auxiliar
- a camada administrativa do servidor está atualmente centrada no ecossistema **CS2-SimpleAdmin**
- existem dois plugins core relevantes carregados:
  - `PlayerSettings [Core]`
  - `MenuManager [Core]`

Leitura canônica:

- o inventário real já não está mais apenas parcialmente presumido
- ele já pode ser tratado como reconciliado para os plugins atualmente carregados
- ainda podem existir arquivos e diretórios auxiliares no disco, mas o runtime validado é a principal fonte de verdade

---

## Source of truth / evidências

As principais evidências deste documento, nesta fase de reconciliação, são:

- documentação consolidada atual do ecossistema HSC
- blueprint técnico consolidado do HSC
- documentação reconciliada da camada AMP/CS2
- saída direta de `css_plugins list`
- inspeção real dos diretórios em:
  - `/home/amp/.ampdata/instances/MixHAXIXE01/counter-strike2/730/game/csgo/addons/counterstrikesharp/plugins`
- evidência reconciliada da presença de subárvores de dados, runtimes, linguagem e persistência auxiliar no host

Enquanto o runtime real permanecer neste formato, a saída do `css_plugins list` prevalece como source of truth principal da camada.

---

## Relações com outros documentos

Este arquivo é complementar a:

- `docs/02-game-panel/README.md`
- `docs/02-game-panel/game-panel-architecture-runtime.md`
- `docs/02-game-panel/instance-mixhaxixe01.md`
- `docs/02-game-panel/cs2-server-configuration.md`
- `docs/02-game-panel/matchzy.md`
- `docs/02-game-panel/mariadb-runtime.md`
- `docs/02-game-panel/operational-runbooks.md`
- `docs/02-game-panel/observability-troubleshooting.md`
- `docs/02-game-panel/game-panel-references-inventory.md`

Este documento descreve o inventário real e o papel da camada de plugins.  
Ele não substitui os documentos de MatchZy, configuração do servidor, runbooks ou troubleshooting aprofundado.

---

## CounterStrikeSharp não é “mais um plugin”

O CounterStrikeSharp deve ser tratado como a base do ecossistema de plugins do lado jogo.

Isso significa:

- ele não é apenas mais um item do inventário
- ele é a camada sobre a qual plugins como MatchZy, WeaponPaints e CS2-SimpleAdmin operam
- falhas nessa base podem aparecer como “falha de plugin”, mesmo quando o plugin em si não é a causa raiz

Regra canônica:

- CounterStrikeSharp é framework/base de runtime
- os plugins carregados são componentes que vivem sobre essa base

---

## Inventário real de plugins carregados

A saída reconciliada de `css_plugins list` no runtime atual é:

### 1. WeaponPaints

- Nome no runtime: `WeaponPaints`
- Versão: `3.2b`
- Autor: `Nereziel & daffyy`
- Descrição reconciliada:
  - `Skin, gloves, agents and knife selector, standalone and web-based`

Papel:
- personalização visual de skins
- gloves, agents e knife selector
- plugin auxiliar com persistência/configuração própria
- parte real da experiência dos jogadores no servidor

Diretório reconciliado no host:
- `WeaponPaints/`

Subárvores observadas:
- `WeaponPaints/data`
- `WeaponPaints/gamedata`
- `WeaponPaints/lang`
- `WeaponPaints/runtimes`

---

### 2. PlayerSettings [Core]

- Nome no runtime: `PlayerSettings [Core]`
- Versão: `0.9.3`
- Autor: `Nick Fox`
- Descrição reconciliada:
  - `One storage for player's settings (aka ClientCookies)`

Papel:
- storage unificado de settings dos jogadores
- base de “client cookies” / persistência de preferências
- plugin core de suporte a outros componentes do runtime

Diretório reconciliado no host:
- `PlayerSettings/`

Leitura canônica:
- este plugin deve ser tratado como core auxiliar de persistência de preferências do jogador

---

### 3. CS2-SimpleAdmin Fun Commands

- Nome no runtime: `CS2-SimpleAdmin Fun Commands`
- Versão: `1.0.0`
- Autor: `Your Name`
- Descrição reconciliada:
  - `Fun commands extension for CS2-SimpleAdmin`

Papel:
- extensão funcional do ecossistema CS2-SimpleAdmin
- comandos recreativos / auxiliares de administração
- plugin dependente da presença da camada SimpleAdmin

Diretório reconciliado no host:
- `CS2-SimpleAdmin_FunCommands/`

Subárvore observada:
- `CS2-SimpleAdmin_FunCommands/lang`

Observação importante:
- o autor exibido no runtime foi reconciliado exatamente como retornado pelo servidor
- qualquer estranheza nesse metadado deve ser tratada como detalhe do próprio plugin/build, não como erro documental

---

### 4. MatchZy

- Nome no runtime: `MatchZy`
- Versão: `0.8.15`
- Autor: `WD- (https://github.com/shobhit-pathak/)`
- Descrição reconciliada:
  - `A plugin for running and managing CS2 practice/pugs/scrims/matches!`

Papel:
- plugin competitivo central do ecossistema
- gestão de practice, pugs, scrims e matches
- produtor do `matchzy.db`
- ponte entre runtime do jogo e cadeia pública de dados do portal

Diretório reconciliado no host:
- `MatchZy/`

Subárvores observadas:
- `MatchZy/lang`
- `MatchZy/runtimes`
- `MatchZy/spawns`

Leitura canônica:
- este continua sendo o plugin mais estrutural do contexto `02-game-panel`

---

### 5. MenuManager [Core]

- Nome no runtime: `MenuManager [Core]`
- Versão: `1.4.1`
- Autor: `Nick Fox`
- Descrição reconciliada:
  - `All menus interacts in one core`

Papel:
- core de menus do runtime
- base de interação unificada de menus
- plugin de infraestrutura funcional para outros componentes

Diretório reconciliado no host:
- `MenuManagerCore/`

Subárvore observada:
- `MenuManagerCore/lang`

Observação importante:
- existe divergência nominal natural entre:
  - nome exibido no runtime: `MenuManager [Core]`
  - diretório no disco: `MenuManagerCore`
- isso não deve ser tratado como inconsistência; é apenas diferença entre display name e diretório técnico

---

### 6. [CS2-SimpleAdmin] Stealth Module

- Nome no runtime: `[CS2-SimpleAdmin] Stealth Module`
- Versão: `v1.0.2`
- Autor: `daffyy`

Papel:
- módulo auxiliar do ecossistema CS2-SimpleAdmin
- extensão administrativa especializada
- parte da malha real de administração do servidor

Diretório reconciliado no host:
- `CS2-SimpleAdmin_StealthModule/`

Leitura canônica:
- este plugin deve ser tratado como módulo auxiliar da camada administrativa, e não como sistema isolado

---

### 7. CS2-SimpleAdmin (RELEASE)

- Nome no runtime: `CS2-SimpleAdmin (RELEASE)`
- Versão: `1.7.8-beta-10b`
- Autor: `daffyy`
- Descrição reconciliada:
  - `Simple admin plugin for Counter-Strike 2 :)`

Papel:
- camada administrativa principal hoje reconciliada no servidor
- base das capacidades de admin do runtime
- plugin estrutural para governança prática da sessão e administração do servidor

Diretório reconciliado no host:
- `CS2-SimpleAdmin/`

Subárvores observadas:
- `CS2-SimpleAdmin/data`
- `CS2-SimpleAdmin/Database`
- `CS2-SimpleAdmin/lang`
- `CS2-SimpleAdmin/runtimes`

Leitura canônica:
- esta camada já não deve ser tratada como mera hipótese; ela está claramente ativa e central na administração do servidor

---

## Resumo do inventário carregado

O runtime atual do servidor possui **7 plugins carregados**, distribuídos conceitualmente assim:

### Competitivo

- `MatchZy` (`0.8.15`)

### Personalização / persistência auxiliar

- `WeaponPaints` (`3.2b`)
- `PlayerSettings [Core]` (`0.9.3`)

### Administração

- `CS2-SimpleAdmin (RELEASE)` (`1.7.8-beta-10b`)
- `[CS2-SimpleAdmin] Stealth Module` (`v1.0.2`)
- `CS2-SimpleAdmin Fun Commands` (`1.0.0`)

### Infraestrutura funcional de UI / menus

- `MenuManager [Core]` (`1.4.1`)

---

## Diretórios reconciliados no host

A inspeção do host reconciliou os seguintes diretórios relevantes sob:

```text
/home/amp/.ampdata/instances/MixHAXIXE01/counter-strike2/730/game/csgo/addons/counterstrikesharp/plugins
```

Diretórios observados:

- `CS2-SimpleAdmin/`
- `CS2-SimpleAdmin_FunCommands/`
- `CS2-SimpleAdmin_StealthModule/`
- `MatchZy/`
- `MenuManagerCore/`
- `PlayerSettings/`
- `WeaponPaints/`
- `gamedata/`

Leitura canônica:

- todos os plugins carregados reconciliados possuem correspondente materializado no disco
- além deles, existe também um diretório genérico `gamedata/`, que deve ser tratado como suporte técnico do ecossistema de plugins, e não como plugin carregado independente

---

## Relação entre inventário carregado e inventário no disco

A regra canônica desta camada passa a ser:

### Fonte principal da verdade

- `css_plugins list`

### Fonte de apoio estrutural

- diretórios presentes no filesystem de plugins

Isso significa:

- plugin em runtime vale mais do que plugin apenas presente no disco
- diretório no disco sem presença no runtime não basta para promover algo a “plugin ativo”
- por outro lado, o disco continua importante para troubleshooting e inventário estrutural

---

## Persistência auxiliar observada

Além do `matchzy.db`, o inventário em disco mostrou sinais de persistência/configuração auxiliar em:

### CS2-SimpleAdmin

Subárvores observadas:
- `CS2-SimpleAdmin/data`
- `CS2-SimpleAdmin/Database`

### WeaponPaints

Subárvores observadas:
- `WeaponPaints/data`
- `WeaponPaints/gamedata`

### PlayerSettings

Leitura funcional reconciliada:
- storage central de settings de jogadores

Leitura canônica:

- já existe evidência suficiente para afirmar que o lado jogo possui persistências auxiliares além do `matchzy.db`
- isso **não** muda o fato de que a persistência estrutural principal da cadeia pública continua sendo o SQLite do MatchZy
- mas agora o contexto já deve tratar a camada administrativa e de preferências como tendo storage auxiliar real

---

## Relação com `mariadb-runtime.md`

Este inventário não prova, por si só, que a camada MariaDB local do Game Panel seja central.

Mas ele reforça que:

- existem persistências auxiliares além do banco principal do MatchZy
- existe pelo menos uma subárvore `Database` ligada ao ecossistema administrativo
- a pergunta sobre backend relacional auxiliar do lado jogo continua legítima
- essa camada ainda precisa ser tratada com honestidade editorial, sem promoção automática a pilar estrutural do contexto

Leitura correta:

- `matchzy.db` continua sendo a persistência estrutural principal
- persistências auxiliares administrativas/cosméticas existem
- a natureza exata dessas persistências ainda pode exigir reconciliação adicional

---

## Dependências da camada de plugins

A camada de plugins depende, no mínimo, de:

- runtime do servidor CS2 saudável
- CounterStrikeSharp saudável
- instância `MixHAXIXE01` íntegra
- paths e permissões coerentes
- configs compatíveis
- ausência de conflito destrutivo entre plugins
- operadores entendendo o comportamento dos componentes realmente carregados

Dependências específicas importantes:

### MatchZy
- depende da base de plugins
- depende do runtime competitivo
- depende da persistência local

### WeaponPaints
- depende do runtime de plugin saudável
- depende de persistência/configuração coerente
- depende de assets/dados auxiliares reais em disco

### CS2-SimpleAdmin
- depende da base CounterStrikeSharp
- depende de sua própria árvore de dados e módulos auxiliares
- sustenta parte essencial da administração do servidor

### PlayerSettings + MenuManager
- dependem do runtime base
- funcionam como camadas core auxiliares para experiência e integração de outros plugins

---

## Invariantes operacionais

Os invariantes conhecidos desta camada incluem:

- o servidor possui atualmente **7 plugins carregados**
- o inventário carregado principal é:
  - WeaponPaints
  - PlayerSettings [Core]
  - CS2-SimpleAdmin Fun Commands
  - MatchZy
  - MenuManager [Core]
  - [CS2-SimpleAdmin] Stealth Module
  - CS2-SimpleAdmin (RELEASE)
- MatchZy continua sendo o componente competitivo central
- WeaponPaints continua sendo o plugin de personalização mais relevante
- CS2-SimpleAdmin já deve ser tratado como camada administrativa real e ativa
- `css_plugins list` é a fotografia principal do runtime
- diretório no disco sem presença no runtime não deve ser tratado automaticamente como plugin ativo
- existem sinais reais de persistência auxiliar além do `matchzy.db`

Esses invariantes ajudam a manter a documentação fiel ao estado real do servidor.

---

## Sinais de saúde da camada

Os sinais de saúde esperados incluem:

- `css_plugins list` refletindo exatamente o inventário carregado esperado
- MatchZy carregado e funcional
- WeaponPaints carregado quando a personalização estiver em uso
- CS2-SimpleAdmin carregado e utilizável
- PlayerSettings e MenuManager carregados como bases auxiliares
- ausência de erro estrutural recorrente de plugin
- coerência entre plugin carregado e diretório correspondente no disco
- persistência auxiliar funcionando quando o plugin depende dela

A saúde dessa camada deve ser inferida tanto pelo inventário quanto pelo comportamento real do servidor.

---

## Comandos de validação

Os comandos abaixo representam verificações típicas da camada de plugins.

### Listar plugins carregados

```bash
css_plugins list
```

### Inspecionar diretórios de plugins no host

```bash
find /home/amp/.ampdata/instances/MixHAXIXE01/counter-strike2/730/game/csgo/addons/counterstrikesharp/plugins -maxdepth 2 -type d | sort
```

### Validar MatchZy no runtime

```bash
css_plugins list
```

### Validar MatchZy no disco

```bash
ls -lah /home/amp/.ampdata/instances/MixHAXIXE01/counter-strike2/730/game/csgo/addons/counterstrikesharp/plugins/MatchZy
```

### Validar WeaponPaints no runtime

```bash
css_plugins list
```

### Validar WeaponPaints no disco

```bash
ls -lah /home/amp/.ampdata/instances/MixHAXIXE01/counter-strike2/730/game/csgo/addons/counterstrikesharp/plugins/WeaponPaints
```

### Validar CS2-SimpleAdmin no runtime

```bash
css_plugins list
```

### Validar CS2-SimpleAdmin no disco

```bash
ls -lah /home/amp/.ampdata/instances/MixHAXIXE01/counter-strike2/730/game/csgo/addons/counterstrikesharp/plugins/CS2-SimpleAdmin
```

### Validar persistência estrutural do MatchZy

```bash
sqlite3 /home/amp/.ampdata/instances/MixHAXIXE01/counter-strike2/730/game/csgo/addons/counterstrikesharp/plugins/MatchZy/matchzy.db ".tables"
```

Regra prática:

- o comando principal de inventário é `css_plugins list`
- o host ajuda a confirmar materialização e subárvores auxiliares
- troubleshooting sério deve sempre olhar runtime e disco em conjunto

---

## Problemas comuns

### 1. Plugin existe no disco, mas não aparece em `css_plugins list`

Causas comuns:

- plugin não carregou
- incompatibilidade de runtime
- problema na base CounterStrikeSharp
- arquivo presente, mas instalação incompleta

Impacto:
- funcionalidade do plugin não está realmente ativa

---

### 2. Plugin aparece como carregado, mas não funciona

Causas comuns:

- configuração incorreta
- persistência/configuração auxiliar quebrada
- dependência de core ausente ou degradada
- conflito com outro plugin

Impacto:
- falsa percepção de saúde
- inventário parece correto, mas o comportamento continua ruim

---

### 3. MatchZy saudável, mas camada administrativa ruim

Causas comuns:

- problema em CS2-SimpleAdmin
- problema em módulos auxiliares
- permissões/admins mal alinhados
- persistência auxiliar degradada

Impacto:
- fluxo competitivo pode até existir
- mas a operação prática do servidor degrada

---

### 4. Plugin core cai e vários sintomas aparecem ao mesmo tempo

Causas comuns:

- falha em `PlayerSettings [Core]`
- falha em `MenuManager [Core]`
- problema estrutural da base de plugins

Impacto:
- sintomas “espalhados”
- falsa suspeita de múltiplos bugs independentes

---

### 5. Inventário carregado muda sem atualização documental

Causas comuns:

- plugin adicionado/removido sem reconciliação
- update de runtime
- manutenção informal no servidor

Impacto:
- documentação perde aderência
- troubleshooting fica mais lento
- runbooks passam a assumir ambiente errado

---

## Itens pendentes de confirmação

Os itens abaixo ainda podem ser refinados diretamente no ambiente real para aumentar o grau de precisão do contexto.

### 1. Inventário de arquivos específicos por plugin

A lista de diretórios já está reconciliada, mas uma fotografia mais detalhada de DLLs/configs por plugin ainda pode ser útil em troubleshooting futuro.

### 2. Natureza exata da subárvore `CS2-SimpleAdmin/Database`

Já há evidência suficiente de persistência auxiliar administrativa, mas o papel exato dessa camada ainda pode ser detalhado melhor no futuro.

### 3. Relação operacional fina entre `PlayerSettings [Core]` e outros plugins

O papel conceitual está claro, mas a malha exata de dependências entre PlayerSettings, MenuManager e plugins auxiliares ainda pode ser refinada.

### 4. Eventuais plugins instalados no disco, mas não carregados

A inspeção atual foi suficiente para o inventário vivo, mas uma auditoria futura ainda pode mapear artefatos órfãos ou componentes stale no disco.

---

## Limites deste documento

Este documento não detalha:

- configuração individual linha a linha de cada plugin
- comandos operacionais internos de cada plugin
- troubleshooting profundo de CS2-SimpleAdmin
- troubleshooting profundo de WeaponPaints
- engenharia reversa da persistência auxiliar administrativa
- todas as dependências internas entre plugins core e plugins finais

Esses tópicos pertencem a documentos específicos do contexto ou exigem reconciliação adicional.

---

## Critério de pronto deste documento

Este documento pode ser considerado maduro quando:

- o inventário real carregado estiver explícito sem ambiguidade
- nomes, versões e autores estiverem reconciliados com o runtime atual
- os diretórios correspondentes no host estiverem corretamente posicionados
- a camada administrativa baseada em CS2-SimpleAdmin estiver formalizada
- os riscos de inventário stale estiverem bem representados
- ele puder ser usado como referência canônica da camada de plugins do Game Panel sem depender do master legado

---

## Última revisão

- Status: ativo
- Classificação: canônico
- Contexto: game panel / plugins installed
- Última revisão: 2026-03-18