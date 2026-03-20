# Network, DNS and TLS

## Objetivo

Documentar a camada de entrada pública da Infra Hostinger do ecossistema HSC, registrando a superfície de rede, o papel do DNS, a terminação TLS e os cuidados operacionais ligados à exposição externa do lado game + portal.

Este documento existe para registrar, de forma estável e auditável:

- quais superfícies públicas pertencem ao lado Hostinger do HSC
- quais hostnames públicos estão realmente ativos no runtime atual
- como o DNS participa da exposição do portal e da Static API v2
- como o TLS é terminado na borda do host
- quais portas públicas são esperadas
- como validar a camada de entrada pública deste lado do ecossistema
- quais riscos e cuidados operacionais existem nessa borda

---

## Escopo

Este documento cobre:

- superfícies públicas do lado Hostinger
- DNS associado ao host Hostinger
- fluxo de entrada HTTP/HTTPS
- terminação TLS via Nginx/Certbot
- portas públicas esperadas
- relação entre borda pública e workloads publicados
- validações operacionais da camada de entrada
- riscos e cuidados dessa borda

Este documento não cobre em profundidade:

- configuração textual completa do Nginx
- publishing detalhado do portal e da v2
- operação do AMP e do servidor CS2
- Auth API no AWS Lightsail
- política de CORS da aplicação dinâmica
- troubleshooting aprofundado do host

Esses tópicos vivem em documentos próprios deste contexto ou em outros contextos canônicos.

---

## Estado atual

O estado operacional conhecido e reconciliado da borda pública do contexto Hostinger é:

- a camada Hostinger expõe superfícies públicas do portal e da Static API v2
- o Nginx é a borda pública dessa camada
- o TLS é encerrado no host por meio da integração entre Nginx e Certbot
- o tráfego externo esperado entra prioritariamente por HTTPS
- os workloads publicados nesse lado não devem expor diretamente suas estruturas internas ao público
- a integridade pública do portal depende da combinação entre DNS, TLS, Nginx, filesystem e publishing íntegros

Também ficou reconciliado no runtime real que os hostnames públicos canônicos ativos do lado Hostinger são:

- `haxixesmokeclub.com`
- `www.haxixesmokeclub.com`

E que os hostnames abaixo **não** estão ativos como entrada pública canônica do portal no estado atual:

- `portal.haxixesmokeclub.com`
- `api.haxixesmokeclub.com`

Também existe hostname técnico do host:

- `srv1353392.hstgr.cloud`

Leitura correta:

- `srv1353392.hstgr.cloud` é hostname técnico de infraestrutura
- `haxixesmokeclub.com` e `www.haxixesmokeclub.com` são hostnames públicos canônicos do lado Hostinger
- a Auth API canônica não pertence a esta borda, e sim ao Lightsail

---

## Source of truth / evidências

As principais evidências deste documento, nesta fase de reconciliação, são:

- documentação consolidada atual do ecossistema HSC
- blueprint técnico consolidado do HSC
- documentação reconciliada da Infra Hostinger
- documentação reconciliada do Portal Estático e da Static API v2
- runtime real da Hostinger
- saída direta de `nginx -T`
- validação pública via `curl -I` dos hostnames reconciliados
- inventário real de arquivos em `/etc/nginx`

Enquanto o runtime real permanecer neste formato, essas evidências prevalecem como source of truth operacional.

---

## Relações com outros documentos

Este arquivo é complementar a:

- `docs/01-infra-hostinger/README.md`
- `docs/01-infra-hostinger/infra-hostinger-architecture-runtime.md`
- `docs/01-infra-hostinger/nginx-static-serving.md`
- `docs/01-infra-hostinger/certbot.md`
- `docs/01-infra-hostinger/observability-troubleshooting.md`
- `docs/03-portal-estatico/nginx-publishing-cache.md`
- `docs/04-infra-aws-lightsail/network-dns-tls.md`

Este documento descreve a borda de rede, DNS e TLS do lado Hostinger.  
Ele não substitui os documentos de serving estático, publicação, Certbot ou troubleshooting.

---

## Superfícies públicas do contexto

A superfície pública do lado Hostinger existe para expor:

- portal público do HSC
- Static API v2
- conteúdos públicos same-origin, quando aplicável
- edge HTTP/HTTPS da metade game + portal do ecossistema

Regra canônica:

- a superfície pública pertence ao Nginx do host Hostinger
- o filesystem operacional, o runtime do jogo e a árvore AMP não devem ser tratados como superfícies públicas diretas

---

## DNS oficial

Os hostnames públicos canônicos reconciliados do lado Hostinger são:

- `haxixesmokeclub.com`
- `www.haxixesmokeclub.com`

Esses hostnames devem ser tratados como:

- entrada pública oficial do portal
- entrada pública oficial da Static API v2
- referência canônica do lado game + portal do ecossistema

Hostname técnico do host:

- `srv1353392.hstgr.cloud`

Esse hostname deve ser tratado como:

- hostname técnico/de infraestrutura
- útil para operação do host
- não como hostname público canônico principal do ecossistema

Hostnames não ativos para a borda pública do portal no estado atual:

- `portal.haxixesmokeclub.com`
- `api.haxixesmokeclub.com`

Regra importante:

- não documentar hostnames não resolvidos como se fossem ativos
- quando houver drift entre documentação histórica e DNS real, prevalece o runtime validado

---

## Subdomínios e papéis

A camada de DNS do lado Hostinger deve distinguir, conceitualmente, entre:

### domínio principal do portal

Hostnames canônicos:

- `haxixesmokeclub.com`
- `www.haxixesmokeclub.com`

Usados para:

- acesso ao portal público
- acesso aos assets públicos
- acesso à Static API v2
- acesso ao conteúdo same-origin quando aplicável

### hostname técnico do host

Hostname:

- `srv1353392.hstgr.cloud`

Usado para:

- identificação técnica da máquina
- operação administrativa do host

### hostnames não ativos no estado atual

- `portal.haxixesmokeclub.com`
- `api.haxixesmokeclub.com`

Regra de separação:

- só pertence ao contexto público ativo aquilo que resolve e responde como borda real
- hostname técnico não substitui hostname público canônico

---

## Fluxo de entrada HTTP/HTTPS

O fluxo esperado da borda pública do lado Hostinger é:

1. o cliente resolve `haxixesmokeclub.com` ou `www.haxixesmokeclub.com`
2. o cliente abre conexão HTTP ou, preferencialmente, HTTPS
3. o tráfego chega ao host Hostinger
4. o Nginx recebe a conexão
5. o Nginx encerra TLS
6. o Nginx resolve o path solicitado
7. o Nginx serve portal, Static API v2 ou conteúdo same-origin público
8. a resposta retorna ao cliente

Esse desenho garante que:

- a camada pública é servida pelo edge do host
- o cliente não acessa diretamente estruturas operacionais internas
- TLS e publishing público ficam concentrados na mesma borda do lado Hostinger

---

## TLS / Let's Encrypt

O estado conhecido do contexto Hostinger pressupõe TLS ativo na borda pública do serviço.

O papel do TLS neste contexto é:

- proteger o tráfego entre cliente e edge do host
- validar a identidade pública do domínio do portal
- permitir consumo confiável do portal e da Static API v2
- reduzir exposição de conteúdo e navegação em trânsito

A terminação TLS acontece no Nginx do contexto, com suporte operacional do Certbot.

Implicações operacionais:

- falha de certificado afeta imediatamente a disponibilidade pública percebida
- portal e API estática podem parecer saudáveis no disco, mas indisponíveis do ponto de vista do usuário
- TLS saudável é parte da saúde real do lado Hostinger, e não um detalhe cosmético

---

## Portas expostas

As portas públicas esperadas para este contexto são, em regra:

- `80/tcp`
- `443/tcp`

Papel típico de cada porta:

### Porta 80

- suporte a HTTP
- redirecionamento para HTTPS, quando aplicável
- apoio operacional à emissão/renovação de certificados

### Porta 443

- entrada HTTPS principal do portal e da Static API v2
- tráfego produtivo esperado do lado Hostinger

Regra canônica:

- o host Hostinger não deve expor como superfície pública arbitrária os paths operacionais do ETL, a árvore AMP ou o banco SQLite
- a borda pública deve permanecer mínima e deliberada

---

## Relação entre DNS, TLS e publishing

No lado Hostinger, DNS e TLS não podem ser separados da publicação pública.

Isso porque:

- o portal depende de hostname público correto
- a Static API v2 depende de paths públicos previsíveis sob esse hostname
- o browser depende de HTTPS válido para consumo saudável da camada pública
- troubleshooting de portal precisa distinguir erro de publishing de erro de borda

Regra operacional:

- DNS correto sem publishing correto não resolve o contexto
- publishing correto sem TLS válido também não resolve o contexto
- a borda pública precisa estar íntegra como conjunto

---

## Validação operacional

As validações mínimas da camada de entrada devem incluir:

### Resolução DNS

Substitua pelo hostname que deseja validar.

```bash
nslookup haxixesmokeclub.com
nslookup www.haxixesmokeclub.com
```

### Health público do portal

```bash
curl -I https://haxixesmokeclub.com/
curl -I https://www.haxixesmokeclub.com/
```

### Verificação local do Nginx

```bash
sudo nginx -t
sudo systemctl status nginx
```

### Verificação de configuração relevante do Nginx

```bash
sudo nginx -T | grep -nE "server_name|root |alias "
```

### Inventário de arquivos do Nginx

```bash
find /etc/nginx -maxdepth 3 -type f | sort
```

Regra prática:

- `portal.haxixesmokeclub.com` e `api.haxixesmokeclub.com` não devem ser usados como smoke test principal enquanto não forem hostnames ativos do runtime
- a validação principal da borda deve mirar os hostnames canônicos reais

---

## Sinais de saúde da camada de entrada

Os sinais de saúde esperados desta camada incluem:

- `haxixesmokeclub.com` resolvendo corretamente
- `www.haxixesmokeclub.com` resolvendo corretamente
- conexão HTTPS estabelecida sem erro de certificado
- portal público respondendo
- Nginx ativo e íntegro
- coerência entre `server_name`, DNS e resposta pública real
- ausência de exposição indevida de arquivos sensíveis
- distinção clara entre hostname técnico e hostname público

Uma borda saudável depende tanto da camada de rede quanto da integridade do serving estático.

---

## Problemas comuns

### 1. DNS resolve para destino incorreto

Causas comuns:

- registro DNS desatualizado
- mudança de IP sem atualização correspondente
- configuração incorreta da zona

Impacto:
- cliente não chega ao host correto
- portal parece “fora” mesmo com o host saudável

---

### 2. TLS inválido ou expirado

Causas comuns:

- renovação não executada corretamente
- hostname não compatível com o certificado
- edge servindo certificado errado ou antigo

Impacto:
- navegador acusa falha de confiança
- portal e v2 ficam operacionalmente indisponíveis ao usuário

---

### 3. Hostname histórico ainda circula, mas não resolve

Causas comuns:

- documentação antiga
- memória operacional stale
- referência a subdomínio que já não participa da borda

Impacto:
- troubleshooting em hostname errado
- falsas suspeitas de indisponibilidade

Exemplos já reconciliados:

- `portal.haxixesmokeclub.com`
- `api.haxixesmokeclub.com`

---

### 4. Arquivo sensível exposto na borda

Causas comuns:

- proteção de borda ausente
- alias/root amplo demais
- publicação de diretório errado
- falta de deny para `.db`, `.sqlite` ou dotfiles

Impacto:
- risco operacional e de segurança
- quebra da separação entre público e operacional

---

### 5. Drift entre Hostinger e Lightsail

Causas comuns:

- configuração residual de hostname dinâmico no host errado
- borda histórica não limpa
- documentação antiga ainda influenciando leitura atual

Impacto:
- confusão sobre onde cada camada realmente roda
- manutenção no host errado
- troubleshooting mais lento

Regra canônica:

- a borda do portal pertence à Hostinger
- a borda da Auth API pertence ao Lightsail

---

## Riscos e cuidados

Os principais riscos desta camada incluem:

- drift entre DNS e host real
- TLS aparentemente ativo, mas servindo borda errada
- Nginx saudável sem publishing útil
- hostname não ativo continuar sendo citado como canônico
- exposição indevida da árvore operacional do host
- confusão entre problemas da borda Hostinger e da Auth API em Lightsail

Cuidados permanentes:

- tratar hostname público como parte do contrato operacional
- validar DNS e TLS após qualquer mudança relevante
- manter a borda pública claramente separada da árvore operacional
- revisar deny rules e exposição de paths
- testar o portal com os hostnames realmente ativos do runtime

---

## Limites deste documento

Este documento não detalha:

- conteúdo completo do vhost Nginx
- sintaxe completa de configuração TLS
- política específica de cache por path
- runbooks detalhados de Certbot
- troubleshooting aprofundado do portal
- detalhes finos do cleanup de configuração residual da Auth API na Hostinger

Esses pontos devem ser tratados em documentos complementares.

---

## Critério de pronto deste documento

Este documento pode ser considerado maduro quando:

- os hostnames públicos ativos do lado Hostinger estiverem validados sem ambiguidade
- a distinção entre hostname técnico e hostname público estiver clara
- a relação entre DNS, Nginx, Certbot e publishing estiver reconciliada com o host real
- a política de portas públicas estiver confirmada
- os riscos de borda e exposição indevida estiverem claros
- ele puder ser usado como referência da entrada pública do lado Hostinger sem depender do master legado

---

## Última revisão

- Status: ativo
- Classificação: canônico
- Contexto: infraestrutura Hostinger / network, DNS and TLS
- Última revisão: 2026-03-18