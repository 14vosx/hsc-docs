# Operational Runbooks

## Objetivo

Documentar os procedimentos operacionais recorrentes do contexto Game Panel do ecossistema HSC, com foco na administração prática da instância `MixHAXIXE01`, na operação do servidor CS2, no uso do MatchZy e nos fluxos competitivos reais do lado jogo.

Este documento existe para registrar, de forma estável e auditável:

- como validar rapidamente se o servidor está operacional
- como iniciar e conduzir fluxos como scrim, BO1 e BO3
- como lidar com warmup, veto, coach e match flow
- como validar plugins e persistência estatística
- como operar fallback administrativo quando a sessão não está no estado ideal
- como reduzir dependência de memória informal na administração do servidor

---

## Escopo

Este documento cobre:

- runbook de validação inicial do servidor
- runbook de checagem do MatchZy
- runbook de scrim
- runbook de BO1
- runbook de BO3
- runbook de veto
- runbook de coach
- runbook de warmup
- runbook de fallback admin
- runbook de validação de persistência estatística
- regras de ouro operacionais do lado jogo

Este documento não cobre em profundidade:

- configuração linha a linha de todos os arquivos do servidor
- troubleshooting aprofundado do AMP
- inventário detalhado de todos os plugins
- pipeline ETL do Portal Estático
- contratos JSON da Static API v2
- Auth API no AWS Lightsail

Esses tópicos vivem em documentos próprios dos contextos `01-infra-hostinger`, `02-game-panel` e `03-portal-estatico`.

---

## Estado atual

O estado operacional conhecido do lado jogo exige que a operação prática seja pensada em camadas:

- Infra Hostinger saudável
- Docker host saudável
- AMP e instância `MixHAXIXE01` coerentes
- servidor CS2 funcional
- MatchZy carregado
- plugins relevantes carregados
- permissões administrativas utilizáveis
- runbook claro para sessão real

Também já existe evidência reconciliada de que o ecossistema usa ou valoriza operacionalmente:

- scrim
- BO1
- BO3
- warmup
- veto
- coach
- fallback administrativo
- presets/aliases auxiliares quando úteis

Este documento assume que o servidor deve ser operado de forma previsível e repetível.

---

## Source of truth / evidências

As principais evidências deste documento, nesta fase de migração documental, são:

- documentação consolidada atual do ecossistema HSC
- blueprint técnico consolidado do HSC
- documentação reconciliada da camada AMP/CS2
- histórico operacional de uso do MatchZy em scrim, BO1, BO3, veto, coach e warmup
- histórico reconciliado de aliases e presets auxiliares do servidor
- histórico reconciliado da necessidade de fallback administrativo

Enquanto a migração canônica do contexto não estiver concluída, essas fontes seguem sendo usadas como base de reconciliação do estado real.

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
- `docs/02-game-panel/observability-troubleshooting.md`
- `docs/03-portal-estatico/data-sources-matchzy-sqlite.md`

Este documento descreve procedimentos recorrentes de operação do lado jogo.  
Ele não substitui os documentos de arquitetura, configuração, plugins ou troubleshooting aprofundado.

---

## Princípios operacionais do contexto

Os runbooks deste contexto seguem estes princípios:

- nunca assumir que “servidor visível” significa “servidor pronto para partida”
- nunca assumir que MatchZy carregado significa fluxo competitivo saudável
- sempre validar a base antes de iniciar uma sessão real
- sempre distinguir problema de infraestrutura de problema de operação de partida
- sempre preferir sequência explícita a improviso administrativo
- sempre tratar fallback admin como parte real da operação
- sempre validar persistência estatística após partidas relevantes

Regra de ouro:

**o lado jogo só está saudável quando a instância existe, o servidor opera, o MatchZy responde e a sessão real acontece como esperado**

---

## Runbook de validação inicial do servidor

Este runbook deve ser usado:

- antes de scrim
- antes de BO1
- antes de BO3
- após reboot
- após update de plugin
- após intervenção manual no servidor
- quando houver dúvida sobre o estado real da sessão

### Objetivo

Confirmar rapidamente que o lado jogo está operacional.

### Sequência lógica

1. validar Infra Hostinger
2. validar Docker host
3. validar instância `MixHAXIXE01`
4. validar camada de plugins
5. validar MatchZy
6. validar se o modo operacional esperado pode ser iniciado

### Validar Docker host

```bash
sudo systemctl status docker
docker ps --format "table {{.Names}}\t{{.Image}}\t{{.Ports}}"
```

### Validar presença estrutural da instância

```bash
ls -lah /home/amp/.ampdata/instances/
ls -lah /home/amp/.ampdata/instances/MixHAXIXE01/
```

### Validar plugins carregados

```bash
css_plugins list
```

### Critério de sucesso

A validação inicial só deve ser tratada como satisfatória quando:

- o substrate do host está íntegro
- a instância oficial existe
- o runtime esperado está presente
- a camada de plugins está carregada
- MatchZy aparece no runtime
- o staff consegue seguir para o modo de operação desejado sem sinais óbvios de drift

---

## Runbook de checagem do MatchZy

Este runbook deve ser usado:

- antes de qualquer partida organizada
- quando o fluxo competitivo parecer estranho
- quando comandos esperados não funcionarem
- quando houver dúvida se o plugin realmente está carregado

### Objetivo

Confirmar que o MatchZy está presente e utilizável no runtime real.

### Sequência lógica

1. listar plugins carregados
2. confirmar presença do MatchZy
3. verificar se comandos/fluxos esperados estão disponíveis
4. confirmar que o contexto competitivo do servidor está coerente com a sessão desejada

### Verificar plugins carregados

```bash
css_plugins list
```

### Observação operacional

O inventário real do runtime é sempre mais confiável do que memória ou print antigo.  
Se houver dúvida sobre o estado do MatchZy, a prioridade é verificar o runtime real.

### Critério de sucesso

A checagem do MatchZy só deve ser tratada como satisfatória quando:

- o plugin aparece como carregado
- o staff reconhece que o servidor está em estado compatível com o fluxo competitivo esperado
- não há sintoma imediato de degradação estrutural do flow de partida

---

## Runbook de scrim

Este runbook deve ser usado quando a sessão é uma scrim competitiva padrão.

### Objetivo

Colocar o servidor em um estado previsível de scrim com mínima ambiguidade operacional.

### Pré-condições

Antes de iniciar:

- validação inicial do servidor concluída
- MatchZy confirmado
- staff/admin presente ou fallback conhecido
- sessão alinhada sobre quem opera o fluxo
- warmup entendido como etapa real do processo

### Sequência lógica

1. validar que o servidor está em baseline saudável
2. entrar no fluxo de warmup/preparação
3. alinhar o modo de sessão como scrim
4. validar se os players/admins entendem o estado atual
5. iniciar a sessão competitiva

### Regra operacional

Scrim não deve depender de improviso sobre “qual era mesmo o próximo comando”.  
O runbook existe para reduzir esse custo cognitivo.

### Critério de sucesso

A scrim é considerada corretamente iniciada quando:

- o servidor está em estado competitivo coerente
- os players reconhecem o flow da sessão
- warmup e transição para partida ocorrem de forma previsível
- o staff não precisa improvisar para manter a sessão viva

---

## Runbook de BO1

Este runbook deve ser usado quando a sessão é uma partida única.

### Objetivo

Executar uma BO1 de forma simples, previsível e coerente com o modelo competitivo do servidor.

### Pré-condições

Antes de iniciar:

- validação inicial do servidor concluída
- MatchZy confirmado
- modo BO1 entendido pelo staff
- eventual fase de veto/mapa resolvida conforme o fluxo vigente
- warmup/preparação concluídos

### Sequência lógica

1. confirmar que a sessão é realmente BO1
2. confirmar que o staff não está operando o servidor como se fosse BO3
3. alinhar warmup e transição
4. iniciar a partida única
5. monitorar o flow competitivo até o término

### Regra operacional

BO1 deve ser tratado como modo explícito.  
Não deve existir ambiguidade entre BO1 e “BO3 incompleto”.

### Critério de sucesso

A BO1 é considerada corretamente operada quando:

- a sessão segue fluxo único e coerente
- não há confusão de série multi-mapa
- o staff consegue conduzir o ciclo completo sem improviso estrutural

---

## Runbook de BO3

Este runbook deve ser usado quando a sessão é uma série multi-mapa.

### Objetivo

Executar uma BO3 com previsibilidade, respeitando as transições e a complexidade maior em relação à BO1.

### Pré-condições

Antes de iniciar:

- validação inicial do servidor concluída
- MatchZy confirmado
- staff consciente de que se trata de série
- eventual fluxo de veto/picks/bans entendido
- warmup e preparação concluídos

### Sequência lógica

1. confirmar que o modo real da sessão é BO3
2. alinhar o flow de mapa, picks e bans
3. garantir que o servidor não está em estado residual de outro modo
4. iniciar a série
5. acompanhar progressão entre mapas de forma disciplinada

### Regra operacional

BO3 não deve ser conduzida como se fosse apenas “uma BO1 com continuação”.  
Ela precisa ser tratada como série.

### Critério de sucesso

A BO3 é considerada corretamente iniciada e conduzida quando:

- o flow multi-mapa está claro
- picks/bans e transições são compreendidos pelo staff
- o servidor se comporta como série, e não como sessão isolada improvisada

---

## Runbook de veto

Este runbook deve ser usado quando a sessão depende de picks e bans de mapa.

### Objetivo

Conduzir a fase de veto sem ambiguidade operacional.

### Pré-condições

Antes de iniciar:

- MatchZy confirmado
- staff consciente do formato da sessão
- mapa/pool relevante conhecidos pelos participantes
- sessão ainda em fase preparatória, não em partida já iniciada de forma incorreta

### Sequência lógica

1. confirmar que o formato da sessão exige veto
2. alinhar qual fluxo de veto será usado
3. executar picks e bans sem improviso de regra
4. confirmar mapa(s) resultante(s)
5. só então avançar para a partida/série

### Regra operacional

Veto mal conduzido não é detalhe administrativo.  
Ele compromete a legitimidade percebida da sessão competitiva.

### Critério de sucesso

O veto é considerado corretamente operado quando:

- picks e bans são compreendidos pelo staff e pelos times
- o resultado final de mapa(s) fica claro
- não há transição confusa entre fase de veto e início real da partida

---

## Runbook de coach

Este runbook deve ser usado quando o cenário exige participação operacional de coach.

### Objetivo

Validar e conduzir o uso do contexto de coach sem improviso.

### Pré-condições

Antes de iniciar:

- MatchZy confirmado
- staff consciente de que a sessão envolve coach
- regras operacionais dessa sessão entendidas pelo time/operadores

### Sequência lógica

1. confirmar que o cenário realmente exige coach
2. validar que o servidor está em fluxo compatível
3. alinhar o papel do coach na sessão
4. iniciar a sessão somente quando a presença/estado de coach estiver clara

### Regra operacional

Coach é parte do flow da sessão.  
Não deve ser tratado como detalhe informal “ajustado no meio do caminho”.

### Critério de sucesso

O fluxo de coach é considerado saudável quando:

- o contexto da sessão acomoda coach sem improviso
- os operadores entendem o papel do coach no runtime
- o modo competitivo não entra em estado ambíguo por causa dessa participação

---

## Runbook de warmup

Este runbook deve ser usado sempre que houver preparação real antes da partida.

### Objetivo

Tratar warmup como etapa formal do flow, e não como período “solto” antes do jogo.

### Pré-condições

Antes de iniciar:

- validação inicial do servidor concluída
- staff alinhado sobre o estado atual da sessão
- MatchZy confirmado

### Sequência lógica

1. colocar a sessão em estado de preparação
2. permitir entrada/ajuste dos participantes
3. confirmar que a transição para o modo competitivo principal está clara
4. encerrar warmup e iniciar a partida apenas quando o estado for compreendido por todos os operadores necessários

### Regra operacional

Warmup mal encerrado ou mal compreendido costuma gerar:
- confusão de sessão
- início prematuro
- percepção de desorganização
- troubleshooting desnecessário do servidor

### Critério de sucesso

O warmup é considerado saudável quando:

- a preparação ocorre sem improviso
- a transição para o modo principal fica clara
- não resta dúvida sobre se o servidor ainda está em warmup ou já entrou na partida

---

## Runbook de fallback admin

Este runbook deve ser usado quando a sessão depende de ação administrativa e não há admin operando do modo ideal.

### Objetivo

Preservar continuidade operacional mesmo em cenário imperfeito de staff/admin.

### Quando usar

Usar quando:

- ninguém com o perfil ideal está operando
- a sessão depende de ação administrativa real
- o fluxo competitivo travou por ausência de admin funcional
- o servidor precisa de caminho de contingência previsível

### Sequência lógica

1. confirmar que o problema é realmente ausência/degradação de admin e não falha técnica maior
2. identificar o caminho de contingência previsto no ambiente atual
3. aplicar a contingência mínima necessária para destravar a sessão
4. evitar improvisos permanentes ou “gambiarras” que alterem o baseline do servidor
5. registrar a necessidade de normalização posterior

### Regra operacional

Fallback admin existe para continuidade, não para virar modo operacional padrão.

### Critério de sucesso

O fallback é considerado bem-sucedido quando:

- a sessão destrava
- o servidor não entra em drift desnecessário
- a contingência é aplicada com escopo mínimo
- permanece claro que se trata de exceção operacional

---

## Runbook de validação de persistência estatística

Este runbook deve ser usado:

- após partidas relevantes
- após mudança em MatchZy
- após update de plugin
- quando o Portal Estático começar a ficar stale
- quando houver dúvida se a estatística do lado jogo continua sendo produzida

### Objetivo

Confirmar que o MatchZy continua produzindo dados estruturais do ecossistema.

### Sequência lógica

1. localizar o `matchzy.db`
2. validar existência do banco
3. validar tabelas principais
4. validar se o volume de dados evolui conforme esperado
5. só depois concluir se o problema é do lado game ou do portal

### Validar existência do banco

Substitua pelo path real reconciliado do ambiente.

```bash
ls -l /CAMINHO/DO/matchzy.db
```

### Validar tabelas principais

```bash
sqlite3 /CAMINHO/DO/matchzy.db ".tables"
```

### Validar volume básico de registros

```bash
sqlite3 /CAMINHO/DO/matchzy.db "SELECT COUNT(*) FROM matchzy_stats_matches;"
sqlite3 /CAMINHO/DO/matchzy.db "SELECT COUNT(*) FROM matchzy_stats_players;"
sqlite3 /CAMINHO/DO/matchzy.db "SELECT COUNT(*) FROM matchzy_stats_maps;"
```

### Critério de sucesso

A persistência estatística é considerada saudável quando:

- o banco existe
- as tabelas principais existem
- o volume de dados evolui coerentemente após partidas reais
- não há sinal imediato de que o MatchZy deixou de persistir

---

## Runbook de preset auxiliar

Este runbook deve ser usado quando a operação exige preset específico de treino, teste ou sessão reduzida.

### Objetivo

Aplicar preset auxiliar sem contaminar o baseline competitivo principal.

### Exemplo reconciliado no histórico do ecossistema

- `load_1v1.cfg`

### Sequência lógica

1. confirmar que o cenário realmente exige preset auxiliar
2. confirmar que o preset é conhecido/documentado
3. aplicar o preset
4. validar que o servidor entrou no estado esperado
5. encerrar ou normalizar a sessão antes de voltar ao fluxo competitivo principal

### Regra operacional

Preset auxiliar útil é aceitável.  
Preset auxiliar “esquecido” no runtime vira fonte de drift.

### Critério de sucesso

O preset auxiliar é considerado bem utilizado quando:

- resolve o cenário específico
- não cria confusão com o baseline principal
- o staff sabe claramente quando está fora do modo competitivo padrão

---

## Regras de ouro

As regras de ouro operacionais deste contexto são:

1. nunca iniciar sessão competitiva sem validação inicial mínima
2. nunca assumir que MatchZy carregado basta por si só
3. sempre distinguir BO1 de BO3 explicitamente
4. sempre tratar warmup como etapa real do fluxo
5. sempre tratar veto como parte séria da sessão
6. sempre ter fallback admin pensado antes de precisar dele
7. sempre validar persistência estatística após incidentes ou mudanças relevantes
8. nunca deixar preset auxiliar virar baseline informal do servidor

---

## Problemas comuns

### 1. Sessão trava por ausência de admin funcional

Causas comuns:

- modelo de permissão frágil
- falta de fallback
- dependência excessiva de uma pessoa específica

---

### 2. Servidor entra em estado ambíguo entre warmup e partida

Causas comuns:

- transição mal conduzida
- runbook não seguido
- operadores sem alinhamento

---

### 3. Staff acredita estar em BO1, mas o runtime reflete outro modo

Causas comuns:

- drift de configuração
- preset errado
- sessão anterior mal encerrada
- improviso administrativo

---

### 4. MatchZy responde, mas a sessão continua ruim

Causas comuns:

- problema operacional humano
- modo errado do servidor
- coach/veto/warmup mal conduzidos
- fallback admin ausente

---

### 5. Partida acontece, mas a estatística não evolui

Causas comuns:

- MatchZy persistindo mal
- path do banco incorreto
- problema de escrita na instância
- falha estrutural invisível ao operador in-game

---

## Limites deste documento

Este documento não detalha:

- cada comando exato do MatchZy para cada cenário
- cada preset e alias existente no ambiente
- cada diferença fina de setup entre todos os modos possíveis
- troubleshooting aprofundado de plugins ou do AMP
- recuperação avançada após incidentes multi-camada

Esses tópicos devem ser tratados em documentos complementares ou após reconciliação adicional do ambiente real.

---

## Critério de pronto deste documento

Este documento pode ser considerado maduro quando:

- os fluxos principais de scrim, BO1, BO3, veto, coach e warmup estiverem suficientemente claros
- o fallback admin estiver formalizado sem ambiguidade
- a validação de persistência estatística estiver incorporada ao runbook normal do contexto
- os presets auxiliares relevantes estiverem reconciliados com o ambiente real
- ele puder ser usado como guia prático de operação do lado jogo sem depender do master legado

---

## Última revisão

- Status: ativo
- Classificação: canônico
- Contexto: game panel / operational runbooks
- Última revisão: 2026-03-18