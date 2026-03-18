# Node.js via systemd

## Objetivo

Documentar o runtime real da Auth API no contexto Infra AWS Lightsail do ecossistema HSC, registrando a unit `systemd` canônica da aplicação, o usuário de execução, o working directory reconciliado, o comando real de subida do Node.js e os pontos operacionais mínimos para validação dessa camada.

Este documento existe para registrar, de forma estável e auditável:

- qual é a unit real da Auth API no host
- como a aplicação é inicializada via `systemd`
- qual é o usuário real de execução
- qual é o working directory real do runtime
- qual é o `ExecStart` real da aplicação
- como validar se a app está de pé antes de culpar Nginx, DNS ou TLS
- quais riscos existem quando a documentação do runtime diverge do host real

---

## Escopo

Este documento cobre:

- unit `systemd` real da Auth API
- runtime Node.js real do host
- usuário, working directory e `ExecStart` reconciliados
- política básica de restart
- relação entre a unit e a porta `3000`
- validações operacionais mínimas
- riscos e problemas comuns dessa camada

Este documento não cobre em profundidade:

- configuração completa do Nginx reverse proxy
- DNS e TLS da borda pública
- modelagem do MariaDB local
- deploy/release/rollback
- backup/restore
- troubleshooting aprofundado da aplicação Express

Esses tópicos vivem em documentos próprios do contexto `04-infra-aws-lightsail`.

---

## Estado atual

O estado operacional conhecido e reconciliado do runtime Node.js da Auth API no Lightsail é:

- a unit canônica da aplicação é:
  - `hsc-auth-api.service`
- a unit está:
  - `loaded`
  - `active`
  - `running`
- o arquivo da unit reconciliado existe em:
  - `/etc/systemd/system/hsc-auth-api.service`
- o usuário real de execução da aplicação é:
  - `hscadmin`
- o working directory real reconciliado é:
  - `/opt/hsc/hsc-auth-api`
- o comando real de inicialização é:
  - `/usr/bin/node index.js`
- a política de restart reconciliada é:
  - `Restart=always`
- a variável de ambiente explicitamente reconciliada na unit é:
  - `Environment=NODE_ENV=production`

Também ficou reconciliado no runtime real que:

- o processo Node aparece como:
  - `/usr/bin/node index.js`
- a aplicação escuta na porta:
  - `3000`
- o socket observado está em:
  - `0.0.0.0:3000`

Leitura canônica:

- a Auth API roda como processo Node gerenciado por `systemd`
- o runtime real atual não usa mais o path antigo `/opt/hsc-auth-api`
- o runtime real atual usa `/opt/hsc/hsc-auth-api`
- a unit `hsc-auth-api.service` é a fonte de verdade principal do boot da app no host

---

## Source of truth / evidências

As principais evidências deste documento, nesta fase de reconciliação, são:

- runtime real do Lightsail
- `systemctl list-unit-files --type=service`
- `systemctl list-units --type=service --all`
- presença do arquivo:
  - `/etc/systemd/system/hsc-auth-api.service`
- extração reconciliada de:
  - `Description=`
  - `User=`
  - `WorkingDirectory=`
  - `ExecStart=`
  - `Restart=`
  - `Environment=`
  - `WantedBy=`
- processo Node real observado via `ps`
- socket real observado via `ss -ltnp`

Enquanto o runtime real permanecer neste formato, essas evidências prevalecem como source of truth operacional.

---

## Relações com outros documentos

Este arquivo é complementar a:

- `docs/04-infra-aws-lightsail/README.md`
- `docs/04-infra-aws-lightsail/architecture-runtime.md`
- `docs/04-infra-aws-lightsail/nginx-reverse-proxy.md`
- `docs/04-infra-aws-lightsail/network-dns-tls.md`
- `docs/04-infra-aws-lightsail/auth-api-operations.md`
- `docs/04-infra-aws-lightsail/observability-troubleshooting.md`
- `docs/04-infra-aws-lightsail/references-inventory.md`

Este documento descreve o runtime da aplicação via `systemd`.  
Ele não substitui os documentos de borda Nginx, rede, operações da API ou troubleshooting aprofundado.

---

## Unit canônica da aplicação

A unit reconciliada e canônica da Auth API no Lightsail é:

```text
hsc-auth-api.service
```

Estado reconciliado via `systemctl`:

- `enabled`
- `active`
- `running`

Leitura canônica:

- esta é a unit oficial do runtime da aplicação
- qualquer documentação anterior apontando para outro nome de service deve ser tratada como stale até nova validação
- operações de start, stop, restart e status da Auth API devem mirar esta unit

---

## Arquivo real da unit

O arquivo reconciliado da unit existe em:

```text
/etc/systemd/system/hsc-auth-api.service
```

Também foi observada a materialização via symlink/target em:

```text
/etc/systemd/system/multi-user.target.wants/hsc-auth-api.service
```

Leitura canônica:

- a unit é local do host, não herdada de pacote do sistema
- a unit está integrada ao boot normal do host por `multi-user.target`

---

## Campos reconciliados da unit

Os campos principais reconciliados no runtime real são:

### Description

```text
Description=HSC Auth API
```

### User

```text
User=hscadmin
```

### WorkingDirectory

```text
WorkingDirectory=/opt/hsc/hsc-auth-api
```

### ExecStart

```text
ExecStart=/usr/bin/node index.js
```

### Restart

```text
Restart=always
```

### Environment

```text
Environment=NODE_ENV=production
```

### WantedBy

```text
WantedBy=multi-user.target
```

Leitura canônica:

- a aplicação sobe a partir do diretório `/opt/hsc/hsc-auth-api`
- o binário Node é chamado diretamente
- o arquivo de entrada é `index.js` relativo ao working directory
- o runtime explícito é produção
- a app deve voltar automaticamente em caso de queda simples, pela política `Restart=always`

---

## Working directory real reconciliado

O working directory real do runtime é:

```text
/opt/hsc/hsc-auth-api
```

Isso corrige explicitamente leituras antigas ou incorretas que poderiam sugerir:

```text
/opt/hsc-auth-api
```

Leitura canônica:

- `/opt/hsc/hsc-auth-api` é o path correto
- troubleshooting, deploy e validação do runtime devem usar este path
- o `ls` em `/opt/hsc-auth-api` falha porque esse path não é o working directory atual da app

---

## ExecStart real reconciliado

O comando real da unit é:

```text
/usr/bin/node index.js
```

Como a unit define:

```text
WorkingDirectory=/opt/hsc/hsc-auth-api
```

a leitura operacional correta é:

- o `node` é executado a partir de `/opt/hsc/hsc-auth-api`
- o entrypoint esperado é:
  - `/opt/hsc/hsc-auth-api/index.js`

Regra canônica:

- o runtime real do host não depende, nesta leitura reconciliada, de PM2, Docker ou wrapper shell
- ele depende diretamente de `systemd` + `node index.js`

---

## Usuário real de execução

A aplicação roda como:

```text
hscadmin
```

Leitura canônica:

- o runtime da app não está, hoje, sob um usuário dedicado como `hscapi`
- o host real usa `hscadmin` como usuário da unit
- qualquer documento antigo mencionando outro usuário deve ser tratado como stale até nova validação

Isso é importante porque:

- permissões de diretório
- leitura de arquivos locais
- execução de deploy
- troubleshooting de processo

dependem desse usuário real.

---

## Política de restart

A unit reconciliada usa:

```text
Restart=always
```

Leitura canônica:

- a aplicação está configurada para reiniciar automaticamente quando o processo cair
- isso melhora a resiliência básica da camada dinâmica
- ainda assim, restart automático não substitui observabilidade e troubleshooting

Regra prática:

- aplicação reiniciando não implica aplicação saudável
- é sempre necessário validar a porta local e o endpoint `/health`

---

## Ambiente explícito reconciliado

A unit explicita:

```text
Environment=NODE_ENV=production
```

Leitura canônica:

- a app sobe explicitamente em modo de produção
- essa variável é parte do contrato operacional mínimo do runtime
- outras variáveis podem existir fora da unit, mas não foram reconciliadas aqui como fonte primária

Regra importante:

- documentar apenas o que foi efetivamente observado
- não presumir `EnvironmentFile=` ou outras variáveis sem validação direta

---

## Processo real observado

O processo real reconciliado no host foi:

```text
/usr/bin/node index.js
```

Associado ao usuário:

- `hscadmin`

Leitura canônica:

- a evidência do processo confirma a aderência entre a unit e o runtime real
- não há, nesta fotografia reconciliada, desvio entre `systemd` e processo efetivamente executado

---

## Porta local e binding observados

O socket reconciliado da aplicação mostrou:

```text
0.0.0.0:3000
```

Leitura correta:

- a aplicação está escutando na porta `3000`
- o binding observado no runtime atual é em `0.0.0.0`
- o Nginx continua fazendo proxy para `127.0.0.1:3000`, o que é compatível com o serviço estar ouvindo no host nessa porta

Regra de precisão:

- a borda pública canônica continua sendo o Nginx
- mas o binding real do processo não deve ser documentado como `127.0.0.1` sem prova
- o que foi observado nesta reconciliação foi `0.0.0.0:3000`

---

## Relação entre systemd, Node e Nginx

A leitura arquitetural correta desta camada é:

1. `systemd` mantém a app viva via `hsc-auth-api.service`
2. a unit sobe o processo Node em `/opt/hsc/hsc-auth-api`
3. a aplicação escuta na porta `3000`
4. o Nginx recebe o tráfego público em `auth-api.haxixesmokeclub.com`
5. o Nginx encaminha para `127.0.0.1:3000`
6. a resposta volta pela borda pública

Leitura canônica:

- `systemd` sustenta o runtime
- Node executa a app
- Nginx expõe a app ao público
- falha em qualquer uma dessas camadas pode degradar a API

---

## Comandos de validação

Os comandos abaixo representam a validação mínima desta camada.

### Validar se a unit existe

```bash
systemctl list-unit-files --type=service --no-pager | grep -Ei 'hsc|auth|api|node'
```

### Validar se a unit está carregada e ativa

```bash
systemctl list-units --type=service --all --no-pager | grep -Ei 'hsc|auth|api|node'
```

### Validar o arquivo real da unit

```bash
find /etc/systemd /lib/systemd/system -maxdepth 2 -type f | grep -Ei 'hsc|auth|api|node' | sort
```

### Validar campos principais da unit

```bash
grep -RInE 'Description=|ExecStart=|WorkingDirectory=|User=|Group=|Environment=|EnvironmentFile=|Restart=|WantedBy=' /etc/systemd /lib/systemd/system 2>/dev/null | grep -Ei 'hsc|auth|api|node'
```

### Validar status da unit canônica

```bash
systemctl status hsc-auth-api.service --no-pager
systemctl cat hsc-auth-api.service
```

### Validar processo real

```bash
ps -ef | grep -Ei 'node|hsc-auth-api|index.js' | grep -v grep
```

### Validar porta local

```bash
ss -ltnp | grep ':3000'
```

### Validar working directory real

```bash
ls -lah /opt/hsc/hsc-auth-api
```

### Validar health local e público

```bash
curl -I http://127.0.0.1:3000/health
curl -sS http://127.0.0.1:3000/health
curl -I https://auth-api.haxixesmokeclub.com/health
curl -sS https://auth-api.haxixesmokeclub.com/health
```

---

## Problemas comuns

### 1. Unit existe, mas a aplicação não sobe

Causas comuns:

- `WorkingDirectory` incorreto
- `index.js` ausente
- dependências da app quebradas
- permissões inadequadas para o usuário da unit

Impacto:
- `systemd` acusa falha
- Nginx pode continuar de pé, mas a API pública degrada

---

### 2. Documentação aponta para path antigo

Causas comuns:

- migração de diretório não reconciliada documentalmente
- documentação herdada de fase anterior
- confusão entre `/opt/hsc-auth-api` e `/opt/hsc/hsc-auth-api`

Impacto:
- troubleshooting no diretório errado
- falsa percepção de que o deploy “sumiu”

---

### 3. Porta `3000` não responde

Causas comuns:

- unit parada
- processo crashando
- app não iniciou corretamente
- conflito de porta

Impacto:
- Nginx devolve erro de upstream
- a borda pública fica indisponível

---

### 4. Processo Node responde, mas borda pública falha

Causas comuns:

- problema em Nginx
- problema de DNS
- problema de TLS
- vhost incorreto

Impacto:
- a app está viva localmente
- a API pública continua degradada

---

### 5. Binding real diferente do presumido

Causas comuns:

- documentação antiga
- mudança de configuração da aplicação
- suposição de que a app escuta apenas em loopback

Impacto:
- documentação imprecisa
- decisões erradas de hardening ou troubleshooting

Regra canônica:

- documentar o binding observado
- não documentar binding presumido

---

## Invariantes operacionais

Os invariantes conhecidos desta camada incluem:

- a unit canônica da app é `hsc-auth-api.service`
- o arquivo real da unit vive em `/etc/systemd/system/hsc-auth-api.service`
- o usuário real da unit é `hscadmin`
- o working directory real é `/opt/hsc/hsc-auth-api`
- o `ExecStart` real é `/usr/bin/node index.js`
- a política de restart é `Restart=always`
- o ambiente explicitamente reconciliado inclui `NODE_ENV=production`
- a app escuta na porta `3000`
- o binding observado no runtime atual é `0.0.0.0:3000`

Esses invariantes ajudam a preservar a leitura correta do runtime Node.js no Lightsail.

---

## Limites deste documento

Este documento não detalha:

- todas as variáveis de ambiente da aplicação
- estratégia completa de deploy
- política completa de logs
- hardening futuro do binding da porta
- troubleshooting profundo de dependências Node
- modelagem do banco local

Esses tópicos pertencem a documentos complementares ou a futuras reconciliações finas.

---

## Critério de pronto deste documento

Este documento pode ser considerado maduro quando:

- o nome real da unit estiver explícito sem ambiguidade
- o usuário, working directory e `ExecStart` reais estiverem corretos
- a relação entre `systemd`, Node e Nginx estiver clara
- a porta local estiver corretamente reconciliada com o runtime
- ele puder ser usado como referência confiável do runtime Node.js da Auth API sem depender do master legado

---

## Última revisão

- Status: ativo
- Classificação: canônico
- Contexto: infraestrutura AWS Lightsail / Node.js via systemd
- Última revisão: 2026-03-18