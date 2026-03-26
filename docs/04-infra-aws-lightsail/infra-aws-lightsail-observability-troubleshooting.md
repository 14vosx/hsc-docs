# Observability and Troubleshooting

## Navegação rápida

- [Home da documentação](../README.md)
- [Infra AWS Lightsail](./README.md)
- [Master Index](../00-governance/99-master-index.md)

---
## Objetivo

Documentar os sinais de saúde, os pontos de observabilidade e a sequência padrão de diagnóstico do contexto AWS Lightsail da Auth API.

Este documento existe para registrar, de forma estável e auditável:

- como validar a saúde do contexto
- quais sinais e artefatos devem ser observados primeiro
- como separar falhas de edge, aplicação e banco
- quais comandos operacionais são usados na triagem
- quais incidentes são mais prováveis neste runtime
- como reduzir tempo de diagnóstico em produção

---

## Escopo

Este documento cobre:

- sinais de saúde do contexto
- logs principais
- smoke checks operacionais
- triagem de incidentes
- falhas comuns do edge
- falhas comuns da aplicação
- falhas comuns de banco
- falhas comuns de deploy
- sequência recomendada de diagnóstico

Este documento não cobre em profundidade:

- configuração completa do Nginx
- unit file detalhado do serviço
- fluxo detalhado de deploy e rollback
- modelagem de domínio da Auth API
- dumps completos de banco
- processo formal de backup e restore

Esses tópicos vivem em documentos próprios do contexto.

---

## Estado atual

O contexto `04-infra-aws-lightsail` possui, no estado operacional conhecido, os seguintes componentes críticos de observabilidade:

- Nginx como edge público
- serviço `hsc-auth-api` via systemd
- aplicação Node.js com health local em `127.0.0.1:3000`
- MariaDB local bound em `127.0.0.1`
- smoke tests de rotas públicas relevantes
- logs do serviço via `journalctl`
- validação operacional por health local e health público

A observabilidade atual é orientada principalmente por:

- health checks
- `systemctl`
- `journalctl`
- `curl`
- validação funcional mínima das superfícies mais críticas

---

## Source of truth / evidências

As principais evidências deste documento, nesta fase de migração documental, são:

- manual operacional da Auth API em AWS Lightsail
- documentação consolidada do ecossistema HSC
- blueprint técnico consolidado
- fluxos de deploy e release por TAG
- impl-logs ligados a CORS, sessões administrativas, schema snapshot e fail-closed
- padrões de validação reconciliados durante a reorganização documental

Enquanto a migração canônica não estiver concluída, essas fontes seguem sendo usadas como base de reconciliação do estado operacional real.

---

## Relações com outros documentos

Este arquivo é complementar a:

- `docs/04-infra-aws-lightsail/README.md`
- `docs/04-infra-aws-lightsail/infra-aws-lightsail-architecture-runtime.md`
- `docs/04-infra-aws-lightsail/infra-aws-lightsail-network-dns-tls.md`
- `docs/04-infra-aws-lightsail/nginx-reverse-proxy.md`
- `docs/04-infra-aws-lightsail/node-systemd.md`
- `docs/04-infra-aws-lightsail/mariadb-local.md`
- `docs/04-infra-aws-lightsail/deploy-release-rollback.md`
- `docs/04-infra-aws-lightsail/auth-api-operations.md`

Este documento descreve triagem, observabilidade e troubleshooting.  
Ele não substitui os documentos de runtime, proxy, banco ou deploy.

---

## Sinais de saúde do contexto

Os sinais de saúde esperados do contexto incluem:

- hostname público resolvendo corretamente
- TLS funcional na borda pública
- Nginx ativo
- serviço `hsc-auth-api` ativo
- health local respondendo
- health público respondendo
- MariaDB acessível pela aplicação
- logs sem erro crítico recorrente
- rotas públicas principais respondendo corretamente
- superfícies administrativas funcionando de acordo com o estado esperado da release

A saúde do contexto deve ser avaliada em camadas, e não com um único indicador isolado.

---

## Camadas de observabilidade

### Camada 1 — Borda pública

Objetivo:
- validar se o serviço é alcançável externamente

Sinais típicos:
- DNS correto
- TLS válido
- resposta do health público
- headers básicos coerentes

---

### Camada 2 — Edge local

Objetivo:
- validar se o Nginx está saudável

Sinais típicos:
- `nginx -t` sem erro
- serviço ativo
- ausência de erro crítico recorrente no Nginx
- upstream configurado de forma coerente

---

### Camada 3 — Aplicação

Objetivo:
- validar se a Auth API está realmente viva

Sinais típicos:
- `systemctl status hsc-auth-api` saudável
- `journalctl` sem crash loop
- health local respondendo em `127.0.0.1:3000`

---

### Camada 4 — Persistência

Objetivo:
- validar se o banco está acessível e compatível com a release

Sinais típicos:
- serviço MariaDB ativo
- ausência de erro de conexão no log da aplicação
- rotas dependentes de persistência funcionando

---

### Camada 5 — Saúde funcional

Objetivo:
- validar se o contexto está operacionalmente útil, e não apenas “de pé”

Sinais típicos:
- `/content/news` responde
- `/content/seasons` responde
- `/content/seasons/active` responde
- fluxos administrativos críticos funcionam quando o escopo da mudança exigir

---

## Logs principais

Os logs principais do contexto são:

### Logs do serviço da aplicação

```bash
sudo journalctl -u hsc-auth-api -f
```

Uso principal:
- crash de bootstrap
- erro de banco
- erro de configuração
- regressão pós-deploy
- erro de sessão, autenticação ou mutação administrativa

---

### Logs do Nginx

```bash
sudo journalctl -u nginx -n 100 --no-pager
```

Uso principal:
- falha de start/reload
- erro de sintaxe
- edge disponível, mas upstream com problema
- troubleshooting da borda

Observação:
- quando necessário, consultar também access log e error log efetivamente configurados no host

---

### Status de serviços

```bash
sudo systemctl status nginx
sudo systemctl status hsc-auth-api
sudo systemctl status mariadb
```

Uso principal:
- triagem rápida
- confirmação de processo ativo
- confirmação de erro evidente de serviço

---

## Smoke checks

Os smoke checks mínimos deste contexto devem ser executados em ordem lógica.

### Health local da aplicação

```bash
curl -sS http://127.0.0.1:3000/health
```

Objetivo:
- validar a aplicação sem depender da borda pública

---

### Health público

Substitua pelo hostname vigente do contexto.

```bash
curl -sS https://SEU_DOMINIO/health
```

Objetivo:
- validar borda + proxy + aplicação

---

### Health público com headers

```bash
curl -I https://SEU_DOMINIO/health
```

Objetivo:
- inspecionar resposta HTTP da borda

---

### Conteúdo público principal

```bash
curl -sS https://SEU_DOMINIO/content/news
curl -sS https://SEU_DOMINIO/content/seasons
curl -sS https://SEU_DOMINIO/content/seasons/active
```

Objetivo:
- validar que o backend está funcionalmente útil

---

### Resolução DNS

```bash
nslookup SEU_DOMINIO
```

Objetivo:
- validar resolução pública do hostname

---

## Sequência de diagnóstico

Quando houver incidente neste contexto, usar a seguinte ordem.

### Etapa 1 — Confirmar borda pública

1. validar DNS
2. validar TLS/health público
3. confirmar que o hostname responde no edge esperado

Comandos típicos:

```bash
nslookup SEU_DOMINIO
curl -I https://SEU_DOMINIO/health
curl -sS https://SEU_DOMINIO/health
```

---

### Etapa 2 — Confirmar Nginx

1. validar sintaxe do Nginx
2. validar status do serviço
3. verificar log do Nginx

Comandos típicos:

```bash
sudo nginx -t
sudo systemctl status nginx
sudo journalctl -u nginx -n 100 --no-pager
```

---

### Etapa 3 — Confirmar aplicação

1. validar status do serviço `hsc-auth-api`
2. validar logs do serviço
3. validar health local

Comandos típicos:

```bash
sudo systemctl status hsc-auth-api
sudo journalctl -u hsc-auth-api -n 100 --no-pager
curl -sS http://127.0.0.1:3000/health
```

---

### Etapa 4 — Confirmar banco

1. validar status do MariaDB
2. procurar erro de conexão ou schema no log da aplicação
3. confirmar comportamento das rotas dependentes de persistência

Comandos típicos:

```bash
sudo systemctl status mariadb
sudo journalctl -u hsc-auth-api -n 100 --no-pager
```

Quando necessário, validar acesso local administrativo ao banco de forma controlada.

---

### Etapa 5 — Confirmar saúde funcional

1. testar `/content/news`
2. testar `/content/seasons`
3. testar `/content/seasons/active`
4. validar fluxo administrativo afetado, se a mudança envolver admin

Comandos típicos:

```bash
curl -sS https://SEU_DOMINIO/content/news
curl -sS https://SEU_DOMINIO/content/seasons
curl -sS https://SEU_DOMINIO/content/seasons/active
```

---

## Falhas comuns

### App não sobe

Sintomas:

- `systemctl status hsc-auth-api` mostra falha
- `journalctl` mostra erro logo no bootstrap
- health local não responde

Causas comuns:

- `.env` inválido ou ausente
- `node` indisponível
- `index.js` ausente ou incorreto
- dependência quebrada
- erro de conexão com banco no start
- drift após deploy

Triagem:

```bash
sudo systemctl status hsc-auth-api
sudo journalctl -u hsc-auth-api -n 100 --no-pager
```

---

### Health público falha, mas local funciona

Sintomas:

- `curl http://127.0.0.1:3000/health` responde
- `curl https://SEU_DOMINIO/health` falha

Causas comuns:

- Nginx parado
- erro de proxy
- problema de TLS
- problema de DNS
- firewall/borda inconsistente

Triagem:

```bash
sudo nginx -t
sudo systemctl status nginx
sudo journalctl -u nginx -n 100 --no-pager
nslookup SEU_DOMINIO
```

---

### CORS quebra no frontend legítimo

Sintomas:

- health público funciona
- chamada em browser falha por origem
- backend parece funcional por `curl`

Causas comuns:

- allowlist de origem incompleta
- domínio/subdomínio mudou sem atualização da aplicação
- regressão de leitura do `.env`

Triagem:
- confirmar domínio efetivamente usado pelo frontend
- reconciliar com allowlist ativa da aplicação
- verificar impl-log relacionado a CORS
- validar logs e comportamento do backend

Observação:
- este é, em regra, um problema da aplicação, não do Nginx em si

---

### Deploy não aplicou a TAG esperada

Sintomas:

- serviço sobe
- endpoints não refletem a mudança esperada
- comportamento não bate com a release publicada

Causas comuns:

- checkout incorreto
- drift no working directory
- restart não efetivo
- operação concorrente
- `npm ci` não refletiu a release desejada

Triagem:

```bash
cd /opt/hsc/hsc-auth-api
git describe --tags --exact-match 2>/dev/null || true
sudo systemctl status hsc-auth-api
sudo journalctl -u hsc-auth-api -n 100 --no-pager
```

Também verificar:

- conteúdo de `/opt/hsc/.deploy-auth-last-tag`
- consistência entre TAG, código em disco e comportamento observado

---

### MariaDB indisponível

Sintomas:

- app sobe parcialmente ou falha
- rotas dependentes de banco quebram
- logs mostram erro de conexão
- health pode ou não indicar degradação, dependendo da implementação ativa

Causas comuns:

- serviço MariaDB parado
- credenciais incorretas
- bind/configuração local incorreta
- schema esperado ausente ou incompatível

Triagem:

```bash
sudo systemctl status mariadb
sudo journalctl -u hsc-auth-api -n 100 --no-pager
```

---

### Rota admin negada indevidamente

Sintomas:

- autenticação aparentemente válida
- rota administrativa retorna negação inesperada
- mutação não é executada

Causas comuns:

- sessão não reconhecida
- identidade administrativa inconsistente
- auditoria indisponível
- regra fail-closed acionada corretamente
- regressão de autorização

Triagem:
- validar contexto de sessão
- validar logs da aplicação
- conferir se a mutação depende de trilha de auditoria
- verificar se a release alterou guardas ou política de write

---

### Health responde, mas funcionalidade principal falha

Sintomas:

- health local e/ou público responde
- `/content/news` ou `/content/seasons` falham
- superfícies administrativas quebram
- backend parece “de pé”, mas degradado

Causas comuns:

- banco degradado
- schema incompatível
- regressão funcional
- falha parcial da aplicação
- release incompleta

Triagem:
- validar endpoints reais de negócio
- ler logs da aplicação
- validar banco
- reconciliar release ativa com a mudança implantada

---

## Interpretação de sintomas

Alguns padrões úteis de interpretação:

### Público falha + local funciona
Provável problema de borda:
- DNS
- TLS
- Nginx
- firewall

### Público falha + local falha
Provável problema de aplicação ou banco:
- serviço `hsc-auth-api`
- bootstrap
- `.env`
- MariaDB

### Health funciona + conteúdo falha
Provável problema funcional:
- banco
- schema
- regressão de rota
- dependência parcial

### Sessão falha + break-glass funciona
Provável problema no fluxo normal de autenticação/sessão:
- persistência
- sessão
- guarda
- regressão de auth

### Admin write falha + leitura funciona
Provável problema em:
- autorização
- auditoria
- regra fail-closed
- write path da release ativa

---

## Lacunas atuais de observabilidade

As lacunas conhecidas ou prováveis desta camada incluem:

- observabilidade ainda muito baseada em `curl`, `systemctl` e `journalctl`
- ausência, até onde reconciliado, de stack formal de métricas e alertas
- health possivelmente insuficiente para detectar toda degradação funcional
- dependência alta de inspeção manual em incidentes
- necessidade de disciplina humana para interpretar borda, app e banco em conjunto

Essas lacunas não invalidam o runtime, mas explicam por que a sequência de diagnóstico precisa ser explícita.

---

## Regras de ouro de troubleshooting

As regras de ouro deste contexto são:

1. nunca assumir que “serviço ativo” significa “contexto saudável”
2. sempre comparar health público com health local
3. sempre distinguir problema de edge de problema de upstream
4. sempre ler `journalctl` antes de concluir causa raiz
5. nunca tratar CORS como se fosse automaticamente problema de Nginx
6. sempre validar banco quando a falha for funcional e não apenas de processo
7. sempre reconciliar a TAG ativa quando a falha surgir após deploy
8. em mutações administrativas, considerar auditoria e fail-closed como parte do diagnóstico

---

## Limites deste documento

Este documento não detalha:

- configuração textual completa do Nginx
- conteúdo do `.env`
- comandos completos de backup/restore
- dump de banco
- fluxo integral de deploy com todos os cenários
- runbooks de domínio específicos de News, Seasons ou Events

Esses detalhes vivem em documentos complementares.

---

## Critério de pronto deste documento

Este documento pode ser considerado maduro quando:

- os sinais de saúde do contexto estiverem alinhados com a prática real
- a sequência de diagnóstico refletir o uso operacional real do host
- as falhas comuns cobrirem os incidentes mais prováveis do contexto
- os comandos aqui descritos forem suficientes para triagem inicial sem depender do master legado
- a distinção entre edge, app, banco e funcionalidade estiver clara
- ele puder ser usado como runbook de triagem real do contexto Lightsail

---

## Última revisão

- Status: ativo
- Classificação: canônico
- Contexto: infraestrutura AWS Lightsail / observability and troubleshooting
- Última revisão: 2026-03-18