# Backoffice Admin — README

## Objetivo

Documentar o contexto canônico do Backoffice Admin do ecossistema HSC, incluindo sua arquitetura administrativa, sua relação com a Auth API e os limites operacionais desta camada.

Este contexto existe para registrar, de forma estável e auditável:

- o papel do Backoffice dentro do ecossistema HSC
- a separação entre Backoffice administrativo e Portal público
- a relação entre SPA administrativa, Auth API e domínios administrativos
- os princípios de autenticação, autorização e mutação administrativa
- os limites, responsabilidades e dependências desta camada

---

## Escopo

Este contexto cobre exclusivamente a camada de Backoffice administrativo do HSC.

Estão dentro do escopo deste contexto:

- a SPA administrativa do HSC
- shell administrativo
- layout, rotas e navegação admin
- autenticação administrativa no frontend
- guards e RBAC
- superfícies administrativas para `news`, `seasons` e `events`
- contratos de integração com a Auth API
- runbooks e troubleshooting ligados ao Backoffice como produto administrativo

Este contexto não cobre, em profundidade:

- o Portal público estático
- a Static API v2
- a VPS Hostinger como infraestrutura base
- o runtime detalhado da Auth API no AWS Lightsail
- o Game Panel / AMP / servidor CS2
- ETL Bash e publicação de JSONs públicos
- operação detalhada de banco de dados
- credenciais e arquivos sensíveis

Esses tópicos vivem em seus próprios contextos canônicos.

---

## Estado atual

O estado conhecido deste contexto, nesta fase, é:

- o Backoffice Admin já existe como necessidade arquitetural explícita do ecossistema
- o contexto canônico `05-backoffice-admin` está sendo formalizado agora
- a Auth API já sustenta superfícies administrativas relevantes
- o Portal público permanece separado e não deve absorver responsabilidades administrativas
- a separação de camadas futura já foi prevista no legado reconciliado
- o desenho deve respeitar o modelo session-first com fallback break-glass controlado
- mutações administrativas sensíveis devem permanecer alinhadas à postura fail-closed da Auth API
- `news` e `seasons` já possuem base funcional/documental mais madura do que `events`
- `events` já aparece como domínio administrativo previsto, mas ainda pede formalização incremental mais cuidadosa

Regra importante:

- este contexto nasce para organizar e sustentar o produto administrativo do HSC
- ele não nasce como camada de operação de host ou como extensão improvisada do Portal

---

## O que é o Backoffice Admin do HSC

O Backoffice Admin do HSC é a camada administrativa do ecossistema.

Seu papel é:

- oferecer superfície administrativa protegida para staff e administradores
- consumir a Auth API para operações administrativas
- centralizar gestão de domínios administrativos do projeto
- refletir permissões reais do backend
- oferecer experiência de administração consistente, previsível e de baixo acoplamento

Em termos arquiteturais, o Backoffice é uma SPA administrativa separada do Portal público e dependente da Auth API para leitura administrativa, mutações e validação de autorização.

---

## O que o Backoffice administra

O Backoffice Admin é a superfície de gestão para domínios administrativos do ecossistema.

Neste estágio, os domínios prioritários do contexto são:

- `news`
- `seasons`
- `events`

Leitura inicial recomendada dos domínios:

### News

Domínio editorial administrativo já previsto com maior maturidade.

Fluxos esperados:
- listar
- criar
- editar
- publicar
- despublicar
- excluir

### Seasons

Domínio competitivo / estrutural já formalizado no legado reconciliado.

Fluxos esperados:
- listar
- criar
- editar
- ativar
- fechar

### Events

Domínio administrativo previsto para agenda e organização de eventos.

Fluxos administrativos esperados:
- listar
- criar
- editar
- excluir ou cancelar

Regra importante:
- a gestão administrativa de `events` pertence ao Backoffice
- a eventual confirmação de presença por usuário final não pertence ao Backoffice como fluxo principal de UX administrativa

---

## O que o Backoffice não faz

O Backoffice Admin não é responsável por:

- renderização pública do portal
- servir páginas públicas para usuários anônimos
- substituir a Static API v2
- operar ETL Bash
- publicar JSONs estáticos
- operar diretamente o host, Nginx, systemd ou banco
- substituir os runbooks da Auth API
- assumir autoridade de segurança acima do backend
- decidir autorização apenas no frontend

Esses papéis pertencem a outros contextos do ecossistema.

---

## Relação com a Auth API

A Auth API é a camada dinâmica central consumida pelo Backoffice.

A relação entre Backoffice e Auth API é:

- o Backoffice consome superfícies administrativas
- a Auth API é a autoridade final para autenticação e autorização
- o frontend reflete permissões; ele não define permissões
- mutações administrativas relevantes passam pela Auth API
- o comportamento de auditoria, sessão e fail-closed é herdado da camada dinâmica

Princípios operacionais dessa relação:

- o frontend deve ser session-first
- o backend continua sendo source of truth para RBAC
- o fallback break-glass existe como contingência operacional, não como UX principal
- erros de autorização devem ser tratados com clareza
- a UI não deve assumir sucesso sem resposta explícita do backend

---

## Relação com o Portal público

O Portal público e o Backoffice Admin são contextos separados.

A relação correta entre eles é:

- o Portal continua sendo camada pública estática
- o Backoffice continua sendo camada administrativa protegida
- mutações administrativas não pertencem ao Portal
- consumo público de conteúdo não deve depender do Backoffice
- o Backoffice não substitui a experiência pública nem a camada estática

Essa separação existe para:

- reduzir acoplamento
- simplificar a superfície pública
- manter segurança administrativa sob fronteira própria
- permitir evolução independente entre produto público e produto administrativo

---

## Princípios do contexto

Os princípios centrais deste contexto são:

- separação explícita entre público e administrativo
- integração por contrato com a Auth API
- session-first no fluxo administrativo
- break-glass apenas como contingência controlada
- UI subordinada à autorização do backend
- evolução incremental por domínio
- MVP antes de sofisticação
- baixo acoplamento entre features administrativas
- documentação canônica viva desde o início

Regra importante:

- o Backoffice deve nascer como produto administrativo disciplinado
- ele não deve nascer como coleção solta de telas CRUD

---

## Documentos canônicos deste contexto

Os documentos canônicos previstos para este contexto são:

- `docs/05-backoffice-admin/README.md`
- `docs/05-backoffice-admin/architecture-runtime.md`
- `docs/05-backoffice-admin/frontend-structure.md`
- `docs/05-backoffice-admin/auth-rbac-and-guards.md`
- `docs/05-backoffice-admin/admin-api-contracts.md`
- `docs/05-backoffice-admin/operational-runbooks.md`
- `docs/05-backoffice-admin/references-inventory.md`

Este `README.md` funciona como porta de entrada e índice local do contexto.

---

## Relação com outros contextos

Este contexto se relaciona diretamente com:

### `docs/00-governance/`

Relação estrutural obrigatória.

A governança documental define:
- como o contexto deve existir
- como seus documentos devem ser mantidos
- como novas superfícies devem ser reconciliadas com o runtime real

---

### `docs/03-portal-estatico/`

Relação forte por separação de produto.

O Portal público:
- continua sendo camada pública
- não absorve responsabilidades administrativas
- pode consumir outputs de domínios administrados em outra camada, mas não substitui o Backoffice

---

### `docs/04-infra-aws-lightsail/`

Relação estrutural crítica.

A Infra AWS Lightsail sustenta:
- a Auth API
- o modelo session-first
- o fallback break-glass
- a postura fail-closed para mutações administrativas sensíveis
- o runtime real consumido pelo Backoffice

---

### `docs/98-legacy/`

Relação de reconciliação histórica.

O legado segue sendo fonte importante para:
- roadmap de Backoffice
- separação de camadas futura
- RBAC inicial
- superfícies administrativas previstas
- prioridades de domínio

O legado não governa este contexto, mas sustenta sua reconciliação inicial.

---

## Validação de fronteira do contexto

A revisão de fronteira do contexto `05-backoffice-admin` confirma que este contexto:

- **não cobre operação de host**
- **não cobre runtime profundo da Auth API**
- **não cobre Portal público estático**
- **não cobre ETL / publicação de conteúdo público**
- **não cobre fluxos públicos de usuário final**

### Leitura correta da fronteira

O contexto `05-backoffice-admin` cobre exclusivamente:

- a SPA administrativa do HSC
- sua estrutura frontend
- sua camada de autenticação e guards no frontend
- seus contratos administrativos com a Auth API
- seus domínios administrativos prioritários
- seus runbooks funcionais como produto administrativo

Dependências externas continuam documentadas em seus contextos próprios.

---

## Ordem de leitura recomendada deste contexto

A ordem recomendada de leitura é:

1. `docs/05-backoffice-admin/README.md`
2. `docs/05-backoffice-admin/architecture-runtime.md`
3. `docs/05-backoffice-admin/frontend-structure.md`
4. `docs/05-backoffice-admin/auth-rbac-and-guards.md`
5. `docs/05-backoffice-admin/admin-api-contracts.md`
6. `docs/05-backoffice-admin/operational-runbooks.md`
7. `docs/05-backoffice-admin/references-inventory.md`

---

## Próxima fase natural do contexto

A próxima fase natural deste contexto é:

- consolidar a arquitetura mínima da SPA administrativa
- formalizar rotas, shell, guards e RBAC
- reconciliar contratos administrativos com a Auth API
- iniciar backlog incremental por domínio
- começar por `seasons`, seguir por `news` e depois `events`

---

## Resumo executivo

O contexto `05-backoffice-admin` existe para registrar a camada administrativa do HSC como produto próprio.

Sua função é:

- separar formalmente o administrativo do público
- documentar o Backoffice como SPA administrativa
- ancorar sua integração na Auth API
- sustentar a evolução de `news`, `seasons` e `events`
- manter a camada administrativa previsível, auditável e coerente com o restante do ecossistema
