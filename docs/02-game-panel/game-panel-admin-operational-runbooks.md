# Admin Operational Runbooks

## Objetivo

Documentar os runbooks canônicos de manutenção da autoridade administrativa do contexto Game Panel do HSC após a equalização baseada no CounterStrikeSharp.

Este documento existe para registrar, de forma operacionalmente útil:

- como manter a baseline administrativa de forma sustentável
- como adicionar, editar e remover admins
- como criar ou alterar grupos administrativos
- como validar mudanças sem reintroduzir drift
- como reagir quando a autoridade aparente divergir do runtime

---

## Escopo

Este documento cobre:

- manutenção de `admins.json`
- manutenção de `admin_groups.json`
- uso recomendado de `admin_overrides.json`
- validação após mudança administrativa
- boas práticas de governança de staff do lado jogo

Este documento não cobre em profundidade:

- implementação linha a linha de cada plugin
- troubleshooting amplo do AMP
- fluxo competitivo de MatchZy em detalhes
- inventário completo de comandos

Esses tópicos vivem em documentos especializados deste contexto.

---

## Estado atual

A baseline administrativa atual do HSC é simples:

- fonte de verdade: CounterStrikeSharp
- grupos vivos: `admin_groups.json`
- admins vivos: `admins.json`
- MatchZy sem lista paralela de admins
- CS2-SimpleAdmin sem cadastro paralelo vivo de admins/grupos
- SimpleAdmin operando sobre SQLite local
- recarga automática por mudança de mapa desabilitada no SimpleAdmin

Leitura operacional importante:

mudança administrativa no HSC deve ser pensada como **mudança de baseline + restart + smoke test**.

---

## Source of truth / evidências

A baseline operacional deste documento se apoia em fatos já validados no runtime:

- CounterStrikeSharp carregando `admins.json` no boot
- CounterStrikeSharp carregando `admin_groups.json` no boot
- CS2-SimpleAdmin funcional via `!css_admin`
- MatchZy funcional via `.whitelist`
- `cfg/MatchZy/admins.json` vazio
- `CS2-SimpleAdmin/data/admins.json` vazio
- `CS2-SimpleAdmin/data/groups.json` vazio
- `DatabaseType: SQLite` no `CS2-SimpleAdmin.json`
- `ReloadAdminsEveryMapChange: false` no `CS2-SimpleAdmin.json`

---

## Relações com outros documentos

Este arquivo é complementar a:

- `docs/02-game-panel/game-panel-admin-authority-runtime.md`
- `docs/02-game-panel/game-panel-admin-command-profiles.md`
- `docs/02-game-panel/game-panel-admin-references-inventory.md`
- `docs/02-game-panel/plugins-installed.md`
- `docs/02-game-panel/game-panel-observability-troubleshooting.md`
- `docs/02-game-panel/game-panel-operational-runbooks.md`

Este documento descreve **como operar e manter** a autoridade administrativa do lado jogo.  
Ele não substitui a arquitetura do modelo nem o inventário de comandos.

---

## Princípios operacionais

Os runbooks administrativos do HSC seguem estes princípios:

- sempre mudar a autoridade na origem canônica
- nunca confiar em estado implícito de plugin como cadastro persistente
- sempre preferir grupos a flags repetidas por usuário
- nunca tratar comandos administrativos de plugin como substitutos de governança de arquivo
- sempre validar o runtime após mudanças
- sempre preservar simplicidade antes de sofisticar o modelo com novos perfis

Regra de ouro:

**no HSC, governança de admin não é improviso de runtime; é baseline explícita de configuração.**

---

## Runbook de manutenção normal

### Objetivo

Manter a autoridade administrativa coerente com o modelo canônico.

### Arquivos-alvo

- `addons/counterstrikesharp/configs/admins.json`
- `addons/counterstrikesharp/configs/admin_groups.json`
- `addons/counterstrikesharp/configs/admin_overrides.json` (somente quando necessário)

### Arquivos que não devem ser usados como origem

- `cfg/MatchZy/admins.json`
- `addons/counterstrikesharp/plugins/CS2-SimpleAdmin/data/admins.json`
- `addons/counterstrikesharp/plugins/CS2-SimpleAdmin/data/groups.json`
- SQLite do SimpleAdmin como workflow de cadastro persistente de admins

### Sequência canônica

1. tirar backup do arquivo que será alterado
2. editar a baseline no CounterStrikeSharp
3. salvar a mudança com JSON válido
4. reiniciar a instância pelo Game Panel
5. validar logs de carga administrativa
6. executar smoke tests em jogo

---

## Runbook de adicionar admin

### Objetivo

Adicionar um novo admin ao HSC sem reintroduzir drift.

### Caminho correto

Adicionar a entrada em `configs/admins.json` apontando para grupo(s) existente(s).

### Exemplo de modelagem recomendada

```json
{
  "Novo Admin": {
    "identity": "7656119XXXXXXXXXX",
    "groups": [
      "#hsc/root"
    ]
  }
}
```

### Boas práticas

- usar o identificador Steam64 em `identity`
- usar grupos em vez de repetir flags por usuário, sempre que possível
- manter o nome da chave como rótulo humano legível
- evitar flags inline duplicadas enquanto a baseline for simples

### Após adicionar

- reiniciar a instância
- validar carga de `admins.json` e `admin_groups.json`
- validar `!css_admin`
- validar `.whitelist`

---

## Runbook de editar admin

### Objetivo

Alterar um cadastro administrativo sem criar múltiplas fontes de verdade.

### Casos comuns

#### Trocar nickname/document label

Pode-se alterar a chave humana do objeto sem alterar a identidade real, desde que `identity` continue correto.

#### Trocar SteamID

Troca crítica. Deve ser feita com atenção e validada imediatamente após restart.

#### Trocar perfil/grupo

Deve ser feita mudando `groups` no `admins.json`, e não criando autoridade local no plugin.

### Exemplo

```json
{
  "Admin Moderado": {
    "identity": "7656119YYYYYYYYYY",
    "groups": [
      "#hsc/mod"
    ]
  }
}
```

---

## Runbook de remover admin

### Objetivo

Remover autoridade de forma limpa e definitiva.

### Caminho correto

- remover a entrada correspondente de `configs/admins.json`
- salvar JSON válido
- reiniciar a instância
- validar que o usuário perdeu acesso aos comandos de staff

### Regra importante

Não remover admin apenas do MatchZy ou apenas do SimpleAdmin.  
No HSC, a remoção canônica nasce em `admins.json`.

---

## Runbook de criar novo grupo administrativo

### Objetivo

Introduzir perfis intermediários sem quebrar a simplicidade da baseline.

### Caminho correto

Adicionar um novo grupo em `configs/admin_groups.json`.

### Exemplo

```json
{
  "#hsc/match-admin": {
    "flags": [
      "@css/generic",
      "@css/chat",
      "@css/config"
    ],
    "immunity": 60
  }
}
```

### Critérios para criar grupo novo

Só criar grupo novo quando houver necessidade operacional real, por exemplo:

- separar moderação de administração total
- separar admin de partida de manutenção técnica do servidor
- delegar Fun Commands sem conceder root pleno

### O que evitar

- criar muitos grupos sem necessidade prática
- usar grupos com semântica ambígua
- replicar `#hsc/root` com nomes diferentes mas mesmo efeito

---

## Runbook de alterar flags de um grupo

### Objetivo

Refinar o alcance de um perfil sem editar dezenas de usuários individualmente.

### Caminho correto

Alterar `flags` dentro de `configs/admin_groups.json`.

### Quando usar

- quando o grupo precisa ganhar ou perder capacidade administrativa transversal
- quando o HSC decide formalizar perfis novos
- quando for necessário reduzir escopo de privilégios

### Exemplo de uso correto

- incluir `@css/cheats` num grupo técnico que precisa de comandos utilitários
- remover `@css/rcon` de um grupo que não deve agir no servidor de forma profunda

---

## Uso de `admin_overrides.json`

### Objetivo

Refinar a permissão exigida por comando específico na camada CounterStrikeSharp.

### Quando usar

Usar `admin_overrides.json` apenas quando houver necessidade real de política transversal por comando, por exemplo:

- restringir fortemente um comando específico
- exigir uma flag customizada para uma operação sensível
- ajustar uma divergência entre superfície de comando e política do HSC

### Quando não usar

Não usar `admin_overrides.json` para compensar modelagem ruim de grupos.  
Primeiro, resolver com grupos claros.  
Só depois, se necessário, usar override fino.

### Estado atual

No baseline atual, `admin_overrides.json` não faz parte da operação viva.

---

## O que não fazer

### Não usar `css_addadmin` / `css_deladmin` como rotina canônica

Esses comandos existem no ecossistema SimpleAdmin, mas o uso deles como governança oficial é incompatível com o modelo atual do HSC.

### Não cadastrar admin em `cfg/MatchZy/admins.json`

Esse arquivo foi mantido vazio de propósito para evitar drift.

### Não cadastrar admin em `CS2-SimpleAdmin/data/admins.json`

Esse arquivo deve permanecer vazio como baseline canônica.

### Não voltar a usar config legado de banco externo morto

O `CS2-SimpleAdmin` deve permanecer coerente com o modo atual em SQLite local, sem resíduo ativo de Railway.

---

## Runbook de validação após mudança

### Objetivo

Confirmar que a mudança administrativa chegou ao runtime real.

### Validação mínima obrigatória

1. reiniciar a instância via Game Panel
2. verificar log do CounterStrikeSharp
3. confirmar carga de `admins.json`
4. confirmar carga de `admin_groups.json`
5. confirmar que os plugins administrativos carregaram sem erro
6. entrar com um admin válido e testar `!css_admin`
7. testar um comando de MatchZy, preferencialmente `.whitelist`

### Sinais de sucesso

- log mostra carga de admins e grupos
- `!css_admin` abre menu administrativo
- `.whitelist` responde normalmente para admin válido

### Sinais de problema

- log não carrega `admin_groups.json`
- plugin administrativo falha no boot
- `!css_admin` não abre
- MatchZy ignora comando administrativo
- usuário esperado aparece sem autoridade

---

## Runbook de troubleshooting de drift administrativo

### Sintoma

Comando funciona em um plugin, mas não em outro.

### Perguntas corretas

- o usuário está em `configs/admins.json`?
- o grupo correto existe em `configs/admin_groups.json`?
- o boot carregou admins e grupos?
- o MatchZy está sem lista paralela?
- o SimpleAdmin continua sem admins/grupos locais ativos?
- a mudança foi seguida de restart?

### Diagnóstico típico

O drift costuma nascer de uma destas causas:

- edição fora da origem canônica
- ausência de restart
- reintrodução de lista paralela no MatchZy
- uso indevido de cadastro persistente do SimpleAdmin
- arquivo JSON inválido ou mal formatado

---

## Política de sustentabilidade do HSC

Para manter o sistema sustentável ao longo do tempo:

- manter poucos grupos, com semântica clara
- documentar qualquer novo grupo imediatamente
- preferir alterações pequenas e reversíveis
- evitar cadastrar staff “temporário” em múltiplas camadas
- validar runtime sempre no mesmo ritual
- remover resíduos de config morta assim que identificados
- preservar a fronteira entre governança e improviso de runtime

---

## Critério de sucesso

A manutenção administrativa do HSC só deve ser tratada como correta quando:

- toda mudança nasce em `admins.json` ou `admin_groups.json`
- MatchZy e SimpleAdmin seguem consumindo a autoridade sem competir com ela
- os testes de runtime passam
- não existem cadastros administrativos paralelos vivos no disco

---

## Resumo executivo

O workflow sustentável do HSC é:

- editar o CounterStrikeSharp
- reiniciar a instância
- validar o runtime
- evitar atalhos de plugin que reintroduzem drift

Esse é o runbook que deve orientar a manutenção administrativa do lado jogo daqui para frente.
