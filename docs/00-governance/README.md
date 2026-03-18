# Governance — README

## Objetivo

Esta pasta concentra a governança do sistema documental do ecossistema HSC.

Ela existe para registrar, de forma estável e auditável:

- como a documentação do HSC é organizada
- quais são os contextos canônicos oficiais
- qual é o ponto de entrada recomendado do repositório
- como navegar a documentação sem depender dos antigos masters amplos
- quais regras governam criação, atualização e reconciliação dos documentos

---

## Estado atual

O sistema documental do HSC já saiu da fase de intenção e entrou em estruturação real.

Os **4 contextos canônicos iniciais já foram estruturados**:

1. Infra Hostinger
2. Game Panel
3. Portal Estático
4. Infra AWS Lightsail

Além deles, o contexto administrativo do produto já foi formalizado como novo contexto canônico:

5. Backoffice Admin

Isso significa que:

- a navegação principal do repositório já deve acontecer por contexto
- os antigos documentos amplos deixam de ser a navegação principal
- a documentação nova já possui base suficiente para crescer de forma incremental
- o Backoffice Admin já passa a existir como camada documental própria
- a próxima fase passa a ser manutenção, reconciliação fina, impl-log, auditoria e expansão controlada

---

## Regra canônica de navegação

A documentação do HSC agora deve ser lida e mantida por contexto.

A ordem recomendada de navegação é:

1. esta pasta `00-governance/`
2. `docs/00-governance/documentation-system.md`
3. `docs/00-governance/99-master-index.md`
4. o contexto técnico que você precisa operar, revisar ou atualizar

Regra importante:

- o índice mestre é o mapa do repositório
- o `documentation-system.md` é a regra do jogo
- os contextos são a documentação operacional real

---

## Contextos canônicos ativos

## `01-infra-hostinger/`

Contexto da infraestrutura base do lado Hostinger.

Cobre:

- Debian
- Nginx
- Docker host
- Certbot
- systemd
- filesystem, paths e permissões
- observabilidade da camada base

---

## `02-game-panel/`

Contexto operacional do servidor CS2.

Cobre:

- AMP
- instância `MixHAXIXE01`
- MatchZy
- plugins
- configuração do servidor
- runbooks do lado jogo
- troubleshooting do Game Panel

---

## `03-portal-estatico/`

Contexto do portal público e da Static API v2.

Cobre:

- ETL Bash
- Static API v2
- fonte SQLite do MatchZy
- contratos JSON
- frontend estático
- publishing e cache
- troubleshooting do portal

---

## `04-infra-aws-lightsail/`

Contexto da Auth API e da camada dinâmica.

Cobre:

- host Lightsail
- Node.js via systemd
- MariaDB local
- deploy/release/rollback
- operações da Auth API
- observabilidade e backup/restore

---

## `05-backoffice-admin/`

Contexto da SPA administrativa do ecossistema HSC.

Cobre:

- shell administrativo
- estrutura frontend Angular 20 + TypeScript + Signals
- autenticação administrativa no frontend
- guards e RBAC
- contratos administrativos com a Auth API
- runbooks funcionais do Backoffice
- superfícies administrativas de `seasons`, `news` e `events`

Regra importante:

- este contexto existe para a camada administrativa protegida
- ele não substitui o Portal público
- ele não absorve operação de host, ETL ou runtime profundo da Auth API

---

## O que mudou no modelo documental

O HSC deixa de depender de um modelo centrado em:

- um master grande
- documentos amplos tentando falar de tudo
- memória informal
- reconstruções periódicas pesadas

E passa a operar com um modelo centrado em:

- contexto
- documento canônico especializado
- inventário explícito
- runbook operacional
- troubleshooting por camada
- reconciliação contínua com o runtime real

Essa mudança existe para reduzir:

- perda de contexto
- drift documental
- duplicação confusa
- dependência de memória humana
- fragilidade de manutenção

---

## Documentos centrais desta pasta

## `docs/00-governance/README.md`

Este arquivo.

Papel:
- porta de entrada da governança documental
- orientação inicial de navegação
- resumo do estado da estrutura atual

---

## `docs/00-governance/documentation-system.md`

Papel:
- documento principal de regras do sistema documental
- convenções de escrita
- tipos de documento
- regras de manutenção
- tratamento de fatos confirmados, parcialmente confirmados e não confirmados

---

## `docs/00-governance/99-master-index.md`

Papel:
- índice mestre canônico do repositório
- mapa global dos contextos
- referência de navegação
- fotografia atual da estrutura documental

---

## Estado da migração

A migração documental atual pode ser lida assim:

- governança → ativa
- 4 contextos canônicos iniciais → estruturados
- contexto `05-backoffice-admin` → formalizado
- documentação antiga ampla → ainda útil como apoio histórico
- legado → deixa de governar o sistema documental
- próxima fase → manutenção incremental e expansão disciplinada

Em termos práticos:

- a base nova já existe
- o contexto administrativo já nasceu formalmente
- agora o objetivo é evitar voltar ao padrão antigo de acúmulo difuso

---

## O que ainda não é a fase principal

Apesar de os contextos principais já estarem estruturados, ainda não estamos, por padrão, na fase de:

- publicação web da documentação
- expansão indiscriminada de subcontextos
- criação massiva de ADRs sem necessidade real
- migração completa de todo o material legado sem critério

A prioridade agora é:

- consolidar o canônico criado
- revisar pendências reais
- registrar mudanças novas via impl-log
- auditar divergências quando elas aparecerem
- manter o índice e os contextos alinhados ao runtime

---

## Próxima fase recomendada do repositório

Com os contextos canônicos ativos já formalizados, a próxima fase natural do sistema documental é:

### 1. ativar o diretório `95-impl-log/`

Para registrar:
- mudanças incrementais
- ajustes reais de runtime
- pequenas evoluções sem poluir o canônico

### 2. ativar o diretório `97-audit/`

Para registrar:
- gaps
- reconciliações pendentes
- divergências entre docs e ambiente real

### 3. migrar material legado relevante para `98-legacy/`

Para:
- preservar histórico útil
- reduzir dependência dos antigos masters como navegação viva
- manter apoio histórico sem deixar o legado governar o presente

---

## Regras de leitura e manutenção

A leitura e a manutenção da documentação devem seguir estas regras:

- começar pela governança
- navegar pelo índice mestre
- entrar no contexto correto antes de alterar qualquer documento
- atualizar o documento canônico do tema antes de criar texto paralelo
- tratar o ambiente real validado como prioridade máxima
- explicitar lacunas em vez de escondê-las

Regra canônica:

**o objetivo do sistema documental não é parecer completo; é permanecer confiável**

---

## Relação com os antigos masters

Os antigos documentos amplos ainda podem existir como:

- fonte histórica
- apoio de reconciliação
- memória arquitetural antiga

Mas não devem mais ser tratados como:

- navegação principal
- índice oficial do ecossistema
- verdade canônica sem validação

Regra importante:

- documento antigo útil continua útil
- documento antigo útil não governa sozinho o sistema atual

---

## Critério de manutenção desta pasta

Esta pasta deve ser revisada quando houver:

- criação de novo contexto canônico
- mudança na estrutura do repositório documental
- mudança nas regras editoriais
- mudança na relação entre canônico, impl-log, audit e legacy
- necessidade de tornar explícita uma nova fase do sistema documental

---

## Critério de pronto desta pasta

A governança documental do HSC pode ser considerada saudável quando:

- qualquer pessoa consegue entender por onde começar
- os contextos canônicos ativos estão claramente posicionados
- o índice mestre está coerente com a estrutura real
- o sistema documental está explícito e utilizável
- o repositório consegue crescer sem voltar a depender de masters monolíticos

---

## Última revisão

- Status: ativo
- Classificação: canônico
- Contexto: governança / porta de entrada
- Última revisão: 2026-03-18