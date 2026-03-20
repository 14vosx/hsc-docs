# Nginx Publishing and Cache

## Objetivo

Documentar como a camada Nginx do lado Hostinger publica o Portal Estático e a Static API v2 do ecossistema HSC, registrando os paths HTTP canônicos, os aliases/roots reconciliados, as regras principais de cache e os comportamentos de compatibilidade hoje ativos no runtime real.

Este documento existe para registrar, de forma estável e auditável:

- quais paths públicos do portal e da v2 estão realmente ativos
- como o Nginx publica os artefatos do portal
- como o Nginx publica os JSONs da Static API v2
- como o mirror de conteúdo same-origin é servido
- quais regras de cache já estão reconciliadas no ambiente real
- quais redirects e compatibilidades históricas ainda existem
- quais riscos operacionais surgem quando publishing, cache ou alias divergem do esperado

---

## Navegação

### Entrada
- [Home da documentação](../README.md)
- [Portal Estático](./README.md)
- [Master Index](../00-governance/99-master-index.md)

### Origem e artefatos publicados
- [ETL Bash Pipeline](./etl-bash-pipeline.md)
- [Static API v2](./static-api-v2.md)
- [JSON Contracts](./json-contracts.md)
- [Frontend Structure](./frontend-structure.md)

### Infraestrutura de entrega
- [Nginx Static Serving](../01-infra-hostinger/nginx-static-serving.md)
- [Systemd Automation](../01-infra-hostinger/systemd-automation.md)
- [Network, DNS and TLS](../01-infra-hostinger/network-dns-tls.md)

### Operação e suporte
- [Operational Runbooks](./operational-runbooks.md)
- [Observability and Troubleshooting](./observability-troubleshooting.md)
- [Documentation System](../00-governance/documentation-system.md)

---

## Escopo

Este documento cobre:

- publishing do portal sob `/portal/cs2/`
- publishing da Static API v2 sob `/api/cs2/v2/`
- publishing dos assets públicos do portal
- mirror same-origin de `/content/news/`
- regras principais de cache e no-store
- redirects e compatibilidades ainda ativos
- validações mínimas da camada de publishing
- riscos e problemas comuns dessa borda

Este documento não cobre em profundidade:

- configuração completa do Nginx linha a linha
- infraestrutura geral da Hostinger
- implementação do ETL Bash
- contratos JSON campo a campo
- frontend em detalhe estrutural
- Auth API no Lightsail

Esses tópicos vivem em documentos próprios dos contextos `01-infra-hostinger`, `03-portal-estatico` e `04-infra-aws-lightsail`.

---

## Estado atual

O estado operacional conhecido e reconciliado desta camada é:

- o portal público é servido sob:
  - `/portal/cs2/`
- os assets estáveis do portal são servidos sob:
  - `/portal/cs2/assets/`
- a Static API v2 é servida publicamente sob:
  - `/api/cs2/v2/`
- o mirror same-origin de News é servido sob:
  - `/content/news/`
- o Nginx usa `alias` para grande parte da árvore do portal
- o Nginx usa `alias` para a árvore principal da v2
- o Nginx aplica política de cache curta para assets estáveis do portal
- o Nginx aplica política de `no-store` para a v2 e para o mirror de conteúdo
- endpoints antigos ainda possuem redirects de compatibilidade para a v2
- `/api/cs2/v1/` está explicitamente desativado e responde `404`

Também ficou reconciliado no runtime real que:

- os hostnames públicos canônicos dessa borda são:
  - `haxixesmokeclub.com`
  - `www.haxixesmokeclub.com`

Leitura canônica:

- o portal e a v2 vivem sob o mesmo domínio principal da Hostinger
- a v2 não usa mais caminhos “genéricos” como borda principal
- a leitura correta atual é `/portal/cs2/` + `/api/cs2/v2/`

---

## Source of truth / evidências

As principais evidências deste documento, nesta fase de reconciliação, são:

- documentação consolidada atual do ecossistema HSC
- blueprint técnico consolidado do HSC
- documentação reconciliada da Infra Hostinger
- documentação reconciliada do Portal Estático
- runtime real da Hostinger
- saída direta de `nginx -T`
- validação pública dos hostnames canônicos
- leitura direta dos blocos `location`, `root` e `alias` relevantes no host

Enquanto o runtime real permanecer neste formato, essas evidências prevalecem como source of truth operacional.

---

## Relações com outros documentos

Este arquivo é complementar a:

- `docs/03-portal-estatico/README.md`
- `docs/03-portal-estatico/portal-estatico-architecture-runtime.md`
- `docs/03-portal-estatico/static-api-v2.md`
- `docs/03-portal-estatico/frontend-structure.md`
- `docs/03-portal-estatico/operational-runbooks.md`
- `docs/03-portal-estatico/observability-troubleshooting.md`
- `docs/03-portal-estatico/portal-estatico-references-inventory.md`
- `docs/01-infra-hostinger/nginx-static-serving.md`
- `docs/01-infra-hostinger/network-dns-tls.md`

Este documento descreve a publicação pública e a estratégia de cache da camada portal/v2.  
Ele não substitui os documentos de topologia geral, ETL, frontend ou troubleshooting aprofundado.

---

## Hostnames públicos da borda

Os hostnames públicos canônicos reconciliados desta camada são:

- `haxixesmokeclub.com`
- `www.haxixesmokeclub.com`

Esses hostnames devem ser tratados como:

- entrada pública oficial do portal
- entrada pública oficial da Static API v2
- entrada pública oficial do mirror same-origin de conteúdo

Hostnames não ativos para esta borda no estado atual:

- `portal.haxixesmokeclub.com`
- `api.haxixesmokeclub.com`

Regra importante:

- a documentação do portal deve usar os hostnames realmente ativos
- subdomínios históricos não resolvidos não devem ser promovidos como canônicos

---

## Publicação do portal

O portal público do HSC é servido sob:

```text
/portal/cs2/
```

A árvore de filesystem reconciliada para essa publicação é:

```text
/var/www/portal/cs2/
```

No runtime real, o Nginx usa:

- `alias /var/www/portal/cs2/;`
- fallback SPA com `try_files $uri $uri/ /portal/cs2/index.html;`

Leitura canônica:

- `/portal/cs2/` é a porta principal do frontend estático
- a navegação do portal depende do fallback SPA configurado no Nginx
- o portal não deve ser tratado como aplicação publicada na raiz `/`

---

## Publicação dos assets do portal

Os assets estáveis do portal são servidos sob:

```text
/portal/cs2/assets/
```

A árvore de filesystem reconciliada para esses assets é:

```text
/var/www/portal/cs2/assets/
```

No runtime real, o Nginx usa:

- `alias /var/www/portal/cs2/assets/;`
- `expires 1h;`
- `Cache-Control: public, max-age=3600`
- `try_files $uri =404;`

Leitura canônica:

- os assets estáveis do portal têm cache curto reconciliado de 1 hora
- esses assets não devem depender de caminhos `current/` ou `releases/` na borda pública atual

---

## Assets legados desativados

No runtime real, os caminhos abaixo estão explicitamente desativados:

- `/portal/cs2/assets/releases/`
- `/portal/cs2/assets/current/`

Ambos retornam:

- `410 Gone`

Leitura canônica:

- esses caminhos pertencem à camada legada/desativada
- não devem ser reutilizados como convenção viva da publicação
- sua presença existe para falha explícita e visível, não para uso funcional

Esse comportamento é útil porque:

- evita fallback silencioso para estrutura antiga
- acelera diagnóstico de cliente ou script stale
- impede que publishing legado continue “funcionando por acidente”

---

## Publicação da Static API v2

A Static API v2 é servida publicamente sob:

```text
/api/cs2/v2/
```

A árvore de filesystem reconciliada para essa publicação é:

```text
/var/www/api/cs2/v2/
```

No runtime real, o Nginx usa:

- `alias /var/www/api/cs2/v2/;`
- `default_type application/json;`
- `Cache-Control: no-store, no-cache, max-age=0, must-revalidate`
- `Pragma: no-cache`
- `Expires: 0`
- `X-HSC-API: v2`
- `try_files $uri =404;`

Leitura canônica:

- `/api/cs2/v2/` é a borda pública principal dos JSONs do ecossistema
- a v2 é tratada como artefato de dados dinâmico-estático com política de não-cache forte
- a leitura pública correta da camada de dados deve partir desse prefixo

---

## Normalização de JSON com trailing slash

No runtime real, existe normalização para evitar acesso incorreto a JSON com barra final:

- requisições do tipo:
  - `/api/cs2/v2/<arquivo>.json/`
- são redirecionadas para:
  - `/api/cs2/v2/<arquivo>.json`

Leitura canônica:

- o path público correto de um JSON da v2 não termina com `/`
- essa regra existe para evitar respostas HTML indevidas e erros do tipo:
  - `Unexpected token <`

Esse comportamento é parte útil do endurecimento da borda.

---

## Static API v1 desativada

No runtime real, a rota:

```text
/api/cs2/v1/
```

retorna:

- `404`

Leitura canônica:

- não existe mais v1 ativa
- a camada pública oficial do ecossistema é somente a Static API v2
- qualquer cliente ainda apontando para v1 deve ser tratado como stale

---

## Endpoints antigos com redirect de compatibilidade

O runtime atual mantém alguns redirects de compatibilidade para clientes ou referências antigas.

### Redirects reconciliados

- `/api/ranking.json` → `/api/cs2/v2/ranking.json`
- `/api/matches.json` → `/api/cs2/v2/matches.json`
- `/api/health.json` → `/api/cs2/v2/health.json`
- `/api/steam/<id>.json` → `/api/cs2/v2/player/<id>.json`

Leitura canônica:

- esses redirects existem como camada de compatibilidade
- eles não substituem o prefixo oficial da v2
- a documentação nova deve sempre preferir `/api/cs2/v2/`

---

## Mirror same-origin de content/news

O mirror same-origin de conteúdo é servido sob:

```text
/content/news/
```

No runtime real:

- `/content/news` redireciona para `/content/news/`
- `/content/news/` serve:
  - `/var/www/api/cs2/v2/content/news/index.json`
- `/content/news/<slug>` redireciona para `/content/news/<slug>/`
- `/content/news/<slug>/` serve:
  - `/var/www/api/cs2/v2/content/news/<slug>.json`

A árvore base reconciliada é:

```text
/var/www/api/cs2/v2/
```

com uso de:

- `root /var/www/api/cs2/v2;`
- `try_files /content/news/index.json =503;`
- `try_files /content/news/$1.json =404;`

Política de cache reconciliada:

- `Cache-Control: no-store, max-age=0`

Leitura canônica:

- o mirror de News é publicado same-origin no domínio do portal
- ele usa a mesma base pública da v2
- ele não deve ser tratado como borda dinâmica da Auth API
- do ponto de vista do browser, ele se comporta como publicação estática de conteúdo JSON

---

## Portal player SPA

O runtime real também tem handling específico para a área de player do portal.

Paths reconciliados:

- `/portal/cs2/player/index.html`
- `/portal/cs2/player/`

A publicação usa:

- `alias /var/www/portal/cs2/player/index.html;`
- `alias /var/www/portal/cs2/player/;`
- fallback SPA com:
  - `try_files $uri $uri/ /portal/cs2/player/index.html;`

Leitura canônica:

- a área de player do portal possui rota específica reforçada na borda
- isso reduz risco de fallback incorreto em navegação profunda dessa área

---

## Compatibilidades antigas do portal

O runtime atual mantém alguns redirects de compatibilidade do lado do portal:

- `/portal/` → `/portal/cs2/`
- `/portal/ranking/` → `/portal/cs2/ranking/`
- `/portal/matches/` → `/portal/cs2/matches/`

Leitura canônica:

- esses redirects ajudam a transição histórica
- o path oficial do portal continua sendo `/portal/cs2/`
- a documentação nova não deve tratar `/portal/` como borda principal

---

## Service Worker neutralizado

No runtime real, existe uma regra explícita para:

```text
/ServiceWorker.js
```

servindo:

- `/var/www/portal/sw-kill-ServiceWorker.js`

com:

- `Cache-Control: no-store, max-age=0`

Leitura canônica:

- essa regra existe para neutralizar interferência de Service Worker no domínio
- o objetivo é evitar cache indevido afetando:
  - `/portal`
  - `/api`
  - e demais rotas same-origin

Esse detalhe é importante porque ajuda a explicar troubleshooting relacionado a estado stale em cliente.

---

## Estratégia de cache reconciliada

A estratégia hoje reconciliada pode ser resumida assim:

### Portal assets estáveis
- cache curto
- `expires 1h`
- `Cache-Control: public, max-age=3600`

### Static API v2
- sem cache útil de borda/browser
- `no-store, no-cache, max-age=0, must-revalidate`
- `Pragma: no-cache`
- `Expires: 0`

### Content mirror `/content/news/`
- sem cache útil
- `Cache-Control: no-store, max-age=0`

### Service Worker neutralizado
- sem cache útil
- `Cache-Control: no-store, max-age=0`

Leitura canônica:

- assets do frontend podem ser reutilizados por curto prazo
- dados públicos e mirror de conteúdo devem refletir o estado mais recente
- a política atual privilegia previsibilidade e troubleshooting em vez de agressividade de cache sobre JSON

---

## Relação entre publishing e ETL

A relação estrutural é:

- o ETL produz arquivos finais em `/var/www/api/cs2/v2/`
- o Nginx publica esses arquivos sob `/api/cs2/v2/`
- o mirror `/content/news/` também lê da mesma base pública
- o portal consome essa camada same-origin

Isso significa:

- problema de ETL pode parecer problema de Nginx
- problema de cache pode parecer problema de ETL
- problema de browser pode parecer problema da v2
- a leitura correta precisa sempre considerar geração + publicação + consumo

---

## Comandos de validação

Os comandos abaixo representam a validação mínima desta camada.

### Validar hostnames públicos principais

```bash
curl -I https://haxixesmokeclub.com/
curl -I https://www.haxixesmokeclub.com/
```

### Validar prefixo público da v2

```bash
curl -I https://haxixesmokeclub.com/api/cs2/v2/health.json
curl -sS https://haxixesmokeclub.com/api/cs2/v2/health.json
```

### Validar ranking público da v2

```bash
curl -I https://haxixesmokeclub.com/api/cs2/v2/ranking.json
curl -sS https://haxixesmokeclub.com/api/cs2/v2/ranking.json
```

### Validar mirror same-origin de News

```bash
curl -I https://haxixesmokeclub.com/content/news/
curl -sS https://haxixesmokeclub.com/content/news/
```

### Validar redirect de compatibilidade da API

```bash
curl -I https://haxixesmokeclub.com/api/health.json
```

### Validar desativação explícita da v1

```bash
curl -I https://haxixesmokeclub.com/api/cs2/v1/
```

### Validar sintaxe do Nginx no host

```bash
sudo nginx -t
```

### Validar configuração relevante do Nginx

```bash
sudo nginx -T | grep -nE "server_name|root |alias "
```

### Validar bloco relevante do vhost

```bash
sudo nginx -T | sed -n '200,320p'
```

Regra prática:

- smoke test principal deve mirar `/portal/cs2/` e `/api/cs2/v2/`
- caminhos antigos só devem ser usados para validar compatibilidade ou detectar clientes stale

---

## Problemas comuns

### 1. Cliente ainda usa caminho antigo da API

Causas comuns:

- documentação antiga
- bookmark antigo
- script legado
- frontend stale

Impacto:
- cliente pode depender de redirect
- ou falhar se estiver mirando v1

---

### 2. JSON parece stale mesmo com ETL saudável

Causas comuns:

- cache do navegador
- estado antigo em cliente
- Service Worker residual
- troubleshooting sem invalidação de estado local

Impacto:
- falsa suspeita de falha da v2

---

### 3. Script ou operador usa assets legacy

Causas comuns:

- referência antiga a `assets/current/`
- referência antiga a `assets/releases/`

Impacto:
- resposta `410 Gone`
- quebra explícita de consumidor stale

---

### 4. Mirror de News falha, mas v2 geral parece saudável

Causas comuns:

- ausência do arquivo `content/news/index.json`
- problema específico do mirror
- detalhe de geração do conteúdo, não da v2 inteira

Impacto:
- portal pode degradar parcialmente
- troubleshooting precisa isolar `content/news/` do restante da API estática

---

### 5. Publishing local correto, mas borda pública incorreta

Causas comuns:

- alias/root divergente
- vhost errado
- caminho público interpretado de forma antiga
- teste feito em hostname não canônico

Impacto:
- o arquivo existe
- o consumidor continua falhando

---

## Invariantes operacionais

Os invariantes conhecidos desta camada incluem:

- o portal público vive em `/portal/cs2/`
- a Static API oficial vive em `/api/cs2/v2/`
- `/api/cs2/v1/` não está ativa
- a borda pública usa `haxixesmokeclub.com` e `www.haxixesmokeclub.com`
- assets estáveis do portal usam cache curto
- JSONs da v2 usam política forte de não-cache
- `/content/news/` é same-origin e publicado a partir da base pública da v2
- caminhos antigos de assets `current/` e `releases/` estão explicitamente desativados
- caminhos antigos do portal e da API sobrevivem apenas como camada de compatibilidade

Esses invariantes ajudam a preservar a leitura correta do publishing da camada pública.

---

## Limites deste documento

Este documento não detalha:

- todos os blocos do vhost linha a linha
- toda a estratégia de cache do frontend em nível de browser
- contratos JSON detalhados
- troubleshooting aprofundado do ETL
- troubleshooting aprofundado do Service Worker
- limpeza futura de compatibilidades históricas

Esses tópicos pertencem a documentos complementares ou a futuras revisões.

---

## Critério de pronto deste documento

Este documento pode ser considerado maduro quando:

- os paths públicos canônicos do portal e da v2 estiverem explícitos sem ambiguidade
- a política de cache reconciliada estiver clara
- o mirror `/content/news/` estiver corretamente posicionado
- assets legados desativados estiverem claramente documentados
- redirects de compatibilidade estiverem formalizados
- ele puder ser usado como referência confiável da borda pública do Portal Estático sem depender do master legado

---

## Última revisão

- Status: ativo
- Classificação: canônico
- Contexto: portal estático / Nginx publishing and cache
- Última revisão: 2026-03-18