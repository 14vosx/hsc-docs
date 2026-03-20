# CS2 Server Configuration

## Objetivo

Documentar a camada de configuração do servidor Counter-Strike 2 no contexto Game Panel do ecossistema HSC, registrando quais tipos de configuração governam o runtime do jogo, como elas se organizam e quais cuidados operacionais precisam existir para manter o servidor previsível.

Este documento existe para registrar, de forma estável e auditável:

- quais camadas de configuração compõem o runtime do servidor
- como configs base, configs de modo de jogo e configs de plugin se relacionam
- por que configuração do servidor não pode ficar dependente apenas de memória informal
- como o servidor deve ser operado em modos como scrim, BO1, BO3, warmup e cenários auxiliares
- quais invariantes e riscos existem quando a configuração real diverge do que os operadores acreditam estar rodando

---

## Escopo

Este documento cobre:

- a configuração do runtime do servidor CS2 em alto nível
- a distinção entre config base do servidor e configs operacionais de partida
- a relação entre config do servidor, MatchZy e plugins
- a relação entre configuração e runbooks operacionais
- os princípios de versionamento e previsibilidade da configuração
- riscos de drift de configuração
- validações mínimas da camada de configuração

Este documento não cobre em profundidade:

- configuração linha a linha de todos os arquivos do servidor
- inventário completo de todos os plugins
- troubleshooting aprofundado do AMP
- pipeline ETL do Portal Estático
- contratos JSON da Static API v2
- Auth API no AWS Lightsail

Esses tópicos vivem em documentos próprios dos contextos `01-infra-hostinger`, `02-game-panel` e `03-portal-estatico`.

---

## Estado atual

O estado operacional conhecido da configuração do servidor HSC é:

- o runtime do servidor CS2 depende de uma base de configuração coerente com a instância `MixHAXIXE01`
- MatchZy participa diretamente do comportamento operacional das partidas
- existem modos operacionais distintos, como scrim, BO1, BO3, warmup e cenários auxiliares de treino
- aliases, comandos e presets operacionais já foram tratados como parte do fluxo real de administração do servidor
- a operação prática do servidor depende tanto da configuração quanto dos runbooks usados pelos admins
- a configuração real do servidor não deve depender apenas da interface do painel nem apenas de memória informal

Também já existe evidência reconciliada de que o ecossistema considera importante:

- configs versionadas
- modos operacionais claros
- fallback administrativo quando necessário
- previsibilidade para treinos, scrims e setup de partida

---

## Source of truth / evidências

As principais evidências deste documento, nesta fase de migração documental, são:

- documentação consolidada atual do ecossistema HSC
- blueprint técnico consolidado do HSC
- documentação reconciliada da camada AMP/CS2
- histórico operacional do servidor ligado a MatchZy, warmup, coach, veto e presets
- histórico reconciliado de aliases e configs auxiliares discutidos na operação do servidor

Enquanto a migração canônica do contexto não estiver concluída, essas fontes seguem sendo usadas como base de reconciliação do estado real.

---

## Relações com outros documentos

Este arquivo é complementar a:

- `docs/02-game-panel/README.md`
- `docs/02-game-panel/game-panel-architecture-runtime.md`
- `docs/02-game-panel/amp-instance-manager.md`
- `docs/02-game-panel/instance-mixhaxixe01.md`
- `docs/02-game-panel/matchzy.md`
- `docs/02-game-panel/plugins-installed.md`
- `docs/02-game-panel/game-panel-operational-runbooks.md`
- `docs/02-game-panel/game-panel-observability-troubleshooting.md`

Este documento descreve a configuração do runtime do servidor CS2 em nível canônico.  
Ele não substitui os documentos de MatchZy, inventário de plugins ou runbooks operacionais detalhados.

---

## Papel da configuração no ecossistema

No HSC, a configuração do servidor não é apenas um conjunto de arquivos técnicos.

Ela define, na prática:

- como o servidor se comporta
- como partidas são operadas
- como warmups, scrims e séries são iniciados
- como plugins se integram ao runtime
- como admins interagem com a instância
- como o servidor mantém previsibilidade entre uma sessão e outra

Em termos arquiteturais:

- infraestrutura saudável sem configuração coerente ainda produz servidor degradado
- MatchZy saudável sem configuração coerente ainda produz fluxo operacional ruim
- por isso, a configuração do servidor é parte central da verdade do contexto

---

## Camadas de configuração

A configuração do servidor pode ser entendida em camadas.

### 1. Configuração base do servidor

Responsável por:

- parâmetros gerais do runtime do CS2
- comportamento base da instância
- inicialização do ambiente do jogo
- coerência mínima do servidor antes de qualquer modo operacional específico

Essa camada define o “chão” do servidor.

---

### 2. Configuração de modo operacional

Responsável por:

- scrim
- BO1
- BO3
- warmup
- cenários auxiliares como treino e presets específicos

Essa camada define o comportamento da sessão atual do servidor.

---

### 3. Configuração de plugin

Responsável por:

- MatchZy
- plugins auxiliares
- comportamento administrativo
- integrações e persistência complementar

Essa camada define como os componentes do runtime efetivamente se comportam.

---

### 4. Configuração operacional de apoio

Responsável por:

- aliases
- presets de carregamento rápido
- comandos padronizados
- atalhos de operação no painel ou em rotinas administrativas

Essa camada reduz atrito operacional do staff.

---

## Configuração base do servidor

A configuração base do servidor deve garantir que o runtime do CS2 suba de forma previsível.

Isso implica:

- ambiente coerente com a instância oficial
- compatibilidade com CounterStrikeSharp
- compatibilidade com MatchZy
- compatibilidade com os plugins instalados
- ausência de conflito estrutural entre a base do servidor e os modos operacionais carregados depois

Regra importante:

- a configuração base não deve ficar sendo “mutada” informalmente a cada partida
- ela deve servir de baseline estável para os modos operacionais do servidor

---

## Configuração de scrim

O modo scrim é um dos fluxos centrais do servidor HSC.

A configuração de scrim deve garantir:

- ambiente competitivo coerente
- preparação adequada para uso real do servidor
- transição previsível entre warmup e partida
- comportamento alinhado com o MatchZy e com o runbook operacional do staff

Em termos documentais:

- este documento reconhece scrim como modo operacional central
- o passo a passo prático de uso vive em `operational-runbooks.md`

---

## Configuração de BO1

O modo BO1 deve ser tratado como configuração operacional distinta.

Seu papel é:

- suportar partida única
- manter simplicidade operacional
- reduzir ambiguidade no fluxo competitivo
- alinhar expectativas de admins e players

A camada de configuração precisa garantir que BO1 não seja apenas “BO3 operando pela metade”, mas um modo coerente em si.

---

## Configuração de BO3

O modo BO3 exige mais disciplina operacional do que BO1.

Seu papel é:

- suportar série de mapas
- alinhar o fluxo competitivo a picks, bans e progressão da série
- depender mais fortemente de MatchZy e do runbook correto

Regra importante:

- a configuração de BO3 precisa estar explícita e previsível
- não deve depender de improviso administrativo a cada sessão

---

## Configuração de warmup

O warmup é parte importante do fluxo real do servidor.

Seu papel é:

- preparar a sessão antes da partida
- dar espaço para organização, entrada e estabilização dos jogadores
- sustentar a transição para o modo competitivo principal

A configuração de warmup deve ser tratada como parte do fluxo do servidor, e não apenas como estado “solto” antes da partida.

---

## Configurações auxiliares de treino e presets

O ecossistema HSC já tratou, operacionalmente, de presets auxiliares para cenários específicos de treino.

Isso inclui a ideia de configs auxiliares dedicadas para:

- treino rápido
- sessões reduzidas
- cenários de teste
- presets de operação simplificada

Exemplo já presente no histórico operacional do ecossistema:

- `load_1v1.cfg`

Regra importante:

- presets auxiliares só devem permanecer no runtime se estiverem documentados e fizerem sentido para a operação real
- config auxiliar não deve virar fonte de drift ou confusão com o modo principal competitivo

---

## Aliases e atalhos operacionais

O ecossistema já tratou aliases e atalhos operacionais como parte útil da administração do servidor.

O papel dessa camada é:

- reduzir repetição manual
- padronizar comandos frequentes
- facilitar execução de modos operacionais conhecidos
- reduzir improviso do staff

Exemplos de uso conceitual:

- carregar preset conhecido
- iniciar modo de treino
- preparar sessão de scrim
- facilitar ações recorrentes dentro do fluxo operacional do servidor

Regra importante:

- alias útil deve ser documentado
- alias não documentado tende a virar dependência escondida de memória informal

---

## Relação entre configuração e MatchZy

A configuração do servidor e o MatchZy não podem ser pensados separadamente.

Essa relação inclui:

- modo da partida
- fluxo competitivo esperado
- warmup
- veto
- transição entre estados
- comportamento administrativo e experiência dos players

Regra canônica:

- MatchZy saudável com configuração errada ainda produz servidor degradado
- configuração correta sem MatchZy saudável também não resolve o fluxo competitivo

---

## Relação entre configuração e plugins

Os plugins instalados dependem do estado correto do runtime para operar como esperado.

Isso implica:

- configuração incompatível pode degradar plugins
- plugin carregado pode comportar-se mal se o modo do servidor estiver errado
- conflitos de expectativa entre configs e plugins podem gerar sintomas difíceis de diagnosticar

Regra importante:

- a configuração do servidor deve ser lida como parte da malha de runtime, e não como arquivo isolado

---

## Relação entre configuração e admins

A configuração só tem valor prático quando os admins conseguem operá-la de forma previsível.

Isso inclui:

- saber qual modo está ativo
- saber qual preset carregar
- saber como iniciar scrim, BO1 ou BO3
- saber como lidar com warmup, coach e fallback operacional
- evitar intervenção improvisada baseada em memória parcial

Regra operacional:

- servidor bem configurado, mas sem runbook claro para admins, continua sendo servidor operacionalmente degradado

---

## Fallback administrativo

O ecossistema já tratou a necessidade de fallback administrativo quando não há admins operando no modelo ideal.

Isso é relevante para a camada de configuração porque:

- parte do fluxo competitivo depende de ações administrativas
- ausência de admin pode bloquear operação prática do MatchZy e da sessão
- fallback precisa ser compatível com a realidade do servidor e com suas permissões

Regra importante:

- fallback admin não é detalhe periférico
- ele faz parte da resiliência operacional do servidor

---

## Configuração como artefato versionado

A configuração do servidor deve ser tratada como artefato versionado do ecossistema.

Isso significa:

- configs principais não devem viver apenas “no painel”
- presets relevantes não devem viver apenas em histórico de comandos
- mudanças relevantes precisam deixar rastro documental
- a configuração real deve poder ser reconciliada com o que o time acredita que está rodando

Regra canônica:

- configuração crítica sem documentação ou sem trilha de mudança é fonte de drift operacional

---

## Riscos de drift de configuração

Os principais riscos desta camada incluem:

- arquivo/config alterado manualmente sem reconciliação
- preset antigo continuando a ser usado sem validade atual
- aliases úteis existirem apenas na memória de quem opera
- MatchZy esperar um fluxo e o servidor estar em outro
- admins acreditarem estar em BO1, mas o runtime refletir outra coisa
- warmup, veto ou coach divergirem do que o runbook assume

Esses riscos aumentam quando:

- a operação depende de memória informal
- a config real não está versionada
- não há documento canônico dizendo o que é baseline e o que é preset auxiliar

---

## Invariantes operacionais

Os invariantes conhecidos desta camada incluem:

- o servidor possui uma base de configuração que precisa ser estável
- MatchZy é dependente do contexto configurado corretamente
- scrim, BO1, BO3 e warmup são modos reais do contexto
- aliases e presets operacionais já fazem parte da realidade do ecossistema
- fallback administrativo precisa existir como preocupação real
- configuração crítica deve ser reconciliável, e não apenas lembrada

Esses invariantes ajudam a preservar previsibilidade do servidor.

---

## Sinais de saúde da configuração

Os sinais de saúde esperados incluem:

- o servidor sobe no estado esperado
- MatchZy opera no modo esperado
- admins conseguem executar os fluxos previstos no runbook
- presets conhecidos funcionam como documentado
- warmup e transição para partida se comportam de forma previsível
- não há discrepância evidente entre o modo que os operadores acreditam estar ativo e o comportamento real do servidor

A saúde da configuração deve ser inferida pelo comportamento operacional real, e não apenas pela existência de arquivos no disco.

---

## Comandos e validações representativas

Os comandos abaixo representam verificações típicas ligadas à configuração do servidor.

### Validar plugins carregados

```bash
css_plugins list
```

### Validar árvore estrutural da instância oficial

```bash
ls -lah /home/amp/.ampdata/instances/MixHAXIXE01/
```

### Validar container relevante no host

```bash
docker ps --format "table {{.Names}}\t{{.Image}}\t{{.Ports}}"
```

### Validar uso de preset auxiliar conhecido, quando aplicável

Exemplo representativo já reconciliado no histórico do ecossistema:

```bash
load_1v1.cfg
```

Observação importante:
- o comando acima representa um preset auxiliar reconhecido no histórico operacional
- o inventário completo dos presets e aliases em uso real ainda deve ser consolidado em runbooks e validação de ambiente

---

## Problemas comuns

### 1. Servidor sobe, mas o modo operacional está errado

Causas comuns:

- config base divergente
- preset errado carregado
- runbook não seguido
- mudança manual não documentada

Impacto:
- admins e players operam com expectativa errada sobre o servidor

---

### 2. MatchZy carrega, mas o fluxo competitivo continua ruim

Causas comuns:

- configuração do modo de partida incompatível
- warmup ou transição mal configurados
- staff operando em fluxo diferente do esperado
- fallback administrativo ausente

Impacto:
- MatchZy “presente”, mas servidor continua ruim na prática

---

### 3. Preset auxiliar interfere no modo principal

Causas comuns:

- preset antigo ou de teste ainda em uso
- alias acionado em contexto errado
- ausência de separação clara entre config base e config auxiliar

Impacto:
- runtime do servidor entra em estado confuso
- troubleshooting fica caro

---

### 4. Servidor aparentemente correto, mas comportamento difere do runbook

Causas comuns:

- drift de configuração
- mudança manual não reconciliada
- documentação desatualizada
- memória informal substituindo configuração versionada

Impacto:
- operação perde previsibilidade
- aumenta a dependência de tentativa e erro

---

### 5. Fallback admin não funciona no momento necessário

Causas comuns:

- modelo de permissão não alinhado
- runbook incompleto
- dependência excessiva de uma pessoa específica
- operação real diferente do que a configuração pressupõe

Impacto:
- a sessão trava operacionalmente mesmo com servidor e plugins saudáveis

---

## Sequência básica de diagnóstico

Quando houver suspeita de problema na configuração do servidor, a sequência recomendada é:

1. validar a saúde da Infra Hostinger
2. validar a saúde do Docker host
3. validar a saúde do AMP e da instância `MixHAXIXE01`
4. validar a presença do MatchZy e dos plugins esperados
5. confirmar qual modo operacional o servidor deveria estar executando
6. comparar o comportamento real do servidor com o runbook do modo esperado
7. investigar presets, aliases e mudanças manuais recentes
8. só depois concluir que a falha é “do jogo” ou “do plugin”

Essa sequência ajuda a responder rapidamente:

- o problema é de infraestrutura, de plugin ou de configuração?
- o servidor está no modo que o time acha que está?
- há drift entre runbook e runtime real?

---

## Itens pendentes de confirmação

Os itens abaixo ainda devem ser confirmados diretamente no ambiente real para elevar o grau de confiança desta camada:

### 1. Inventário completo de presets operacionais em uso

Já há evidência de presets e aliases úteis, mas a lista completa ainda precisa ser consolidada diretamente a partir do ambiente real.

### 2. Caminho final e organização dos arquivos de configuração mais críticos

A estrutura conceitual está reconciliada, mas a fotografia completa dos paths de config relevantes ainda deve ser formalizada com validação direta do runtime.

### 3. Baseline exata entre config base e configs auxiliares

É desejável fechar explicitamente quais arquivos/presets são:
- baseline do servidor
- presets de scrim
- presets de treino
- presets auxiliares

---

## Limites deste documento

Este documento não detalha:

- todos os arquivos de configuração linha a linha
- todos os aliases e presets em uso
- todo o fluxo operacional de BO1, BO3, veto, coach e warmup
- troubleshooting aprofundado de MatchZy
- troubleshooting aprofundado da base de plugins

Esses tópicos pertencem a documentos específicos do contexto ou dependem de validação adicional do ambiente real.

---

## Critério de pronto deste documento

Este documento pode ser considerado maduro quando:

- a distinção entre config base, config de modo e config auxiliar estiver clara
- a relação entre configuração e MatchZy estiver reconciliada sem ambiguidade
- os presets operacionais relevantes estiverem formalizados
- os riscos de drift de configuração estiverem bem representados
- ele puder ser usado como referência canônica da camada de configuração do servidor sem depender do master legado

---

## Última revisão

- Status: ativo
- Classificação: canônico
- Contexto: game panel / configuração do servidor CS2
- Última revisão: 2026-03-18