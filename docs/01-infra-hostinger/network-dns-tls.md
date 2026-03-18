# Network, DNS and TLS

## Objetivo

Documentar a camada de entrada pública da Infra Hostinger do ecossistema HSC, registrando a superfície de rede, o papel do DNS, a terminação TLS e os cuidados operacionais ligados à exposição externa do lado game + portal.

Este documento existe para registrar, de forma estável e auditável:

- quais superfícies públicas pertencem ao lado Hostinger do HSC
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

- configuração textual do Nginx
- publishing detalhado do portal e da v2
- operação do AMP e do servidor CS2
- Auth API no AWS Lightsail
- política de CORS da aplicação dinâmica
- troubleshooting aprofundado do host

Esses tópicos vivem em documentos próprios deste contexto ou em outros contextos canônicos.

---

## Estado atual

O estado operacional conhecido da borda pública do contexto Hostinger é:

- a camada Hostinger expõe superfícies públicas do portal e da Static API v2
- existe domínio público do ecossistema apontando para o lado Hostinger
- o Nginx é a borda pública dessa camada
- o TLS é encerrado no host por meio da integração entre Nginx e Certbot
- o tráfego externo esperado entra prioritariamente por HTTPS
- os workloads publicados nesse lado não devem expor diretamente suas estruturas internas ao público
- a integridade pública do portal depende da combinação entre DNS, TLS, Nginx, filesystem e publishing íntegros

Esse desenho preserva a separação entre:

- borda pública do lado Hostinger
- runtime interno do host
- workloads publicados sobre ele
- camada dinâmica separada no AWS Lightsail

---

## Source of truth / evidências

As principais evidências deste documento, nesta fase de migração documental, são:

- documentação consolidada atual do ecossistema HSC
- blueprint técnico consolidado do HSC
- documentação reconciliada da Infra Hostinger
- documentação reconciliada do Portal Estático e da Static API v2
- reconciliação do uso de Nginx e Certbot como borda pública do lado Hostinger

Enquanto a migração canônica do contexto não estiver concluída, essas fontes seguem sendo usadas como base de reconciliação do estado real.

---

## Relações com outros documentos

Este arquivo é complementar a:

- `docs/01-infra-hostinger/README.md`
- `docs/01-infra-hostinger/architecture-runtime.md`
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

O contexto Hostinger depende de resolução DNS correta para publicar o portal e a Static API v2.

A modelagem documental atual reconhece que o domínio principal do ecossistema participa diretamente dessa camada pública do lado Hostinger.

Em termos documentais, o lado Hostinger é a referência da camada pública para:

- portal público do HSC
- recursos estáticos
- paths públicos associados ao portal e à v2

Regra importante:

- este documento deve refletir o DNS efetivamente vigente no lado Hostinger
- quando houver mais de um hostname histórico na documentação, o runtime atual validado deve prevalecer
- mudança de hostname, alias ou subdomínio público do lado Hostinger deve ser tratada como alteração relevante de contexto

Enquanto a reconciliação final não estiver concluída, o canônico deve evitar afirmar hostnames além dos já validados no ambiente real.

---

## Subdomínios e papéis

A camada de DNS do lado Hostinger deve distinguir, conceitualmente, entre:

### domínio principal do portal

Usado para:

- acesso ao portal público
- acesso a assets públicos
- navegação pública principal

### paths públicos da Static API v2

Usados para:

- acesso aos JSONs públicos da v2
- consumo pelo frontend do portal
- validação operacional dos artefatos publicados

### conteúdos auxiliares same-origin

Usados para:

- conteúdo público complementar sob a mesma borda, quando esse fluxo estiver ativo

Regra de separação:

- um hostname ou alias só pertence ao contexto Hostinger quando resolve para a borda pública do host que serve o portal e a Static API v2
- a borda da Auth API em Lightsail não deve ser confundida com a borda pública do lado Hostinger

---

## Fluxo de entrada HTTP/HTTPS

O fluxo esperado da borda pública do lado Hostinger é:

1. o cliente resolve o hostname público do portal
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

## Firewall e borda

O host Hostinger deve manter coerência entre:

- DNS
- IP público do host
- portas liberadas
- Nginx funcional
- Certbot funcional
- publishing público válido
- proteção de arquivos internos

Cuidados operacionais importantes:

- `443/tcp` deve estar disponível para o edge público
- `80/tcp` deve estar coerente com política de redirecionamento e renovação
- a borda pública não deve expor `.db`, `.sqlite`, dotfiles ou paths operacionais
- o lado público do host deve publicar apenas portal, v2 e conteúdo público explicitamente permitido

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

Substitua pelo domínio vigente do portal.

```bash
nslookup SEU_DOMINIO
```

### Health público do portal

```bash
curl -I https://SEU_DOMINIO/
```

### Verificação pública de artefato da v2

Substitua pelo path público real da v2.

```bash
curl -I https://SEU_DOMINIO/SEU_PATH_PUBLICO_DA_V2/health.json
```

### Verificação pública de conteúdo principal

```bash
curl -sS https://SEU_DOMINIO/SEU_PATH_PUBLICO_DA_V2/health.json
curl -sS https://SEU_DOMINIO/SEU_PATH_PUBLICO_DA_V2/ranking.json
```

### Verificação local do Nginx

```bash
sudo nginx -t
sudo systemctl status nginx
```

A combinação entre DNS, acesso público e validação local do edge ajuda a separar falhas de:

- DNS
- TLS
- Nginx
- publishing
- ou filesystem/permissões

---

## Sinais de saúde da camada de entrada

Os sinais de saúde esperados desta camada incluem:

- hostname público resolvendo corretamente
- conexão HTTPS estabelecida sem erro de certificado
- portal público respondendo
- artefatos públicos da v2 respondendo
- Nginx ativo e íntegro
- ausência de exposição indevida de arquivos sensíveis
- coerência entre domínio, TLS, edge e publishing público

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

### 3. Domínio principal responde, mas JSON da v2 falha

Causas comuns:

- path público da v2 incorreto
- problema de publishing
- alias/root incorreto no Nginx
- permissões de leitura incorretas

Impacto:
- frontend pode carregar
- mas a camada de dados pública quebra

---

### 4. Público responde, mas browser mostra dado stale

Causas comuns:

- cache do navegador
- política de cache inadequada
- troubleshooting baseado em um único cliente sem invalidação de estado

Impacto:
- o host parece não refletir atualização recente
- o problema pode ser confundido com ETL ou publishing

---

### 5. Arquivo sensível exposto na borda

Causas comuns:

- proteção de borda ausente
- alias/root amplo demais
- publicação de diretório errado
- falta de deny para `.db`, `.sqlite` ou dotfiles

Impacto:
- risco operacional e de segurança
- quebra da separação entre público e operacional

---

## Riscos e cuidados

Os principais riscos desta camada incluem:

- drift entre DNS e host real
- TLS aparentemente ativo, mas servindo borda errada
- Nginx saudável sem publishing útil
- path público mudando sem atualização documental
- exposição indevida da árvore operacional do host
- confusão entre problemas da borda Hostinger e da Auth API em Lightsail

Cuidados permanentes:

- tratar hostname público como parte do contrato operacional
- validar DNS e TLS após qualquer mudança relevante
- manter a borda pública claramente separada da árvore operacional
- revisar deny rules e exposição de paths
- testar tanto o portal quanto a v2 após mudanças de borda

---

## Limites deste documento

Este documento não detalha:

- conteúdo do vhost Nginx
- sintaxe completa de configuração TLS
- política específica de cache por path
- runbooks detalhados de Certbot
- troubleshooting aprofundado do portal
- detalhes de publishing da v2

Esses pontos devem ser tratados em documentos complementares.

---

## Critério de pronto deste documento

Este documento pode ser considerado maduro quando:

- o domínio público vigente do lado Hostinger estiver validado sem ambiguidade
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