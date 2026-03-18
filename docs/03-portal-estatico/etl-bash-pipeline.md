# ETL Bash Pipeline

## Objetivo

Documentar o pipeline ETL Bash responsável por gerar a Static API v2 do contexto Portal Estático do ecossistema HSC.

Este documento existe para registrar, de forma estável e auditável:

- o papel do ETL na arquitetura pública do portal
- a ordem normativa de geração dos artefatos da v2
- os scripts principais da pipeline
- o uso de lock global para evitar concorrência
- o uso de statefiles para fluxo incremental
- a regra de escrita atômica dos JSONs públicos
- a geração de `health.json`
- os invariantes operacionais que mantêm a publicação previsível

---

## Escopo

Este documento cobre:

- o papel do ETL Bash no contexto do portal
- a sequência normativa de geração
- os scripts principais conhecidos da pipeline
- o lock global do processo
- os statefiles incrementais
- a regra de escrita atômica
- a relação entre ETL e publicação pública da v2
- integração com conteúdo same-origin, quando aplicável
- sinais de saúde e problemas comuns da pipeline

Este documento não cobre em profundidade:

- queries SQL linha a linha
- estrutura campo a campo de cada JSON
- configuração completa do Nginx
- troubleshooting geral da VPS Hostinger
- operação do AMP e do MatchZy em profundidade
- Auth API no Lightsail

Esses tópicos vivem em documentos específicos deste contexto ou em outros contextos canônicos.

---

## Estado atual

O estado operacional conhecido da pipeline ETL do portal é:

- a geração da Static API v2 é feita por scripts Bash
- a fonte primária de dados é o `matchzy.db`
- a pipeline é organizada em etapas normativas
- existe lock global para impedir concorrência indevida
- a geração combina outputs agregados e incrementais
- a publicação final depende de escrita atômica
- existe artefato de saúde do contexto (`health.json`)
- há integração auxiliar com espelho same-origin de conteúdo, quando aplicável
- os scripts operacionais vivem na base do contexto do portal e/ou em `/usr/local/bin/`

Esse desenho existe para manter a camada pública previsível, barata e fácil de operar.

---

## Source of truth / evidências

As principais evidências deste documento, nesta fase de migração documental, são:

- documentação consolidada atual do ecossistema HSC
- blueprint técnico consolidado do HSC
- reconciliação documental da ordem normativa do ETL
- reconciliação dos locks, statefiles e paths operacionais do contexto
- inventário dos scripts `gen-*` já consolidados na arquitetura da v2
- registros de integração com conteúdo same-origin no contexto do portal

Enquanto a migração canônica do contexto não estiver concluída, essas fontes seguem sendo usadas como base de reconciliação do estado real.

---

## Relações com outros documentos

Este arquivo é complementar a:

- `docs/03-portal-estatico/README.md`
- `docs/03-portal-estatico/architecture-runtime.md`
- `docs/03-portal-estatico/static-api-v2.md`
- `docs/03-portal-estatico/data-sources-matchzy-sqlite.md`
- `docs/03-portal-estatico/sql-queries-and-views.md`
- `docs/03-portal-estatico/json-contracts.md`
- `docs/03-portal-estatico/nginx-publishing-cache.md`
- `docs/03-portal-estatico/operational-runbooks.md`
- `docs/03-portal-estatico/observability-troubleshooting.md`

Este documento descreve o pipeline de geração da v2.  
Ele não substitui os documentos de contrato, SQL, publicação final ou troubleshooting geral.

---

## Papel do ETL no contexto

O ETL Bash é o mecanismo que transforma a fonte de dados operacional em artefatos públicos consumidos pelo Portal Estático.

Seu papel é:

- ler o `matchzy.db`
- executar queries versionadas ou lógica equivalente de transformação
- gerar coleções e detalhes incrementais
- atualizar o conjunto público da Static API v2
- preservar ordem de execução consistente
- evitar concorrência entre execuções
- evitar publicação parcial de arquivos
- sinalizar saúde da camada gerada

Em termos arquiteturais, o ETL é a ponte entre o mundo do runtime do jogo e o mundo da publicação pública estática.

---

## Estrutura operacional da pipeline

A pipeline é composta por:

- scripts Bash especializados por artefato
- diretório operacional do contexto do portal
- lock global para serializar a execução
- statefiles para incremental
- diretório público final da v2
- convenções de escrita atômica
- etapa de health/checks
- publicação e serving posteriores via Nginx

Representação lógica resumida:

- SQLite de origem
- scripts `gen-*`
- estado incremental
- artefatos temporários
- move atômico para publicação
- Nginx servindo os arquivos finais

---

## Ordem normativa do pipeline

A ordem normativa conhecida da pipeline da v2 é:

1. `gen-matches.sh`
2. `gen-match-details-incremental.sh`
3. `gen-ranking.sh`
4. `gen-players-incremental.sh`
5. `gen-players-from-ranking.sh`
6. `gen-maps.sh`

Essa ordem não é arbitrária.

Ela existe para:

- garantir que coleções-base sejam geradas antes dos detalhes dependentes
- permitir que ranking alimente páginas de player
- preservar consistência entre agregados e recursos incrementais
- reduzir chance de publicação incoerente entre artefatos relacionados

Regra canônica:
- qualquer mudança dessa ordem deve ser tratada como mudança relevante do contexto

---

## Scripts principais

### `gen-matches.sh`

Papel:
- gerar a coleção principal de partidas públicas

Responsabilidades típicas:
- consultar a base de matches
- produzir `matches.json` ou artefato equivalente da coleção
- fornecer base para etapas posteriores de detalhe por match

Importância arquitetural:
- é uma das etapas-base da publicação pública da v2

---

### `gen-match-details-incremental.sh`

Papel:
- gerar ou atualizar detalhes incrementais por partida

Responsabilidades típicas:
- identificar matches que precisam de detalhe
- produzir recursos como `match/{id}.json`
- reduzir necessidade de regeneração completa de todos os detalhes a cada ciclo

Importância arquitetural:
- permite equilíbrio entre custo de geração e riqueza de detalhe público

---

### `gen-ranking.sh`

Papel:
- gerar o ranking público consolidado

Responsabilidades típicas:
- aplicar regras do ranking v2
- produzir `ranking.json`
- servir como fonte derivada para outras páginas, especialmente de jogadores

Importância arquitetural:
- é um dos artefatos públicos mais centrais do portal

---

### `gen-players-incremental.sh`

Papel:
- gerar ou atualizar dossiês incrementais de jogadores

Responsabilidades típicas:
- identificar players relevantes para atualização
- produzir `player/{steamid64}.json`
- manter a camada incremental de player alinhada ao estado recente da base

Importância arquitetural:
- evita recomputação desnecessária de todos os dossiês em cada rodada

---

### `gen-players-from-ranking.sh`

Papel:
- complementar a geração de players a partir do universo já presente no ranking

Responsabilidades típicas:
- garantir cobertura de players relevantes expostos publicamente
- alinhar dossiês com a camada de ranking
- preencher lacunas da geração incremental quando necessário

Importância arquitetural:
- ajuda a manter coerência entre `ranking.json` e `player/{steamid64}.json`

---

### `gen-maps.sh`

Papel:
- gerar artefatos agregados e/ou detalhados por mapa

Responsabilidades típicas:
- produzir `maps.json`
- produzir `map/{map}.json`
- consolidar visão pública de stats por mapa

Importância arquitetural:
- fecha a pipeline principal da v2 conhecida nesta fase

---

## Lock global

A pipeline conhecida da v2 usa lock global para impedir execução concorrente.

Lock reconciliado:

- `/opt/cs2-portal/locks/gen-all-v2.lock`

Objetivo do lock:

- garantir serialização da pipeline
- evitar duas execuções escrevendo na mesma árvore pública ao mesmo tempo
- reduzir risco de artefatos parciais
- preservar previsibilidade da publicação

Regra canônica:
- o lock global é parte essencial da saúde da pipeline
- ignorar o lock é desvio operacional

---

## Papel do lock na consistência

Sem lock global, a pipeline fica exposta a riscos como:

- geração duplicada concorrente
- overwrite parcial entre execuções
- mix de artefatos de ciclos diferentes
- publicação inconsistente de ranking, players e matches
- troubleshooting muito mais difícil

Por isso:

- o lock não é detalhe de implementação
- ele é parte do contrato operacional da geração da v2

---

## Incremental e statefiles

A pipeline conhecida usa statefiles para sustentar geração incremental.

Base operacional reconciliada:

- `/opt/cs2-portal/state/`

Papel dos statefiles:

- registrar progresso incremental
- evitar recomputação total quando não necessária
- identificar itens novos ou pendentes
- sustentar geração eficiente de match details, players e outros recursos incrementais

Regra operacional:
- statefile inconsistente pode causar tanto perda de atualização quanto regeneração excessiva
- por isso, statefiles precisam ser tratados como artefatos operacionais sensíveis do contexto

---

## Escrita atômica

A publicação da v2 depende de escrita atômica dos arquivos públicos.

O princípio operacional é:

- nunca expor ao Nginx um JSON parcialmente escrito
- gerar artefato em área temporária ou fluxo seguro
- validar minimamente
- mover para o destino final de forma atômica

Objetivos:

- evitar JSON truncado
- evitar respostas públicas parcialmente geradas
- reduzir risco de quebra do frontend durante janelas de publicação
- preservar consistência entre ciclo de geração e ciclo de consumo

Regra canônica:
- a escrita da v2 não deve depender de overwrite ingênuo diretamente no path público final

---

## Publicação da saída

A saída final da pipeline é publicada em:

- `/var/www/api/cs2/v2/`

Isso significa:

- o ETL escreve a camada pública da v2 nessa árvore
- o Nginx serve dessa árvore
- o frontend consome a partir dessa árvore pública

Implicações:

- permissão de escrita do ETL e permissão de leitura do Nginx precisam coexistir
- drift de ownership/grupo pode quebrar o contexto mesmo com pipeline logicamente correta
- artefatos temporários não devem permanecer como estado público final

---

## Health generation

A pipeline conhecida inclui geração de artefato de saúde do contexto.

Artefato relevante:

- `health.json`

Função do `health.json`:

- sinalizar estado recente da geração
- registrar checks operacionais da cadeia pública
- ajudar a diferenciar falha de ETL de falha de frontend
- apoiar troubleshooting e automação de validação

Regra operacional:
- `health.json` deve refletir a saúde da camada de geração/publicação, não apenas a existência do diretório público

---

## Integração com cache de News

O contexto admite integração com conteúdo same-origin de News.

Quando essa integração está ativa, a pipeline ou seus fluxos associados precisam considerar:

- obtenção/espelhamento do conteúdo
- publicação em path público coerente
- compatibilidade com o frontend
- previsibilidade do serving same-origin

Regra importante:
- o espelho de News é complementar à pipeline principal da v2
- ele não substitui a cadeia estatística baseada em `matchzy.db`

---

## Paths operacionais relevantes

Os paths principais conhecidos ligados à pipeline incluem:

### Base operacional do portal

- `/opt/cs2-portal/`

### Locks

- `/opt/cs2-portal/locks/`

### Lock global principal

- `/opt/cs2-portal/locks/gen-all-v2.lock`

### Statefiles

- `/opt/cs2-portal/state/`

### Publicação pública final

- `/var/www/api/cs2/v2/`

### Scripts operacionais

- `/usr/local/bin/*.sh`

Esses paths precisam permanecer coerentes com a Infra Hostinger e com a topologia real do ambiente.

---

## Dependências operacionais da pipeline

A pipeline depende, no mínimo, de:

- acesso ao `matchzy.db`
- schema compatível com as queries
- SQLite com suporte necessário, incluindo JSON1 quando exigido
- scripts íntegros e executáveis
- lock global funcional
- statefiles íntegros
- permissões corretas em paths operacionais
- permissões corretas na publicação pública
- espaço em disco suficiente
- Nginx lendo os arquivos finais corretos
- frontend compatível com o contrato publicado

Essas dependências explicam por que falhas do portal nem sempre têm origem no frontend.

---

## Invariantes operacionais

Os invariantes conhecidos da pipeline incluem:

- a geração da v2 é feita por Bash
- a ordem normativa deve ser respeitada
- o lock global deve serializar a execução
- a escrita deve ser atômica
- artefatos públicos não devem ser publicados parcialmente
- statefiles devem refletir incremental de forma coerente
- a saúde da pipeline deve ser observável via `health.json`
- o ETL deve produzir outputs compatíveis com o frontend ativo

Esses invariantes são parte do contrato operacional do contexto.

---

## Sinais de saúde da pipeline

Os sinais de saúde esperados incluem:

- lock global funcionando sem ficar preso indevidamente
- scripts `gen-*` executando com sucesso
- atualização recente dos artefatos públicos
- `health.json` coerente
- `ranking.json`, `matches.json`, `maps.json` presentes e consumíveis
- recursos incrementais sendo gerados conforme esperado
- ausência de JSON truncado
- ausência de artefatos temporários órfãos na publicação
- frontend consumindo a v2 sem quebra estrutural

A saúde da pipeline deve ser avaliada tanto pelo lado dos arquivos quanto pelo lado do consumo real.

---

## Problemas comuns

### 1. ETL não roda

Causas comuns:

- script ausente ou não executável
- ambiente incorreto
- lock preso
- path do banco incorreto
- erro de permissão

Impacto:
- a v2 deixa de atualizar
- o portal passa a operar com dados stale

---

### 2. Lock preso

Causas comuns:

- execução anterior abortada
- falha sem limpeza adequada
- concorrência indevida
- intervenção manual incompleta

Impacto:
- novas execuções ficam bloqueadas
- dados públicos não são renovados

---

### 3. `ranking.json` atualiza, mas players não

Causas comuns:

- falha em `gen-players-incremental.sh`
- statefile inconsistente
- quebra em `gen-players-from-ranking.sh`
- problema de permissão ou de output incremental

Impacto:
- frontend fica incoerente entre lista de ranking e dossiês públicos

---

### 4. `matches.json` atualiza, mas detalhes por match não

Causas comuns:

- falha em `gen-match-details-incremental.sh`
- incremental inconsistente
- problema de path de output
- erro em query de detalhe

Impacto:
- página agregada funciona
- página de detalhe quebra ou fica stale

---

### 5. JSON parcial ou inválido aparece em produção

Causas comuns:

- quebra de atomicidade
- escrita direta no path final
- concorrência entre execuções
- falha no meio da publicação

Impacto:
- portal quebra no browser
- respostas públicas ficam inválidas
- troubleshooting se torna urgente

---

### 6. Health parece bom, mas dados estão stale

Causas comuns:

- `health.json` não capturando o problema real
- statefile travado
- fonte de dados não atualizando
- parte da pipeline executando sem erro, mas sem refletir mudança real

Impacto:
- falsa sensação de saúde
- frontend continua servindo dados antigos

---

## Sequência mínima de validação

A validação mínima da pipeline deve incluir:

1. verificar existência/saúde do lock
2. verificar acesso ao `matchzy.db`
3. verificar timestamps dos artefatos públicos principais
4. validar `health.json`
5. validar recursos públicos centrais
6. confirmar consistência entre agregados e detalhes
7. só depois aprofundar em SQL, incremental ou frontend

Comandos representativos:

```bash
ls -l /opt/cs2-portal/locks/
ls -l /var/www/api/cs2/v2/
cat /var/www/api/cs2/v2/health.json
```

Quando necessário, complementar com inspeções de scripts, statefiles e logs da automação real do host.

---

## Limites deste documento

Este documento não detalha:

- queries SQL específicas de cada recurso
- conteúdo de cada script linha a linha
- shape de cada JSON
- troubleshooting aprofundado por artefato
- configuração de timers ou systemd em detalhes-
- runbook completo de regeneração manual

Esses detalhes pertencem a documentos específicos do contexto.

---

## Critério de pronto deste documento

Este documento pode ser considerado maduro quando:

- a ordem normativa da pipeline estiver confirmada no ambiente real
- os scripts principais estiverem reconciliados sem ambiguidade
- o lock global e os statefiles estiverem formalizados corretamente
- a regra de escrita atômica estiver clara e aderente ao runtime
- a relação entre ETL, publicação e frontend estiver suficientemente representada
- ele puder ser lido como a definição operacional da geração da v2 sem depender do master legado

---

## Última revisão

- Status: ativo
- Classificação: canônico
- Contexto: portal estático / ETL Bash pipeline
- Última revisão: 2026-03-18