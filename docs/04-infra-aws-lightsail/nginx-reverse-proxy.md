# Nginx Reverse Proxy

## Objetivo

Documentar o papel do Nginx como edge público e reverse proxy da Auth API no contexto AWS Lightsail.

Este documento existe para registrar, de forma estável e auditável:

- a função do Nginx na borda pública da Auth API
- a relação entre TLS termination e upstream interno
- o modelo de reverse proxy adotado no contexto
- os limites entre responsabilidades do edge e da aplicação
- os cuidados operacionais necessários para manter a exposição pública saudável
- os problemas mais comuns ligados a proxy, upstream e headers

---

## Escopo

Este documento cobre:

- o papel do Nginx no contexto AWS Lightsail
- o modelo de reverse proxy para a Auth API
- a relação entre hostname público e upstream local
- TLS termination na borda
- headers relevantes do edge
- interação entre Nginx e CORS da aplicação
- validações operacionais do proxy
- sintomas e falhas comuns do edge

Este documento não cobre em profundidade:

- DNS e política geral de TLS
- unit file da aplicação
- contratos HTTP completos da Auth API
- fluxo de deploy e rollback
- troubleshooting geral do host
- modelagem funcional do backend

Esses assuntos vivem em documentos próprios do contexto.

---

## Estado atual

O estado operacional conhecido do edge deste contexto é:

- o Nginx é a camada pública de entrada da Auth API
- o Nginx recebe tráfego HTTP/HTTPS do hostname público do contexto
- o TLS é encerrado no Nginx
- o Nginx encaminha requisições para a aplicação Node.js rodando localmente
- a aplicação permanece atrás do proxy e não deve ser tratada como superfície pública direta
- o banco MariaDB não participa da borda pública
- a saúde pública do backend depende da combinação entre Nginx funcional e upstream local saudável

Esse desenho preserva a separação entre:

- borda pública
- aplicação interna
- banco local

---

## Source of truth / evidências

As principais evidências deste documento, nesta fase de migração documental, são:

- manual operacional da Auth API em AWS Lightsail
- documentação consolidada do ecossistema HSC
- blueprint técnico consolidado
- reconciliação do health local em `127.0.0.1:3000`
- documentação operacional que posiciona o Nginx como edge público do contexto
- impl-logs ligados a CORS e endurecimento operacional da aplicação

Enquanto a migração canônica do contexto não estiver concluída, essas fontes seguem sendo usadas como base de reconciliação.

---

## Relações com outros documentos

Este arquivo é complementar a:

- `docs/04-infra-aws-lightsail/README.md`
- `docs/04-infra-aws-lightsail/architecture-runtime.md`
- `docs/04-infra-aws-lightsail/network-dns-tls.md`
- `docs/04-infra-aws-lightsail/node-systemd.md`
- `docs/04-infra-aws-lightsail/auth-api-operations.md`
- `docs/04-infra-aws-lightsail/observability-troubleshooting.md`

Este documento descreve o reverse proxy do contexto.  
Ele não substitui os documentos de DNS/TLS, runtime da aplicação ou operação funcional.

---

## Papel do Nginx no contexto

O Nginx é o edge oficial do contexto AWS Lightsail.

Suas responsabilidades neste contexto são:

- receber tráfego público HTTP/HTTPS
- encerrar TLS
- expor o hostname público da Auth API
- encaminhar requisições para a aplicação local
- servir como primeira camada de validação operacional da borda
- isolar a aplicação Node.js do tráfego direto da internet

O Nginx não é o responsável primário pela lógica de negócio da aplicação, pela persistência em banco ou pela política completa de CORS do backend.

---

## Modelo de reverse proxy

O modelo canônico do contexto é:

- cliente externo acessa o hostname público da Auth API
- Nginx recebe a requisição
- Nginx encerra TLS
- Nginx encaminha a requisição para o upstream local da aplicação
- a aplicação Node.js processa a requisição
- a resposta retorna ao cliente via Nginx

Esse desenho implica:

- a aplicação roda em porta interna/local
- o cliente não deve consumir diretamente essa porta
- o proxy é a camada oficial de exposição pública do backend

---

## Upstream da Auth API

O upstream local conhecido do contexto é a aplicação Node.js escutando em loopback, com health local observado em:

- `http://127.0.0.1:3000/health`

Isso indica que o upstream interno esperado é, conceitualmente:

- host local: `127.0.0.1`
- porta interna da aplicação: `3000`

Regra operacional:
- o upstream deve permanecer acessível localmente
- a indisponibilidade do upstream transforma o edge em proxy sem backend útil
- um Nginx “de pé” não significa backend saudável se o upstream estiver falhando

---

## Fluxo do proxy

O fluxo esperado do reverse proxy é:

1. cliente resolve o hostname público
2. cliente conecta ao Nginx
3. Nginx aceita conexão e encerra TLS
4. Nginx encaminha a requisição para o upstream local
5. aplicação Node.js responde
6. Nginx devolve a resposta ao cliente

Esse fluxo permite diferenciar duas classes de incidente:

- problema de edge/proxy
- problema de upstream/aplicação

---

## TLS termination

A terminação TLS acontece no Nginx do contexto.

Isso significa:

- o certificado público pertence à borda Nginx
- a aplicação não precisa expor TLS diretamente na malha interna local
- o tráfego público seguro é responsabilidade do edge

Implicações operacionais:

- falhas de certificado afetam a disponibilidade percebida externamente
- falhas no Nginx podem indisponibilizar o backend mesmo com a aplicação viva localmente
- troubleshooting público deve sempre considerar a separação entre edge e upstream

---

## Headers e segurança

O Nginx participa da construção do contexto HTTP entregue ao backend e ao cliente.

Em nível conceitual, essa camada deve preservar:

- encaminhamento correto para o upstream
- integridade de headers necessários ao backend
- consistência entre esquema HTTPS público e percepção interna da aplicação
- postura mínima de segurança no edge

Headers e políticas específicos devem ser mantidos coerentes com o runtime real, mas este documento evita congelar uma configuração textual completa sem validação direta do host.

Princípios esperados:

- o backend deve receber contexto suficiente para interpretar corretamente a requisição original
- o edge deve evitar se tornar uma fonte de ambiguidade entre HTTP interno e HTTPS público
- políticas básicas de segurança de borda devem ser mantidas de forma consistente

---

## CORS: o que é do app e o que é do edge

A distinção correta neste contexto é:

- Nginx = edge público e reverse proxy
- aplicação = autoridade principal sobre política de CORS

Isso significa:

- o Nginx participa da entrega pública do serviço
- mas a semântica principal de quais origens de browser podem consumir a API pertence ao runtime da aplicação
- mudanças em allowlist de origem devem ser tratadas prioritariamente no backend, não apenas na borda

O edge e a aplicação precisam permanecer coerentes, mas o Nginx não deve ser tratado como substituto da política de CORS da Auth API.

---

## Responsabilidades do edge

As responsabilidades esperadas do Nginx neste contexto incluem:

- expor a Auth API por hostname público correto
- encerrar TLS corretamente
- encaminhar o tráfego ao upstream local correto
- preservar headers essenciais
- responder de forma previsível quando o upstream estiver indisponível
- manter clara a separação entre borda e runtime interno

---

## Responsabilidades que não pertencem ao edge

Não pertencem primariamente ao Nginx, neste contexto:

- lógica de autenticação administrativa
- decisão de autorização funcional
- persistência em banco
- modelagem de domínio de News, Seasons ou Events
- semântica principal de CORS
- registro de auditoria administrativa
- controle de sessão do backend

Esses pontos pertencem à aplicação e às suas dependências internas.

---

## Validação operacional

As validações mínimas do reverse proxy devem incluir:

### Validar health público

Substitua pelo hostname vigente do contexto.

```bash
curl -sS https://SEU_DOMINIO/health
```

### Verificar headers da resposta pública

```bash
curl -I https://SEU_DOMINIO/health
```

### Validar health local do upstream

```bash
curl -sS http://127.0.0.1:3000/health
```

### Validar status do Nginx

```bash
sudo systemctl status nginx
```

### Validar sintaxe da configuração

```bash
sudo nginx -t
```

### Ver logs recentes do Nginx

O path exato pode variar conforme o host.  
Quando necessário, consultar os logs padrão do serviço e do access/error log vigentes no sistema.

Exemplo representativo:

```bash
sudo journalctl -u nginx -n 100 --no-pager
```

A combinação entre health público, health local e status do Nginx é a forma mais eficiente de separar falhas de edge e upstream.

---

## Sinais de saúde do proxy

Os sinais de saúde esperados desta camada incluem:

- `nginx -t` sem erro
- serviço Nginx ativo
- health público respondendo corretamente
- health local do upstream respondendo corretamente
- ausência de erro recorrente de upstream nos logs
- TLS funcionando sem erro para o hostname público
- ausência de exposição direta indevida da porta interna da aplicação

---

## Problemas comuns

### 1. Nginx ativo, mas upstream indisponível

Causas comuns:

- aplicação Node.js parada
- serviço `hsc-auth-api` falhando
- porta interna incorreta
- upstream local inacessível

Impacto:
- borda responde com erro de upstream
- health público falha, mesmo com o Nginx ativo

---

### 2. Upstream responde localmente, mas público falha

Causas comuns:

- Nginx parado
- configuração incorreta do proxy
- TLS quebrado
- hostname apontando para edge incorreto
- firewall/borda em estado inconsistente

Impacto:
- aplicação viva localmente
- indisponibilidade pública do backend

---

### 3. Configuração do Nginx inválida

Causas comuns:

- sintaxe incorreta
- diretiva inválida
- edição manual malformada
- conflito entre arquivos de configuração

Impacto:
- reload/restart falha
- risco de indisponibilidade pública imediata

---

### 4. Proxy aponta para upstream errado

Causas comuns:

- porta incorreta
- host interno incorreto
- drift entre configuração do Nginx e runtime atual da aplicação

Impacto:
- tráfego encaminhado para destino inválido
- health público quebra
- troubleshooting fica confuso

---

### 5. Browser consome o endpoint, mas falha por origem

Causas comuns:

- problema de CORS no backend
- allowlist não atualizada
- mudança de hostname sem coordenação com a aplicação

Impacto:
- Nginx e upstream podem estar corretos
- integração em browser continua quebrada

---

### 6. Certificado válido, mas serviço funcionalmente degradado

Causas comuns:

- upstream parcial
- banco degradado
- regressão da aplicação
- health insuficiente para detectar problema funcional

Impacto:
- borda saudável do ponto de vista de TLS
- backend ainda assim problemático

---

## Sequência básica de diagnóstico

Quando houver falha pública no contexto, a sequência recomendada é:

1. validar `nginx -t`
2. validar status do serviço Nginx
3. testar health público
4. testar health local do upstream
5. verificar logs do Nginx
6. verificar status e logs do serviço `hsc-auth-api`
7. só depois aprofundar em banco, CORS ou semântica funcional

Essa sequência ajuda a responder rapidamente:

- o edge está vivo?
- o upstream está vivo?
- o problema é de proxy ou de aplicação?

---

## Riscos e cuidados

Os principais riscos desta camada incluem:

- proxy apontando para upstream incorreto
- edição manual não validada com `nginx -t`
- edge saudável mascarando backend parcialmente degradado
- interpretação errada de CORS como se fosse problema do Nginx
- drift entre hostname público, TLS e vhost ativo
- exposição indevida de superfície interna fora do proxy

Cuidados operacionais permanentes:

- testar sintaxe antes de reload/restart
- manter o edge alinhado ao upstream real
- preservar a separação entre responsabilidades do Nginx e da aplicação
- validar público e local sempre em conjunto
- tratar falha de proxy e falha de upstream como problemas distintos

---

## Limites deste documento

Este documento não detalha:

- arquivo completo de configuração do Nginx
- cada diretiva do vhost
- política detalhada de headers de segurança
- emissão e renovação de certificados
- troubleshooting completo do serviço
- modelagem funcional da Auth API

Esses tópicos devem ser tratados em documentos complementares ou impl-logs quando necessário.

---

## Critério de pronto deste documento

Este documento pode ser considerado maduro quando:

- o upstream local estiver confirmado sem ambiguidade
- a relação entre hostname público, Nginx e aplicação estiver reconciliada com o runtime real
- a distinção entre responsabilidades do edge e da aplicação estiver clara
- os comandos de validação refletirem a prática operacional real
- os problemas de proxy mais comuns estiverem representados de forma útil
- ele puder ser usado como referência do reverse proxy sem depender do master legado

---

## Última revisão

- Status: ativo
- Classificação: canônico
- Contexto: infraestrutura AWS Lightsail / Nginx reverse proxy
- Última revisão: 2026-03-18