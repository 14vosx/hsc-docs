# Observability and Troubleshooting

## Objetivo

Documentar os sinais de saúde, os pontos de observabilidade e a sequência padrão de diagnóstico da camada-base da Infra Hostinger no ecossistema HSC.

Este documento existe para registrar, de forma estável e auditável:

- como validar a saúde da VPS Hostinger como substrate do lado game + portal
- como separar falhas de borda pública, filesystem, automação, Docker e workloads superiores
- quais comandos operacionais são usados na triagem inicial
- quais incidentes são mais prováveis nessa camada-base
- como reduzir o tempo de diagnóstico quando o problema ainda não está claramente localizado

---

## Escopo

Este documento cobre:

- sinais de saúde da Infra Hostinger
- observabilidade da borda pública via Nginx
- observabilidade do filesystem público e operacional
- observabilidade da automação do host
- observabilidade da camada Docker host
- sequência recomendada de diagnóstico
- falhas comuns do substrate do lado Hostinger
- interpretação de sintomas recorrentes

Este documento não cobre em profundidade:

- troubleshooting detalhado do AMP Instance Manager
- troubleshooting detalhado do servidor CS2
- troubleshooting detalhado do MatchZy
- troubleshooting detalhado do ETL script por script
- troubleshooting detalhado do frontend do portal
- Auth API no AWS Lightsail

Esses tópicos vivem em documentos próprios dos contextos `02-game-panel`, `03-portal-estatico` e `04-infra-aws-lightsail`.

---

## Estado atual

O contexto `01-infra-hostinger` possui, no estado operacional conhecido, as seguintes camadas críticas de observabilidade:

- host Debian
- Nginx como edge público do lado Hostinger
- Certbot como suporte da camada TLS
- Docker como substrate do lado game
- systemd e timers do host
- filesystem público e operacional
- publishing do portal e da Static API v2
- árvores operacionais sob `/opt/cs2-portal/` e `/home/amp/.ampdata/instances/`

A observabilidade dessa camada é orientada principalmente por:

- `systemctl`
- `journalctl`
- `nginx -t`
- inspeção de filesystem e permissões
- verificação de timers
- verificação do estado do Docker
- validação pública do portal e da v2

A Infra Hostinger precisa ser analisada como base compartilhada.  
Um sintoma público no portal pode nascer no Nginx, no filesystem, na automação, no Docker host ou até em uma dependência do lado game.

---

## Source of truth / evidências

As principais evidências deste documento, nesta fase de migração documental, são:

- documentação consolidada atual do ecossistema HSC
- blueprint técnico consolidado do HSC
- documentação reconciliada da Infra Hostinger
- documentação reconciliada do Portal Estático e do Game Panel
- histórico operacional já consolidado do lado Hostinger com Nginx, Docker, Certbot, systemd e filesystem público

Enquanto a migração canônica do contexto não estiver concluída, essas fontes seguem sendo usadas como base de reconciliação do estado real.

---

## Relações com outros documentos

Este arquivo é complementar a:

- `docs/01-infra-hostinger/README.md`
- `docs/01-infra-hostinger/infra-hostinger-architecture-runtime.md`
- `docs/01-infra-hostinger/network-dns-tls.md`
- `docs/01-infra-hostinger/nginx-static-serving.md`
- `docs/01-infra-hostinger/docker-host.md`
- `docs/01-infra-hostinger/certbot.md`
- `docs/01-infra-hostinger/systemd-automation.md`
- `docs/01-infra-hostinger/filesystem-paths-permissions.md`
- `docs/02-game-panel/observability-troubleshooting.md`
- `docs/03-portal-estatico/observability-troubleshooting.md`

Este documento descreve triagem, observabilidade e troubleshooting da camada-base Hostinger.  
Ele não substitui os documentos especializados dos workloads sustentados por essa infraestrutura.

---

## Sinais de saúde do contexto

Os sinais de saúde esperados da Infra Hostinger incluem:

- hostname público resolvendo corretamente para o lado Hostinger
- Nginx ativo e íntegro
- TLS funcional na borda pública
- portal público respondendo
- recursos públicos da Static API v2 respondendo
- diretórios públicos presentes e legíveis pelo edge
- diretórios operacionais íntegros
- timers e services do host coerentes com a automação esperada
- Docker ativo e sem degradação estrutural evidente
- ausência de `permission denied` nos fluxos normais do host
- ausência de exposição indevida de arquivos e diretórios internos

A saúde da infraestrutura não deve ser inferida por um único indicador.  
O host pode estar “de pé” e ainda assim estar degradando portal, ETL ou lado game.

---

## Camadas de observabilidade

### Camada 1 — Entrada pública

Objetivo:
- validar se o lado Hostinger está alcançável externamente

Sinais típicos:
- DNS correto
- HTTPS válido
- portal respondendo
- recursos públicos da v2 respondendo

---

### Camada 2 — Edge Nginx

Objetivo:
- validar se o Nginx está saudável e servindo os paths corretos

Sinais típicos:
- `nginx -t` sem erro
- serviço `nginx` ativo
- ausência de erro crítico recorrente nos logs
- serving correto do portal e da v2

---

### Camada 3 — Filesystem e permissões

Objetivo:
- validar se a árvore pública e a árvore operacional continuam utilizáveis pelos processos corretos

Sinais típicos:
- diretórios presentes
- `www-data` com leitura nas áreas públicas
- ETL com escrita nas áreas operacionais e públicas quando necessário
- ausência de bloqueio por ownership ou grupo incorretos

---

### Camada 4 — Automação do host

Objetivo:
- validar se systemd/timers continuam mantendo a camada pública atualizada

Sinais típicos:
- timers presentes
- services executando
- outputs públicos atualizando
- `health.json` coerente com a cadência esperada

---

### Camada 5 — Docker host

Objetivo:
- validar se o substrate do lado game está íntegro

Sinais típicos:
- daemon Docker ativo
- container relevante presente
- ausência de falha estrutural do runtime de containers

---

### Camada 6 — Saúde indireta dos workloads

Objetivo:
- entender se o host está saudável, mas algum workload superior está degradado

Sinais típicos:
- Nginx saudável, mas v2 stale
- Docker saudável, mas o servidor CS2 degradado
- automação saudável, mas frontend quebrado por contrato

Essa camada existe para evitar atribuição prematura de causa raiz à infraestrutura base.

---

## Logs e artefatos principais

Os pontos de observação mais relevantes do contexto incluem:

### Nginx

```bash
sudo nginx -t
sudo systemctl status nginx
sudo journalctl -u nginx -n 100 --no-pager
```

Uso principal:
- borda pública
- sintaxe de configuração
- falhas de serving
- indícios de problemas de edge

---

### Docker

```bash
sudo systemctl status docker
sudo journalctl -u docker -n 100 --no-pager
docker ps --format "table {{.Names}}\t{{.Image}}\t{{.Ports}}"
```

Uso principal:
- substrate de containers
- presença do workload do lado game
- falhas do daemon Docker

---

### systemd e timers

```bash
systemctl list-timers --all
sudo systemctl status SEU_TIMER.timer
sudo systemctl status SEU_SERVICE.service
sudo journalctl -u SEU_SERVICE.service -n 100 --no-pager
```

Uso principal:
- automação do host
- ETL
- health generation
- rotinas recorrentes

---

### Filesystem público

```bash
ls -lah /var/www/portal/cs2/
ls -lah /var/www/api/cs2/v2/
```

Uso principal:
- existência dos artefatos públicos
- timestamps
- triagem de publishing e permissionamento

---

### Filesystem operacional

```bash
ls -lah /opt/cs2-portal/
ls -lah /opt/cs2-portal/locks/
ls -lah /opt/cs2-portal/state/
ls -lah /opt/cs2-portal/sql/
ls -lah /usr/local/bin/
```

Uso principal:
- triagem da camada operacional do portal
- lock global
- statefiles
- scripts

---

### Árvore AMP

```bash
ls -lah /home/amp/.ampdata/instances/
```

Uso principal:
- validar a existência da árvore operacional do lado game
- confirmar que o substrate do host continua compatível com a camada AMP

---

## Smoke checks

Os smoke checks mínimos desta camada devem incluir:

### Validar portal público

Substitua pelo domínio público vigente.

```bash
curl -I https://SEU_DOMINIO/
```

### Validar recurso público da v2

Substitua pelo path público real da v2.

```bash
curl -I https://SEU_DOMINIO/SEU_PATH_PUBLICO_DA_V2/health.json
```

### Validar integridade do Nginx

```bash
sudo nginx -t
sudo systemctl status nginx
```

### Validar árvores públicas

```bash
ls -lah /var/www/portal/cs2/
ls -lah /var/www/api/cs2/v2/
```

### Validar Docker host

```bash
sudo systemctl status docker
docker ps --format "table {{.Names}}\t{{.Image}}\t{{.Ports}}"
```

### Validar automação do host

```bash
systemctl list-timers --all
```

### Validar artefato público básico da v2 no host

```bash
jq . /var/www/api/cs2/v2/health.json
```

Esses checks ajudam a separar falhas de:

- borda
- filesystem
- automação
- Docker host
- publishing
- ou workload superior

---

## Sequência de diagnóstico

Quando houver incidente no lado Hostinger, a sequência recomendada é:

### Etapa 1 — Confirmar borda pública

1. validar DNS
2. validar HTTPS do portal
3. validar HTTPS de um artefato da v2

Comandos típicos:

```bash
nslookup SEU_DOMINIO
curl -I https://SEU_DOMINIO/
curl -I https://SEU_DOMINIO/SEU_PATH_PUBLICO_DA_V2/health.json
```

---

### Etapa 2 — Confirmar Nginx

1. validar sintaxe
2. validar serviço
3. validar logs

Comandos típicos:

```bash
sudo nginx -t
sudo systemctl status nginx
sudo journalctl -u nginx -n 100 --no-pager
```

---

### Etapa 3 — Confirmar filesystem público

1. verificar árvores públicas
2. verificar arquivos esperados
3. verificar ownership e grupo

Comandos típicos:

```bash
ls -lah /var/www/portal/cs2/
ls -lah /var/www/api/cs2/v2/
stat /var/www/api/cs2/v2/health.json
```

---

### Etapa 4 — Confirmar automação do host

1. listar timers
2. verificar services ligados ao portal/v2
3. verificar atualização dos outputs públicos

Comandos típicos:

```bash
systemctl list-timers --all
ls -lah /var/www/api/cs2/v2/
cat /var/www/api/cs2/v2/health.json
```

---

### Etapa 5 — Confirmar Docker host

1. validar daemon Docker
2. validar container relevante
3. verificar logs do daemon

Comandos típicos:

```bash
sudo systemctl status docker
docker ps --format "table {{.Names}}\t{{.Image}}\t{{.Ports}}"
sudo journalctl -u docker -n 100 --no-pager
```

---

### Etapa 6 — Isolar workload superior

Se o substrate parecer saudável, aprofundar nos contextos superiores:

- `02-game-panel` se o problema estiver no lado jogo
- `03-portal-estatico` se o problema estiver na geração, contrato ou consumo da v2

Essa etapa evita continuar investigando a infraestrutura base quando a raiz já está claramente acima dela.

---

## Falhas comuns

### Nginx não sobe

Sintomas:

- `nginx -t` falha
- `systemctl status nginx` mostra erro
- portal e v2 indisponíveis publicamente

Causas comuns:

- sintaxe inválida
- path de serving incorreto
- problema de certificado
- drift na configuração de borda

Triagem:

```bash
sudo nginx -t
sudo systemctl status nginx
sudo journalctl -u nginx -n 100 --no-pager
```

---

### Certificado expirado ou inválido

Sintomas:

- navegador acusa falha de segurança
- `curl` HTTPS falha ou retorna problema de certificado
- portal e v2 parecem “fora” para o usuário final

Causas comuns:

- renovação não ocorreu
- mismatch entre hostname e certificado
- problema na borda após alteração do vhost

Triagem:

```bash
curl -I https://SEU_DOMINIO/
sudo certbot certificates
sudo systemctl status nginx
```

---

### Path publicado sem leitura

Sintomas:

- arquivo existe
- público retorna erro ou não serve corretamente
- troubleshooting inicialmente parece problema de Nginx

Causas comuns:

- `www-data` sem leitura
- traversal de diretório bloqueado
- ownership/grupo incorretos

Triagem:

```bash
ls -lah /var/www/api/cs2/v2/
stat /var/www/api/cs2/v2/health.json
```

---

### Timer não executa

Sintomas:

- v2 não atualiza
- `health.json` fica stale
- nenhuma mudança recente aparece nos outputs públicos

Causas comuns:

- timer desabilitado
- service falhando
- script operacional quebrado
- lock preso
- permissão incorreta

Triagem:

```bash
systemctl list-timers --all
ls -lah /var/www/api/cs2/v2/
ls -l /opt/cs2-portal/locks/
```

---

### Docker indisponível

Sintomas:

- daemon Docker parado
- container relevante ausente
- lado game comprometido
- cadeia de dados do portal tende a parar de evoluir depois

Causas comuns:

- falha do daemon
- problema de host
- runtime do Docker degradado

Triagem:

```bash
sudo systemctl status docker
docker ps -a --format "table {{.Names}}\t{{.Status}}\t{{.Image}}"
sudo journalctl -u docker -n 100 --no-pager
```

---

### Alias ou path incorreto

Sintomas:

- domínio responde
- assets ou JSONs específicos retornam `404`
- portal carrega parcialmente
- v2 parece “sumida”

Causas comuns:

- root/alias incorreto no Nginx
- path público divergente do esperado pelo frontend
- mudança não coordenada de diretório público

Triagem:

```bash
sudo nginx -t
ls -lah /var/www/portal/cs2/
ls -lah /var/www/api/cs2/v2/
```

---

### Cache ou arquivo estático não servido como esperado

Sintomas:

- arquivo local correto
- público continua entregando estado antigo ou errado
- troubleshooting fica ambíguo entre cache e publishing

Causas comuns:

- política de cache inadequada
- cliente com cache antigo
- troubleshooting sem teste isolado
- artifact stale realmente publicado

Triagem:

```bash
curl -I https://SEU_DOMINIO/
curl -I https://SEU_DOMINIO/SEU_PATH_PUBLICO_DA_V2/health.json
ls -lah /var/www/api/cs2/v2/
```

---

## Interpretação de sintomas

Alguns padrões úteis de interpretação:

### Portal e v2 falham juntos
Provável problema de borda ou filesystem público:
- Nginx
- TLS
- path de serving
- permissões

### Portal responde, mas v2 falha
Provável problema concentrado em:
- publishing da v2
- ETL
- path público da API estática
- permissions da árvore da v2

### Docker falha, mas portal ainda responde
Provável problema do substrate do lado game:
- impacto imediato no jogo
- impacto indireto posterior na atualização dos dados públicos

### Timers falham, mas Nginx segue saudável
Provável problema de automação:
- portal continua acessível
- dados públicos ficam stale
- health pode ou não refletir a degradação

### Arquivo existe no disco, mas não serve
Provável problema de:
- permissões
- root/alias
- leitura pelo Nginx
- publishing na árvore errada

---

## Lacunas atuais de observabilidade

As lacunas conhecidas ou prováveis desta camada incluem:

- observabilidade ainda fortemente baseada em diagnóstico manual
- ausência, até onde reconciliado, de stack formal de métricas e alertas
- dependência alta de `systemctl`, `journalctl`, `curl`, `ls` e `stat`
- necessidade de interpretar em conjunto host, publishing e workload
- risco de atribuição incorreta de causa raiz entre infraestrutura e contexto superior

Essas lacunas explicam por que a sequência de diagnóstico precisa ser explícita e disciplinada.

---

## Regras de ouro de troubleshooting

As regras de ouro desta camada são:

1. nunca assumir que o problema está “no portal” antes de validar a base
2. nunca assumir que Nginx ativo significa serving saudável
3. sempre validar público e local quando possível
4. sempre validar filesystem e permissões ao investigar serving
5. sempre considerar timers quando a camada pública estiver stale
6. sempre considerar Docker quando o lado game ou a origem de dados parar de evoluir
7. nunca confundir problema da Infra Hostinger com problema da Auth API em Lightsail
8. escalar para o contexto superior correto assim que a causa provável sair claramente da camada-base

---

## Limites deste documento

Este documento não detalha:

- troubleshooting profundo do ETL
- troubleshooting profundo do frontend
- troubleshooting profundo do servidor CS2
- debugging de plugins
- troubleshooting da Auth API
- mitigação avançada de incidentes multi-camada

Esses tópicos vivem em documentos complementares.

---

## Critério de pronto deste documento

Este documento pode ser considerado maduro quando:

- os sinais de saúde da Infra Hostinger estiverem alinhados com a prática real
- a sequência de diagnóstico refletir o uso operacional real do host
- as falhas comuns cobrirem os incidentes mais prováveis dessa camada-base
- os comandos aqui descritos forem suficientes para triagem inicial sem depender do master legado
- a distinção entre problema de infraestrutura e problema de workload superior estiver clara
- ele puder ser usado como runbook real de triagem da Infra Hostinger

---

## Última revisão

- Status: ativo
- Classificação: canônico
- Contexto: infraestrutura Hostinger / observability and troubleshooting
- Última revisão: 2026-03-18