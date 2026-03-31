# HSC — Estado Atual Oficial da Arquitetura Pública

## Status do documento

- **Tipo:** documento canônico de estado atual
- **Escopo:** superfícies públicas, entrypoints operacionais e arquitetura exposta ao usuário
- **Status:** ativo
- **Atualizado em:** 2026-03-28
- **Owner:** HSC
- **Relacionados:**
  - `docs/00-governance/99-master-index.md`
  - `docs/03-portal-estatico/brand-hub-root-product-and-surface-decisions.md`
  - `docs/03-portal-estatico/brand-hub-root-publishing-and-cutover-runtime.md`
  - `docs/03-portal-estatico/brand-hub-root-frontend-implementation-runtime.md`
  - `docs/02-game-panel/game-panel-public-access-and-ops-entrypoint.md`

---

## 1. Objetivo

Este documento consolida o **estado atual oficial da arquitetura pública do HSC** após a separação entre:

- marca pública
- produto público
- operação administrativa

A função deste documento é registrar, de forma canônica, **quais superfícies existem**, **qual o papel de cada uma** e **como elas se relacionam** no runtime atual.

---

## 2. Princípio central da arquitetura pública

A arquitetura pública atual do HSC segue a regra:

**marca no apex, produto no portal, operação no ops**

Em termos práticos:

- **apex** entrega marca, narrativa e entrada
- **portal** entrega produto público operacional
- **ops** entrega administração e operação
- **api/content** entregam suporte público desacoplado

---

## 3. Superfícies oficiais

### 3.1 Apex público

**URL:** `https://haxixesmokeclub.com`

**Função oficial:** Brand Hub Root

**Papel:**
- hub principal da marca
- porta de entrada pública do ecossistema
- camada institucional, cultural e narrativa
- ponte de entrada para o Portal CS2

**Não é mais:**
- endpoint do AMP
- endpoint do Game Panel
- fallback operacional do painel
- simples redirect para o portal

---

### 3.2 Portal público

**URL:** `https://haxixesmokeclub.com/portal/cs2/`

**Função oficial:** Portal CS2

**Papel:**
- superfície pública operacional do HSC no contexto atual
- leitura de ranking
- leitura de partidas
- leitura de jogadores
- presença pública viva do produto

**Características:**
- orientado a leitura e navegação funcional
- subordinado à camada de marca no apex
- não substitui o papel institucional do root

---

### 3.3 API pública

**URL base:** `https://haxixesmokeclub.com/api/cs2/v2/`

**Função oficial:** API pública estática JSON v2

**Papel:**
- sustentar o Portal CS2
- fornecer dados públicos desacoplados
- servir endpoints estáticos para ranking, matches, health e player

---

### 3.4 Content mirror público

**URL base:** `https://haxixesmokeclub.com/content/news/`

**Função oficial:** content mirror público

**Papel:**
- servir conteúdo público de forma resiliente
- reduzir dependência direta da camada administrativa
- manter leitura pública de news e conteúdo institucional

---

### 3.5 Entry point operacional do AMP

**URL:** `https://ops.haxixesmokeclub.com`

**Função oficial:** acesso operacional ao AMP / Game Panel

**Papel:**
- restaurar o acesso administrativo ao painel
- separar operação do domínio público principal
- evitar conflito entre hub da marca e painel operacional

**Observação:**
esta superfície existe porque o apex deixou de servir o AMP após a publicação do Brand Hub Root.

---

### 3.6 Backoffice administrativo

**URL:** `https://backoffice.haxixesmokeclub.com`

**Função oficial:** superfície administrativa de conteúdo e gestão interna

**Estado:**
- separado da experiência pública
- não é protagonista da arquitetura pública principal

---

### 3.7 Área futura de conta

**URL:** `https://account.haxixesmokeclub.com`

**Função oficial:** futura área do usuário logado

**Estado:**
- reservado
- ainda não materializado como superfície viva

---

## 4. Modelo de separação por camada

### 4.1 Camada de marca

**Superfície:** apex

**Responsabilidade:**
- posicionamento
- manifesto
- narrativa
- entrada
- identidade

**Natureza:**
- editorial
- institucional
- cultural

---

### 4.2 Camada de produto público

**Superfície:** `/portal/cs2/`

**Responsabilidade:**
- leitura operacional do universo CS2
- navegação funcional
- exposição pública da operação atual

**Natureza:**
- funcional
- informacional
- operacional

---

### 4.3 Camada de suporte público

**Superfícies:**
- `/api/cs2/v2/`
- `/content/news/`

**Responsabilidade:**
- fornecer dados e conteúdo públicos desacoplados
- sustentar leitura e resiliência do portal e da camada editorial

---

### 4.4 Camada operacional

**Superfície:** `ops.haxixesmokeclub.com`

**Responsabilidade:**
- acesso ao AMP
- administração de instâncias
- operação de infraestrutura de jogo

**Natureza:**
- administrativa
- não pública no sentido narrativo
- crítica operacionalmente

---

## 5. Runtime atual do Brand Hub Root

### 5.1 Publicação

O root público da marca é publicado em modelo versionado com releases em filesystem.

**Estrutura:**
- releases em `/var/www/brand-hub/releases/...`
- symlink ativo em `/var/www/brand-hub/current`

### 5.2 Estratégia de publicação

O modelo atual de cutover do root é:

1. criar nova release
2. validar conteúdo
3. trocar symlink
4. validar smoke test
5. executar rollback simples se necessário

### 5.3 Consequência arquitetural

O root deixou de ser compatível com o fallback anterior do AMP no `location /`, pois passou a servir explicitamente o hub da marca.

---

## 6. Runtime atual do Portal CS2

O Portal CS2 continua sendo servido como aplicação estática via Nginx.

**Origem principal:**
- `/var/www/portal/cs2/`

**Rota pública:**
- `/portal/cs2/`

**Papel atual:**
- prova viva da camada pública operacional do HSC

---

## 7. Runtime atual da API pública

A API pública JSON v2 continua sendo servida a partir de estrutura estática.

**Origem principal:**
- `/var/www/api/cs2/v2/`

**Rota pública:**
- `/api/cs2/v2/`

---

## 8. Runtime atual do content mirror

O content mirror público continua ativo sob:

- `/content/news/`

**Função:**
- desacoplar leitura pública da gestão administrativa
- sustentar resiliência e previsibilidade de consumo

---

## 9. Runtime atual do AMP / Game Panel

### 9.1 Situação anterior

Antes da mudança do root, o AMP era acessível pelo domínio principal porque o fallback do `location /` fazia reverse proxy para o painel.

### 9.2 Situação atual

Após a implantação do Brand Hub Root, o apex deixou de servir o AMP.

Para restaurar o acesso operacional, foi criado o entrypoint:

- `https://ops.haxixesmokeclub.com`

### 9.3 Implementação

O host `ops.haxixesmokeclub.com` foi configurado para responder como reverse proxy do AMP, apontando para:

- `http://127.0.0.1:8080`

### 9.4 Resultado

A operação administrativa foi restaurada sem reverter o root da marca e sem recolocar o painel no apex.

---

## 10. Situação do botão “Gerenciar painel” da Hostinger

### 10.1 Estado atual

O botão interno da Hostinger pode continuar abrindo a lógica antiga associada ao domínio histórico do painel.

### 10.2 Impacto real

Isso deixou de ser bloqueio crítico porque o acesso ao AMP já foi restabelecido por:

- `https://ops.haxixesmokeclub.com`

### 10.3 Prioridade

Ajustar o comportamento interno desse botão é desejável, mas não crítico no runtime atual.

---

## 11. Identidade oficial da camada pública

### 11.1 Logo de sistema

**Logo oficial:** `HSC Master Digital Logo v1`

### 11.2 Regra de uso

- **navbar do hub:** mono light
- **hero do hub:** primary accent
- **footer do hub:** mono light
- **portal:** mesma identidade com peso mais funcional
- **ops/AMP:** sem obrigação de aderência visual ao hub; o ponto principal é o entrypoint

### 11.3 Direção visual consolidada

- dark premium
- accent cyan-blue
- marca antes de plataforma
- portal como prova viva
- operação fora do apex

---

## 12. O que está resolvido

### Resolvido
- apex deixou de redirecionar seco para o portal
- root passou a operar como hub da marca
- logo digital oficial foi definido
- operação do AMP deixou de depender do apex
- acesso ao AMP foi restaurado via `ops`
- a arquitetura pública ficou separada por função

---

## 13. O que permanece em aberto

### Em aberto, mas não bloqueante
- refinamento visual da homepage v4
- redução da sensação de “app padrão/dashboard” no root
- ajuste interno do botão da Hostinger
- definição futura da superfície `account`
- expansão futura de superfícies editoriais e comunitárias

---

## 14. Regra canônica para decisões futuras

Toda evolução pública do HSC deve respeitar este arranjo:

### Root
vende marca, cultura, manifesto e entrada

### Portal
entrega produto público operacional

### Ops
entrega acesso administrativo e operação

### API / Content
sustentam leitura pública desacoplada

---

## 15. Resumo executivo

O HSC agora possui separação formal entre:

- **marca pública**
- **produto público**
- **operação administrativa**

Esse arranjo reduz ambiguidade de superfície, preserva o papel do apex como hub da marca e mantém a operação acessível sem contaminar a experiência pública principal.