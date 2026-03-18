# Deploy, Release and Rollback

## Objetivo

Documentar o fluxo oficial de release, deploy e rollback da Auth API no contexto AWS Lightsail.

Este documento existe para registrar, de forma estável e auditável:

- a estratégia normativa de publicação da Auth API
- o uso de release por TAG como mecanismo de versionamento operacional
- o processo de deploy manual em produção
- os smoke tests obrigatórios após publicação
- o fluxo de rollback quando a release ativa precisa ser revertida
- os guardrails necessários para manter o runtime consistente

---

## Escopo

Este documento cobre:

- estratégia de branch e release da Auth API
- publicação por TAG
- deploy manual no host Lightsail
- restart controlado do serviço `hsc-auth-api`
- smoke tests pós-deploy
- rollback operacional
- state file da última TAG conhecida
- cuidados com drift entre código, TAG e serviço ativo

Este documento não cobre em profundidade:

- detalhes do unit file do systemd
- configuração detalhada do Nginx
- contratos HTTP completos da Auth API
- troubleshooting geral do runtime
- backup e restore do banco

Esses assuntos vivem em documentos próprios do contexto.

---

## Estado atual

O modelo operacional conhecido para a Auth API é:

- repositório Git versionado
- release determinística por TAG
- working directory de produção em `/opt/hsc/hsc-auth-api`
- instalação de dependências via `npm ci`
- restart controlado do serviço `hsc-auth-api`
- registro da última TAG implantada em `/opt/hsc/.deploy-auth-last-tag`
- logging operacional de deploy em `/var/log/hsc/deploy-auth.log`
- validação pós-deploy por smoke tests

O objetivo deste modelo é garantir que produção rode exatamente uma release identificável e reversível.

---

## Source of truth / evidências

As evidências principais deste fluxo, nesta fase de migração documental, são:

- manuais operacionais de deploy da Auth API
- documentação consolidada do ecossistema HSC
- documentos específicos de git flow, release por TAG e workflow de deploy
- impl-logs ligados a release, deploy e hardening operacional da Auth API

Enquanto a migração canônica não estiver concluída, essas fontes seguem sendo usadas para reconciliação do estado real.

---

## Relações com outros documentos

Este arquivo é complementar a:

- `docs/04-infra-aws-lightsail/README.md`
- `docs/04-infra-aws-lightsail/architecture-runtime.md`
- `docs/04-infra-aws-lightsail/node-systemd.md`
- `docs/04-infra-aws-lightsail/auth-api-operations.md`
- `docs/04-infra-aws-lightsail/observability-troubleshooting.md`
- `docs/95-impl-log/`

Este documento descreve o fluxo de publicação operacional da Auth API.  
Ele não substitui os documentos de runtime, edge, banco ou troubleshooting.

---

## Princípios normativos

O fluxo de deploy da Auth API deve respeitar estes princípios:

- produção deve rodar uma release identificável por TAG
- deploy deve ser determinístico
- restart do serviço deve acontecer de forma explícita e controlada
- smoke test pós-deploy é obrigatório
- rollback deve ser rápido, claro e reproduzível
- produção não deve depender de branch solta como referência operacional
- o runtime ativo deve poder ser reconciliado com Git, TAG e estado do serviço

Regra de ouro:

**produção deve refletir uma TAG conhecida, e não um estado ambíguo do repositório**

---

## Estratégia oficial de branches

A estratégia operacional conhecida deste contexto é:

- `main` representa a linha principal de release
- `develop` pode ser usada como linha de evolução contínua
- release para produção deve ser consolidada em TAG
- produção deve operar sobre TAG publicada, preferencialmente em detached HEAD

Esse desenho existe para:

- reduzir ambiguidade sobre o que está em produção
- facilitar rollback
- tornar troubleshooting mais previsível
- preservar rastreabilidade entre código e runtime

---

## Release por TAG

A release por TAG é o mecanismo oficial de publicação da Auth API em produção.

Objetivos da TAG:

- identificar uma versão exata
- permitir deploy reproduzível
- permitir rollback objetivo
- separar claramente desenvolvimento de runtime produtivo

Regras:

- a TAG deve apontar para o commit aprovado para produção
- a TAG deve ser fetchada explicitamente no host
- a aplicação em produção deve ser alinhada à TAG escolhida
- mudanças em produção sem TAG devem ser evitadas

Formato de tag:
- seguir o padrão adotado pelo repositório da Auth API
- manter consistência entre changelog, impl-log e release operacional

---

## Working directory oficial

O diretório oficial de deploy e runtime da aplicação é:

- `/opt/hsc/hsc-auth-api`

Esse diretório deve conter:

- código da release ativa
- `package.json`
- `package-lock.json`
- `node_modules` compatíveis com a release ativa
- entrypoint `index.js`
- arquivo `.env`
- working tree coerente com a TAG implantada

Qualquer divergência entre esse diretório e a TAG esperada compromete a previsibilidade do deploy.

---

## Artefatos operacionais do fluxo

Os artefatos operacionais conhecidos deste fluxo incluem:

### Diretório da aplicação

- `/opt/hsc/hsc-auth-api`

### Serviço da aplicação

- `hsc-auth-api`

### State file da última TAG implantada

- `/opt/hsc/.deploy-auth-last-tag`

### Log operacional de deploy

- `/var/log/hsc/deploy-auth.log`

Esses artefatos devem ser tratados como parte do fluxo canônico de publicação.

---

## Fluxo de deploy manual no Lightsail

O fluxo manual de deploy deve seguir uma sequência clara e previsível.

### Sequência lógica

1. acessar o host Lightsail
2. entrar no diretório da aplicação
3. sincronizar o repositório remoto e as TAGs
4. identificar a TAG alvo
5. registrar a TAG atualmente ativa antes de alterar o runtime
6. alinhar o working tree à TAG alvo
7. instalar dependências com `npm ci`
8. reiniciar o serviço `hsc-auth-api`
9. executar smoke tests obrigatórios
10. registrar resultado do deploy

---

## Pré-condições de deploy

Antes de iniciar um deploy, validar:

- acesso administrativo ao host
- working directory correto
- serviço `hsc-auth-api` conhecido e íntegro
- TAG alvo existente no repositório
- MariaDB funcional
- `.env` presente e íntegro
- espaço em disco e permissões básicas em ordem
- ausência de intervenção concorrente no mesmo diretório

Se alguma dessas pré-condições falhar, o deploy não deve prosseguir cegamente.

---

## Passos operacionais de deploy

### Entrar no diretório da aplicação

```bash
cd /opt/hsc/hsc-auth-api
```

### Sincronizar o repositório e as TAGs

```bash
git fetch --all --tags --prune
```

### Verificar a TAG alvo

```bash
git tag --list
```

### Registrar a TAG atualmente ativa, se existir

```bash
git describe --tags --exact-match 2>/dev/null || true
```

### Alinhar o diretório à TAG alvo

Substitua `SEU_TAG` pela release desejada.

```bash
git checkout --detach SEU_TAG
```

### Instalar dependências da release ativa

```bash
npm ci
```

### Reiniciar o serviço da aplicação

```bash
sudo systemctl restart hsc-auth-api
```

### Verificar status do serviço

```bash
sudo systemctl status hsc-auth-api
```

### Verificar logs do serviço

```bash
sudo journalctl -u hsc-auth-api -n 100 --no-pager
```

---

## Registro da última TAG implantada

O fluxo deve manter referência da última TAG implantada em:

- `/opt/hsc/.deploy-auth-last-tag`

Esse arquivo existe para:

- facilitar rollback
- tornar a operação mais previsível
- preservar memória operacional fora do estado transitório do shell

Após um deploy bem-sucedido, registrar a TAG ativa:

```bash
printf '%s\n' 'SEU_TAG' | sudo tee /opt/hsc/.deploy-auth-last-tag >/dev/null
```

Observação:
- esse arquivo não substitui Git
- ele serve como apoio operacional rápido para rollback e auditoria local

---

## Logging operacional de deploy

O fluxo conhecido considera o uso de log operacional em:

- `/var/log/hsc/deploy-auth.log`

Esse log deve registrar, quando aplicável:

- data/hora do deploy
- TAG implantada
- operador responsável
- resultado do restart
- resultado dos smoke tests
- eventual rollback executado

O objetivo não é substituir logs do sistema, mas preservar trilha operacional do ato de publicação.

---

## Smoke tests obrigatórios

Depois de qualquer deploy, executar smoke tests mínimos.

### Health local

```bash
curl -sS http://127.0.0.1:3000/health
```

### Health público

Substitua pela URL pública vigente do contexto.

```bash
curl -sS https://SEU_DOMINIO/health
```

### Superfícies públicas relevantes

Quando aplicável, validar também:

- `/content/news`
- `/content/seasons`
- `/content/seasons/active`

Exemplos:

```bash
curl -sS https://SEU_DOMINIO/content/news
curl -sS https://SEU_DOMINIO/content/seasons
curl -sS https://SEU_DOMINIO/content/seasons/active
```

### Superfícies administrativas críticas

Quando o deploy afetar superfícies administrativas, validar também os fluxos administrativos esperados de forma controlada.

---

## Critério de sucesso do deploy

Um deploy só deve ser considerado concluído com sucesso quando:

- a TAG alvo estiver realmente ativa no diretório de produção
- `npm ci` tiver concluído sem erro
- o serviço `hsc-auth-api` estiver ativo
- o health local responder corretamente
- o health público responder corretamente
- os endpoints críticos do escopo alterado responderem como esperado
- não houver erro crítico recorrente em `journalctl`

Se o código foi atualizado, mas os smoke tests falharam, o deploy não deve ser tratado como concluído.

---

## GitHub Actions manual-only

O contexto admite automação auxiliar, mas a regra operacional continua sendo:

- deploy só é válido quando a release ativa do host estiver confirmada
- qualquer automação deve preservar o modelo por TAG
- automação não substitui validação operacional
- o operador continua responsável por confirmar estado final, serviço e smoke tests

Em outras palavras:
- automação pode ajudar
- mas o source of truth operacional continua sendo o host em produção

---

## Rollback

O rollback oficial deve seguir a mesma lógica determinística do deploy.

Objetivo do rollback:

- restaurar rapidamente uma release anterior conhecida
- reduzir tempo de indisponibilidade
- preservar previsibilidade do runtime

O rollback deve usar:

- a TAG anterior conhecida
- o state file local, quando confiável
- e a confirmação da release efetivamente desejada

---

## Sequência lógica de rollback

1. identificar a TAG atualmente ativa
2. identificar a TAG anterior desejada
3. alinhar o working directory à TAG anterior
4. reinstalar dependências com `npm ci`
5. reiniciar o serviço
6. executar os mesmos smoke tests do deploy
7. registrar o rollback no log operacional e no state file

---

## Passos operacionais de rollback

### Entrar no diretório da aplicação

```bash
cd /opt/hsc/hsc-auth-api
```

### Sincronizar tags

```bash
git fetch --all --tags --prune
```

### Verificar a TAG que será restaurada

```bash
git tag --list
```

### Fazer checkout da TAG anterior

Substitua `TAG_ANTERIOR` pela release a restaurar.

```bash
git checkout --detach TAG_ANTERIOR
```

### Reinstalar dependências

```bash
npm ci
```

### Reiniciar o serviço

```bash
sudo systemctl restart hsc-auth-api
```

### Validar o runtime após rollback

```bash
sudo systemctl status hsc-auth-api
curl -sS http://127.0.0.1:3000/health
```

### Atualizar o state file

```bash
printf '%s\n' 'TAG_ANTERIOR' | sudo tee /opt/hsc/.deploy-auth-last-tag >/dev/null
```

---

## Rollback e banco de dados

Rollback de aplicação não implica automaticamente rollback de banco.

Isso significa:

- se a release nova mudou comportamento de schema ou dados
- a reversão de código pode não ser suficiente
- qualquer mudança de banco precisa ser pensada separadamente

Regra importante:
- rollback de aplicação é seguro apenas quando compatível com o estado atual do banco
- se houver migração de schema com impacto, a estratégia deve ser tratada explicitamente no impl-log e, se necessário, em ADR

---

## Sync pós-release

Após uma release estável, pode existir necessidade de sincronização entre linhas de trabalho, por exemplo:

- alinhar `develop` com o que foi publicado em `main`
- preservar consistência entre release e linha de desenvolvimento
- evitar reintrodução acidental de drift

Essa sincronização deve acontecer no repositório de desenvolvimento, não diretamente no working tree de produção como substituto do deploy formal.

Produção continua sendo guiada por TAG.

---

## Lock anti-concorrência

Deploy e rollback não devem ocorrer concorrentemente no mesmo host e diretório.

Riscos de concorrência:

- checkout interrompido
- `node_modules` incoerente
- state file incorreto
- serviço reiniciado sobre working tree inconsistente
- diagnóstico confuso após incidente

Regra operacional:
- só uma operação de publicação por vez
- se necessário, registrar lock operacional explícito no workflow do time ou em script próprio

Mesmo sem mecanismo automatizado de lock, a regra deve ser tratada como obrigatória.

---

## Problemas comuns

### 1. TAG errada implantada

Causas comuns:

- seleção incorreta da release
- falta de conferência antes do checkout
- state file desatualizado

Impacto:
- produção passa a rodar versão diferente da esperada

---

### 2. `npm ci` falha

Causas comuns:

- lockfile inconsistente
- dependência quebrada
- ambiente Node incompatível
- problema de rede ou registry

Impacto:
- deploy interrompido antes do restart seguro

---

### 3. Serviço reinicia, mas health falha

Causas comuns:

- aplicação subiu parcialmente
- erro interno no bootstrap
- quebra de compatibilidade com `.env`
- falha de conexão com banco

Impacto:
- deploy tecnicamente executado, mas funcionalmente inválido

---

### 4. Código no diretório não corresponde à release esperada

Causas comuns:

- checkout incompleto
- alteração manual no working tree
- operação concorrente
- falha de disciplina operacional

Impacto:
- perda de rastreabilidade
- troubleshooting mais difícil
- rollback menos confiável

---

### 5. Rollback não resolve o problema

Causas comuns:

- problema real estava no banco
- Nginx ou edge também estava afetado
- `.env` estava inconsistente
- mudança de schema tornou o código anterior incompatível

Impacto:
- necessidade de diagnóstico além da simples reversão de TAG

---

## Sequência mínima de verificação pós-publicação

Após qualquer deploy ou rollback, a sequência mínima de verificação deve ser:

1. confirmar TAG ativa no diretório
2. confirmar serviço `hsc-auth-api` ativo
3. confirmar health local
4. confirmar health público
5. validar endpoints críticos afetados
6. ler `journalctl` para erro crítico recorrente
7. registrar estado final da operação

---

## Limites deste documento

Este documento não detalha:

- conteúdo do `.env`
- configuração textual do Nginx
- política de CORS em profundidade
- backup e restore do banco
- troubleshooting completo do runtime
- estratégia completa de branches no repositório de desenvolvimento

Esses detalhes vivem em documentos complementares deste contexto.

---

## Critério de pronto deste documento

Este documento pode ser considerado maduro quando:

- o fluxo real de deploy estiver validado contra a prática atual
- a disciplina por TAG estiver consolidada
- o uso do state file estiver confirmado
- os smoke tests refletirem exatamente a rotina operacional
- o rollback estiver documentado de forma suficientemente reproduzível
- ele puder ser usado como runbook real de publicação sem depender do master legado

---

## Última revisão

- Status: ativo
- Classificação: canônico
- Contexto: infraestrutura AWS Lightsail / deploy, release e rollback
- Última revisão: 2026-03-18