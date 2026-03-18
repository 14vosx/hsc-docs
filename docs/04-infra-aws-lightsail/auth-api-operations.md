# Auth API Operations

## Objetivo

Documentar a operação funcional da Auth API no contexto AWS Lightsail, registrando as superfícies relevantes, os invariantes operacionais e os fluxos essenciais de validação do backend dinâmico do ecossistema HSC.

Este documento existe para registrar, de forma estável e auditável:

- o papel operacional da Auth API no ecossistema
- as superfícies públicas e administrativas mais relevantes
- o modelo administrativo session-first com caminho break-glass
- os princípios de CORS e consumidores autorizados
- a obrigatoriedade de auditoria administrativa
- os smoke tests mínimos de operação
- as dependências cruzadas do runtime

---

## Escopo

Este documento cobre:

- o papel operacional da Auth API
- endpoints públicos operacionais
- endpoints administrativos operacionais
- autenticação administrativa baseada em sessão
- caminho break-glass administrativo
- regras operacionais de CORS
- auditoria administrativa
- invariantes de operação
- smoke tests e operações recorrentes

Este documento não cobre em profundidade:

- configuração detalhada do Nginx
- unit file do serviço
- fluxo completo de deploy e rollback
- backup e restore
- modelagem completa de domínio
- backlog funcional futuro

Esses assuntos vivem em documentos próprios do contexto.

---

## Estado atual

A Auth API opera como backend dinâmico central do HSC no contexto AWS Lightsail.

O estado operacional conhecido desta camada inclui:

- execução em Node.js via systemd
- exposição pública por Nginx com TLS e reverse proxy
- persistência em MariaDB local
- superfícies públicas de conteúdo
- superfícies administrativas protegidas
- modelo administrativo session-first
- caminho break-glass por chave administrativa
- trilha de auditoria para mutações administrativas relevantes
- política de CORS orientada por allowlist explícita de origens

A Auth API sustenta a metade dinâmica do ecossistema HSC, enquanto a Hostinger sustenta jogo, ETL e portal estático.

---

## Source of truth / evidências

As principais evidências deste documento, nesta fase de migração documental, são:

- documentação consolidada atual do ecossistema HSC
- blueprint técnico consolidado do HSC
- manual operacional da Auth API em AWS Lightsail
- impl-logs de CORS, sessões administrativas, schema snapshot e fail-closed
- documentação operacional de deploy e release por TAG

Enquanto a migração canônica não estiver concluída, essas fontes seguem sendo usadas para reconciliação do estado operacional real.

---

## Relações com outros documentos

Este arquivo é complementar a:

- `docs/04-infra-aws-lightsail/README.md`
- `docs/04-infra-aws-lightsail/architecture-runtime.md`
- `docs/04-infra-aws-lightsail/node-systemd.md`
- `docs/04-infra-aws-lightsail/mariadb-local.md`
- `docs/04-infra-aws-lightsail/deploy-release-rollback.md`
- `docs/04-infra-aws-lightsail/observability-troubleshooting.md`
- `docs/95-impl-log/`

Este documento descreve a operação funcional da Auth API.  
Ele não substitui a documentação de runtime do host, banco, edge ou deploy.

---

## Papel operacional da Auth API

A Auth API é a camada dinâmica do HSC responsável por:

- expor saúde operacional do backend
- servir conteúdo dinâmico e administrável
- sustentar autenticação administrativa
- persistir sessões
- registrar auditoria administrativa
- suportar superfícies administrativas de conteúdo
- preservar separação entre portal estático e backend dinâmico

Do ponto de vista do ecossistema, a Auth API é o backend central das superfícies que não pertencem ao portal puramente estático.

---

## Endpoints públicos operacionais

As superfícies públicas operacionais conhecidas incluem:

### Health

- `/health`

Função:
- validar disponibilidade do runtime
- expor resposta básica de saúde da aplicação
- ajudar a separar falhas entre edge, app e banco

Observação operacional:
- o contexto também admite health local via `127.0.0.1:3000/health`

---

### Conteúdo News

Endpoints públicos ligados a conteúdo de News são parte do escopo funcional conhecido da Auth API.

Exemplo de superfície pública relevante:

- `/content/news`

Função:
- expor conteúdo público gerenciado na camada dinâmica
- servir como origem para consumo externo e, em alguns fluxos, para espelhamento no lado Hostinger

---

### Conteúdo Seasons

Superfícies públicas ligadas a Seasons fazem parte do estado operacional conhecido da Auth API.

Exemplos de superfícies públicas relevantes:

- `/content/seasons`
- `/content/seasons/active`
- `/content/seasons/:slug`

Função:
- expor catálogo ou estado ativo de temporadas
- sustentar consumo público ou integração com outras camadas do ecossistema

---

### Estruturas futuras ou incrementais

O backend já prevê base estrutural para outras superfícies dinâmicas, como Events, mas este documento não deve assumir maturidade funcional completa sem validação real da release ativa.

Regra editorial:
- documentar apenas o que já está presente no estado operacional reconciliado

---

## Endpoints administrativos operacionais

As superfícies administrativas conhecidas da Auth API incluem operações ligadas a:

- conteúdo News
- conteúdo Seasons
- autenticação administrativa
- verificação de identidade/sessão administrativa
- inspeções técnicas ou administrativas restritas

A nomenclatura exata de cada rota administrativa deve ser mantida alinhada com o runtime real e, quando necessário, com impl-logs e documentos específicos do domínio.

Pontos operacionais importantes:

- superfícies administrativas não são parte do portal estático
- mutações administrativas devem respeitar autenticação e auditoria
- uma rota administrativa disponível não implica que ela possa ser usada sem sessão ou sem trilha adequada

---

## Session-first + break-glass

O modelo administrativo atual é session-first com caminho break-glass.

### Session-first

O modelo prioritário de operação administrativa é baseado em sessão persistida.

Isso significa:

- o fluxo normal administrativo deve privilegiar autenticação por sessão
- a aplicação deve reconhecer contexto autenticado persistido
- superfícies administrativas devem operar sob identidade autenticada rastreável
- o uso administrativo normal não deve depender exclusivamente de cabeçalho estático

Esse modelo favorece:

- melhor rastreabilidade
- menor dependência de segredo bruto por request
- maior aderência a operação administrativa real

---

### Break-glass administrativo

O contexto mantém um caminho break-glass baseado em chave administrativa para contingência e operação controlada.

Características esperadas:

- uso excepcional, não rotina
- escopo administrativo restrito
- necessidade de cuidado operacional
- relevância alta para troubleshooting, acesso de emergência e operações de continuidade

Regra operacional:
- break-glass existe como fallback controlado
- não deve substituir o fluxo normal session-first

---

## Fluxos de autenticação administrativa

Os fluxos conhecidos do contexto incluem:

- início de autenticação administrativa
- emissão e validação de magic link, quando aplicável
- persistência de sessão
- leitura de identidade administrativa corrente
- proteção de rotas administrativas
- fallback break-glass para contingência controlada

A evolução desses fluxos já envolveu:

- persistência de sessões
- uso de tabelas ligadas a `magic_links`
- uso de tabela `sessions`
- fortalecimento de guardas administrativas

---

## CORS e consumers autorizados

A política operacional conhecida de CORS da Auth API é orientada por allowlist explícita.

Implicações:

- nem toda origem deve consumir a API
- consumidores legítimos precisam estar explicitamente autorizados
- a resposta operacional da aplicação pode refletir as origens permitidas
- o controle de CORS pertence prioritariamente ao runtime da aplicação, e não apenas ao edge

Consumidores autorizados podem incluir:

- superfícies oficiais do ecossistema HSC
- portal e domínios/subdomínios operacionais aprovados
- interfaces administrativas autorizadas
- camadas futuras de backoffice ou account, quando formalizadas

Regra operacional:
- alteração de CORS é mudança relevante
- mudanças de allowlist devem ser registradas de forma rastreável

---

## Auditoria administrativa

A auditoria administrativa é componente central do contexto.

O estado operacional conhecido já indica:

- existência de trilha administrativa persistida
- tabela operacional `admin_audit_log`
- endurecimento do princípio de auditoria em mutações sensíveis

Princípio operacional:

**mutações administrativas relevantes devem gerar rastreabilidade adequada**

Essa trilha é importante para:

- investigação
- conformidade interna
- reconstrução de ações
- análise de incidentes
- validação de comportamento administrativo

---

## Invariante fail-closed para mutações

O contexto já evoluiu para postura fail-closed em mutações administrativas sensíveis.

Interpretação operacional:

- não basta a rota estar exposta
- não basta haver autenticação parcial
- não basta haver intenção de escrita

Se a mutação administrativa não puder cumprir os pré-requisitos de segurança e rastreabilidade esperados, o comportamento saudável do sistema é negar a operação.

Em termos práticos, isso significa:

- autenticação, autorização e auditoria precisam estar coerentes
- o backend não deve aceitar mutações críticas em estado operacional ambíguo
- operação “sem audit, sem write” é a referência de endurecimento conhecida deste contexto

---

## Invariantes operacionais

Os invariantes operacionais conhecidos da Auth API incluem:

### 1. A aplicação deve responder health local e público

Sem isso, a camada dinâmica do contexto não pode ser considerada saudável.

### 2. Rotas públicas devem respeitar o contrato ativo da release

Especialmente em superfícies como:

- News
- Seasons

### 3. Rotas administrativas devem operar sob guarda adequada

Isso inclui:

- sessão válida ou caminho break-glass controlado
- políticas de autorização coerentes
- rastreabilidade mínima necessária

### 4. Mutações sensíveis devem preservar auditoria

Sem isso, o contexto entra em estado operacional inadequado.

### 5. A aplicação deve conseguir acessar o MariaDB local

Sem persistência funcional, parte importante das superfícies dinâmicas degrada ou falha.

### 6. CORS deve refletir apenas consumidores autorizados

Origem indevida não deve ser aceita silenciosamente.

---

## Smoke tests

Depois de deploy, rollback ou mudança relevante no contexto, os smoke tests mínimos devem incluir:

### Health local

```bash
curl -sS http://127.0.0.1:3000/health
```

### Health público

Substitua pelo domínio público vigente do contexto.

```bash
curl -sS https://SEU_DOMINIO/health
```

### Conteúdo público principal

```bash
curl -sS https://SEU_DOMINIO/content/news
curl -sS https://SEU_DOMINIO/content/seasons
curl -sS https://SEU_DOMINIO/content/seasons/active
```

### Logs do serviço

```bash
sudo journalctl -u hsc-auth-api -n 100 --no-pager
```

Quando a mudança afetar superfícies administrativas, também é necessário validar o fluxo administrativo correspondente de forma controlada.

---

## Operações recorrentes

As operações recorrentes mais relevantes deste contexto incluem:

- validar health da aplicação
- validar serviço `hsc-auth-api`
- validar comportamento de rotas públicas críticas
- validar comportamento administrativo após release
- conferir logs após mudança operacional
- confirmar integridade de sessão administrativa
- verificar se a política de CORS continua coerente
- investigar trilha de auditoria após mutações relevantes

A Auth API é uma camada dinâmica; por isso, seu estado operacional não deve ser inferido apenas por processo ativo.  
É necessário validar também comportamento funcional.

---

## Dependências cruzadas

Este contexto depende, em nível operacional, de:

- Nginx funcional
- serviço `hsc-auth-api` funcional
- MariaDB local funcional
- working directory íntegro
- `.env` íntegro
- política de CORS coerente com consumidores atuais
- superfícies administrativas preservando guarda e auditoria
- disciplina de release por TAG
- troubleshooting orientado por logs e smoke tests

Também há dependências arquiteturais indiretas com outros contextos:

### Relação com Portal Estático

O portal ou seus espelhos podem depender de conteúdo público exposto pela Auth API.

### Relação com governança documental

Mudanças nesta camada devem refletir:
- impl-log, quando aplicável
- atualização do documento canônico correspondente
- atualização do índice mestre, se a topologia global mudar

---

## Sinais de saúde do contexto

Os principais sinais de saúde esperados são:

- health local respondendo
- health público respondendo
- serviço `hsc-auth-api` ativo
- logs sem erro crítico recorrente
- MariaDB local acessível pela aplicação
- rotas públicas críticas respondendo corretamente
- superfícies administrativas protegidas funcionando como esperado
- comportamento coerente de sessão e auditoria

---

## Problemas comuns

### 1. Health responde, mas rotas de conteúdo falham

Causas comuns:

- falha de banco
- regressão de release
- schema incompatível
- erro específico do módulo de conteúdo

Impacto:
- contexto aparentemente vivo, mas funcionalmente degradado

---

### 2. Sessão administrativa não persiste

Causas comuns:

- regressão em lógica de sessão
- configuração inconsistente
- problema de banco
- quebra em fluxo de autenticação

Impacto:
- dificuldade ou impossibilidade de operar superfícies administrativas pelo fluxo normal

---

### 3. Break-glass funciona, mas fluxo normal falha

Causas comuns:

- problema em sessão
- problema em autenticação por magic link
- problema em persistência de sessão
- regressão de guarda administrativa

Impacto:
- operação administrativa degradada, ainda que exista contingência

---

### 4. Mutações administrativas são negadas indevidamente

Causas comuns:

- regra fail-closed atuando sobre contexto inconsistente
- auditoria indisponível
- identidade administrativa não reconhecida
- erro de autorização

Impacto:
- proteção ativa, mas necessidade de investigação operacional

---

### 5. CORS quebra consumidores legítimos

Causas comuns:

- allowlist incompleta
- mudança de domínio/subdomínio sem atualização da configuração
- regressão de leitura do `.env`

Impacto:
- frontends legítimos deixam de consumir a API corretamente

---

## Limites deste documento

Este documento não detalha:

- todos os contratos HTTP campo a campo
- configuração completa do Nginx
- fluxo detalhado de deploy
- estrutura detalhada do banco
- backup e restore
- design completo de domínio de News, Seasons ou Events

Esses detalhes devem viver em documentos específicos ou impl-logs próprios.

---

## Critério de pronto deste documento

Este documento pode ser considerado maduro quando:

- as superfícies públicas e administrativas mais relevantes estiverem reconciliadas com a release ativa
- o modelo session-first + break-glass estiver formalizado sem ambiguidade
- a exigência de auditoria administrativa estiver clara
- a política de CORS estiver descrita em nível operacional
- os smoke tests cobrirem corretamente o núcleo funcional da Auth API
- ele puder ser usado como referência de operação funcional sem depender do master legado

---

## Última revisão

- Status: ativo
- Classificação: canônico
- Contexto: infraestrutura AWS Lightsail / operação funcional da Auth API
- Última revisão: 2026-03-18