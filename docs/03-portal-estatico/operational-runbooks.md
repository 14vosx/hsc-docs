# Operational Runbooks

## Objetivo

Documentar os procedimentos operacionais recorrentes do contexto Portal Estático do ecossistema HSC, com foco em regeneração da Static API v2, publicação pública, validação funcional e rollback simples da camada.

Este documento existe para registrar, de forma estável e auditável:

- como operar a regeneração da camada pública de dados
- como validar a publicação do portal e da v2
- como executar smoke tests mínimos após mudança operacional
- como verificar integridade básica dos JSONs publicados
- como validar o fluxo de conteúdo same-origin, quando aplicável
- como executar rollback simples da camada pública quando necessário

---

## Escopo

Este documento cobre:

- runbook de regeneração do ETL
- runbook de publicação do portal
- runbook de smoke test
- runbook de validação de JSON
- runbook de validação de conteúdo same-origin
- rollback simples da camada pública
- regras de ouro operacionais do contexto

Este documento não cobre em profundidade:

- troubleshooting avançado da VPS Hostinger
- configuração detalhada do Nginx
- queries SQL linha a linha
- operação detalhada do MatchZy
- Auth API no Lightsail
- mudanças estruturais de arquitetura

Esses tópicos vivem em documentos próprios deste contexto ou em outros contextos canônicos.

---

## Estado atual

O estado operacional conhecido deste contexto é:

- a camada pública do portal depende da geração da Static API v2
- a geração é feita por ETL Bash
- a publicação pública depende de diretórios servidos por Nginx
- a pipeline usa lock global
- a publicação deve preservar escrita atômica
- a validação operacional precisa considerar tanto o filesystem quanto o consumo público real
- o portal continua estático mesmo quando há espelho same-origin de conteúdo auxiliar

A operação saudável do contexto depende de disciplina em três pontos:

- geração
- publicação
- validação

---

## Source of truth / evidências

As principais evidências deste documento, nesta fase de migração documental, são:

- documentação consolidada atual do ecossistema HSC
- blueprint técnico consolidado do HSC
- reconciliação documental da pipeline ETL Bash
- reconciliação dos paths públicos de portal e Static API v2
- runbooks históricos já embutidos na operação prática do ecossistema

Enquanto a migração canônica do contexto não estiver concluída, essas fontes seguem sendo usadas como base de reconciliação do estado real.

---

## Relações com outros documentos

Este arquivo é complementar a:

- `docs/03-portal-estatico/README.md`
- `docs/03-portal-estatico/architecture-runtime.md`
- `docs/03-portal-estatico/etl-bash-pipeline.md`
- `docs/03-portal-estatico/json-contracts.md`
- `docs/03-portal-estatico/nginx-publishing-cache.md`
- `docs/03-portal-estatico/observability-troubleshooting.md`
- `docs/01-infra-hostinger/systemd-automation.md`

Este documento descreve procedimentos recorrentes de operação do portal e da v2.  
Ele não substitui os documentos de arquitetura, pipeline, contratos ou troubleshooting aprofundado.

---

## Princípios operacionais do contexto

Os runbooks deste contexto seguem estes princípios:

- nunca publicar artefato parcial deliberadamente
- nunca assumir que arquivo existente significa publicação saudável
- sempre diferenciar geração bem-sucedida de publicação acessível
- sempre validar público e local quando houver alteração relevante
- sempre tratar lock e statefiles como artefatos operacionais reais
- sempre preferir validação objetiva a percepção visual isolada do portal

Regra de ouro:

**o contexto só está saudável quando o ETL gera, o Nginx publica e o frontend consome corretamente**

---

## Runbook de regeneração ETL

Este runbook deve ser usado quando houver necessidade de:

- regenerar a v2
- validar que a pipeline continua funcional
- refletir novos dados públicos
- recuperar a camada pública após falha parcial de geração
- verificar consistência após mudança em scripts, SQL ou paths

### Pré-condições

Antes de iniciar:

- confirmar acesso ao host
- confirmar acesso aos paths do contexto
- confirmar que o `matchzy.db` está acessível
- confirmar que não há execução concorrente da pipeline
- confirmar que o lock global não está preso indevidamente
- confirmar que há espaço e permissões adequadas na árvore pública

### Paths relevantes

- `/opt/cs2-portal/`
- `/opt/cs2-portal/locks/`
- `/opt/cs2-portal/state/`
- `/var/www/api/cs2/v2/`

### Sequência lógica

1. verificar lock global
2. verificar acesso à fonte de dados
3. executar a pipeline de geração conforme o fluxo operacional vigente
4. validar atualização dos artefatos públicos principais
5. validar `health.json`
6. validar recursos públicos centrais
7. só depois validar o frontend

### Verificar lock global

```bash
ls -l /opt/cs2-portal/locks/
```

### Verificar publicação atual antes da regeneração

```bash
ls -lah /var/www/api/cs2/v2/
```

### Verificar a saúde atual da camada

```bash
cat /var/www/api/cs2/v2/health.json
```

### Observação operacional

A forma exata de invocação da pipeline pode variar entre:

- um script agregador do contexto
- uma chamada manual dos scripts `gen-*`
- automação via timers do host

Enquanto o comando agregador real não estiver fixado aqui com validação definitiva, este runbook documenta a lógica operacional, não um comando inventado.

Regra editorial:
- não registrar comando final de regeneração sem reconciliação com o host real
- registrar a lógica e os checks é preferível a inventar uma linha de execução

---

## Runbook de publicação do portal

Este runbook deve ser usado quando houver necessidade de:

- validar a publicação pública do frontend
- verificar se o portal está servindo assets corretamente
- confirmar se a árvore pública do portal continua acessível
- validar se a borda pública está coerente após alteração em assets ou publishing

### Pré-condições

Antes de iniciar:

- confirmar que o Nginx está ativo
- confirmar que a árvore pública do portal existe
- confirmar que os assets esperados estão presentes
- confirmar que não houve alteração não coordenada de path público

### Path relevante

- `/var/www/portal/cs2/`

### Verificar a árvore pública do portal

```bash
ls -lah /var/www/portal/cs2/
```

### Validar Nginx

```bash
sudo nginx -t
sudo systemctl status nginx
```

### Validar acesso público ao portal

Substitua pela URL pública vigente do portal.

```bash
curl -I https://SEU_DOMINIO/
```

### Validar assets publicamente, quando necessário

Substitua pelos paths públicos reais de assets conhecidos.

```bash
curl -I https://SEU_DOMINIO/SEU_ASSET.css
curl -I https://SEU_DOMINIO/SEU_MODULO.js
```

### Critério de sucesso

A publicação do portal deve ser tratada como saudável quando:

- a árvore pública existe
- o Nginx está íntegro
- o portal responde publicamente
- os assets principais são servidos corretamente
- o frontend carrega no browser sem erro estrutural evidente

---

## Runbook de smoke test

Este runbook deve ser usado:

- após regeneração relevante da v2
- após alteração em publishing
- após ajuste de Nginx
- após mudança estrutural no portal
- após incidente operacional

### Objetivo

Validar rapidamente que o contexto continua útil publicamente.

### Sequência mínima

1. validar portal público
2. validar `health.json`
3. validar `ranking.json`
4. validar `matches.json`
5. validar um recurso de detalhe conhecido
6. validar que o frontend consome esses artefatos sem quebra estrutural

### Comandos representativos

Substitua pelo hostname público e path real da v2.

```bash
curl -I https://SEU_DOMINIO/
curl -sS https://SEU_DOMINIO/SEU_PATH_PUBLICO_DA_V2/health.json
curl -sS https://SEU_DOMINIO/SEU_PATH_PUBLICO_DA_V2/ranking.json
curl -sS https://SEU_DOMINIO/SEU_PATH_PUBLICO_DA_V2/matches.json
```

### Validação estrutural de JSON com `jq`

```bash
jq . /var/www/api/cs2/v2/health.json
jq . /var/www/api/cs2/v2/ranking.json
jq . /var/www/api/cs2/v2/matches.json
```

### Critério de sucesso

O smoke test só deve ser tratado como bem-sucedido quando:

- o portal responde
- a v2 responde
- os JSONs são válidos
- pelo menos uma rota agregada e uma rota de detalhe passam na validação
- não há sinal imediato de shape quebrado ou artefato parcial

---

## Runbook de validação de JSON

Este runbook deve ser usado quando houver suspeita de:

- JSON inválido
- JSON truncado
- shape incompatível com o frontend
- publicação parcial
- falha isolada em um recurso da v2

### Objetivo

Confirmar rapidamente se o problema está no artefato publicado.

### Sequência lógica

1. localizar o arquivo público
2. validar sintaxe JSON
3. validar shape macro esperado
4. validar coerência com recurso relacionado
5. validar acesso público ao mesmo recurso

### Exemplos de validação local

```bash
jq . /var/www/api/cs2/v2/health.json
jq . /var/www/api/cs2/v2/ranking.json
jq . /var/www/api/cs2/v2/matches.json
```

### Exemplo de validação pública

Substitua pelo path público real.

```bash
curl -sS https://SEU_DOMINIO/SEU_PATH_PUBLICO_DA_V2/ranking.json
```

### Casos de coerência cruzada

Alguns exemplos importantes:

- item de `ranking.json` deve apontar para `player/{steamid64}.json`
- item de `matches.json` deve apontar para `match/{id}.json`
- item de `maps.json` deve apontar para `map/{map}.json`

### Critério de sucesso

A validação do JSON só é satisfatória quando:

- o arquivo é JSON válido
- o shape macro continua útil ao frontend
- o recurso relacionado também existe e está coerente
- o mesmo artefato é acessível localmente e publicamente

---

## Runbook de validação de News same-origin

Este runbook deve ser usado quando houver fluxo same-origin de conteúdo ativo e for necessário validar:

- se o conteúdo de News está acessível publicamente
- se a publicação same-origin está íntegra
- se o portal consegue consumir essa camada complementar

### Objetivo

Separar problema de espelho/publicação same-origin de problema do restante do portal.

### Sequência lógica

1. identificar o path público vigente do conteúdo same-origin
2. validar resposta pública
3. validar estrutura mínima do artefato retornado
4. validar consumo no frontend, se aplicável

### Exemplo representativo

Substitua pelo path real vigente.

```bash
curl -sS https://SEU_DOMINIO/SEU_PATH_DE_NEWS
curl -I https://SEU_DOMINIO/SEU_PATH_DE_NEWS
```

### Critério de sucesso

O fluxo same-origin de News deve ser tratado como saudável quando:

- o recurso responde publicamente
- a estrutura mínima esperada está preservada
- o portal consome esse conteúdo sem erro estrutural
- a quebra dessa camada não afeta indevidamente o restante da v2 estatística

---

## Runbook de rollback simples

Este runbook deve ser usado quando houver necessidade de:

- voltar rapidamente ao último estado público estável
- desfazer mudança de asset estático
- desfazer publicação pública incorreta
- recuperar a camada após publishing problemático

### Importante

Este rollback é de camada pública estática.  
Ele não deve ser confundido com:

- rollback de aplicação no Lightsail
- restore de banco
- correção de schema SQLite
- reversão do runtime do jogo

### Estratégia mínima

1. identificar qual camada quebrou:
   - assets do portal
   - artefatos da v2
   - same-origin de conteúdo
2. identificar o último estado público estável conhecido
3. restaurar esse estado na árvore pública correspondente
4. validar Nginx
5. reexecutar smoke tests mínimos

### Validações após rollback

```bash
sudo nginx -t
sudo systemctl status nginx
curl -I https://SEU_DOMINIO/
curl -sS https://SEU_DOMINIO/SEU_PATH_PUBLICO_DA_V2/health.json
```

### Critério de sucesso

O rollback simples só é considerado concluído quando:

- a publicação pública volta a responder
- a v2 volta a estar acessível
- o frontend volta a consumir a camada sem quebra estrutural evidente
- o problema que motivou o rollback deixa de se reproduzir

### Regra importante

Se a quebra veio de:

- fonte de dados
- schema
- ETL lógico
- statefile inconsistente
- lock preso

então rollback de publishing sozinho pode não resolver.  
Nesses casos, é necessário seguir para troubleshooting do pipeline.

---

## Regras de ouro

As regras de ouro operacionais deste contexto são:

1. nunca assumir que Nginx saudável significa portal saudável
2. nunca assumir que JSON existente significa JSON válido
3. nunca assumir que `health.json` saudável significa contrato íntegro em todos os recursos
4. sempre validar pelo menos uma coleção e um detalhe
5. sempre diferenciar problema de ETL, publishing e frontend
6. sempre considerar lock e statefiles no diagnóstico da geração
7. sempre testar acesso público real após alteração
8. nunca documentar “comando final” não validado no host real

---

## Problemas comuns

### 1. Regeneração executa, mas dados não mudam

Causas comuns:

- fonte de dados stale
- statefile inconsistente
- pipeline parcial
- lock preso anteriormente

---

### 2. Publicação local existe, mas público não responde

Causas comuns:

- problema de Nginx
- alias/root incorreto
- permissão de leitura
- cache enganando a validação

---

### 3. JSON local é válido, mas frontend quebra

Causas comuns:

- shape incompatível
- dependência de detalhe ausente
- recurso incremental não gerado
- módulo JS esperando campo diferente

---

### 4. Mesmo após rollback simples, erro persiste

Causas comuns:

- problema não estava na árvore pública
- falha real está em ETL, schema ou fonte de dados
- cache do cliente ou path público incorreto continua interferindo

---

## Limites deste documento

Este documento não detalha:

- cada comando real de automação do host ainda não reconciliado
- a execução script por script do ETL em ambiente real
- restore de statefiles
- recovery avançado de publicação
- troubleshooting aprofundado do browser
- rollback complexo multi-camada

Esses pontos podem ser formalizados posteriormente após reconciliação adicional com o runtime real.

---

## Critério de pronto deste documento

Este documento pode ser considerado maduro quando:

- os procedimentos aqui descritos refletirem a prática operacional real do host
- o comando agregador real de regeneração, se existir, estiver confirmado
- os smoke tests cobrirem corretamente a camada pública
- a validação same-origin estiver reconciliada com o path real do ambiente
- o rollback simples estiver claro e usável
- ele puder ser usado como runbook prático do contexto sem depender do master legado

---

## Última revisão

- Status: ativo
- Classificação: canônico
- Contexto: portal estático / operational runbooks
- Última revisão: 2026-03-18