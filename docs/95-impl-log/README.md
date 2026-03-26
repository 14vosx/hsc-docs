# Impl Log — README

## Objetivo

Esta pasta existe para concentrar o histórico incremental de implementação do ecossistema HSC em formato navegável, auditável e cronológico.

Ela serve para:

- registrar checkpoints de evolução técnica
- preservar contexto de decisões e cutovers
- manter trilha histórica de mudanças relevantes
- apoiar auditoria e reconciliação documental
- evitar que logs de implementação fiquem espalhados fora de um índice local claro

---

## Navegação rápida

### Entrada
- [Home da documentação](../README.md)
- [Governance](../00-governance/README.md)
- [Master Index](../00-governance/99-master-index.md)

### Documentos desta pasta
- [2026-03-18 — Initial Canonical Context Migration](./2026-03-18-initial-canonical-context-migration.md)
- [2026-03-19 — Auth Admin Magic Link and SQL Migrations Cutover](./2026-03-19-auth-admin-magic-link-and-sql-migrations-cutover.md)
- [2026-03-24 — Local Auth Users Management Checkpoint](./2026-03-24-local-auth-users-management-checkpoint.md)

### Contextos relacionados
- [Governance](../00-governance/README.md)
- [Audit](../97-audit/README.md)
- [Legacy](../98-legacy/README.md)

---

## Regra de leitura

Este contexto não substitui a documentação canônica por contexto.

A leitura correta é:

1. começar pela Home ou pelo contexto canônico principal
2. usar os impl logs como trilha histórica de execução, migração e checkpoint
3. voltar para os documentos canônicos quando a dúvida for estrutural, operacional ou arquitetural

---

## Organização

Os arquivos desta pasta devem permanecer preferencialmente:

- datados no nome
- orientados a checkpoint real
- focados em implementação, migração, correção ou cutover
- escritos de forma que favoreça leitura cronológica no Obsidian

Este `README.md` funciona como porta de entrada e índice local do contexto.
