# MariaDB Local

## Objetivo

Documentar o papel, a topologia operacional e os limites do MariaDB local utilizado pela Auth API no contexto AWS Lightsail.

Este documento existe para registrar, de forma estável e auditável:

- como o banco local participa do runtime da Auth API
- onde ele se posiciona na topologia do host
- como a aplicação se conecta a ele
- quais entidades e superfícies operacionais dependem dele
- quais riscos e cuidados existem no uso de um banco local bound em loopback
- como snapshots de schema e verificações estruturais devem ser tratados no sistema documental

---

## Escopo

Este documento cobre:

- o papel do MariaDB no contexto AWS Lightsail
- bind e exposição de rede do banco
- database operacional da Auth API
- relação entre aplicação e banco local
- entidades e tabelas operacionais relevantes
- política de acesso
- riscos de drift de schema
- relação entre schema snapshot, impl-log e operação

Este documento não cobre em profundidade:

- backup e restore detalhados
- dump completo de DDL
- modelagem funcional completa do domínio
- troubleshooting geral do host
- fluxo de deploy e rollback da aplicação

Esses assuntos vivem em documentos próprios do contexto.

---

## Estado atual

O estado operacional conhecido do banco deste contexto é:

- engine: MariaDB
- engine de armazenamento observada: InnoDB
- modo de uso: banco local da Auth API
- host de conexão da aplicação: `127.0.0.1`
- database principal conhecida: `hsc_auth`
- exposição externa: não prevista como parte do desenho canônico
- papel arquitetural: persistência local da camada dinâmica da Auth API

O desenho atual favorece simplicidade operacional e menor superfície de exposição, mantendo a aplicação e o banco no mesmo host Lightsail.

---

## Source of truth / evidências

As principais evidências deste documento, nesta fase de migração documental, são:

- manual operacional da Auth API em AWS Lightsail
- documentação consolidada do ecossistema HSC
- impl-logs ligados a schema snapshot, sessões administrativas e hardening
- observações reconciliadas sobre bind local, database `hsc_auth` e superfícies operacionais dependentes do banco

Enquanto a migração canônica do contexto não estiver concluída, essas fontes continuam servindo como base de reconciliação.

---

## Relações com outros documentos

Este arquivo é complementar a:

- `docs/04-infra-aws-lightsail/README.md`
- `docs/04-infra-aws-lightsail/architecture-runtime.md`
- `docs/04-infra-aws-lightsail/node-systemd.md`
- `docs/04-infra-aws-lightsail/deploy-release-rollback.md`
- `docs/04-infra-aws-lightsail/backup-restore.md`
- `docs/04-infra-aws-lightsail/auth-api-operations.md`
- `docs/04-infra-aws-lightsail/observability-troubleshooting.md`
- `docs/95-impl-log/`

Este documento descreve o banco local como componente de runtime.  
Ele não substitui snapshots técnicos, impl-logs estruturais nem runbooks de backup.

---

## Papel do MariaDB neste contexto

O MariaDB local é a camada oficial de persistência da Auth API no contexto AWS Lightsail.

Seu papel é sustentar:

- autenticação e fluxos administrativos
- sessões persistidas
- auditoria administrativa
- conteúdo administrável da Auth API
- entidades de suporte ao domínio já em produção
- evolução incremental de superfícies administrativas e de conteúdo

Em termos arquiteturais, o banco local sustenta a metade dinâmica do HSC, em contraste com a camada Hostinger, que concentra jogo, ETL e portal estático.

---

## Topologia do banco no host

O desenho canônico conhecido deste contexto é:

- aplicação e banco rodam no mesmo host Lightsail
- a aplicação acessa o banco por loopback
- o banco não é parte da superfície pública do contexto
- o Nginx não se comunica com o banco diretamente
- a aplicação Node.js é a única consumidora operacional principal do banco neste contexto

Representação lógica resumida:

- internet
- Nginx
- Auth API
- MariaDB local em `127.0.0.1`

Esse desenho reduz exposição de rede e simplifica a administração do contexto.

---

## Bind e exposição de rede

O estado operacional esperado para o MariaDB deste contexto é:

- acesso da aplicação por `127.0.0.1`
- banco não tratado como serviço público do host
- uso local restrito ao contexto da Auth API

Implicações operacionais:

- falhas de rede externa não afetam diretamente a comunicação app → banco
- o risco principal está em disponibilidade local do serviço MariaDB, credenciais, schema e integridade do runtime
- qualquer abertura indevida de bind externo deve ser tratada como desvio arquitetural

Regra canônica:
- o banco local não deve ser promovido a endpoint de rede pública sem decisão arquitetural explícita

---

## Database principal

A database principal conhecida deste contexto é:

- `hsc_auth`

Esse nome deve ser tratado como referência canônica até revisão explícita.

A Auth API depende desse banco para persistir dados estruturais do runtime administrativo e de conteúdo.

---

## Relação com a Auth API

A Auth API é a consumidora operacional principal do MariaDB local.

A relação conhecida entre aplicação e banco inclui:

- leitura e escrita de dados de autenticação
- persistência de sessões
- persistência de auditoria administrativa
- persistência de conteúdo administrável
- leitura e escrita de entidades operacionais do domínio já disponíveis no backend

Essa relação implica que:

- o serviço da aplicação depende do banco para parte relevante de suas superfícies
- a indisponibilidade do banco tende a degradar ou impedir rotas administrativas e partes das rotas públicas dinâmicas
- o health do serviço pode continuar respondendo em alguns cenários, mas o contexto pode estar funcionalmente degradado

---

## Entidades e tabelas operacionais relevantes

As entidades e tabelas abaixo já aparecem como relevantes no estado conhecido do contexto.

### `users`

Tabela estrutural de identidade do sistema.

Papel operacional conhecido:

- base de usuários do ecossistema dinâmico
- suporte a dados administrativos e de autenticação
- expansão incremental conforme novas superfícies foram adicionadas

---

### `magic_links`

Tabela ligada ao fluxo de autenticação por magic link.

Papel operacional conhecido:

- persistir emissão e uso de links de autenticação
- sustentar fluxo autenticado e controles temporais associados

---

### `sessions`

Tabela ligada à persistência de sessão.

Papel operacional conhecido:

- sustentar modelo session-first
- registrar sessões administrativas ou autenticadas
- participar do controle de superfícies protegidas

---

### `admin_audit_log`

Tabela ligada à auditoria administrativa.

Papel operacional conhecido:

- registrar ações administrativas relevantes
- apoiar rastreabilidade de mutações
- sustentar o princípio operacional de auditoria obrigatória

Esse ponto é especialmente importante porque o contexto já evoluiu para uma postura fail-closed em mutações administrativas sensíveis.

---

### Tabelas de conteúdo e domínio

O contexto também sustenta superfícies de conteúdo, especialmente relacionadas a:

- News
- Seasons
- estruturas futuras ou incrementais como Events

A documentação canônica deste arquivo não deve presumir o desenho completo de todas essas tabelas sem validação real do schema atual.  
Quando necessário, o detalhamento estrutural deve ser referenciado via impl-log ou snapshot técnico específico.

---

## Observações sobre schema conhecido

O schema conhecido deste contexto foi parcialmente consolidado a partir de snapshots e impl-logs estruturais.

Isso significa:

- existe evidência concreta de tabelas ligadas a autenticação, sessão e auditoria
- existe histórico de expansão do schema
- ainda pode haver lacunas entre documentação legada e estado atual validado do banco
- o documento canônico deve descrever apenas o que já foi reconciliado com confiança suficiente

Regra editorial:
- quando houver incerteza sobre tabela, coluna ou constraint, documentar como “pendente de validação” em vez de inventar completude

---

## Política de acesso

A política operacional esperada deste contexto é:

- acesso principal ao banco feito pela Auth API
- acesso administrativo direto apenas para manutenção, validação, snapshot ou troubleshooting controlado
- nenhuma dependência canônica de acesso remoto público ao MariaDB
- credenciais de acesso mantidas fora da documentação canônica

Acesso manual ao banco deve ser tratado como operação de manutenção, não como fluxo primário do sistema.

---

## Segurança operacional

Os principais cuidados de segurança deste contexto são:

- manter o banco bound em loopback
- não versionar credenciais
- não replicar conteúdo sensível do `.env` nos documentos
- evitar exposição de dumps completos sem necessidade
- tratar snapshots técnicos com classificação adequada
- preservar consistência entre app, banco e fluxo de auditoria

Também é importante observar:

- rollback de aplicação não implica rollback de banco
- mudança de schema exige disciplina documental própria
- uma mutação administrativa sem trilha de auditoria aceitável não deve ser tratada como comportamento saudável do sistema

---

## Snapshot de schema e auditoria estrutural

O HSC já possui evidência de uso de snapshots estruturais do banco como instrumento de verificação técnica.

No sistema documental novo, a regra é:

- snapshot de schema não vive como documento canônico principal deste contexto
- snapshot deve ficar registrado como impl-log técnico ou documento de apoio auditável
- este documento canônico deve apontar o papel desses snapshots, sem virar dump permanente de DDL

Finalidade dos snapshots:

- validar estado real do schema
- detectar drift entre documentação e banco
- registrar expansão estrutural importante
- apoiar troubleshooting e auditoria técnica

---

## Drift de schema

Os principais riscos de drift de schema neste contexto são:

- documentação descrever entidades que não existem mais
- runtime depender de tabelas ou colunas não documentadas
- mudança de banco entrar em produção sem impl-log correspondente
- rollback de código assumir schema antigo sem compatibilidade
- snapshots ficarem desatualizados sem revisão cruzada

Mitigações recomendadas:

- registrar mudanças estruturais relevantes em `95-impl-log/`
- usar snapshots técnicos em marcos importantes
- reconciliar documentos canônicos após mudanças reais no banco
- não presumir schema completo com base apenas em memória ou blueprint antigo

---

## Impacto operacional de indisponibilidade

Se o MariaDB local estiver indisponível, os impactos esperados incluem:

- degradação de rotas públicas dinâmicas dependentes de dados persistidos
- falha de rotas administrativas
- falha em autenticação e/ou sessão
- falha em auditoria administrativa
- erros de bootstrap ou de runtime na aplicação, dependendo do ponto de dependência

A severidade do impacto depende do quanto a aplicação exige banco já no bootstrap e do escopo funcional da release ativa.

---

## Sinais de saúde do banco no contexto

Os sinais de saúde esperados deste componente incluem:

- serviço MariaDB ativo no host
- aplicação conseguindo conectar via `127.0.0.1`
- ausência de erro recorrente de conexão nos logs da Auth API
- endpoints dependentes de persistência respondendo corretamente
- comportamento coerente entre login, sessão, conteúdo e superfícies administrativas

Saúde do banco não deve ser inferida apenas pela existência do processo.  
A relação app → banco continua sendo parte do diagnóstico real.

---

## Comandos de validação

Os comandos abaixo representam verificações operacionais típicas do contexto.

### Verificar status do serviço MariaDB

```bash
sudo systemctl status mariadb
```

### Verificar se a aplicação consegue subir sem erro persistente de banco

```bash
sudo journalctl -u hsc-auth-api -n 100 --no-pager
```

### Verificar health local da aplicação

```bash
curl -sS http://127.0.0.1:3000/health
```

### Acessar o banco local manualmente, quando necessário

O método exato de autenticação depende da configuração real do host.  
A forma abaixo é apenas representativa do acesso local administrativo:

```bash
mysql -h 127.0.0.1 -u SEU_USUARIO -p hsc_auth
```

Regra importante:
- não registrar credenciais reais na documentação canônica

---

## Problemas comuns

### 1. Aplicação não conecta ao banco

Causas comuns:

- MariaDB parado
- credenciais incorretas no `.env`
- database esperada ausente
- bind ou configuração local incorreta

Impacto:
- falha em rotas dependentes de persistência
- possível crash no bootstrap, dependendo da release

---

### 2. Banco ativo, mas schema incompatível

Causas comuns:

- release nova espera tabela ou coluna inexistente
- rollback de aplicação encontra schema mais novo
- drift entre documentação e banco real

Impacto:
- erros de runtime
- rotas específicas quebradas
- comportamento inconsistente entre módulos

---

### 3. Auditoria administrativa deixa de persistir

Causas comuns:

- falha em `admin_audit_log`
- schema incompleto
- erro de permissão ou escrita
- mudança parcial em mutações administrativas

Impacto:
- quebra do princípio operacional de rastreabilidade
- necessidade de investigação imediata

---

### 4. Snapshot técnico não reflete o banco real

Causas comuns:

- snapshot antigo
- extração feita fora do momento correto
- mudança estrutural sem atualização documental

Impacto:
- documentação persuasiva, mas incorreta
- troubleshooting mais difícil

---

## Limites deste documento

Este documento não detalha:

- dumps completos de DDL
- estrutura coluna a coluna de todas as tabelas
- credenciais de acesso
- backup/restore detalhado
- tuning avançado do MariaDB
- troubleshooting detalhado do serviço do banco

Esses tópicos devem ser tratados em documentos específicos, impl-logs ou rotinas operacionais apropriadas.

---

## Critério de pronto deste documento

Este documento pode ser considerado maduro quando:

- o nome da database e o bind local estiverem confirmados no runtime real
- a lista de entidades operacionais relevantes estiver reconciliada com o schema vigente
- a relação entre Auth API e banco estiver clara
- os riscos de drift estiverem explicitados
- snapshots e mudanças estruturais relevantes estiverem sendo tratados fora do canônico principal, de forma organizada
- ele puder ser lido como visão operacional do banco local sem depender do master legado

---

## Última revisão

- Status: ativo
- Classificação: canônico
- Contexto: infraestrutura AWS Lightsail / MariaDB local
- Última revisão: 2026-03-18