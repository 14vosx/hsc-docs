# References and Inventory

## Objetivo

Consolidar o inventário de referências, evidências, artefatos operacionais, paths críticos e itens pendentes de validação do contexto Portal Estático do ecossistema HSC.

Este documento existe para registrar, de forma estável e auditável:

- quais documentos alimentam este contexto
- quais artefatos reais do ambiente são source of truth operacional
- quais paths e recursos públicos são críticos para a camada
- quais comandos ajudam a validar o contexto
- quais pontos ainda dependem de confirmação direta no ambiente real
- quais são os limites documentais deste contexto

---

## Escopo

Este documento cobre:

- documentos de origem do contexto
- artefatos reais do runtime do portal
- paths críticos conhecidos
- recursos públicos relevantes da v2
- componentes operacionais centrais
- comandos de validação
- itens pendentes de confirmação
- limites documentais do contexto

Este documento não cobre em profundidade:

- explicação detalhada de cada script do ETL
- contratos JSON campo a campo
- troubleshooting aprofundado
- infraestrutura completa da VPS Hostinger
- operação detalhada do Game Panel
- Auth API no Lightsail

Esses assuntos vivem nos documentos especializados do contexto ou em outros contextos canônicos.

---

## Estado atual

O contexto `03-portal-estatico` já possui estrutura canônica definida para documentar:

- arquitetura de runtime do portal
- Static API v2
- fonte de dados baseada em MatchZy SQLite
- pipeline ETL Bash
- contratos JSON
- estrutura do frontend
- publishing e cache via Nginx
- runbooks operacionais
- observabilidade e troubleshooting

Neste estágio da migração, o contexto já foi estruturado documentalmente, mas ainda depende de reconciliação progressiva com o ambiente real para fechar alguns pontos específicos de path, comandos operacionais agregadores e publishing público final.

---

## Source of truth / evidências

As evidências que sustentam este contexto se dividem em quatro grupos principais.

### 1. Documentação consolidada do ecossistema

Usada para:

- visão macro do papel do Portal Estático
- separação entre portal, Hostinger, Game Panel e Auth API
- topologia geral da cadeia pública de dados

### 2. Documentação específica do portal e da v2

Usada para:

- estrutura da Static API v2
- pipeline ETL
- paths públicos
- regras de geração
- relação entre portal e JSONs publicados

### 3. Impl-logs e registros incrementais

Usados para:

- rastrear mudanças de publishing
- rastrear integração de conteúdo same-origin
- registrar alterações relevantes de ETL, contratos ou borda pública

### 4. Ambiente real do host

Usado para:

- validar paths efetivos
- validar comandos reais de geração
- confirmar estrutura de publicação
- confirmar presença e integridade dos recursos públicos
- confirmar dependências entre Nginx, ETL, SQLite e frontend

Regra principal:
- quando houver conflito entre documento histórico e ambiente real validado, prevalece o ambiente real validado

---

## Relações com outros documentos

Este arquivo é complementar a:

- `docs/03-portal-estatico/README.md`
- `docs/03-portal-estatico/architecture-runtime.md`
- `docs/03-portal-estatico/static-api-v2.md`
- `docs/03-portal-estatico/data-sources-matchzy-sqlite.md`
- `docs/03-portal-estatico/etl-bash-pipeline.md`
- `docs/03-portal-estatico/sql-queries-and-views.md`
- `docs/03-portal-estatico/json-contracts.md`
- `docs/03-portal-estatico/frontend-structure.md`
- `docs/03-portal-estatico/nginx-publishing-cache.md`
- `docs/03-portal-estatico/operational-runbooks.md`
- `docs/03-portal-estatico/observability-troubleshooting.md`

Este documento não substitui nenhum dos arquivos acima.  
Ele funciona como fechamento de inventário e referência do contexto.

---

## Documentos de origem

Os documentos abaixo são as principais fontes de extração e reconciliação deste contexto.

### Fontes primárias

- documentação específica do portal e da Static API v2
- documentação consolidada atual do ecossistema HSC
- blueprint técnico consolidado do HSC

### Fontes secundárias

- impl-logs ligados a mirror same-origin de conteúdo
- impl-logs ligados a ETL e publishing
- documentação reconciliada da Infra Hostinger
- documentação reconciliada da camada Game Panel, especialmente no que toca à origem do `matchzy.db`

### Fontes de apoio histórico

- documentos legados do portal
- master antigo
- blueprint histórico
- materiais `_old` que ajudem a reconciliar paths, decisões e fluxos

Regra documental:
- fontes históricas ajudam a reconciliar
- mas não governam o canônico sozinhas

---

## Artefatos reais do contexto

Os artefatos reais conhecidos ou esperados do contexto Portal Estático incluem:

### Portal público

- árvore pública do portal em `/var/www/portal/cs2/`

### Static API v2

- árvore pública da v2 em `/var/www/api/cs2/v2/`

### Base operacional do ETL

- `/opt/cs2-portal/`

### Diretório de locks

- `/opt/cs2-portal/locks/`

### Lock global principal

- `/opt/cs2-portal/locks/gen-all-v2.lock`

### Diretório de statefiles

- `/opt/cs2-portal/state/`

### SQLs versionados

- `/opt/cs2-portal/sql/`

### Scripts operacionais

- `/usr/local/bin/*.sh`

### Fonte de dados

- `matchzy.db` sob a árvore operacional da instância `MixHAXIXE01`

### Artefato de saúde

- `health.json` na árvore pública da v2

Esses artefatos devem ser tratados como inventário-base do contexto até revisão explícita.

---

## Componentes operacionais centrais

Os componentes principais conhecidos deste contexto são:

### Portal estático

Papel:
- interface pública do HSC
- consumo de dados da v2
- renderização em HTML/CSS/JS ESModules

### Static API v2

Papel:
- camada pública de dados
- contrato entre ETL e frontend
- publicação por arquivos JSON

### ETL Bash

Papel:
- transformar o SQLite em JSON público
- orquestrar geração e incremental
- preservar lock e atomicidade

### SQLite MatchZy

Papel:
- fonte primária de dados estatísticos
- origem de ranking, matches, players e maps

### Nginx

Papel:
- servir o portal
- servir a v2
- publicar same-origin de conteúdo quando aplicável
- proteger artefatos não públicos

---

## Paths críticos

Os paths críticos conhecidos deste contexto incluem:

### Publicação do portal

- `/var/www/portal/cs2/`

### Publicação da v2

- `/var/www/api/cs2/v2/`

### Base operacional do portal

- `/opt/cs2-portal/`

### Locks

- `/opt/cs2-portal/locks/`

### Lock global

- `/opt/cs2-portal/locks/gen-all-v2.lock`

### Statefiles

- `/opt/cs2-portal/state/`

### SQLs

- `/opt/cs2-portal/sql/`

### Scripts operacionais

- `/usr/local/bin/`

### Banco de origem

- path do `matchzy.db` dentro da árvore da instância `MixHAXIXE01`

Regra prática:
- qualquer mudança nesses paths deve ser tratada como alteração relevante e refletida no canônico e, quando necessário, no impl-log

---

## Recursos públicos principais

Os recursos públicos já reconciliados como centrais neste contexto incluem:

### Saúde

- `health.json`

### Coleções públicas

- `ranking.json`
- `matches.json`
- `maps.json`

### Recursos de detalhe

- `match/{id}.json`
- `map/{map}.json`
- `player/{steamid64}.json`

### Recursos auxiliares

- `steam-cache/{steamid64}.json`

### Conteúdo same-origin, quando aplicável

- path público de News ou conteúdo espelhado sob o domínio do portal

Regra editorial:
- este documento inventaria recursos
- a semântica operacional detalhada vive em `static-api-v2.md` e `json-contracts.md`

---

## Tabelas e entidades operacionais relevantes

As estruturas de persistência já reconciliadas como relevantes para a v2 incluem:

- `matchzy_stats_matches`
- `matchzy_stats_maps`
- `matchzy_stats_players`

Essas tabelas sustentam, em nível macro:

- listagem e detalhe de partidas
- agregados e detalhe por mapa
- ranking e dossiê público de jogador

Também há dependência de identificadores estruturais, especialmente:

- `steamid64`
- identificadores de match
- identificadores/nomes públicos de mapa

---

## Serviços e dependências indiretas

Embora o contexto do portal não tenha backend dinâmico próprio, ele depende indiretamente de:

### Nginx

- serving público do portal e da v2

### Automação do host

- timers e/ou execução operacional do ETL

### Runtime do jogo

- produção do `matchzy.db`

### Filesystem e permissões

- escrita pelo ETL
- leitura pelo Nginx

### Browser

- consumo dos JSONs e assets públicos

Essas dependências explicam por que o contexto precisa de documentação separada da Infra Hostinger e do Game Panel, mesmo rodando sobre eles.

---

## Comandos de validação

Os comandos abaixo formam um kit mínimo de validação do contexto.

### Validar árvore pública do portal

```bash
ls -lah /var/www/portal/cs2/
```

### Validar árvore pública da v2

```bash
ls -lah /var/www/api/cs2/v2/
```

### Validar `health.json`

```bash
cat /var/www/api/cs2/v2/health.json
```

### Validar JSONs principais com `jq`

```bash
jq . /var/www/api/cs2/v2/health.json
jq . /var/www/api/cs2/v2/ranking.json
jq . /var/www/api/cs2/v2/matches.json
jq . /var/www/api/cs2/v2/maps.json
```

### Validar lock global

```bash
ls -l /opt/cs2-portal/locks/
```

### Validar statefiles

```bash
ls -lah /opt/cs2-portal/state/
```

### Validar árvore de SQLs

```bash
ls -lah /opt/cs2-portal/sql/
```

### Validar Nginx

```bash
sudo nginx -t
sudo systemctl status nginx
```

### Validar acesso público ao portal

Substitua pelo domínio público vigente.

```bash
curl -I https://SEU_DOMINIO/
```

### Validar acesso público à v2

Substitua pelo path público real da v2.

```bash
curl -sS https://SEU_DOMINIO/SEU_PATH_PUBLICO_DA_V2/health.json
curl -sS https://SEU_DOMINIO/SEU_PATH_PUBLICO_DA_V2/ranking.json
curl -sS https://SEU_DOMINIO/SEU_PATH_PUBLICO_DA_V2/matches.json
```

### Validar fonte SQLite

Substitua pelo path real validado do banco.

```bash
ls -l /CAMINHO/DO/matchzy.db
sqlite3 /CAMINHO/DO/matchzy.db ".tables"
```

### Validar state do recurso detalhado

Exemplo representativo:

```bash
jq . /var/www/api/cs2/v2/player/SEU_STEAMID64.json
```

---

## Itens pendentes de confirmação

Os itens abaixo ainda devem ser confirmados diretamente no ambiente real para elevar o grau de confiança do contexto.

### 1. Path final público da v2 sob o domínio

O path em filesystem está reconciliado.  
O path HTTP público final da v2 ainda deve ser fixado sem ambiguidade no ambiente real.

### 2. Comando agregador real de regeneração

A lógica da pipeline está clara, mas o comando operacional único ou fluxo real exato de execução agregada ainda precisa ser fixado diretamente no host.

### 3. Path exato do `matchzy.db`

A dependência estrutural da instância `MixHAXIXE01` já está reconciliada, mas o path final completo do banco deve ser confirmado de forma explícita no ambiente real.

### 4. Diretório e convenção final dos scripts `gen-*`

A existência da camada de scripts está reconciliada, mas o inventário final do local exato de cada script precisa ser confirmado contra o host.

### 5. Path público vigente do conteúdo same-origin de News

A existência do fluxo está reconciliada, mas o path final canônico deve ser fixado no ambiente real.

### 6. Eventuais logs adicionais além de `journalctl`

É útil confirmar se há logs locais específicos do ETL ou da publicação que devam ser citados formalmente no contexto.

---

## Itens fora do escopo deste contexto

Os itens abaixo não pertencem ao inventário canônico do contexto Portal Estático:

- detalhes da infraestrutura base da VPS Hostinger
- configuração completa de Certbot e TLS da Hostinger
- operação do AMP e do servidor CS2
- MatchZy como ferramenta operacional de jogo
- runbooks de scrim, veto, coach ou warmup
- Auth API no Lightsail
- credenciais e arquivos sensíveis
- documentação histórica não reconciliada

Esses itens pertencem a outros contextos ou devem permanecer fora do fluxo normal do repositório documental.

---

## Limites documentais do contexto

Os limites documentais deste contexto são:

- ele documenta a camada pública do portal e sua cadeia de dados
- ele não documenta sozinho a infraestrutura base do host
- ele não documenta sozinho a origem operacional do jogo
- ele não substitui o índice mestre
- ele não substitui impl-logs históricos
- ele depende de confirmação periódica contra o ambiente real para permanecer confiável

---

## Critério de manutenção

Este documento deve ser atualizado quando houver:

- mudança de path crítico
- mudança de path público da v2
- mudança de localização do `matchzy.db`
- mudança relevante no inventário de scripts
- alteração dos recursos públicos centrais
- confirmação ou resolução de item pendente listado aqui

Mudanças pequenas de comportamento funcional devem ser refletidas primeiro no documento especializado correspondente, e não necessariamente aqui.

---

## Critério de pronto deste documento

Este documento pode ser considerado maduro quando:

- os artefatos reais do contexto estiverem confirmados sem ambiguidade
- os paths críticos estiverem validados no ambiente real
- o path público final da v2 estiver fixado
- o path final do `matchzy.db` estiver fixado
- os itens pendentes estiverem resolvidos ou explicitamente mantidos como pendência consciente
- ele puder ser usado como inventário confiável do contexto sem depender do master legado

---

## Última revisão

- Status: ativo
- Classificação: canônico
- Contexto: portal estático / references and inventory
- Última revisão: 2026-03-18