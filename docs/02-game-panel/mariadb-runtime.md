# MariaDB Runtime

## Navegação rápida

- [Home da documentação](../README.md)
- [Game Panel](./README.md)
- [Master Index](../00-governance/99-master-index.md)

---
## Objetivo

Documentar, de forma honesta e explícita, o estado atual da camada MariaDB no contexto Game Panel do ecossistema HSC.

Este documento existe para registrar:

- o que já está confirmado sobre persistência SQL no lado jogo
- o que ainda não está confirmado como runtime canônico
- a diferença entre a persistência principal já reconciliada e integrações auxiliares ainda pendentes
- por que este tema não deve ser tratado com suposição

---

## Escopo

Este documento cobre:

- o estado atual da camada MariaDB no lado game
- a distinção entre SQLite confirmado e MariaDB/MySQL ainda não consolidado como camada central
- o papel de integrações auxiliares que possam usar banco relacional externo
- o que deve e o que não deve ser afirmado como verdade canônica neste momento

Este documento não cobre em profundidade:

- Auth API no Lightsail
- MariaDB local da camada dinâmica
- pipeline ETL do Portal Estático
- configuração detalhada de WeaponPaints
- troubleshooting detalhado de banco externo
- credenciais, hosts ou strings de conexão sensíveis

Esses tópicos vivem em outros contextos ou dependem de validação adicional.

---

## Estado atual

O estado atual reconciliado do lado jogo é:

- a persistência estatística principal e confirmada do contexto Game Panel é **SQLite**
- o artefato estrutural central já confirmado é o **`matchzy.db`**
- o ecossistema público do portal depende dessa persistência SQLite, e não de MariaDB, para ranking, matches, maps e players
- não há, nesta fase documental, confirmação suficiente para tratar um **MariaDB local do lado Game Panel** como camada central e canônica do runtime

Em outras palavras:

- **SQLite do MatchZy = confirmado e estrutural**
- **MariaDB no Game Panel = ainda não consolidado como runtime central canônico**

---

## O que está confirmado

## 1. Persistência estatística principal do lado jogo

O que está confirmado como verdade operacional do contexto é:

- o MatchZy produz persistência estatística local
- essa persistência vive em `matchzy.db`
- as tabelas principais já reconciliadas são:
  - `matchzy_stats_matches`
  - `matchzy_stats_maps`
  - `matchzy_stats_players`
- o Portal Estático consome essa base via ETL Bash

Conclusão canônica:

**o banco estrutural principal do lado jogo, hoje, é SQLite e não MariaDB**

---

## 2. A cadeia pública de dados não depende de MariaDB

Também está confirmado que a cadeia pública que alimenta a Static API v2 depende da persistência do MatchZy em SQLite.

Isso significa que:

- o ranking público não depende de MariaDB do lado game
- a listagem pública de partidas não depende de MariaDB do lado game
- os agregados por mapa não dependem de MariaDB do lado game
- os dossiês públicos de jogador não dependem de MariaDB do lado game

Conclusão canônica:

**MariaDB não é, hoje, a fonte principal da camada pública de stats**

---

## O que está parcialmente evidenciado, mas ainda não consolidado

Existe evidência histórica de uso de banco relacional compatível com MySQL/MariaDB em integrações auxiliares do lado jogo, especialmente em cenários ligados a plugins.

O caso mais relevante já aparecido no ecossistema é:

- integração auxiliar associada ao **WeaponPaints**
- uso de banco remoto compatível com MySQL/MariaDB
- evidência de conexão externa fora do host principal do Game Panel

Isso sugere que:

- pode existir uso de banco relacional no lado game
- esse uso pode ser relevante para plugins auxiliares
- mas esse uso **não está reconciliado como camada central do runtime do Game Panel**

Conclusão editorial:

**há evidência de uso auxiliar de banco compatível com MariaDB/MySQL, mas não evidência suficiente para promovê-lo, neste momento, a componente canônico central do contexto**

---

## O que não está confirmado

Os itens abaixo **não devem ser tratados como verdade canônica** neste momento:

### 1. MariaDB local dentro da VPS Hostinger como parte obrigatória do Game Panel

Não está confirmado, nesta fase documental, que o lado jogo dependa de:

- serviço local `mariadb` na Hostinger
- database local dedicada ao runtime do servidor CS2
- schema local MariaDB como base principal do lado game

---

### 2. MariaDB como dependência estrutural do MatchZy

Não está confirmado que:

- o MatchZy dependa de MariaDB para sua persistência principal
- a operação competitiva do servidor dependa de MariaDB
- a cadeia de stats pública derive de MariaDB

O que está confirmado, ao contrário, é a dependência do `matchzy.db`.

---

### 3. Modelo canônico de banco auxiliar do lado game

Ainda não está confirmado, de forma suficiente para canônico, qual é exatamente o desenho atual das integrações auxiliares relacionais no lado jogo:

- se existe apenas banco externo
- se existe algum banco local auxiliar
- quais plugins realmente dependem dele hoje
- quais schemas estão ativos
- qual parte disso é produção real versus experimento antigo

---

## Leitura canônica atual

A leitura canônica atual deste contexto deve ser:

### Camada confirmada e central
- SQLite do MatchZy
- `matchzy.db`
- tabelas estatísticas do MatchZy
- dependência direta do Portal Estático nessa base

### Camada auxiliar potencial, ainda não consolidada
- integrações compatíveis com MySQL/MariaDB para plugins específicos
- possível uso externo/remoto de banco relacional
- persistência auxiliar fora da cadeia estatística principal

---

## Implicação prática para a documentação

Enquanto não houver validação adicional do ambiente real, este documento estabelece a seguinte regra:

**não documentar MariaDB do Game Panel como se fosse pilar estrutural do contexto**

Em vez disso, documentar assim:

- a persistência estrutural principal do lado jogo é SQLite
- integrações MariaDB/MySQL podem existir como camada auxiliar de plugins
- a camada MariaDB do Game Panel permanece em estado de reconciliação

Isso evita drift documental e falsa confiança.

---

## Quando este documento deve ser expandido

Este documento deve ser revisado e expandido se alguma das condições abaixo for confirmada diretamente no ambiente real:

### 1. existência de serviço MariaDB local no lado Hostinger/Game Panel

Exemplos de evidência útil:

- `systemctl status mariadb`
- database local claramente associada ao lado game
- schema ativo de plugin do servidor dentro da VPS

### 2. plugin auxiliar com dependência canônica de MariaDB/MySQL

Exemplos de evidência útil:

- plugin em produção que depende de banco relacional
- configuração ativa e estável
- banco usado no dia a dia operacional do servidor

### 3. relevância operacional recorrente dessa camada

Exemplos de evidência útil:

- troubleshooting real recorrente dessa persistência
- dependência real para admins/players
- impacto concreto dessa camada no funcionamento do servidor

Se essas evidências forem consolidadas, este arquivo pode deixar de ser “estado atual e limites” e virar documentação plena de runtime MariaDB do lado game.

---

## Validações úteis para próxima revisão

As verificações abaixo ajudam a decidir se este documento precisa evoluir no futuro.

### Validar se existe serviço local MariaDB no host

```bash
sudo systemctl status mariadb
```

### Procurar evidência de uso de banco relacional por plugins

A validação exata depende do runtime real e dos arquivos efetivamente ativos no ambiente.

### Validar se o lado game continua dependendo estruturalmente apenas do `matchzy.db`

Substitua pelo path real reconciliado do ambiente.

```bash
ls -l /CAMINHO/DO/matchzy.db
sqlite3 /CAMINHO/DO/matchzy.db ".tables"
```

Regra importante:

- enquanto a prova estrutural continuar apontando para `matchzy.db` como base principal, SQLite permanece sendo a verdade canônica do contexto

---

## Riscos de documentar errado esta camada

Os principais riscos de documentar MariaDB incorretamente no contexto Game Panel incluem:

- confundir persistência auxiliar com persistência estrutural
- induzir troubleshooting para o banco errado
- criar falsa dependência arquitetural no contexto
- poluir os runbooks do lado jogo com um componente que pode nem ser central
- aumentar drift entre documentação e runtime real

Por isso, este documento deliberadamente prefere:

- explicitar a lacuna
- separar confirmado de não confirmado
- evitar “completar” a arquitetura com suposição

---

## Estado editorial deste documento

Este arquivo deve ser lido como:

- documento canônico válido
- documento curto por design
- documento de contenção de ambiguidade
- documento que protege a qualidade da arquitetura documental

Ele existe para dizer com clareza:

**hoje, o lado jogo do HSC é estruturalmente SQLite-first; MariaDB no contexto Game Panel ainda não está consolidado como camada central**

---

## Critério de pronto deste documento

Este documento pode ser considerado adequado neste estágio quando:

- ele deixar claro que SQLite é a persistência principal confirmada
- ele não inventar MariaDB local como componente central sem prova
- ele registrar honestamente a possibilidade de integrações auxiliares relacionais
- ele reduzir ambiguidade para futuras revisões

---

## Última revisão

- Status: ativo
- Classificação: canônico
- Contexto: game panel / MariaDB runtime
- Última revisão: 2026-03-18