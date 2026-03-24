# Deploy, Release and Rollback

## Objetivo

Documentar o fluxo oficial de release, deploy e rollback da Auth API no contexto AWS Lightsail, refletindo o modelo atual de publicação por TAG, migrations explícitas antes do restart e runtime desacoplado da evolução de schema.

Este documento existe para registrar, de forma estável e auditável:

- a estratégia normativa de publicação da Auth API
- o uso de release por TAG como mecanismo de versionamento operacional
- o fluxo oficial de release local e deploy em produção
- a obrigatoriedade de `npm run db:migrate` antes do restart do serviço
- o conjunto mínimo de smoke tests pós-deploy
- o fluxo de rollback operacional
- os guardrails para evitar drift entre código, TAG, banco e serviço ativo

---

## Escopo

Este documento cobre:

- estratégia de branch e release da Auth API
- geração de release por TAG a partir de `main`
- deploy manual no host Lightsail
- aplicação explícita de migrations no release e no deploy
- restart controlado do serviço `hsc-auth-api`
- smoke tests obrigatórios
- rollback operacional
- state file da última TAG conhecida
- relação entre release, banco e runtime

Este documento não cobre em profundidade:

- configuração detalhada do systemd
- contratos HTTP completos da Auth API
- troubleshooting profundo de host/Nginx/TLS
- backup e restore do banco

Esses assuntos vivem em documentos próprios.

---

## Estado atual

Aqui está a consolidação deste bloco, mantendo a clareza técnica e agrupando o modelo geral de infraestrutura com o histórico recente de entregas e suas implicações:

***

## Estado atual do Modelo Operacional e Deploy

O modelo operacional reconciliado da Auth API baseia-se em uma separação clara de três etapas fundamentais: **1. release do código**, **2. migração do banco** e **3. execução do runtime**. 

As características e diretrizes deste modelo incluem:

**Infraestrutura e Execução**
- repositório Git versionado com release determinística por TAG;
- deploy manual controlado no host Lightsail;
- diretório de produção consolidado em `/opt/hsc/hsc-auth-api`;
- instalação rigorosa de dependências via `npm ci --omit=dev`;
- restart controlado do serviço `hsc-auth-api`;
- runtime da aplicação restrito a *readiness check*, não sendo mais responsável pela evolução do schema.

**Migrações e Verificações**
- execução explícita de `npm run db:migrate` sempre antes do restart do serviço;
- smoke tests obrigatórios após o deploy.

**Estado e Logging**
- state file da última TAG implantada mantido em `/opt/hsc/.deploy-auth-last-tag`;
- logging operacional de deploy registrado em `/var/log/hsc/deploy-auth.log`.

### Histórico Recente: Rollout de Admin Users Management
A linha publicada já avançou além da `v0.4.7`. É importante notar que a entrega funcional de *users management* não foi concluída em uma única tag, exigindo três releases sequenciais para a sua reconciliação e estabilização:

- **`v0.4.8`**:
  - publicou as superfícies administrativas de users;
  - publicou a migration `0003_admin_audit_log.sql`.
- **`v0.4.9`**:
  - publicou a migration `0004_users_role_enum_reconcile.sql`;
  - reconciliou o legado de `users.role`, convertendo o valor `user` para `viewer`.
- **`v0.4.10`**:
  - publicou o hotfix de política de CORS liberando o método `PATCH`;
  - destravou as mutações da área `/users` no Backoffice publicado.

### Consequências Operacionais Críticas
- A linha funcional final para a entrega de gestão de usuários só é considerada **completa a partir da tag `v0.4.10`**.
- **Atenção a Rollbacks:** Qualquer operação de rollback para uma tag anterior a `v0.4.10` terá impacto funcional direto, quebrando as operações de mutação na área `/users` do Backoffice.

---

## Source of truth / evidências

As evidências principais deste fluxo são:

- `ops/release.sh`
- `ops/deploy-auth.sh`
- `scripts/migrate.js`
- `db/migrations/*.sql`
- tabela `schema_migrations`
- release line reconciliada até `v0.4.7`
- deploys reais realizados no Lightsail com migration explícita
- impl-log do checkpoint de auth admin + cutover de migrations

---

## Relações com outros documentos

Este arquivo é complementar a:

- `docs/04-infra-aws-lightsail/infra-aws-lightsail-architecture-runtime.md`
- `docs/04-infra-aws-lightsail/node-systemd.md`
- `docs/04-infra-aws-lightsail/auth-api-operations.md`
- `docs/04-infra-aws-lightsail/mariadb-local.md`
- `docs/04-infra-aws-lightsail/backup-restore.md`
- `docs/04-infra-aws-lightsail/infra-aws-lightsail-references-inventory.md`
- `docs/95-impl-log/2026-03-19-auth-admin-magic-link-and-sql-migrations-cutover.md`

---

## Estratégia normativa de release

### Branch de release

A release da Auth API só pode ser gerada a partir de `main`.

Regras operacionais:

- feature/fix nasce em branch própria
- branch é mergeada primeiro em `develop`
- promoção para `main` acontece por branch/PR de promoção
- a TAG de release só nasce depois de `main` reconciliada

### Release por TAG

Cada release operacional deve ter TAG própria.

Exemplos recentes do checkpoint:

- `v0.4.0`
- `v0.4.1`
- `v0.4.2`
- `v0.4.3`
- `v0.4.4`
- `v0.4.5`
- `v0.4.6`
- `v0.4.7`

Regra importante:

- produção deve ser interpretada como uma TAG específica, nunca como “branch solta”

---

## Fluxo oficial de release local

O script oficial é:

```bash
./ops/release.sh <TAG>
```

Exemplo:

```bash
./ops/release.sh v0.4.7
```

### Pré-condições

- estar no repositório correto
- estar em `main`
- working tree limpa
- `main` local igual a `origin/main`
- TAG ainda inexistente
- `.env.local` presente para validação local de migrations e smoke

### O que o script faz

1. valida branch atual (`main`)
2. valida working tree limpa
3. garante sincronização com `origin/main`
4. valida inexistência prévia da TAG
5. executa migrations locais explicitamente:

```bash
ENV_FILE=.env.local npm run db:migrate
```

6. roda smoke local obrigatório
7. cria TAG anotada
8. faz push da TAG

### Consequência importante

O release local agora valida o banco explicitamente antes da TAG. Isso evita gerar release de código que ainda não passou pelo fluxo real de migrations.

---

## Fluxo oficial de deploy em produção

O script oficial é:

```bash
sudo -u hscadmin -H /opt/hsc/hsc-auth-api/ops/deploy-auth.sh <TAG>
```

Exemplo:

```bash
sudo -u hscadmin -H /opt/hsc/hsc-auth-api/ops/deploy-auth.sh v0.4.7
```

### Sequência operacional reconciliada

1. valida host correto
2. valida lock anti-concorrência
3. resolve TAG alvo
4. registra TAG anterior em `/opt/hsc/.deploy-auth-last-tag`
5. faz `git fetch --tags --prune`
6. faz checkout forçado da TAG
7. executa:

```bash
npm ci --omit=dev
```

8. executa migrations explicitamente:

```bash
ENV_FILE=.env npm run db:migrate
```

9. reinicia o serviço `hsc-auth-api`
10. roda smoke tests pós-restart

### Mudança estrutural importante

A evolução de schema não acontece mais no boot da app como mecanismo principal.

O passo oficial é:

- release/deploy executam migrations explicitamente
- runtime depois sobe já contra um banco reconciliado

---

## Fluxo oficial de rollback

O rollback continua baseado no state file:

```text
/opt/hsc/.deploy-auth-last-tag
```

Comando:

```bash
sudo -u hscadmin -H /opt/hsc/hsc-auth-api/ops/deploy-auth.sh --rollback
```

### Comportamento esperado

- lê a última TAG conhecida
- faz checkout da TAG anterior
- reinstala dependências
- roda migrations pendentes da release alvo, se houver
- reinicia o serviço
- executa smoke tests novamente

### Observação importante

Rollback de código não implica desfazer automaticamente schema change já aplicado.

Regra prática:

- migrations devem ser projetadas para compatibilidade progressiva sempre que possível
- rollback funcional precisa considerar banco e aplicação juntos

---

## Migrations como etapa oficial de release

### Regra atual

Toda release e todo deploy devem executar:

```bash
npm run db:migrate
```

Isso vale para:

- validação local antes da TAG
- deploy em produção antes do restart

### Fonte de verdade do schema

O caminho canônico agora é:

- `db/migrations/*.sql`
- `scripts/migrate.js`
- `schema_migrations`

### Papel de `schema.js`

`src/db/schema.js` foi congelado como compatibilidade legada.

Ele não é mais o mecanismo principal de evolução do schema.

---

## Runtime não evolui mais schema no boot

### Estado anterior

Historicamente, o bootstrap da app fazia `ensureSchema()` como mecanismo principal de evolução.

### Estado atual

O bootstrap do banco faz apenas readiness check.

Comportamento esperado:

- abrir conexão
- validar `SELECT 1`
- validar sanity check mínimo via repositório
- marcar `db.ready=true`

Mensagem operacional esperada no log:

```text
Database readiness check passed.
```

Mensagem antiga que não representa mais o fluxo principal:

```text
Database schema ensured (...)
```

---

## Smoke tests obrigatórios pós-deploy

Os smoke tests mínimos do `deploy-auth.sh` incluem:

### Health

```bash
curl -fsS http://127.0.0.1:3000/health
```

Esperado:

- `"ok":true`

### Conteúdo dinâmico

```bash
curl -fsS http://127.0.0.1:3000/content/news
curl -fsS http://127.0.0.1:3000/content/seasons
curl -fsS http://127.0.0.1:3000/content/seasons/active
```

Esperado:

- `"ok":true`

### Superfície técnica protegida

```bash
curl -fsS http://127.0.0.1:3000/admin/schema -H "X-Admin-Key: <ADMIN_KEY>"
```

Esperado:

- `"ok":true`

---

## Guardrails operacionais

### Guardrails do release

- release só a partir de `main`
- working tree limpa
- `main` alinhada com `origin/main`
- migrations locais executadas antes do smoke
- smoke obrigatório antes da TAG

### Guardrails do deploy

- host correto
- lock anti-concorrência
- `ADMIN_KEY` disponível no `.env`
- migrations executadas antes do restart
- smoke obrigatório após restart

### Guardrails de banco

- ambientes existentes devem estar baselined em `schema_migrations`
- ambientes novos devem nascer pelo diretório `db/migrations`
- nova mudança de schema deve nascer como migration SQL, não em `schema.js`

---

## Troubleshooting resumido

### Sintoma: deploy sobe app, mas backend não serve tráfego corretamente

Verificar:

- saída do `npm run db:migrate`
- `journalctl -u hsc-auth-api`
- `/health`
- schema aplicado na tabela `schema_migrations`

### Sintoma: TAG em produção, mas migration não aplicou

Verificar:

- se a migration existe em `db/migrations/`
- se a TAG contém esse arquivo
- se `scripts/migrate.js` foi executado no deploy
- se `schema_migrations` registrou a filename correta

### Sintoma: release local falha antes da TAG

Verificar:

- `.env.local` presente
- MariaDB local em pé
- `npm run db:migrate` sem pendência quebrada
- smoke local consistente

---

## Critério de pronto deste documento

Este documento pode ser considerado reconciliado quando:

- o fluxo oficial de release/deploy estiver alinhado ao runtime real
- a obrigatoriedade de `npm run db:migrate` estiver explícita
- o papel legado de `schema.js` estiver documentado
- rollback e smoke tests estiverem descritos sem ambiguidade

Neste checkpoint, esse critério está atendido.


## Última revisão

- Status: ativo
- Classificação: canônico
- Contexto: infraestrutura AWS Lightsail / deploy, release e rollback
- Última revisão: 2026-03-19