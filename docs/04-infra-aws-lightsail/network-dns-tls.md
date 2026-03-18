# Network, DNS and TLS

## Objetivo

Documentar a camada de entrada pública do contexto AWS Lightsail da Auth API, registrando a superfície de rede, o papel do DNS, a terminação TLS e os cuidados operacionais ligados à exposição externa do serviço.

Este documento existe para registrar, de forma estável e auditável:

- quais superfícies públicas pertencem a este contexto
- como o DNS participa da exposição da Auth API
- como o TLS é terminado
- quais portas públicas são esperadas
- como validar a camada de entrada do contexto
- quais riscos e cuidados operacionais existem nessa borda

---

## Escopo

Este documento cobre:

- subdomínios e superfícies públicas do contexto
- fluxo de entrada HTTP/HTTPS
- DNS associado ao contexto Lightsail
- terminação TLS
- portas públicas esperadas
- relação entre edge público e runtime interno da Auth API
- validações operacionais da borda
- riscos e cuidados dessa camada

Este documento não cobre em profundidade:

- configuração textual do Nginx
- unit file da aplicação
- contratos HTTP completos da Auth API
- fluxo detalhado de deploy e rollback
- troubleshooting aprofundado do runtime
- configuração completa de CORS da aplicação

Esses tópicos vivem em documentos próprios do contexto.

---

## Estado atual

O estado operacional conhecido da borda pública deste contexto é:

- a Auth API está hospedada em AWS Lightsail
- existe IP estático associado ao host
- a exposição pública ocorre via Nginx
- o tráfego externo esperado entra por HTTPS
- o TLS é encerrado no edge do contexto
- a aplicação Node.js roda atrás do Nginx
- o banco MariaDB não faz parte da superfície pública
- o consumo legítimo da Auth API depende de DNS, TLS e política de origem coerente com os domínios oficiais do ecossistema

Este desenho preserva a separação entre:

- borda pública
- runtime interno da aplicação
- persistência local

---

## Source of truth / evidências

As principais evidências deste documento, nesta fase de migração documental, são:

- manual operacional da Auth API em AWS Lightsail
- documentação consolidada do ecossistema HSC
- blueprint técnico consolidado
- impl-logs ligados a allowlist de CORS e operação da Auth API
- reconciliação documental do papel atual da camada Lightsail dentro da arquitetura híbrida do HSC

Enquanto a migração canônica do contexto não estiver concluída, essas fontes continuam servindo como base de reconciliação.

---

## Relações com outros documentos

Este arquivo é complementar a:

- `docs/04-infra-aws-lightsail/README.md`
- `docs/04-infra-aws-lightsail/architecture-runtime.md`
- `docs/04-infra-aws-lightsail/nginx-reverse-proxy.md`
- `docs/04-infra-aws-lightsail/node-systemd.md`
- `docs/04-infra-aws-lightsail/auth-api-operations.md`
- `docs/04-infra-aws-lightsail/observability-troubleshooting.md`

Este documento descreve a borda de rede, DNS e TLS do contexto.  
Ele não substitui os documentos de proxy, runtime da aplicação ou operação funcional.

---

## Superfícies públicas do contexto

A superfície pública deste contexto corresponde à entrada externa da Auth API.

Essa superfície existe para expor:

- health público do backend
- rotas públicas de conteúdo dinâmico
- superfícies administrativas protegidas que fazem parte do backend
- resposta do edge público sobre o runtime interno da Auth API

Regra canônica:
- a superfície pública pertence ao Nginx do contexto Lightsail
- a aplicação Node.js não deve ser tratada como serviço público exposto diretamente à internet

---

## DNS oficial

O contexto AWS Lightsail depende de resolução DNS correta para funcionar como backend público do ecossistema.

A modelagem documental atual reconhece que a Auth API opera sob subdomínio(s) oficiais do HSC.

Exemplos de nomes já observados historicamente na documentação do ecossistema incluem formas como:

- `auth-api.haxixesmokeclub.com`
- ou superfícies correlatas do domínio oficial do HSC

Regra importante:
- este documento deve refletir apenas o DNS que estiver efetivamente vigente
- quando houver mais de um hostname histórico, o runtime atual validado deve prevalecer
- mudanças de hostname devem ser tratadas como alteração relevante de contexto

Enquanto a reconciliação final não estiver concluída, a documentação canônica deve evitar afirmar mais do que o estado realmente validado.

---

## Subdomínios e papéis

A camada de DNS deste contexto deve distinguir, conceitualmente, entre:

### hostname público da Auth API

Usado para:

- health público
- rotas públicas de conteúdo
- rotas administrativas protegidas
- integrações autorizadas

### outros hostnames do ecossistema

Hostnames como portal público, subdomínios estáticos ou superfícies futuras não pertencem automaticamente a este contexto, mesmo quando interagem com ele.

Regra de separação:
- um hostname só pertence ao contexto AWS Lightsail quando ele resolve para a borda pública da Auth API desse contexto

---

## Fluxo de entrada HTTP/HTTPS

O fluxo esperado da borda pública é:

1. o cliente resolve o hostname público da Auth API via DNS
2. o cliente abre conexão HTTP ou, preferencialmente, HTTPS
3. o tráfego chega ao IP público estático do host Lightsail
4. o Nginx recebe a conexão
5. o Nginx encerra TLS
6. o Nginx encaminha a requisição para o runtime interno da Auth API
7. a aplicação responde
8. a resposta retorna ao cliente pela mesma borda

Esse desenho garante que:

- o cliente não acessa diretamente a porta interna da aplicação
- a segurança de transporte fica concentrada no edge
- a superfície pública do contexto fica estável e previsível

---

## TLS / Let's Encrypt

O estado conhecido do contexto pressupõe TLS ativo na borda pública do serviço.

O papel do TLS neste contexto é:

- proteger o tráfego entre cliente e edge
- validar a identidade do hostname público
- reduzir exposição de credenciais e payloads administrativos em trânsito
- sustentar consumo confiável por frontends e integrações autorizadas

A terminação TLS acontece no Nginx do contexto, e não no processo Node.js da aplicação.

Implicações operacionais:

- a aplicação Node.js pode operar apenas na malha local interna
- certificados e renovação afetam diretamente a disponibilidade pública e a confiança do cliente
- falha de TLS pode tornar o serviço operacionalmente indisponível, mesmo com a aplicação viva localmente

---

## Portas expostas

As portas públicas esperadas para este contexto são, em regra:

- `80/tcp`
- `443/tcp`

Papel típico de cada porta:

### Porta 80

- suporte a HTTP
- redirecionamento para HTTPS, quando aplicável
- apoio operacional à validação/renovação de certificado, conforme a estratégia adotada

### Porta 443

- entrada HTTPS principal do contexto
- tráfego produtivo esperado da Auth API

Regra canônica:
- a porta interna da aplicação não deve ser tratada como porta pública do contexto

Se a aplicação usa porta interna local, ela deve permanecer atrás do Nginx.

---

## Firewall e borda

O host Lightsail deve manter coerência entre:

- DNS
- IP estático
- portas liberadas
- Nginx em funcionamento
- aplicação escutando internamente
- banco restrito ao loopback

Cuidados operacionais importantes:

- `443/tcp` deve estar disponível para o edge público
- `80/tcp` deve estar coerente com a política de redirecionamento/renovação
- portas internas da aplicação não devem ser expostas como superfície pública normal
- MariaDB não deve ser promovido a serviço de rede pública do contexto

A borda pública deve ser a mínima necessária para o funcionamento do backend.

---

## Relação com CORS

CORS não é sinônimo de DNS, mas esta camada interage com a política de origem.

A distinção correta é:

- DNS define onde o serviço é encontrado
- TLS valida a identidade da borda e protege o transporte
- CORS controla quais origens de browser podem consumir a aplicação

Por isso:

- mudança de domínio ou subdomínio do ecossistema pode exigir atualização de CORS
- um DNS correto não implica que o browser poderá consumir a API
- um CORS correto não substitui TLS, DNS ou proxy adequados

Regra operacional:
- mudanças em domínio e origem devem ser tratadas de forma coordenada entre borda e aplicação

---

## Validação operacional

As validações mínimas da camada de entrada devem incluir:

### Resolução DNS

Substitua pelo hostname vigente do contexto.

```bash
nslookup SEU_DOMINIO
```

### Health público

Substitua pela URL pública vigente do contexto.

```bash
curl -sS https://SEU_DOMINIO/health
```

### Verificação manual de headers e resposta

```bash
curl -I https://SEU_DOMINIO/health
```

### Teste local isolando edge e app

```bash
curl -sS http://127.0.0.1:3000/health
```

A diferença entre os testes público e local ajuda a separar falhas de:

- DNS
- TLS
- Nginx
- aplicação
- ou dependências internas

---

## Sinais de saúde da camada de entrada

Os sinais de saúde esperados desta camada incluem:

- hostname público resolvendo para o IP correto
- conexão HTTPS estabelecida sem erro de certificado
- resposta válida do health público
- resposta válida do health local
- diferença clara entre falha de borda e falha de aplicação quando houver incidente
- ausência de exposição indevida de portas internas

Uma borda saudável depende tanto do caminho público quanto da coerência com o runtime interno.

---

## Problemas comuns

### 1. DNS resolve para IP incorreto

Causas comuns:

- mudança de IP sem atualização de zona DNS
- registro antigo ainda ativo
- configuração incorreta do hostname

Impacto:
- clientes chegam ao destino errado
- API aparentemente “fora” mesmo com o host saudável

---

### 2. TLS inválido ou expirado

Causas comuns:

- renovação não executada corretamente
- hostname não compatível com o certificado
- edge servindo certificado errado

Impacto:
- falha de confiança no cliente
- erros em browser e integrações
- indisponibilidade operacional do backend para consumo legítimo

---

### 3. `curl` local funciona, mas público falha

Causas comuns:

- Nginx parado ou mal configurado
- problema de TLS
- firewall ou borda incorreta
- DNS apontando errado

Impacto:
- a aplicação está viva, mas inacessível externamente

---

### 4. Público responde, mas browser bloqueia consumo

Causas comuns:

- problema de CORS
- origem não autorizada
- divergência entre domínio atual e allowlist da aplicação

Impacto:
- backend funcional, mas integração em browser quebrada

---

### 5. Porta interna exposta indevidamente

Causas comuns:

- regra de firewall permissiva
- configuração de aplicação ou host fora do desenho esperado

Impacto:
- aumento da superfície de ataque
- bypass do edge público
- perda de previsibilidade operacional

---

## Riscos e cuidados

Os principais riscos desta camada incluem:

- drift entre DNS e runtime real
- TLS ativo, mas apontando para configuração errada
- borda pública saudável com aplicação degradada internamente
- mudança de domínio sem atualização coordenada de CORS
- exposição indevida da porta interna da aplicação
- abertura indevida do banco como serviço externo

Cuidados permanentes:

- tratar hostname público como parte do contrato operacional
- validar DNS e TLS após mudanças
- manter edge e aplicação claramente separados
- revisar allowlist de origem quando domínios oficiais mudarem
- preservar a política de mínima exposição do contexto

---

## Limites deste documento

Este documento não detalha:

- conteúdo do vhost Nginx
- sintaxe de configuração TLS
- política completa de CORS
- detalhes de renovação de certificado
- troubleshooting completo do edge
- inventário detalhado de firewall do provedor

Esses pontos devem ser tratados em documentos complementares quando necessário.

---

## Critério de pronto deste documento

Este documento pode ser considerado maduro quando:

- o hostname público vigente do contexto estiver validado sem ambiguidade
- a relação entre DNS, IP estático, Nginx e aplicação estiver reconciliada com o runtime real
- a política de portas públicas estiver confirmada
- os testes de validação refletirem a prática operacional real
- os riscos de borda e de coordenação com CORS estiverem claros
- ele puder ser usado como referência da entrada pública do contexto sem depender do master legado

---

## Última revisão

- Status: ativo
- Classificação: canônico
- Contexto: infraestrutura AWS Lightsail / network, DNS e TLS
- Última revisão: 2026-03-18