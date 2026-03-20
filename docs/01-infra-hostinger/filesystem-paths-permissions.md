# Filesystem Paths and Permissions

## Objetivo

Documentar os paths, ownerships, grupos e permissões da camada-base da Infra Hostinger, registrando quais diretórios são críticos para o funcionamento do lado game + portal do ecossistema HSC.

Este documento existe para registrar, de forma estável e auditável:

- quais paths do host são canônicos
- quais áreas são públicas e quais são operacionais
- como ownership e grupos precisam se organizar
- como Nginx, ETL, AMP e demais workloads dependem dessas permissões
- quais riscos surgem quando paths ou permissões divergem do esperado
- quais validações mínimas ajudam a manter a camada íntegra

---

## Escopo

Este documento cobre:

- paths oficiais da camada-base Hostinger
- áreas de publicação pública
- áreas operacionais do portal/ETL
- áreas operacionais do lado AMP/CS2 em nível estrutural
- ownership e grupos relevantes
- convenções de permissão do host
- relação entre `www-data`, `amp` e demais necessidades do runtime
- riscos comuns ligados a path drift e permissionamento incorreto

Este documento não cobre em profundidade:

- operação detalhada do AMP Instance Manager
- operação da instância `MixHAXIXE01`
- runbooks de partida
- queries SQL da v2
- contratos JSON
- Auth API no Lightsail
- credenciais e arquivos sensíveis

Esses tópicos vivem em documentos próprios dos contextos `02-game-panel`, `03-portal-estatico` e `04-infra-aws-lightsail`.

---

## Estado atual

O estado operacional conhecido da camada de filesystem do lado Hostinger inclui:

- publicação pública do portal em `/var/www/portal/cs2/`
- publicação pública da Static API v2 em `/var/www/api/cs2/v2/`
- base operacional do ETL em `/opt/cs2-portal/`
- locks e statefiles do ETL sob `/opt/cs2-portal/`
- SQLs versionados do portal sob `/opt/cs2-portal/sql/`
- scripts operacionais em `/usr/local/bin/`
- árvore operacional do lado AMP sob `/home/amp/.ampdata/instances/`
- dependência de leitura pública por `www-data`
- dependência de escrita por processos operacionais do host e/ou usuários associados ao fluxo de geração

Também já existe evidência reconciliada de que ownership e permissões incorretos causam falhas reais de:

- publicação
- leitura pública
- escrita do ETL
- upload/manual update de assets
- acesso do Nginx aos arquivos finais

---

## Source of truth / evidências

As principais evidências deste documento, nesta fase de migração documental, são:

- documentação consolidada atual do ecossistema HSC
- blueprint técnico consolidado do HSC
- documentação reconciliada da Infra Hostinger
- documentação reconciliada do Portal Estático e da camada ETL
- histórico operacional ligado a ownership, grupos e permissões do lado Hostinger

Enquanto a migração canônica do contexto não estiver concluída, essas fontes seguem sendo usadas como base de reconciliação do estado real.

---

## Relações com outros documentos

Este arquivo é complementar a:

- `docs/01-infra-hostinger/README.md`
- `docs/01-infra-hostinger/infra-hostinger-architecture-runtime.md`
- `docs/01-infra-hostinger/nginx-static-serving.md`
- `docs/01-infra-hostinger/systemd-automation.md`
- `docs/01-infra-hostinger/observability-troubleshooting.md`
- `docs/02-game-panel/instance-mixhaxixe01.md`
- `docs/../03-portal-estatico/portal-estatico-architecture-runtime.md`
- `docs/03-portal-estatico/etl-bash-pipeline.md`
- `docs/03-portal-estatico/nginx-publishing-cache.md`

Este documento descreve a camada de paths e permissões do host Hostinger.  
Ele não substitui os documentos de Game Panel, ETL, publishing ou troubleshooting aprofundado.

---

## Princípios gerais de filesystem do host

A camada Hostinger depende de uma separação clara entre:

- áreas públicas
- áreas operacionais
- áreas de automação
- áreas de runtime do jogo
- áreas sensíveis ou não publicáveis

Princípios canônicos:

- o Nginx deve ler apenas artefatos públicos finais
- o ETL deve escrever em áreas operacionais e publicar em áreas públicas de forma controlada
- o runtime do jogo deve operar em sua árvore própria, sem ser tratado como publishing público
- ownership e grupos devem refletir responsabilidade real de leitura e escrita
- nenhum path crítico deve depender de “acerto informal” não documentado

---

## Paths oficiais do contexto

Os paths abaixo formam o inventário estrutural principal da camada Hostinger.

### Publicação do portal

- `/var/www/portal/cs2/`

Papel:
- servir HTML, CSS, JavaScript, imagens e assets públicos do portal

Natureza:
- público
- legível pelo Nginx

---

### Publicação da Static API v2

- `/var/www/api/cs2/v2/`

Papel:
- servir `health.json`, `ranking.json`, `matches.json`, `maps.json`, recursos de detalhe e demais artefatos públicos da v2

Natureza:
- público
- legível pelo Nginx
- escrito pelo fluxo de geração/publicação do ETL

---

### Base operacional do portal / ETL

- `/opt/cs2-portal/`

Papel:
- concentrar a base operacional do contexto do portal

Natureza:
- operacional
- não pública

Subáreas relevantes:
- `locks/`
- `state/`
- `sql/`

---

### Locks do ETL

- `/opt/cs2-portal/locks/`

Papel:
- serializar execução da pipeline
- evitar concorrência indevida

Natureza:
- operacional
- não pública

---

### Lock global principal

- `/opt/cs2-portal/locks/gen-all-v2.lock`

Papel:
- lock global conhecido da geração da v2

Natureza:
- operacional
- não pública
- sensível ao fluxo correto de execução

---

### Statefiles do ETL

- `/opt/cs2-portal/state/`

Papel:
- sustentar incremental
- registrar progresso/estado da geração

Natureza:
- operacional
- não pública

---

### SQLs do portal

- `/opt/cs2-portal/sql/`

Papel:
- armazenar queries SQL versionadas usadas pela geração da v2

Natureza:
- operacional
- não pública

---

### Scripts operacionais

- `/usr/local/bin/`

Papel:
- concentrar scripts do host, inclusive scripts `gen-*` ou equivalentes usados na automação do portal e de outras camadas do lado Hostinger

Natureza:
- operacional
- não pública

---

### Base estrutural do lado AMP

- `/home/amp/.ampdata/instances/`

Papel:
- sustentar a árvore operacional das instâncias AMP
- hospedar, indiretamente, o contexto do servidor CS2 e a origem do `matchzy.db`

Natureza:
- operacional
- não pública

---

## Áreas de publicação

As áreas de publicação pública já reconciliadas são:

- `/var/www/portal/cs2/`
- `/var/www/api/cs2/v2/`

Regras canônicas dessas áreas:

- devem conter apenas artefatos finais públicos
- precisam ser legíveis por `www-data`
- não devem expor bancos, statefiles, locks, scripts ou materiais internos
- devem permanecer estáveis para evitar quebra de paths públicos

Essas áreas representam a face pública do lado Hostinger.

---

## Áreas operacionais

As áreas operacionais do host já reconciliadas incluem:

- `/opt/cs2-portal/`
- `/opt/cs2-portal/locks/`
- `/opt/cs2-portal/state/`
- `/opt/cs2-portal/sql/`
- `/usr/local/bin/`
- `/home/amp/.ampdata/instances/`

Regras canônicas dessas áreas:

- não são para serving público
- podem exigir escrita por automações, ETL ou runtime do jogo
- devem manter ownership e grupos coerentes com quem realmente executa os fluxos
- precisam ser protegidas contra exposição indevida pelo Nginx

---

## Ownership e grupos

A operação do lado Hostinger já evidencia a importância de alinhar ownership e grupos com a função real de cada diretório.

A convenção mais relevante já reconciliada nesta fase é a coexistência entre:

- `amp`
- `www-data`

Essa coexistência aparece especialmente quando a geração ou manipulação de artefatos públicos precisa ser feita por uma camada operacional que não é a mesma que lê os arquivos publicamente.

### Papel de `www-data`

`www-data` representa a camada de leitura pública do Nginx.

Necessidades típicas:

- ler assets do portal
- ler JSONs da v2
- servir conteúdo público sem bloqueio de permissão

### Papel de `amp`

`amp` representa a camada operacional associada ao lado Game Panel / CS2 e, por extensão, partes do ecossistema que tocam a fonte de dados ou áreas ligadas ao runtime do jogo.

Necessidades típicas:

- manter controle sobre a árvore operacional da instância
- preservar acesso à base de dados do runtime do jogo
- coexistir com fluxos que precisam publicar outputs derivados

---

## Convenção `amp:www-data`

Há evidência reconciliada de uso e importância da convenção:

- `amp:www-data`

Essa convenção é particularmente útil quando:

- o owner operacional precisa continuar sendo `amp`
- o grupo precisa permitir leitura ou colaboração da camada servida/publicada
- o Nginx precisa ler o resultado final
- o ETL ou publicação precisa ocorrer sem quebrar a estrutura operacional do lado game

Regra importante:
- a convenção de owner:group não é cosmética
- ela é parte do contrato operacional entre geração e serving

---

## Convenções de permissão

As permissões do host precisam refletir a natureza de cada área.

### Em áreas públicas

Espera-se, no mínimo:

- leitura pelo Nginx (`www-data`)
- traversal correto de diretórios
- ausência de bloqueio indevido para serving público

### Em áreas operacionais

Espera-se:

- escrita pelos responsáveis reais do fluxo
- leitura adequada por scripts, ETL e runtime necessários
- ausência de exposição pública
- integridade contra mudanças acidentais por usuários ou processos errados

### Em áreas híbridas de publicação derivada

Quando um artefato nasce de uma área operacional e termina numa área pública, a permissão precisa permitir:

- escrita controlada na publicação
- leitura pública estável
- ausência de lacunas entre quem gera e quem serve

---

## Regras para Nginx / `www-data`

A camada pública depende de `www-data` conseguir:

- atravessar os diretórios públicos
- ler os arquivos finais do portal
- ler os arquivos finais da v2
- servir assets, JSONs e conteúdos auxiliares públicos

Se `www-data` não conseguir ler:

- o portal quebra
- a v2 quebra
- o troubleshooting pode parecer de Nginx quando a raiz é permissão

Regra canônica:
- a árvore pública deve sempre ser validada do ponto de vista de leitura do edge

---

## Regras para AMP

A camada AMP depende de a árvore operacional manter:

- ownership coerente com o runtime do jogo
- acesso íntegro à estrutura da instância
- integridade de paths da árvore `~amp/.ampdata`
- preservação do ambiente do lado Game Panel

Regra canônica:
- a infraestrutura base não deve “corrigir” permissões do lado AMP de forma cega e destrutiva
- mudanças nessa árvore precisam respeitar o contexto `02-game-panel`

---

## Relação entre ETL e permissões

O ETL depende fortemente de permissões corretas para:

- ler a fonte de dados
- usar `locks/`
- usar `state/`
- ler SQLs
- escrever artefatos públicos
- permitir leitura posterior pelo Nginx

Isso significa que permissões quebradas podem produzir sintomas como:

- ETL não roda
- ETL roda, mas não publica
- artefatos existem, mas o Nginx não serve
- assets sobem manualmente, mas ficam indisponíveis publicamente

A causa raiz, nesses casos, pode estar inteiramente no filesystem.

---

## Riscos de permissões incorretas

Os principais riscos desta camada incluem:

- Nginx sem leitura sobre arquivos públicos
- ETL sem escrita na árvore pública
- ETL sem acesso a `locks/` ou `state/`
- runtime do jogo sem integridade sobre sua própria árvore
- uploads/copias manuais falhando com “permission denied”
- publicação parcial por ownership divergente
- troubleshooting falso atribuído a Nginx, quando a causa real é permissão

Esses riscos são frequentes e estruturais.

---

## Sinais de saúde do filesystem

Os sinais de saúde esperados incluem:

- diretórios públicos existentes
- diretórios operacionais existentes
- ownership coerente com a função da área
- leitura pública funcionando
- escrita operacional funcionando
- ausência de `permission denied` nos fluxos normais
- ausência de drift entre owner, group e uso real da camada

A saúde do filesystem deve ser inferida não só pela existência dos diretórios, mas pela sua usabilidade real pelos processos corretos.

---

## Comandos de validação

Os comandos abaixo representam verificações típicas da camada de paths e permissões.

### Verificar ownership e permissões da publicação do portal

```bash
ls -ld /var/www/portal/cs2/
ls -lah /var/www/portal/cs2/
```

### Verificar ownership e permissões da publicação da v2

```bash
ls -ld /var/www/api/cs2/v2/
ls -lah /var/www/api/cs2/v2/
```

### Verificar ownership e permissões da base do ETL

```bash
ls -ld /opt/cs2-portal/
ls -lah /opt/cs2-portal/
```

### Verificar locks e statefiles

```bash
ls -ld /opt/cs2-portal/locks/
ls -ld /opt/cs2-portal/state/
ls -lah /opt/cs2-portal/locks/
ls -lah /opt/cs2-portal/state/
```

### Verificar SQLs e scripts

```bash
ls -ld /opt/cs2-portal/sql/
ls -lah /opt/cs2-portal/sql/
ls -lah /usr/local/bin/
```

### Verificar árvore AMP

```bash
ls -ld /home/amp/.ampdata/instances/
ls -lah /home/amp/.ampdata/instances/
```

### Verificar usuário e grupo de arquivos específicos

Exemplo representativo:

```bash
stat /var/www/api/cs2/v2/health.json
stat /var/www/portal/cs2/index.html
```

---

## Problemas comuns

### 1. `permission denied` ao copiar ou publicar assets

Causas comuns:

- owner incorreto
- grupo incorreto
- diretório público sem permissão de escrita adequada ao fluxo operacional

Impacto:
- portal não recebe atualização
- intervenção manual falha

---

### 2. ETL gera parcialmente ou não publica

Causas comuns:

- sem escrita em `/var/www/api/cs2/v2/`
- sem acesso a `locks/` ou `state/`
- owner/group divergentes do esperado

Impacto:
- v2 stale
- artefato parcial
- falsa percepção de falha lógica em script

---

### 3. Nginx não consegue servir arquivo que existe

Causas comuns:

- `www-data` sem leitura
- traversal de diretório bloqueado
- owner/group sem compatibilidade com a camada pública

Impacto:
- arquivo existe no host, mas público responde erro ou indisponibilidade

---

### 4. Mudança no lado AMP quebra a leitura da fonte de dados

Causas comuns:

- alteração de ownership/path na árvore da instância
- ajuste de permissão sem considerar dependências do ETL

Impacto:
- portal quebra indiretamente por não conseguir mais ler sua fonte de dados

---

### 5. Grupo correto, owner errado ou vice-versa

Causas comuns:

- correção manual incompleta
- cópia/extração feita por usuário diferente
- deploy operacional sem padronização

Impacto:
- sintomas intermitentes
- parte do fluxo funciona, parte falha

---

## Limites deste documento

Este documento não detalha:

- valores numéricos exatos de chmod para cada diretório sem validação fina do host
- toda a árvore de arquivos do lado AMP
- a lógica do ETL que opera sobre esses paths
- a configuração textual do Nginx
- os detalhes do runtime do jogo
- os detalhes da Auth API

Esses tópicos pertencem a documentos específicos ou exigem validação adicional do ambiente real.

---

## Critério de pronto deste documento

Este documento pode ser considerado maduro quando:

- os paths críticos estiverem confirmados no host real
- a convenção de ownership e grupos estiver reconciliada sem ambiguidade
- o papel de `amp` e `www-data` estiver claro
- os riscos de permissão incorreta estiverem bem representados
- ele puder ser usado como referência real de paths e permissionamento sem depender do master legado

---

## Última revisão

- Status: ativo
- Classificação: canônico
- Contexto: infraestrutura Hostinger / filesystem paths and permissions
- Última revisão: 2026-03-18