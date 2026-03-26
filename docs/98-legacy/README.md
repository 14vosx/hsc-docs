# Legacy — README

## Objetivo

Esta pasta existe para concentrar o material histórico do ecossistema HSC de forma navegável, auditável e separada da documentação canônica viva.

Ela serve para:

- preservar documentos amplos e históricos
- evitar que arquivos legados continuem governando a navegação principal
- apoiar reconciliação documental quando necessário
- concentrar o acervo antigo para futura extração controlada para outro repositório
- manter memória histórica sem degradar a experiência de navegação no Obsidian

---

## Navegação rápida

### Entrada
- [Home da documentação](../README.md)
- [Governance](../00-governance/README.md)
- [Master Index](../00-governance/99-master-index.md)

### Documentos desta pasta
- [HSC MASTER DOCUMENTATION](./HSC_MASTER_DOCUMENTATION.md)
- [HSC Master Blueprint](./HSC_Master_Blueprint.md)
- [Master Documents Migration Map](./master-documents-migration-map.md)

### Contextos relacionados
- [Impl Log](../95-impl-log/README.md)
- [Audit](../97-audit/README.md)
- [Documentation System](../00-governance/documentation-system.md)

---

## Estado atual

O sistema documental HSC já opera com 5 contextos canônicos ativos:

1. Infra Hostinger
2. Game Panel
3. Portal Estático
4. Infra AWS Lightsail
5. Backoffice Admin

Isso muda o papel do legado.

Os documentos desta pasta continuam úteis para:

- memória arquitetural
- reconciliação histórica
- comparação entre modelo antigo e modelo novo
- auditoria de drift documental
- apoio em migrações e checkpoints

Mas eles não devem mais ser tratados, por padrão, como:

- ponto de entrada principal do vault
- índice oficial do repositório
- verdade canônica automática
- hub vivo de navegação operacional

---

## Regra canônica desta pasta

A regra oficial é:

**documento legado pode ser preservado, consultado e citado, mas não governa o sistema documental novo sem reconciliação explícita**

Ordem de prioridade quando houver dúvida:

1. ambiente real validado
2. documento canônico atual do contexto correto
3. impl-log relevante
4. auditoria relevante
5. documento legado útil

---

## Função desta pasta no Obsidian

No uso diário do Obsidian, esta pasta deve funcionar como:

- hub de acesso ao acervo histórico
- zona de isolamento do legado
- ponto de partida para consulta histórica
- base para futura remoção organizada do material antigo do repositório principal

Ela não deve funcionar como:

- Home alternativa
- índice paralelo ao sistema canônico
- atalho para pular os contextos atuais
- substituto da navegação por contexto

---

## Critério de concentração do legado

Devem permanecer ou ser movidos para esta pasta os documentos que se encaixem em um ou mais destes casos:

- master amplo antigo
- blueprint histórico
- documentação consolidada de fase anterior
- snapshot técnico útil apenas como apoio histórico
- documento ainda relevante, mas incompatível com o modelo canônico atual
- material que precisa ser preservado antes de futura migração para outro repositório

---

## Regra de uso correto

Ao encontrar informação relevante em um documento legado, o fluxo correto é:

1. identificar o contexto canônico atual dono do assunto
2. validar a informação no runtime real, quando necessário
3. migrar a parte útil para o contexto correto, se aplicável
4. registrar reconciliação em impl-log ou audit quando fizer sentido
5. manter o legado preservado como histórico, sem reativá-lo como fonte principal

---

## Critério de pronto desta pasta

Esta pasta estará saudável quando:

- o acervo histórico relevante estiver concentrado aqui
- os documentos legados estiverem fáceis de localizar no Obsidian
- a navegação principal não depender mais de masters antigos
- a futura migração do legado para outro repositório puder ser feita com baixo atrito

---

## Observação operacional

Este `README.md` funciona como porta de entrada do legado dentro do vault atual.

Ele também define a pasta `98-legacy/` como área oficial de concentração de documentação histórica até que o acervo seja movido para um repositório dedicado.
