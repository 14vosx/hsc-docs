# Network, DNS and TLS

## Objetivo

Documentar a camada de entrada pública do contexto Infra AWS Lightsail do ecossistema HSC, registrando a superfície de rede da Auth API, o hostname canônico, a borda TLS e os cuidados operacionais ligados à exposição externa dessa camada dinâmica.

Este documento existe para registrar, de forma estável e auditável:

- qual é a superfície pública canônica da Auth API
- como o DNS participa da exposição da API no Lightsail
- como o TLS é encerrado na borda dessa camada
- quais portas públicas são esperadas
- como validar a camada pública da Auth API
- quais riscos e cuidados operacionais existem nessa borda

---

## Escopo

Este documento cobre:

- superfície pública da Auth API
- DNS associado ao Lightsail
- fluxo de entrada HTTP/HTTPS
- terminação TLS via Nginx
- portas públicas esperadas
- relação entre borda pública e runtime local da aplicação
- validações operacionais da camada de entrada
- riscos e cuidados dessa borda

Este documento não cobre em profundidade:

- configuração textual completa do vhost Nginx
- deploy/release/rollback da aplicação
- modelagem do MariaDB local
- troubleshooting aprofundado da aplicação Node/Express
- infraestrutura da Hostinger
- publishing do Portal Estático

Esses tópicos vivem em documentos próprios deste contexto ou em outros contextos canônicos.

---

## Estado atual

O estado operacional conhecido e reconciliado da borda pública do contexto Lightsail é:

- a Auth API está canonicamente no AWS Lightsail
- o hostname público canônico da Auth API é:
  - `auth-api.haxixesmokeclub.com`
- o Nginx no Lightsail é a borda pública dessa camada
- o TLS é encerrado no próprio Lightsail, na borda Nginx da API
- o tráfego público esperado entra prioritariamente por HTTPS
- a aplicação Node.js não é a superfície pública canônica; ela responde localmente atrás do proxy reverso
- o upstream local reconciliado da aplicação é:
  - `http://127.0.0.1:3000`

Também ficou evidenciado no runtime real que:

- o endpoint `https://auth-api.haxixesmokeclub.com/health` responde com `200 OK`
- o host possui configuração específica da API em:
  - `/etc/nginx/sites-available/hsc-auth-api`

Leitura canônica:

- a borda pública da Auth API pertence ao Lightsail
- a Hostinger não é a camada canônica dessa borda, mesmo que exista configuração residual antiga fora do Lightsail

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
- `docs/04-infra-aws-lightsail/nginx-reverse-proxy.md`
- `docs/04-infra-aws-lightsail/node-systemd.md`
- `docs/04-infra-aws-lightsail/auth-api-operations.md`
- `docs/04-infra-aws-lightsail/infra-aws-lightsail-observability-troubleshooting.md`
- `docs/04-infra-aws-lightsail/infra-aws-lightsail-references-inventory.md`
- `docs/01-infra-hostinger/network-dns-tls.md`

Este documento descreve a camada de rede, DNS e TLS da Auth API no Lightsail.  
Ele não substitui os documentos de reverse proxy, runtime da aplicação, observabilidade ou inventário do contexto.

---

## Superfície pública do contexto

A superfície pública desta camada existe para expor:

- Auth API do ecossistema HSC
- endpoint de health
- endpoints públicos/privados da aplicação sob o domínio oficial da API
- borda dinâmica da metade backend do ecossistema

Regra canônica:

- a superfície pública da Auth API pertence ao Nginx do Lightsail
- o processo Node.js em `127.0.0.1:3000` não deve ser tratado como borda pública
- o runtime local da aplicação é interno ao host

---

## DNS oficial

O hostname público canônico reconciliado desta camada é:

- `auth-api.haxixesmokeclub.com`

Esse hostname deve ser tratado como:

- entrada pública oficial da Auth API
- referência canônica da camada dinâmica do ecossistema
- hostname prioritário em runbooks, troubleshooting e documentação

Regra importante:

- qualquer menção histórica à Auth API como superfície canônica da Hostinger deve ser tratada como estado antigo ou drift residual
- a leitura correta atual é Lightsail-first para a Auth API

---

## Fluxo de entrada HTTP/HTTPS

O fluxo esperado da borda pública do Lightsail é:

1. o cliente resolve `auth-api.haxixesmokeclub.com`
2. o cliente abre conexão HTTP ou, preferencialmente, HTTPS
3. o tráfego chega ao host Lightsail
4. o Nginx recebe a conexão
5. o Nginx encerra TLS
6. o Nginx resolve o vhost da Auth API
7. o Nginx faz proxy para `http://127.0.0.1:3000`
8. a aplicação Node responde
9. o Nginx devolve a resposta ao cliente

Esse desenho garante que:

- a aplicação não fique exposta diretamente
- a borda pública permaneça concentrada no Nginx
- DNS, TLS, proxy e hostname canônico permaneçam alinhados

---

## TLS / HTTPS

O estado reconciliado desta camada pressupõe HTTPS funcional na borda pública da Auth API.

O papel do TLS neste contexto é:

- proteger o tráfego entre cliente e API
- validar a identidade pública de `auth-api.haxixesmokeclub.com`
- permitir consumo confiável da camada dinâmica
- manter a Auth API operacionalmente utilizável por browser, frontend e integrações

Implicações operacionais:

- falha de certificado ou borda segura afeta imediatamente a disponibilidade percebida da API
- a aplicação pode continuar saudável localmente mesmo quando o acesso público em HTTPS estiver quebrado
- TLS saudável é parte do runtime real da Auth API, não detalhe cosmético

---

## Portas expostas

As portas públicas esperadas para este contexto são, em regra:

- `80/tcp`
- `443/tcp`

Papel típico de cada porta:

### Porta 80

- suporte a HTTP
- redirecionamento para HTTPS, quando aplicável
- apoio operacional à emissão e renovação de certificados

### Porta 443

- entrada HTTPS principal da Auth API
- tráfego produtivo esperado da camada dinâmica

Regra canônica:

- a porta `3000` pertence ao runtime interno da aplicação
- ela não deve ser tratada como interface pública canônica

---

## Relação entre DNS, TLS e reverse proxy

No Lightsail, DNS, TLS e reverse proxy precisam ser lidos como um conjunto.

Isso acontece porque:

- o cliente consome o hostname público da API
- o hostname precisa resolver corretamente para o Lightsail
- o TLS precisa ser válido para esse hostname
- o Nginx precisa resolver o vhost correto
- o proxy precisa apontar para o upstream local correto

Regra operacional:

- DNS correto sem proxy correto não resolve a borda
- proxy correto sem TLS funcional também não resolve a borda
- a camada pública só está saudável quando hostname, TLS, Nginx e upstream local estão coerentes juntos

---

## Relação com a aplicação local

A aplicação responde localmente em:

```text
http://127.0.0.1:3000
```

Isso significa que:

- a aplicação é dependência estrutural da borda pública
- ela não deve ser tratada como endpoint público direto
- troubleshooting precisa sempre separar:
  - borda pública
  - upstream local
  - runtime do serviço Node

Regra importante:

- `auth-api.haxixesmokeclub.com` é a borda pública
- `127.0.0.1:3000` é o upstream interno

---

## Validação operacional

As validações mínimas da camada de entrada devem incluir:

### Validar health público da Auth API

```bash
curl -I https://auth-api.haxixesmokeclub.com/health
curl -sS https://auth-api.haxixesmokeclub.com/health
```

### Validar sintaxe do Nginx

```bash
sudo nginx -t
```

### Validar status do Nginx

```bash
sudo systemctl status nginx
```

### Validar configuração ativa relevante

```bash
sudo nginx -T | grep -nE "server_name|proxy_pass|root |alias "
```

### Validar inventário de arquivos de Nginx

```bash
find /etc/nginx -maxdepth 3 -type f | sort
```

### Validar upstream local da aplicação

```bash
curl -I http://127.0.0.1:3000/health
curl -sS http://127.0.0.1:3000/health
```

Regra prática:

- se o upstream local responde, mas o hostname público não, o problema tende a estar em DNS/TLS/Nginx
- se o hostname público responde, mas o upstream local falha, o resultado público tende a degradar logo em seguida
- a borda precisa sempre ser validada por ambos os lados

---

## Sinais de saúde da camada de entrada

Os sinais de saúde esperados desta camada incluem:

- `auth-api.haxixesmokeclub.com` resolvendo corretamente
- conexão HTTPS estabelecida sem erro de certificado
- endpoint `/health` respondendo publicamente
- Nginx ativo e íntegro
- `server_name auth-api.haxixesmokeclub.com` presente no runtime
- proxy apontando para `127.0.0.1:3000`
- ausência de exposição pública indevida do upstream interno

Uma borda saudável depende tanto da camada de rede quanto da integridade do reverse proxy e do serviço local.

---

## Problemas comuns

### 1. Hostname resolve, mas `/health` falha

Causas comuns:

- problema no Nginx
- problema no upstream local
- proxy apontando para porta errada
- aplicação indisponível

Impacto:
- a borda existe, mas a API não responde corretamente

---

### 2. Upstream local responde, mas o hostname público falha

Causas comuns:

- problema de DNS
- problema de TLS
- vhost incorreto
- Nginx não carregou a configuração esperada

Impacto:
- a aplicação continua viva localmente
- a API fica indisponível ao público

---

### 3. `server_name` stale ou divergente

Causas comuns:

- drift de configuração
- hostname antigo mantido em arquivo ativo
- mudança de borda não reconciliada documentalmente

Impacto:
- tráfego pode ir para bloco incorreto
- troubleshooting fica ambíguo
- documentação perde aderência ao runtime

---

### 4. Proxy reverso correto, mas TLS inválido

Causas comuns:

- problema de certificado
- mismatch de hostname
- borda segura incompleta ou stale

Impacto:
- a aplicação pode estar saudável
- mas a experiência pública da API continua degradada

---

### 5. Drift entre Hostinger e Lightsail

Causas comuns:

- configuração residual antiga em outro host
- borda histórica não limpa
- leitura documental antiga ainda circulando

Impacto:
- confusão sobre onde a Auth API realmente roda
- troubleshooting mais lento
- risco de manutenção no host errado

Regra canônica:

- a presença de configuração residual em outro host não invalida a borda canônica reconciliada no Lightsail

---

## Invariantes operacionais

Os invariantes conhecidos desta camada incluem:

- a Auth API canônica está no Lightsail
- o hostname canônico é `auth-api.haxixesmokeclub.com`
- o Nginx do Lightsail é a borda pública dessa camada
- o upstream local reconciliado é `127.0.0.1:3000`
- o endpoint `/health` responde publicamente na borda reconciliada
- DNS, TLS, Nginx e upstream devem ser tratados como conjunto operacional

Esses invariantes ajudam a preservar a leitura correta da entrada pública da Auth API.

---

## Limites deste documento

Este documento não detalha:

- configuração completa do vhost Nginx
- arquivo completo de certificado/TLS
- políticas avançadas de headers de segurança
- deploy e restart da aplicação
- recovery completo após falha de borda
- limpeza técnica do vhost residual da Hostinger

Esses tópicos pertencem a documentos complementares ou a futuras reconciliações finas.

---

## Critério de pronto deste documento

Este documento pode ser considerado maduro quando:

- o hostname canônico da Auth API estiver explícito sem ambiguidade
- a relação entre DNS, TLS e reverse proxy estiver clara
- a distinção entre borda pública e upstream interno estiver formalizada
- os comandos de validação refletirem o runtime real
- ele puder ser usado como referência confiável da camada de rede e borda da Auth API sem depender do master legado

---

## Última revisão

- Status: ativo
- Classificação: canônico
- Contexto: infraestrutura AWS Lightsail / network, DNS and TLS
- Última revisão: 2026-03-18