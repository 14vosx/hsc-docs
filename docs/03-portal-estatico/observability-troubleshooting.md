# Observability and Troubleshooting

## Objetivo

Documentar os sinais de saúde, os pontos de observabilidade e a sequência padrão de diagnóstico do contexto Portal Estático do ecossistema HSC.

Este documento existe para registrar, de forma estável e auditável:

- como validar a saúde do portal público e da Static API v2
- como separar falhas de ETL, publicação, cache, frontend e fonte de dados
- quais artefatos devem ser observados primeiro
- quais comandos operacionais são usados na triagem
- quais incidentes são mais prováveis nesta camada
- como reduzir o tempo de diagnóstico em produção

---

## Escopo

Este documento cobre:

- sinais de saúde do contexto
- observabilidade da Static API v2
- observabilidade da publicação pública via Nginx
- observabilidade do frontend público
- smoke checks operacionais
- falhas comuns de ETL, JSON, cache e publishing
- sequência recomendada de diagnóstico
- interpretação de sintomas recorrentes

Este documento não cobre em profundidade:

- configuração completa do Nginx
- queries SQL linha a linha
- implementação detalhada dos scripts `gen-*`
- troubleshooting aprofundado da VPS Hostinger
- troubleshooting detalhado do Game Panel
- Auth API no Lightsail

Esses tópicos vivem em documentos próprios deste contexto ou em outros contextos canônicos.

---

## Estado atual

O contexto `03-portal-estatico` possui, no estado operacional conhecido, as seguintes camadas críticas de observabilidade:

- Nginx servindo o portal público
- Static API v2 publicada em arquivos JSON
- pipeline ETL Bash gerando a v2
- `health.json` como artefato técnico da camada de dados
- `matchzy.db` como fonte primária da cadeia estatística
- frontend consumindo os JSONs públicos via browser
- fluxo same-origin de conteúdo, quando aplicável

A observabilidade atual é orientada principalmente por:

- existência e integridade dos arquivos publicados
- `health.json`
- inspeção via `curl`, `jq`, `ls` e timestamps
- validação pública real no browser
- validação da cadeia ETL → publishing → consumo

---

## Source of truth / evidências

As principais evidências deste documento, nesta fase de migração documental, são:

- documentação consolidada atual do ecossistema HSC
- blueprint técnico consolidado do HSC
- reconciliação documental da Static API v2
- reconciliação do pipeline ETL Bash
- reconciliação dos paths públicos do portal e da v2
- experiência operacional já consolidada com geração de JSON, lock global e publicação via Nginx

Enquanto a migração canônica do contexto não estiver concluída, essas fontes seguem sendo usadas como base de reconciliação do estado real.

---

## Relações com outros documentos

Este arquivo é complementar a:

- `docs/03-portal-estatico/README.md`
- `docs/03-portal-estatico/architecture-runtime.md`
- `docs/03-portal-estatico/static-api-v2.md`
- `docs/03-portal-estatico/data-sources-matchzy-sqlite.md`
- `docs/03-portal-estatico/etl-bash-pipeline.md`
- `docs/03-portal-estatico/json-contracts.md`
- `docs/03-portal-estatico/frontend-structure.md`
- `docs/03-portal-estatico/nginx-publishing-cache.md`
- `docs/03-portal-estatico/operational-runbooks.md`
- `docs/01-infra-hostinger/nginx-static-serving.md`
- `docs/01-infra-hostinger/systemd-automation.md`

Este documento descreve triagem, observabilidade e troubleshooting do contexto do portal.  
Ele não substitui os documentos de pipeline, contratos, publicação ou infraestrutura base.

---

## Sinais de saúde do contexto

Os sinais de saúde esperados do contexto incluem:

- portal público respondendo corretamente
- `health.json` acessível e coerente
- `ranking.json`, `matches.json` e `maps.json` acessíveis
- recursos de detalhe acessíveis quando esperados
- JSONs válidos e não truncados
- frontend carregando assets e renderizando dados
- `matchzy.db` acessível ao ETL
- lock global funcionando sem concorrência indevida
- ausência de artefatos temporários órfãos na árvore pública
- same-origin de conteúdo funcionando, quando aplicável

A saúde do contexto deve ser lida como uma cadeia, e não como um único indicador isolado.

---

## Camadas de observabilidade

### Camada 1 — Fonte de dados

Objetivo:
- validar se a origem estatística está acessível e coerente

Sinais típicos:
- `matchzy.db` existe
- SQLite acessível
- tabelas principais presentes
- schema compatível com as queries

---

### Camada 2 — ETL

Objetivo:
- validar se a geração da v2 está funcionando

Sinais típicos:
- lock global íntegro
- statefiles coerentes
- artefatos públicos recentes
- ausência de falha estrutural durante geração

---

### Camada 3 — Publicação

Objetivo:
- validar se os outputs do ETL estão de fato expostos ao público

Sinais típicos:
- Nginx saudável
- paths públicos corretos
- JSONs acessíveis publicamente
- ausência de artefato parcial ou recurso ausente

---

### Camada 4 — Frontend

Objetivo:
- validar se o browser está consumindo os dados e renderizando a interface corretamente

Sinais típicos:
- assets sendo servidos
- fetches funcionando
- páginas públicas preenchidas
- navegação entre lista e detalhe funcionando

---

### Camada 5 — Conteúdo complementar

Objetivo:
- validar fluxos auxiliares, como o same-origin de News, quando ativos

Sinais típicos:
- recurso responde publicamente
- estrutura mínima preservada
- frontend consome o conteúdo sem quebra estrutural

---

## Logs e artefatos principais

A camada do Portal Estático depende mais de artefatos e checks do que de um único log central.

Os principais pontos de observação incluem:

### Árvore pública da v2

```bash
ls -lah /var/www/api/cs2/v2/
```

Uso principal:
- verificar presença de recursos
- verificar timestamps
- detectar ausência de artefatos esperados

---

### Árvore pública do portal

```bash
ls -lah /var/www/portal/cs2/
```

Uso principal:
- verificar assets públicos
- verificar publicação da camada estática

---

### Health da v2

```bash
cat /var/www/api/cs2/v2/health.json
```

Uso principal:
- triagem rápida da camada de dados
- verificação de atualização e consistência operacional

---

### Lock global

```bash
ls -l /opt/cs2-portal/locks/
```

Uso principal:
- detectar concorrência
- detectar lock preso
- validar que a serialização da pipeline continua operando

---

### Statefiles

```bash
ls -lah /opt/cs2-portal/state/
```

Uso principal:
- inspecionar estado incremental
- detectar incoerência ou estagnação operacional

---

### Nginx

```bash
sudo nginx -t
sudo systemctl status nginx
sudo journalctl -u nginx -n 100 --no-pager
```

Uso principal:
- validar publishing público
- separar falha de publicação de falha de ETL

---

## Smoke checks

Os smoke checks mínimos do contexto devem incluir:

### Validar portal público

Substitua pelo domínio público vigente.

```bash
curl -I https://SEU_DOMINIO/
```

### Validar `health.json`

Substitua pelo path público real da v2.

```bash
curl -sS https://SEU_DOMINIO/SEU_PATH_PUBLICO_DA_V2/health.json
```

### Validar recursos centrais

```bash
curl -sS https://SEU_DOMINIO/SEU_PATH_PUBLICO_DA_V2/ranking.json
curl -sS https://SEU_DOMINIO/SEU_PATH_PUBLICO_DA_V2/matches.json
curl -sS https://SEU_DOMINIO/SEU_PATH_PUBLICO_DA_V2/maps.json
```

### Validar sintaxe local dos JSONs

```bash
jq . /var/www/api/cs2/v2/health.json
jq . /var/www/api/cs2/v2/ranking.json
jq . /var/www/api/cs2/v2/matches.json
jq . /var/www/api/cs2/v2/maps.json
```

### Validar um recurso de detalhe

Substitua pelo arquivo realmente existente.

```bash
jq . /var/www/api/cs2/v2/player/SEU_STEAMID64.json
```

### Validar same-origin de conteúdo, quando aplicável

Substitua pelo path público vigente.

```bash
curl -I https://SEU_DOMINIO/SEU_PATH_DE_NEWS
curl -sS https://SEU_DOMINIO/SEU_PATH_DE_NEWS
```

---

## Sequência de diagnóstico

Quando houver incidente neste contexto, usar a seguinte ordem.

### Etapa 1 — Confirmar publicação pública

1. validar portal público
2. validar `health.json` público
3. validar um recurso agregado da v2
4. validar um recurso de detalhe

Comandos típicos:

```bash
curl -I https://SEU_DOMINIO/
curl -sS https://SEU_DOMINIO/SEU_PATH_PUBLICO_DA_V2/health.json
curl -sS https://SEU_DOMINIO/SEU_PATH_PUBLICO_DA_V2/ranking.json
```

---

### Etapa 2 — Confirmar Nginx e publishing

1. validar sintaxe do Nginx
2. validar status do serviço
3. validar árvore pública local
4. conferir permissões básicas

Comandos típicos:

```bash
sudo nginx -t
sudo systemctl status nginx
ls -lah /var/www/portal/cs2/
ls -lah /var/www/api/cs2/v2/
```

---

### Etapa 3 — Confirmar integridade dos JSONs

1. validar `health.json`
2. validar JSONs agregados com `jq`
3. validar um artefato detalhado
4. comparar consistência entre coleção e detalhe

Comandos típicos:

```bash
jq . /var/www/api/cs2/v2/health.json
jq . /var/www/api/cs2/v2/ranking.json
jq . /var/www/api/cs2/v2/matches.json
```

---

### Etapa 4 — Confirmar ETL

1. verificar lock global
2. verificar statefiles
3. verificar timestamps dos arquivos públicos
4. investigar se a geração realmente ocorreu

Comandos típicos:

```bash
ls -l /opt/cs2-portal/locks/
ls -lah /opt/cs2-portal/state/
ls -lah /var/www/api/cs2/v2/
```

---

### Etapa 5 — Confirmar fonte de dados

1. verificar existência do `matchzy.db`
2. verificar leitura básica do banco
3. validar tabelas principais
4. investigar drift de schema, quando necessário

Exemplos representativos:

```bash
ls -l /CAMINHO/DO/matchzy.db
sqlite3 /CAMINHO/DO/matchzy.db ".tables"
sqlite3 /CAMINHO/DO/matchzy.db "SELECT COUNT(*) FROM matchzy_stats_matches;"
```

Substitua o path pelo valor real reconciliado do ambiente.

---

### Etapa 6 — Confirmar frontend

1. abrir a página pública afetada
2. observar se assets carregam
3. observar se o fetch do JSON correspondente funciona
4. verificar se a navegação entre coleção e detalhe está coerente

Aqui a observação real no browser complementa a triagem de filesystem e `curl`.

---

## Falhas comuns

### ETL não roda

Sintomas:

- timestamps da v2 não mudam
- `health.json` não atualiza
- dados públicos ficam stale

Causas comuns:

- lock preso
- script não executado
- problema de permissão
- path da fonte de dados incorreto
- statefile inconsistente

Triagem:
- verificar lock
- verificar statefiles
- verificar timestamps dos artefatos

---

### Lock preso

Sintomas:

- pipeline não avança
- nenhuma atualização nova aparece
- execução operacional fica bloqueada

Causas comuns:

- execução anterior abortada
- concorrência indevida
- limpeza incompleta
- automação interrompida

Triagem:

```bash
ls -l /opt/cs2-portal/locks/
```

---

### `ranking.json` desatualizado

Sintomas:

- portal mostra ranking antigo
- outros recursos podem ou não ter atualizado

Causas comuns:

- `gen-ranking.sh` falhou
- fonte de dados stale
- ETL parcial
- cache público ou do navegador enganando o teste

Triagem:
- validar arquivo local
- validar resposta pública
- comparar timestamps
- verificar se outras partes da v2 atualizaram

---

### `player/{steamid64}.json` inconsistente

Sintomas:

- ranking funciona
- página de player quebra ou mostra dados incoerentes

Causas comuns:

- `gen-players-incremental.sh` falhou
- `gen-players-from-ranking.sh` falhou
- `steamid64` inconsistente
- enriquecimento auxiliar ausente ou stale

Triagem:
- validar `ranking.json`
- validar o arquivo do player com `jq`
- validar se o recurso existe publicamente

---

### `match/{id}.json` ausente ou quebrado

Sintomas:

- lista de partidas funciona
- detalhe de match retorna erro ou não carrega

Causas comuns:

- incremental de detalhe falhou
- path do recurso de detalhe está errado
- publicação parcial
- detalhe nunca foi gerado

Triagem:
- validar `matches.json`
- validar presença do detalhe no filesystem
- validar acesso público ao mesmo recurso

---

### `/content/news/` ou fluxo same-origin quebrado

Sintomas:

- portal principal funciona
- conteúdo complementar não aparece
- fetch específico falha

Causas comuns:

- espelho same-origin não foi atualizado
- path público incorreto
- publishing parcial de conteúdo auxiliar
- mudança não coordenada de origem do recurso

Triagem:
- validar o path público exato
- validar resposta via `curl`
- separar falha de conteúdo auxiliar da saúde da v2 estatística

---

### Portal fetch falhando

Sintomas:

- página carrega
- dados não aparecem
- browser mostra erro de carregamento de recurso

Causas comuns:

- recurso não publicado
- path incorreto
- JSON inválido
- shape incompatível
- publicação pública quebrada
- cache devolvendo estado antigo

Triagem:
- testar a mesma URL com `curl`
- validar o arquivo local com `jq`
- comparar público vs local

---

### Cache servindo versão antiga

Sintomas:

- arquivo local foi atualizado
- público continua mostrando estado antigo
- navegador exibe dado stale

Causas comuns:

- cache do browser
- política de cache inadequada
- troubleshooting feito sempre pelo mesmo cliente com cache persistente

Triagem:
- testar via `curl`
- testar em janela anônima
- comparar timestamps locais com o comportamento público
- validar headers públicos

---

## Interpretação de sintomas

Alguns padrões úteis de interpretação:

### Portal público quebra + JSON público também quebra
Provável problema de publishing, ETL ou fonte de dados.

### Portal público carrega + JSON público quebra
Provável problema concentrado na v2 ou em parte do ETL.

### JSON local válido + JSON público falha
Provável problema de Nginx, path público ou permissão.

### JSON público válido + frontend quebra
Provável problema de compatibilidade de contrato ou frontend.

### `health.json` saudável + recurso específico quebra
Provável falha localizada em artefato, incremental, contrato ou detalhe não coberto pelo health básico.

### Coleção funciona + detalhe quebra
Provável problema no incremental de detalhe, não na geração agregada.

---

## Lacunas atuais de observabilidade

As lacunas conhecidas ou prováveis desta camada incluem:

- observabilidade ainda fortemente manual
- dependência alta de checks por `curl`, `jq` e inspeção de timestamps
- ausência, até onde reconciliado, de métricas formais e alertas estruturados
- `health.json` não substitui validação funcional real de todos os recursos
- necessidade de diagnóstico humano para separar ETL, publishing e frontend

Essas lacunas explicam por que a sequência de diagnóstico precisa ser explícita.

---

## Regras de ouro de troubleshooting

As regras de ouro deste contexto são:

1. nunca assumir que portal “abrindo” significa dados saudáveis
2. sempre validar público e local
3. sempre diferenciar ETL de publishing
4. sempre validar pelo menos uma coleção e um detalhe
5. nunca tratar `health.json` como prova completa de integridade funcional
6. sempre considerar lock e statefiles quando houver stale data
7. sempre considerar o `matchzy.db` como possível raiz do problema
8. nunca concluir que o erro é “do frontend” sem testar o JSON real que ele consome

---

## Limites deste documento

Este documento não detalha:

- execução linha a linha dos scripts do ETL
- queries SQL específicas
- configuração textual completa do Nginx
- debugging avançado do browser
- troubleshooting completo da VPS Hostinger
- troubleshooting detalhado do Game Panel e do MatchZy

Esses tópicos devem ser tratados em documentos complementares.

---

## Critério de pronto deste documento

Este documento pode ser considerado maduro quando:

- os sinais de saúde do contexto estiverem alinhados com a prática real
- a sequência de diagnóstico refletir o uso operacional real do host
- as falhas comuns cobrirem os incidentes mais prováveis do portal
- os comandos aqui descritos forem suficientes para triagem inicial sem depender do master legado
- a distinção entre fonte, ETL, publishing, cache e frontend estiver clara
- ele puder ser usado como runbook real de triagem do contexto Portal Estático

---

## Última revisão

- Status: ativo
- Classificação: canônico
- Contexto: portal estático / observability and troubleshooting
- Última revisão: 2026-03-18