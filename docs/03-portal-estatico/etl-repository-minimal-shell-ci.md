# ETL Repository — Minimal Shell CI

## Navegação rápida

- [Home da documentação](../README.md)
- [Portal Estático](./README.md)
- [Master Index](../00-governance/99-master-index.md)

### Documentos adjacentes
- [ETL Bash Pipeline](./etl-bash-pipeline.md)
- [ETL Runtime Materialization Runbook](./etl-runtime-materialization-runbook.md)
- [Portal Estático References Inventory](./portal-estatico-references-inventory.md)
- [2026-04-02 — ETL Minimal Shell CI Workflow](../95-impl-log/2026-04-02-etl-minimal-shell-ci-workflow.md)

---
## Objetivo

Documentar a camada mínima de CI formalizada no repositório `hsc-cs2-etl` para reduzir regressões bobas em scripts shell que sustentam o runtime ETL do contexto Portal Estático.

Este documento existe para registrar, de forma estável e auditável:

- qual é o workflow mínimo atualmente ativo no repositório ETL
- quais arquivos entram no gate de CI
- quais verificações são obrigatórias nesta fase
- quais verificações foram conscientemente deixadas fora de escopo
- como interpretar a política atual de severidade do `shellcheck`
- como reproduzir localmente a validação mínima antes de abrir PR

---

## Escopo

Este documento cobre:

- workflow `.github/workflows/ci-shell.yml`
- triggers em `push` e `pull_request`
- coleta explícita de `bin/*.sh` e `scripts/*.sh`
- checagem de sintaxe com `bash -n`
- lint básico com `shellcheck -S warning`
- validação local mínima correspondente ao workflow
- leitura correta do que a CI protege nesta fase

Este documento não cobre em profundidade:

- execução real do ETL contra a VPS Hostinger
- smoke de runtime com `systemd`
- refactor dos scripts do ETL
- gates adicionais como `shfmt`
- cobertura de `legacy/` ou `_reconcile/`
- matriz de jobs ou automação avançada de release

---

## Estado atual

O estado reconciliado do repositório ETL, após a frente `[F1] / [ETL 01] — CI mínima`, é:

- o repositório `14vosx/hsc-cs2-etl` possui um workflow mínimo de GitHub Actions
- o workflow ativo vive em:
  - `.github/workflows/ci-shell.yml`
- esse workflow dispara em:
  - `push`
  - `pull_request`
- o gate atual varre apenas scripts diretamente em:
  - `bin/*.sh`
  - `scripts/*.sh`
- a CI falha se a coleta vier vazia
- a CI falha se `bash -n` detectar erro de sintaxe
- a CI falha se `shellcheck -S warning` detectar `warning` ou `error`
- mensagens `info` do `shellcheck` não bloqueiam o pipeline nesta fase
- `legacy/` e `_reconcile/` ficaram explicitamente fora da cobertura
- `shfmt -d` permaneceu fora de escopo na entrega atual

Leitura canônica:

- esta CI não substitui validação de runtime
- esta CI funciona como gate mínimo de regressão estrutural do repositório ETL
- a política atual privilegia simplicidade e previsibilidade sobre cobertura maximalista

---

## Source of truth / evidências

As evidências diretas desta frente são:

- repositório `14vosx/hsc-cs2-etl`
- arquivo `.github/workflows/ci-shell.yml`
- branch de trabalho `chore/f1-ci-minima`
- PR `#2` mergeado na `main`
- execução remota validada do workflow `CI Shell`
- validação local mínima de `bash -n` e `shellcheck`
- impl-log `2026-04-02-etl-minimal-shell-ci-workflow.md`

Enquanto o repositório permanecer com esse workflow e esse contrato de severidade, essas evidências prevalecem como source of truth do gate mínimo do ETL.

---

## Relações com outros documentos

Este arquivo é complementar a:

- `docs/03-portal-estatico/README.md`
- `docs/03-portal-estatico/etl-bash-pipeline.md`
- `docs/03-portal-estatico/etl-runtime-materialization-runbook.md`
- `docs/03-portal-estatico/portal-estatico-references-inventory.md`
- `docs/03-portal-estatico/portal-estatico-operational-runbooks.md`
- `docs/01-infra-hostinger/systemd-automation.md`
- `docs/95-impl-log/2026-04-02-etl-minimal-shell-ci-workflow.md`

Este documento descreve o gate mínimo de repositório do ETL.
Ele não substitui o runtime canônico do host nem os runbooks operacionais da VPS.

---

## Repositório-alvo e fronteira do gate

A leitura correta desta frente é:

- o alvo da CI é o repositório `hsc-cs2-etl`
- o gate existe para validar a camada versionada antes do merge
- o gate não precisa reproduzir o runtime completo da Hostinger para ser útil nesta fase

Fronteira atual do gate:

- **inclui** `bin/*.sh`
- **inclui** `scripts/*.sh`
- **não inclui** `legacy/*.sh`
- **não inclui** `_reconcile/**/*.sh`

Motivo operacional dessa fronteira:

- `bin/` é a fonte versionada principal do runtime ETL
- `scripts/` contém utilitários versionados relevantes para materialização e manutenção
- `legacy/` não representa a malha ativa do ETL
- `_reconcile/` preserva evidência importada do host, e não deve governar a CI desta fase

---

## Workflow canônico atual

O workflow ativo da frente é:

- nome: `CI Shell`
- path: `.github/workflows/ci-shell.yml`
- job único: `shell`
- runner: `ubuntu-latest`

Sequência canônica do workflow:

1. `actions/checkout@v4`
2. instalar `shellcheck`
3. coletar scripts em `bin/` e `scripts/` com `find ... -maxdepth 1 -type f -name '*.sh' | sort`
4. falhar se a coleta vier vazia
5. executar `bash -n` em cada arquivo coletado
6. executar `shellcheck -S warning` em cada arquivo coletado

Leitura canônica:

- a solução atual é deliberadamente mínima
- não existe cache, matrix ou step cosmético nesta fase
- o gate foi desenhado para ser claro, conservador e fácil de auditar

---

## Política de severidade do ShellCheck

Durante a primeira execução remota do workflow, a pipeline falhou por um `SC2317 (info)` em `bin/gen-match-details-incremental.sh`.

Leitura correta desse incidente:

- a falha remota mostrou que o `shellcheck` puro estava bloqueando também mensagens `info`
- a frente `[F1]` foi definida como **lint básico**, não como política máxima de estilo
- o ajuste canônico foi mover a execução para:

```bash
shellcheck -S warning "$file"
```

Consequência operacional:

- `error` e `warning` continuam bloqueando o pipeline
- `info` deixa de bloquear nesta fase
- não foi necessário alterar os scripts do ETL para concluir a entrega mínima

Regra importante:

- esse ajuste não deve ser lido como relaxamento geral de qualidade
- ele deve ser lido como alinhamento entre severidade do gate e escopo real da story `[F1]`

---

## Validação local mínima correspondente

A validação local mínima usada para fechar a frente foi, em essência:

1. garantir branch de trabalho dedicada
2. instalar `shellcheck` localmente quando necessário
3. coletar os mesmos arquivos do workflow
4. rodar `bash -n` nesses arquivos
5. rodar `shellcheck` nesses arquivos
6. provar falha controlada de `shellcheck` em arquivo temporário
7. commitar e dar `push` para disparar a Action remota

Leitura correta:

- a validação local não substitui a Action remota
- ela reduz chance de subir workflow quebrado ou alteração regressiva óbvia
- a Action remota continua sendo o gate final do repositório

---

## O que esta CI protege hoje

No estado atual, esta CI protege principalmente contra:

- erro bobo de sintaxe shell
- regressão estrutural óbvia em scripts versionados do ETL
- introdução acidental de lint relevante em `bin/` ou `scripts/`
- merges sem gate mínimo sobre a camada shell do repositório

Ela ainda não protege, por si só, contra:

- drift de comportamento no runtime da VPS
- quebra de integração com `systemd`
- problema de permissão no host
- regressão de dados contra `matchzy.db`
- inconsistência de JSON publicado

---

## Fora de escopo confirmado nesta entrega

Ficou explicitamente fora do escopo da frente `[F1] / [ETL 01]`:

- testar runtime real da VPS dentro da CI
- reescrever scripts do ETL
- refatorar shell scripts apenas para agradar lint
- criar matriz de jobs
- expandir a cobertura para `legacy/` ou `_reconcile/`
- adicionar `shfmt -d` no mesmo workflow

Esses itens continuam possíveis como evolução futura, mas não fazem parte do contrato mínimo atual.
