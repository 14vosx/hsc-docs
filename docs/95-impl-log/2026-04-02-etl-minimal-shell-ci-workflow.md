# 2026-04-02 — ETL Minimal Shell CI Workflow

## Navegação rápida

- [Home da documentação](../README.md)
- [Impl Log](./README.md)
- [Master Index](../00-governance/99-master-index.md)

### Contextos e documentos relacionados
- [Portal Estático](../03-portal-estatico/README.md)
- [ETL Bash Pipeline](../03-portal-estatico/etl-bash-pipeline.md)
- [ETL Repository — Minimal Shell CI](../03-portal-estatico/etl-repository-minimal-shell-ci.md)
- [Portal Estático References Inventory](../03-portal-estatico/portal-estatico-references-inventory.md)

---
## Objetivo

Registrar, de forma estável e auditável, o checkpoint técnico em que o repositório `hsc-cs2-etl` recebeu sua CI mínima para scripts shell.

Este impl-log existe para preservar a trilha incremental desta frente sem substituir os documentos canônicos do contexto `03-portal-estatico`.

---

## Escopo

Este documento cobre:

- inspeção real do repositório ETL antes da implementação
- decisão de escopo do gate mínimo
- criação do workflow `.github/workflows/ci-shell.yml`
- validação local mínima correspondente
- falha inicial da Action remota e ajuste de severidade do `shellcheck`
- branch, PR e merge da entrega

Este documento não cobre em profundidade:

- runtime da VPS Hostinger
- materialização do runtime ETL
- contratos JSON da v2
- SQL detalhado do portal
- backlog completo das próximas histórias do épico ETL Hardening/Ops

---

## Estado inicial do checkpoint

No início desta frente, o repositório `hsc-cs2-etl` apresentava o seguinte quadro:

- não existia `.github/`
- não existia `.github/workflows/`
- a camada shell versionada principal estava distribuída entre `bin/` e `scripts/`
- também existiam shells em `legacy/` e `_reconcile/`, mas fora da malha principal da entrega
- o repositório não possuía gate automático mínimo para sintaxe shell ou lint básico

A pergunta central desta frente era:

- como introduzir uma CI mínima e conservadora no ETL sem expandir escopo para runtime real, refactor ou automação excessiva?

---

## Source of truth / evidências

As evidências diretas deste impl-log são:

- clone local real do repositório `hsc-cs2-etl`
- inventário dos scripts em `bin/`, `scripts/`, `legacy/` e `_reconcile/`
- arquivo `.github/workflows/ci-shell.yml`
- branch `chore/f1-ci-minima`
- PR `#2`
- merge do PR `#2` na `main`
- runs remotos do workflow `CI Shell`
- saída local de `bash -n` e `shellcheck`

Commits relevantes da frente:

- `chore(ci): add minimal shell workflow`
- `chore(ci): ignore shellcheck info level`

---

## Relações com outros documentos

Este impl-log deve ser reconciliado principalmente com:

- `docs/03-portal-estatico/README.md`
- `docs/03-portal-estatico/etl-bash-pipeline.md`
- `docs/03-portal-estatico/etl-repository-minimal-shell-ci.md`
- `docs/03-portal-estatico/portal-estatico-references-inventory.md`
- `docs/95-impl-log/README.md`

---

## Linha incremental do checkpoint

## 1. Inspeção real do repositório

A primeira fase foi de inspeção objetiva do repositório antes de qualquer proposta de workflow.

Constatações confirmadas:

- `bin/` existia e concentrava a maior parte dos scripts `gen-*`
- `scripts/` existia e continha `materialize-etl-runtime.sh`
- `legacy/steam-profile.sh` existia, mas fora do núcleo atual da entrega
- `_reconcile/2026-04-01/vps-runtime/raw/usr/local/bin/*.sh` preservava a baseline importada do host
- `.github/workflows/` não existia

Resultado:

- o gate mínimo pôde ser desenhado com base em estrutura confirmada, e não em suposição

---

## 2. Fechamento do escopo mínimo

A segunda fase consolidou a leitura correta da story `[F1] / [ETL 01] — CI mínima`.

Decisões fechadas:

- cobrir apenas `bin/*.sh`
- cobrir apenas `scripts/*.sh`
- deixar `legacy/` fora do gate
- deixar `_reconcile/` fora do gate
- usar um único workflow
- usar um único job Linux
- rodar `bash -n`
- rodar `shellcheck`
- manter `shfmt -d` fora de escopo

Resultado:

- a frente ficou protegida contra ampliação indevida de cobertura e complexidade

---

## 3. Criação do workflow mínimo

Foi criado o arquivo:

- `.github/workflows/ci-shell.yml`

Leitura operacional do workflow inicial:

- trigger em `push`
- trigger em `pull_request`
- checkout do repositório
- instalação de `shellcheck`
- coleta explícita dos scripts-alvo
- falha se a coleta vier vazia
- execução de `bash -n`
- execução de `shellcheck`

Resultado:

- o repositório ETL passou a ter seu primeiro gate automático mínimo de shell

---

## 4. Validação local mínima

Antes do push, a frente executou uma validação local mínima coerente com o workflow:

- conferência da branch de trabalho
- coleta dos scripts-alvo
- validação positiva de `bash -n`
- validação positiva de `shellcheck` nos arquivos reais
- prova controlada de falha de `shellcheck` em script temporário

Também houve correção de processo:

- a alteração não foi commitada diretamente na `main`
- a branch `chore/f1-ci-minima` foi criada antes do commit final

Resultado:

- o risco de subir uma Action quebrada por erro trivial foi reduzido antes do `push`

---

## 5. Primeira execução remota e falha observada

Após o primeiro `push`, a GitHub Action disparou corretamente.

Leitura importante:

- o trigger em branch funcionou
- checkout e instalação de `shellcheck` passaram
- a falha ocorreu no step `Validate shell scripts`

Erro observado:

- `SC2317 (info)` em `bin/gen-match-details-incremental.sh`

Trecho relevante da leitura técnica:

- o gate inicial estava bloqueando mensagens `info` do `shellcheck`
- isso extrapolava o escopo de **lint básico** definido para a frente

Resultado:

- ficou claro que a correção mínima deveria ocorrer no workflow, e não nos scripts do runtime ETL

---

## 6. Ajuste canônico de severidade

A correção aplicada foi simples e intencional:

- trocar `shellcheck "$file"`
- por `shellcheck -S warning "$file"`

Leitura canônica desse ajuste:

- `warning` e `error` continuam bloqueando
- `info` deixa de bloquear a CI desta fase
- o escopo da story permanece mínimo e conservador
- o runtime ETL não precisou ser alterado para concluir a entrega

Resultado:

- o workflow passou a refletir corretamente o objetivo de “lint básico” da `[F1]`

---

## 7. Push final, PR e merge

Após o ajuste de severidade:

- um novo commit foi criado
- a branch `chore/f1-ci-minima` recebeu `push`
- a Action remota passou com sucesso
- o PR `#2` foi aberto
- o PR `#2` foi mergeado na `main`

Resultado final do checkpoint:

- a CI mínima do ETL ficou ativa na branch principal do repositório

---

## Resultado consolidado da frente

Ao final deste checkpoint, o repositório `hsc-cs2-etl` passou a ter:

- workflow mínimo de GitHub Actions
- trigger em `push` e `pull_request`
- validação de sintaxe shell com `bash -n`
- lint básico com `shellcheck -S warning`
- cobertura explícita apenas de `bin/*.sh` e `scripts/*.sh`
- branch, PR e merge rastreáveis para a entrega

Leitura correta:

- a frente `[F1] / [ETL 01]` foi concluída
- o critério de pronto foi atingido
- o repositório ETL agora possui um gate mínimo real contra regressões shell triviais
