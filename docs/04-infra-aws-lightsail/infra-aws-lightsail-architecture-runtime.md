# Architecture Runtime

## Objetivo

Documentar a topologia real de runtime do contexto AWS Lightsail que hospeda a Auth API do ecossistema HSC.

Este documento existe para registrar, de forma objetiva e auditável:

- o inventário essencial do host
- os componentes reais que compõem o runtime
- a relação entre Nginx, aplicação Node.js via systemd e MariaDB local
- os paths oficiais do contexto
- o fluxo básico de requisição e dependências operacionais
- os failure domains centrais desta camada

---

## Escopo

Este documento cobre:

- provedor e host operacional do contexto
- sistema operacional
- Nginx como edge local do contexto
- serviço `hsc-auth-api` via systemd
- aplicação Node.js da Auth API
- MariaDB local bound em loopback
- paths oficiais do runtime
- dependências e pontos de falha da topologia

Este documento não cobre em profundidade:

- DNS e TLS público
- configuração detalhada do reverse proxy
- contratos HTTP da Auth API
- deploy, release e rollback
- backup e restore
- troubleshooting detalhado

Esses tópicos vivem em documentos próprios deste contexto.

---

## Estado atual

O runtime atual conhecido do contexto `04-infra-aws-lightsail` é:

- provedor: AWS Lightsail
- sistema operacional: Ubuntu 22.04 LTS
- usuário operacional: `hscadmin`
- edge HTTP/HTTPS: Nginx com TLS e reverse proxy
- runtime da aplicação: Node.js via systemd
- nome do serviço principal: `hsc-auth-api`
- banco de dados: MariaDB InnoDB local bound em `127.0.0.1`
- base local da aplicação: `/opt/hsc/hsc-auth-api`
- health local da aplicação: `http://127.0.0.1:3000/health`

A Auth API opera como backend central da camada dinâmica do HSC, separada da infraestrutura Hostinger.

---

## Source of truth / evidências

As evidências principais deste runtime, nesta fase de migração, são:

- manual operacional da Auth API em AWS Lightsail
- documentação consolidada do ecossistema HSC
- blueprint técnico consolidado do HSC
- impl-logs de autenticação, schema, CORS, fail-closed e release/deploy

Os caminhos, nomes de serviço e relações entre componentes descritos aqui foram reconciliados a partir desses documentos.

---

## Relações com outros documentos

Este arquivo é complementar a:

- `docs/04-infra-aws-lightsail/README.md`
- `docs/04-infra-aws-lightsail/network-dns-tls.md`
- `docs/04-infra-aws-lightsail/nginx-reverse-proxy.md`
- `docs/04-infra-aws-lightsail/node-systemd.md`
- `docs/04-infra-aws-lightsail/mariadb-local.md`
- `docs/04-infra-aws-lightsail/deploy-release-rollback.md`
- `docs/04-infra-aws-lightsail/auth-api-operations.md`
- `docs/04-infra-aws-lightsail/infra-aws-lightsail-observability-troubleshooting.md`

Este documento descreve a topologia do runtime.  
Ele não substitui os documentos especializados acima.

---

## Inventário do host

### Provedor

- AWS Lightsail

### Perfil operacional conhecido

- instância dedicada ao contexto da Auth API
- IP estático associado
- host Linux persistente com operação administrativa via usuário `hscadmin`

### Sistema operacional

- Ubuntu 22.04 LTS

### Papel do host

Este host existe para sustentar exclusivamente a camada dinâmica da Auth API, com os seguintes blocos locais:

- edge HTTP/HTTPS via Nginx
- aplicação Node.js da Auth API
- banco MariaDB local
- utilitários e scripts operacionais de deploy e backup

---

## Topologia do runtime

A topologia lógica do contexto é:

- internet
- Nginx
- serviço `hsc-auth-api` via systemd
- processo Node.js da Auth API
- MariaDB local bound em `127.0.0.1`

Representação resumida:

- tráfego externo chega ao Nginx
- o Nginx encerra TLS e faz reverse proxy para a aplicação local
- a aplicação Node.js responde localmente
- a aplicação acessa o MariaDB pela interface loopback
- operações de deploy e rollback atuam sobre o working directory da aplicação e sobre o serviço systemd

Esta topologia mantém o banco inacessível externamente e reduz a superfície de exposição direta do runtime.

---

## Componentes do contexto

### AWS Lightsail

O Lightsail é o substrate de infraestrutura deste contexto.

Responsabilidades nesta camada:

- prover a VM
- prover o IP estático
- controlar a superfície de portas públicas permitidas
- sustentar a execução do Nginx, Node.js e MariaDB no mesmo host

Este componente não documenta a aplicação em si; ele sustenta o runtime local.

---

### Ubuntu

O Ubuntu 22.04 LTS é o sistema operacional base do host.

Responsabilidades nesta camada:

- execução do systemd
- gerenciamento de serviços locais
- filesystem do contexto
- execução do Nginx
- execução do MariaDB
- execução do Node.js
- suporte aos scripts operacionais do contexto

O sistema operacional é parte essencial do runtime porque define o comportamento de boot, serviços, logs e paths operacionais.

---

### Nginx

O Nginx é o edge local do contexto.

Papel no runtime:

- receber tráfego HTTP/HTTPS
- encerrar TLS
- fazer reverse proxy para a Auth API
- servir como camada pública de entrada do contexto

No desenho atual, o Nginx é a única camada que deve receber tráfego público diretamente para este contexto.

A configuração detalhada do reverse proxy e de TLS fica em documentos próprios.

---

### `hsc-auth-api` via systemd

O serviço principal do contexto é `hsc-auth-api`.

Estado operacional conhecido do serviço:

- unit file em `/etc/systemd/system/hsc-auth-api.service`
- usuário de execução: `hscadmin`
- working directory: `/opt/hsc/hsc-auth-api`
- environment file: `/opt/hsc/hsc-auth-api/.env`
- comando de execução: `/usr/bin/node index.js`
- política de reinício: `Restart=always`
- dependência de boot observada: `After=network.target mariadb.service`

Esse arranjo define um runtime de aplicação persistente e controlado pelo systemd.

---

### Aplicação Node.js

A aplicação Node.js representa a Auth API propriamente dita.

Responsabilidades da aplicação no contexto:

- expor endpoints públicos e administrativos
- ler configuração do `.env`
- aplicar regras de CORS do runtime
- operar superfícies administrativas protegidas
- acessar o MariaDB local
- responder ao health local e público

Observação importante:
- a aplicação é executada localmente e fica atrás do Nginx
- o health local documentado usa `http://127.0.0.1:3000/health`, o que indica porta local de aplicação na camada interna

---

### MariaDB local

O MariaDB é o banco de dados do contexto Auth API.

Estado operacional conhecido:

- engine: MariaDB / InnoDB
- host de acesso da aplicação: `127.0.0.1`
- database principal: `hsc_auth`
- exposição externa: bloqueada
- bind: local / loopback

Papel do MariaDB no runtime:

- persistir entidades e tabelas da Auth API
- sustentar autenticação, sessões, auditoria e conteúdo
- servir como base de dados local e acoplada ao host Lightsail

Esse desenho privilegia simplicidade operacional e menor exposição de rede.

---

## Paths oficiais

Os paths oficiais conhecidos deste contexto são:

### Aplicação

- `/opt/hsc/hsc-auth-api`

### Configuração de ambiente

- `/opt/hsc/hsc-auth-api/.env`

### Unit file do serviço

- `/etc/systemd/system/hsc-auth-api.service`

### State file de deploy/rollback

- `/opt/hsc/.deploy-auth-last-tag`

### Log operacional de deploy

- `/var/log/hsc/deploy-auth.log`

### Script de backup do banco

- `/opt/hsc/backup-mariadb.sh`

Esses paths devem ser tratados como referência operacional canônica até revisão explícita.

---

## Fluxo de requisição

### Fluxo externo

1. o cliente acessa o endpoint público da Auth API
2. o tráfego chega ao Nginx
3. o Nginx encerra TLS
4. o Nginx encaminha a requisição para a aplicação local
5. a aplicação processa a requisição
6. quando necessário, a aplicação acessa o MariaDB local
7. a resposta retorna ao cliente via Nginx

---

### Fluxo interno de saúde

1. o processo Node.js expõe health local
2. a validação local pode ser feita em `http://127.0.0.1:3000/health`
3. o health público é derivado do mesmo runtime, mas exposto pelo edge Nginx

Esse padrão ajuda a isolar falhas entre:
- edge
- aplicação
- banco local

---

## Dependências operacionais

Este contexto depende, em nível de runtime, de:

- serviço Nginx funcional
- serviço MariaDB funcional
- unit systemd válida para `hsc-auth-api`
- `.env` íntegro e coerente com o banco local
- working directory correto em `/opt/hsc/hsc-auth-api`
- Node.js disponível no host
- comunicação local entre aplicação e MariaDB via `127.0.0.1`
- fluxo operacional de deploy compatível com detached HEAD por TAG
- state file e logging operacional de deploy em estado íntegro

Dependências adicionais de governança e operação:

- disciplina de atualização por TAG
- sincronização correta entre release e restart do serviço
- cuidado com locks de deploy e com drift de configuração

---

## Failure domains

Os principais failure domains deste contexto são:

### 1. Edge Nginx

Se o Nginx falhar:
- o contexto perde exposição pública
- a aplicação pode continuar viva localmente, mas ficará inacessível externamente

---

### 2. Serviço `hsc-auth-api`

Se o serviço systemd falhar:
- o Nginx permanece respondendo no edge
- mas o upstream da Auth API ficará indisponível
- o health local falhará

---

### 3. MariaDB local

Se o MariaDB falhar:
- a aplicação pode subir parcialmente
- mas endpoints dependentes de banco ficam indisponíveis ou degradados
- superfícies de conteúdo/admin tendem a falhar

---

### 4. Configuração de ambiente

Se o `.env` estiver incorreto:
- a aplicação pode não subir
- pode subir com configuração inconsistente
- pode falhar em conexão com DB, CORS ou autenticação administrativa

---

### 5. Paths operacionais

Se paths críticos mudarem ou estiverem incorretos:
- o serviço pode iniciar no diretório errado
- o deploy pode falhar
- rollback pode perder a referência de última TAG
- logs e scripts operacionais podem deixar de funcionar como esperado

---

### 6. Drift entre deploy e serviço

Se o conteúdo do working directory, a TAG efetiva e o estado do serviço divergirem:
- o runtime real pode não refletir a versão esperada
- rollback pode se tornar confuso
- troubleshooting fica menos previsível

Esse risco é especialmente relevante porque o contexto usa deploy determinístico por TAG e detached HEAD.

---

## Limites deste documento

Este documento não detalha:

- nomes de domínio e política de DNS
- configuração textual do Nginx
- política completa de CORS
- contratos HTTP de cada endpoint
- fluxo detalhado de release e rollback
- estratégia detalhada de backup/restore
- matriz completa de troubleshooting

Esses detalhes vivem nos documentos próprios deste contexto.

---

## Critério de pronto deste documento

Este documento pode ser considerado maduro quando:

- o inventário do host estiver validado contra o runtime real
- os paths oficiais estiverem confirmados
- a relação entre Nginx, systemd, Node.js e MariaDB estiver estável
- os failure domains estiverem corretamente refletidos
- ele puder ser lido como visão topológica do contexto, sem depender do master legado

---

## Última revisão

- Status: ativo
- Classificação: canônico
- Contexto: infraestrutura AWS Lightsail / arquitetura de runtime
- Última revisão: 2026-03-18