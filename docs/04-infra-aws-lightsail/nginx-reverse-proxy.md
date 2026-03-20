# Nginx Reverse Proxy

## Objetivo

Documentar o papel do Nginx como reverse proxy da Auth API no contexto Infra AWS Lightsail do ecossistema HSC.

Este documento existe para registrar, de forma estável e auditável:

- como o Nginx participa da borda pública da Auth API
- qual é o hostname canônico dessa camada
- para qual upstream local o proxy aponta no runtime real
- como o Nginx se relaciona com o serviço Node.js da Auth API
- quais validações mínimas ajudam a manter essa camada íntegra
- quais falhas comuns surgem quando a borda de proxy sai do estado esperado

---

## Navegação

### Entrada
- [Home da documentação](../README.md)
- [Infra AWS Lightsail](./README.md)
- [Master Index](../00-governance/99-master-index.md)

### Infraestrutura imediata
- [Architecture Runtime](./infra-aws-lightsail-architecture-runtime.md)
- [Auth API Operations](./auth-api-operations.md)
- [Node Systemd](./node-systemd.md)
- [Deploy / Release / Rollback](./deploy-release-rollback.md)

### Consumo administrativo
- [Backoffice Admin](../05-backoffice-admin/README.md)
- [Admin API Contracts](../05-backoffice-admin/admin-api-contracts.md)
- [Auth, RBAC and Guards](../05-backoffice-admin/auth-rbac-and-guards.md)
- [Frontend Structure](../05-backoffice-admin/frontend-structure.md)

### Operação e suporte
- [Operational Runbooks](./operational-runbooks.md)
- [Observability and Troubleshooting](./observability-troubleshooting.md)
- [Documentation System](../00-governance/documentation-system.md)

---

## Escopo

Este documento cobre:

- o papel do Nginx no Lightsail
- o hostname público da Auth API
- o upstream local da aplicação
- a relação entre TLS, Nginx e Node.js
- a localização reconciliada da configuração do vhost
- validações operacionais mínimas da borda
- riscos e problemas comuns dessa camada

Este documento não cobre em profundidade:

- deploy/release/rollback da aplicação
- configuração detalhada do systemd do Node.js
- modelagem do MariaDB local
- rotinas de backup/restore
- troubleshooting aprofundado da aplicação Express
- infraestrutura da Hostinger

Esses tópicos vivem em documentos próprios deste contexto ou em outros contextos canônicos.

---

## Estado atual

O estado operacional conhecido e reconciliado desta camada é:

- a Auth API está canonicamente no AWS Lightsail
- o hostname público canônico da Auth API é:
  - `auth-api.haxixesmokeclub.com`
- o Nginx no Lightsail atua como reverse proxy dessa API
- o upstream local reconciliado da aplicação é:
  - `http://127.0.0.1:3000`
- o endpoint `/health` responde publicamente com sucesso por essa borda
- a configuração reconciliada do vhost ativo existe em:
  - `/etc/nginx/sites-available/hsc-auth-api`

Também ficou evidenciado no runtime real que:

- existe um bloco default separado com `server_name _`
- esse bloco default não altera a leitura canônica da Auth API
- a borda pública oficial da API é o vhost específico de `auth-api.haxixesmokeclub.com`

---

## Source of truth / evidências

As principais evidências deste documento, nesta fase de reconciliação, são:

- documentação consolidada atual do ecossistema HSC
- blueprint técnico consolidado do HSC
- documentação reconciliada do contexto AWS Lightsail
- runtime real do Lightsail
- saída direta de `nginx -T`
- resposta pública validada de:
  - `https://auth-api.haxixesmokeclub.com/health`
- inventário real de arquivos em `/etc/nginx`

Enquanto o runtime real permanecer neste formato, essas evidências prevalecem como source of truth operacional.

---

## Relações com outros documentos

Este arquivo é complementar a:

- `docs/04-infra-aws-lightsail/README.md`
- `docs/04-infra-aws-lightsail/infra-aws-lightsail-architecture-runtime.md`
- `docs/04-infra-aws-lightsail/network-dns-tls.md`
- `docs/04-infra-aws-lightsail/node-systemd.md`
- `docs/04-infra-aws-lightsail/auth-api-operations.md`
- `docs/04-infra-aws-lightsail/observability-troubleshooting.md`
- `docs/04-infra-aws-lightsail/infra-aws-lightsail-references-inventory.md`

Este documento descreve a borda Nginx da Auth API no Lightsail.  
Ele não substitui os documentos de runtime da aplicação, TLS, systemd ou troubleshooting aprofundado.

---

## Papel do Nginx no contexto

No contexto Lightsail, o Nginx existe para:

- receber tráfego público da Auth API
- responder pelo hostname canônico da API
- encerrar a camada de borda HTTP/HTTPS
- encaminhar as requisições para a aplicação Node.js local
- isolar a aplicação do acesso público direto ao processo Node
- servir como ponto de validação e troubleshooting da borda da API

Em termos arquiteturais:

- o cliente acessa `auth-api.haxixesmokeclub.com`
- o Nginx recebe essa requisição
- o Nginx encaminha a requisição para `127.0.0.1:3000`
- a aplicação Node responde
- o Nginx devolve a resposta ao cliente

---

## Hostname canônico da Auth API

O hostname público canônico reconciliado da Auth API é:

- `auth-api.haxixesmokeclub.com`

Este hostname deve ser tratado como:

- entrada pública oficial da API
- referência canônica da borda dinâmica do ecossistema HSC
- substituto da leitura antiga que ainda podia deixar resquícios de Hostinger como runtime da Auth API

Regra canônica:

- `auth-api.haxixesmokeclub.com` pertence ao contexto Lightsail
- ele não deve ser documentado como borda canônica da Hostinger

---

## Upstream local da aplicação

O upstream local reconciliado no runtime real é:

```text
http://127.0.0.1:3000
```

Isso significa que:

- a aplicação Node.js da Auth API escuta localmente na porta `3000`
- o Nginx faz proxy reverso para esse processo local
- a porta da aplicação não é a superfície pública canônica da API
- a superfície pública canônica é o hostname servido pelo Nginx

Regra importante:

- `127.0.0.1:3000` é upstream interno
- `auth-api.haxixesmokeclub.com` é borda pública

---

## Configuração reconciliada do vhost

A evidência reconciliada do host mostra a presença de configuração específica da API em:

```text
/etc/nginx/sites-available/hsc-auth-api
```

Também ficou reconciliado em `nginx -T` que existem blocos com:

- `server_name auth-api.haxixesmokeclub.com;`
- `proxy_pass http://127.0.0.1:3000;`

Leitura canônica:

- o vhost específico da Auth API existe no Lightsail
- o reverse proxy ativo da API está no próprio Lightsail
- qualquer configuração residual em outro host deve ser tratada como legado ou cleanup pendente, não como borda canônica ativa

---

## Relação entre Nginx e Node.js

A Auth API não deve ser exposta publicamente como processo Node puro.

A relação correta entre as camadas é:

- Node.js roda localmente
- systemd mantém o processo da aplicação
- Nginx faz a borda pública e o proxy reverso
- o cliente consome a API pelo hostname da borda, não pelo processo interno

Benefícios dessa separação:

- isola o processo da aplicação
- mantém a borda padronizada
- facilita TLS, proxy e observabilidade
- permite troubleshooting por camada

Regra operacional:

- problema de aplicação e problema de proxy não são a mesma coisa
- o Nginx pode estar saudável com a aplicação degradada
- ou a aplicação pode estar saudável com a borda degradada

---

## Relação com TLS

A camada de proxy reverso deve ser lida em conjunto com a camada de rede e TLS do Lightsail.

Isso significa que:

- o hostname canônico da API depende de borda Nginx correta
- a experiência pública saudável depende de HTTPS funcional
- troubleshooting da Auth API precisa distinguir:
  - falha de DNS
  - falha de TLS
  - falha de Nginx
  - falha da aplicação Node

Este documento foca no reverse proxy.  
Os detalhes gerais de DNS/TLS vivem em `network-dns-tls.md`.

---

## Evidência de saúde pública

A borda reconciliada já respondeu com sucesso ao endpoint:

- `https://auth-api.haxixesmokeclub.com/health`

A resposta pública confirmou:

- HTTP `200 OK`
- resposta JSON da aplicação
- presença de headers do Nginx e da aplicação Express
- aplicação respondendo através do proxy reverso do Lightsail

Leitura canônica:

- a borda pública da Auth API está funcional no Lightsail
- o reverse proxy está de fato apontando para uma aplicação ativa

---

## Camada default do Nginx

Também foi identificada no runtime uma configuração default com:

- `server_name _`
- `root /var/www/html`

Esse bloco deve ser tratado como:

- configuração default do host
- não como vhost canônico da Auth API

Regra importante:

- o vhost default não substitui o vhost específico de `auth-api.haxixesmokeclub.com`
- a documentação da Auth API deve continuar centrada no bloco específico da API

---

## Dependências operacionais

Esta camada depende, no mínimo, de:

- DNS correto para `auth-api.haxixesmokeclub.com`
- Nginx funcional no Lightsail
- upstream local `127.0.0.1:3000` ativo
- aplicação Node.js saudável
- systemd mantendo a aplicação disponível
- configuração do vhost coerente com o runtime real
- ausência de drift entre hostname, proxy e app local

Dependências cruzadas importantes:

- se o Node cair, o Nginx pode continuar “de pé” sem entregar valor
- se o Nginx quebrar, a aplicação pode continuar ativa localmente e ainda assim ficar indisponível ao público
- a leitura canônica da borda deve sempre considerar as duas camadas

---

## Comandos de validação

Os comandos abaixo representam a validação mínima desta camada.

### Validar sintaxe do Nginx

```bash
sudo nginx -t
```

### Validar status do Nginx

```bash
sudo systemctl status nginx
```

### Validar configuração ativa da Auth API

```bash
sudo nginx -T | grep -nE "server_name|proxy_pass|root |alias "
```

### Validar presença do vhost reconciliado

```bash
find /etc/nginx -maxdepth 3 -type f | sort
```

### Validar health público da Auth API

```bash
curl -I https://auth-api.haxixesmokeclub.com/health
curl -sS https://auth-api.haxixesmokeclub.com/health
```

### Validar upstream local diretamente no host

```bash
curl -I http://127.0.0.1:3000/health
curl -sS http://127.0.0.1:3000/health
```

Regra prática:

- se o upstream local responde, mas o hostname público não, o problema tende a estar na borda
- se nem o upstream local responde, o problema tende a estar na aplicação ou no systemd

---

## Problemas comuns

### 1. Nginx saudável, aplicação indisponível

Causas comuns:

- processo Node parado
- serviço systemd da app degradado
- app respondendo fora da porta esperada
- falha de boot da aplicação

Impacto:
- a borda pública responde erro de upstream
- o problema parece “de Nginx”, mas a raiz está na aplicação

---

### 2. Aplicação saudável localmente, hostname público falha

Causas comuns:

- problema no vhost
- problema de DNS
- problema de TLS
- Nginx não carregou a config esperada

Impacto:
- o processo Node continua útil apenas localmente
- a API fica indisponível ao público

---

### 3. `server_name` incorreto ou stale

Causas comuns:

- drift de configuração
- hostname antigo mantido no host
- borda reconfigurada sem atualização documental

Impacto:
- tráfego pode cair em bloco errado
- troubleshooting fica ambíguo
- a documentação perde aderência ao runtime

---

### 4. Proxy apontando para porta errada

Causas comuns:

- mudança do runtime da aplicação sem ajuste do vhost
- múltiplas versões da aplicação
- intervenção manual não reconciliada

Impacto:
- hostname público deixa de responder corretamente
- a app pode continuar saudável em outra porta, mas invisível para a borda correta

---

### 5. Drift entre Lightsail e Hostinger

Causas comuns:

- configuração residual antiga no host errado
- vhost legado ainda presente em Hostinger
- arquitetura atual não refletida em limpeza técnica completa

Impacto:
- confusão documental
- troubleshooting mais lento
- percepção errada de onde a Auth API realmente roda

Regra canônica:

- a presença de config residual em outro host não invalida a borda canônica reconciliada no Lightsail

---

## Invariantes operacionais

Os invariantes conhecidos desta camada incluem:

- a Auth API canônica está no Lightsail
- o hostname canônico é `auth-api.haxixesmokeclub.com`
- o Nginx do Lightsail faz proxy reverso para `127.0.0.1:3000`
- o vhost específico da API existe em `/etc/nginx/sites-available/hsc-auth-api`
- o endpoint `/health` responde publicamente pela borda do Lightsail
- a aplicação Node e o Nginx devem ser lidos como camadas separadas, porém acopladas operacionalmente

Esses invariantes ajudam a preservar a leitura correta da borda dinâmica do ecossistema.

---

## Limites deste documento

Este documento não detalha:

- arquivo completo do vhost linha a linha
- configuração detalhada do serviço Node.js
- estratégias completas de reload/restart da app
- política detalhada de TLS
- recovery completo após falha de borda
- cleanup da configuração residual da Hostinger

Esses tópicos pertencem a documentos complementares ou a futuras reconciliações finas.

---

## Critério de pronto deste documento

Este documento pode ser considerado maduro quando:

- o hostname canônico da Auth API estiver explícito sem ambiguidade
- o upstream local reconciliado estiver claro
- a relação entre Nginx e Node.js estiver descrita corretamente
- o vhost da API estiver identificado no runtime
- ele puder ser usado como referência confiável da camada de reverse proxy da Auth API sem depender do master legado

---

## Última revisão

- Status: ativo
- Classificação: canônico
- Contexto: infraestrutura AWS Lightsail / Nginx reverse proxy
- Última revisão: 2026-03-18