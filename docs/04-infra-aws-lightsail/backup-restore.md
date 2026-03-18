# Backup and Restore

## Objetivo

Documentar a camada real de backup do MariaDB no contexto Infra AWS Lightsail do ecossistema HSC, registrando o script real observado no host, o diretório reconciliado dos dumps, o mecanismo exato de agendamento, o formato dos artefatos de backup, a política real de retenção e a validação prática do restore com distinção clara entre o que já foi confirmado e o que ainda depende de validação adicional.

Este documento existe para registrar, de forma estável e auditável:

- onde os dumps reais do banco estão sendo gravados
- qual script real de backup existe no host
- qual é o mecanismo exato que dispara o backup
- qual é o padrão de naming dos dumps observados
- qual é a política real de retenção já confirmada no script
- qual log de backup já foi reconciliado
- como validar a existência e a saúde básica da camada de backup
- como executar um restore de forma controlada
- como o restore já foi validado na prática no host atual

---

## Escopo

Este documento cobre:

- diretório real de dumps do MariaDB no Lightsail
- script real de backup observado no host
- mecanismo exato de agendamento do backup
- arquivos de dump já encontrados
- log real da camada de backup
- estado observado do serviço MariaDB
- convenções reconciliadas de naming dos dumps
- política real de retenção observada no script
- validações operacionais mínimas da camada
- procedimento seguro de restore em nível operacional
- validação prática do restore em base temporária

Este documento não cobre em profundidade:

- modelagem do schema da Auth API
- deploy/release da aplicação Node
- configuração completa do Nginx
- tuning avançado do MariaDB
- estratégia de snapshot do host além dos dumps SQL
- eventual retenção ou cópia off-host, ainda não reconciliada

Esses tópicos vivem em documentos próprios do contexto `04-infra-aws-lightsail` ou exigem nova reconciliação prática.

---

## Estado atual

O estado operacional conhecido e reconciliado da camada de backup do Lightsail é:

- existe um script real de backup em:
  - `/opt/hsc/backup-mariadb.sh`
- existe um diretório real de dumps em:
  - `/opt/hsc/backups/mariadb/`
- existe um log real da camada de backup em:
  - `/opt/hsc/backups/mariadb/backup.log`
- existem múltiplos dumps reais comprimidos no host com naming consistente
- os dumps observados usam extensão:
  - `.sql.gz`
- o prefixo observado dos arquivos é:
  - `hsc_auth_`
- o banco alvo definido no script é:
  - `hsc_auth`
- o MariaDB local está ativo no host
- o serviço reconciliado do banco é:
  - `mariadb.service`
- o banco está aceitando conexões locais via socket:
  - `/run/mysqld/mysqld.sock`

Também ficou reconciliado no host que:

- o mecanismo exato de agendamento do backup é o **crontab do root**
- a linha exata do crontab é:
  - `15 3 * * * /opt/hsc/backup-mariadb.sh`
- o host está em timezone:
  - `Etc/UTC`
- portanto o backup está programado para rodar:
  - **todos os dias às 03:15 UTC**
- a política exata de retenção configurada no script é:
  - `RETENTION_DAYS=14`
- o script usa `mysqldump "$DB_NAME" | gzip -c > "$OUT_FILE"`
- o script aplica:
  - `chown root:adm`
  - `chmod 640`
- `/var/backups` não é o diretório real dos dumps da Auth API
- `/var/backups` contém backups de sistema/pacote, não os dumps principais do banco da Auth API
- o restore já foi **validado na prática** no host atual, em base temporária, sem tocar a produção

Leitura canônica:

- o diretório real dos dumps está fechado sem ambiguidade
- o mecanismo exato de agendamento está fechado sem ambiguidade
- a retenção exata dos dumps também está fechada no nível do script
- o restore já não é mais hipótese documental; ele foi testado com sucesso em base temporária no host real

---

## Source of truth / evidências

As principais evidências deste documento, nesta fase de reconciliação, são:

- presença do script:
  - `/opt/hsc/backup-mariadb.sh`
- presença do diretório:
  - `/opt/hsc/backups/mariadb/`
- presença do log:
  - `/opt/hsc/backups/mariadb/backup.log`
- presença dos dumps reais:
  - `hsc_auth_*.sql.gz`
- conteúdo reconciliado do script real
- presença da entrada exata no crontab do root
- ausência de crontab relevante do `hscadmin`
- ausência de timer `systemd --user` relevante para backup
- saída real de `systemctl status mariadb`
- presença do socket local:
  - `/run/mysqld/mysqld.sock`
- timezone reconciliado do host:
  - `Etc/UTC`
- restore prático realizado em base temporária:
  - `hsc_auth_restore_test`
- inventário restaurado de tabelas e comparação com a produção

Enquanto o runtime real permanecer neste formato, essas evidências prevalecem como source of truth operacional.

---

## Relações com outros documentos

Este arquivo é complementar a:

- `docs/04-infra-aws-lightsail/README.md`
- `docs/04-infra-aws-lightsail/architecture-runtime.md`
- `docs/04-infra-aws-lightsail/mariadb-local.md`
- `docs/04-infra-aws-lightsail/node-systemd.md`
- `docs/04-infra-aws-lightsail/auth-api-operations.md`
- `docs/04-infra-aws-lightsail/observability-troubleshooting.md`
- `docs/04-infra-aws-lightsail/references-inventory.md`

Este documento descreve a camada de backup e restore.  
Ele não substitui os documentos de runtime da aplicação, inventário geral ou operação do MariaDB.

---

## Diretório real dos dumps

O diretório real reconciliado dos dumps do MariaDB da Auth API é:

```text
/opt/hsc/backups/mariadb/
```

Leitura canônica:

- este é o diretório principal da camada de backup do banco
- qualquer documentação anterior apontando para outro local deve ser tratada como stale até nova validação
- este diretório deve ser tratado como o alvo operacional principal de inventário, checagem de espaço, rotação e restore

---

## Script real de backup

O script real reconciliado no host é:

```text
/opt/hsc/backup-mariadb.sh
```

Permissões observadas:

- owner: `root`
- group: `root`
- mode: `0755`

Leitura canônica:

- este script existe materialmente no host
- ele é o entrypoint real da rotina de backup do MariaDB
- ele não delega o backup a outro wrapper reconciliado nesta rodada
- o script já permite fechar com precisão:
  - nome da base
  - diretório de saída
  - retenção
  - naming dos dumps
  - forma de compressão
  - permissões finais do artefato

---

## Conteúdo operacional reconciliado do script

Os parâmetros reais reconciliados do script são:

### Base-alvo

```text
DB_NAME="hsc_auth"
```

### Diretório de saída

```text
BACKUP_DIR="/opt/hsc/backups/mariadb"
```

### Retenção

```text
RETENTION_DAYS=14
```

### Naming do arquivo

O script monta:

```text
${BACKUP_DIR}/${DB_NAME}_${TS}.sql.gz
```

onde:

```text
TS="$(date -u +%Y-%m-%dT%H%M%SZ)"
```

### Log

```text
LOG_FILE="${BACKUP_DIR}/backup.log"
```

### Comando principal de dump

```text
mysqldump "$DB_NAME" | gzip -c > "$OUT_FILE"
```

### Permissões do dump final

```text
chown root:adm "$OUT_FILE"
chmod 640 "$OUT_FILE"
```

### Política de limpeza

```text
find "$BACKUP_DIR" -type f -name "${DB_NAME}_*.sql.gz" -mtime +"$RETENTION_DAYS" -print -delete
```

Leitura canônica:

- a retenção exata está fechada em 14 dias
- a base-alvo está fechada como `hsc_auth`
- a compressão é gzip no próprio pipeline do dump
- o cleanup remove arquivos mais antigos que a janela configurada

---

## Mecanismo exato de agendamento

O mecanismo exato reconciliado do backup é:

- **cron**
- **crontab do root**

A linha exata observada é:

```text
15 3 * * * /opt/hsc/backup-mariadb.sh
```

Leitura canônica:

- o backup **não** está sendo disparado por timer `systemd` reconciliado
- o backup **não** está sendo disparado por `crontab` do `hscadmin`
- o backup **não** está sendo disparado por user timer reconciliado
- o invocador exato atual é o `root` cron

---

## Timezone do agendamento

O host está em:

```text
Etc/UTC
```

Com isso, a entrada:

```text
15 3 * * * /opt/hsc/backup-mariadb.sh
```

deve ser lida como:

- execução diária
- às **03:15 UTC**

Leitura canônica:

- o horário operacional do backup está fechado
- qualquer leitura desse horário em outro fuso deve ser tratada como conversão derivada, não como configuração do host

---

## Log real da camada de backup

O log reconciliado da camada de backup é:

```text
/opt/hsc/backups/mariadb/backup.log
```

Leitura canônica:

- este arquivo é parte real do fluxo de backup
- ele registra o caminho de saída dos dumps
- ele já mostrou evidência de sucesso recorrente
- ele também já mostrou ao menos um episódio histórico de falha de conexão do `mysqldump`

Trechos operacionais relevantes já observados no host:

- o log contém linhas `Out: /opt/hsc/backups/mariadb/...`
- o log registrou erro histórico de conexão ao socket local
- o log mostra também arquivos sendo impressos/removidos pela etapa de retenção

Isso confirma que o log deve ser tratado como artefato importante de troubleshooting da camada.

---

## Artefatos de dump observados

Os dumps reais observados no host incluem, entre outros:

- `hsc_auth_2026-03-04T031501Z.sql.gz`
- `hsc_auth_2026-03-05T031501Z.sql.gz`
- `hsc_auth_2026-03-06T031501Z.sql.gz`
- `hsc_auth_2026-03-07T031501Z.sql.gz`
- `hsc_auth_2026-03-08T031501Z.sql.gz`
- `hsc_auth_2026-03-09T031501Z.sql.gz`
- `hsc_auth_2026-03-10T031501Z.sql.gz`
- `hsc_auth_2026-03-11T031501Z.sql.gz`
- `hsc_auth_2026-03-12T031501Z.sql.gz`
- `hsc_auth_2026-03-13T031501Z.sql.gz`
- `hsc_auth_2026-03-14T031501Z.sql.gz`
- `hsc_auth_2026-03-15T031501Z.sql.gz`
- `hsc_auth_2026-03-16T031501Z.sql.gz`
- `hsc_auth_2026-03-17T031501Z.sql.gz`
- `hsc_auth_2026-03-18T031502Z.sql.gz`

Leitura canônica:

- existe materialidade histórica suficiente para afirmar que a rotina de dump está funcionando de forma recorrente
- o formato comprimido `.sql.gz` está reconciliado
- o prefixo `hsc_auth_` está alinhado com o nome da base do script
- a convenção temporal está em UTC com marca no nome do arquivo

---

## Convenção de naming observada

O padrão observado nos dumps é:

```text
hsc_auth_<timestamp UTC>.sql.gz
```

Exemplo:

```text
hsc_auth_2026-03-18T031502Z.sql.gz
```

Leitura canônica:

- o naming inclui data e hora em UTC
- a extensão final é `.sql.gz`
- isso é adequado para:
  - inventário rápido
  - ordenação cronológica
  - retenção por script
  - restore direcionado por data

---

## Retenção real reconciliada

A retenção exata configurada no script é:

```text
RETENTION_DAYS=14
```

A remoção é feita por:

```text
find "$BACKUP_DIR" -type f -name "${DB_NAME}_*.sql.gz" -mtime +"$RETENTION_DAYS" -print -delete
```

Leitura canônica:

- a política de retenção já não é mais inferência
- ela está confirmada diretamente no script
- a limpeza atua sobre dumps do padrão:
  - `hsc_auth_*.sql.gz`
- arquivos mais antigos que a janela configurada podem ser impressos e removidos durante a execução

---

## Estado reconciliado do MariaDB

O serviço de banco local reconciliado no host é:

```text
mariadb.service
```

Estado observado:

- `active (running)`

Versão observada no `systemctl status`:

- `10.6.23-MariaDB-0ubuntu0.22.04.1`

Socket observado:

```text
/run/mysqld/mysqld.sock
```

Porta observada:

- `3306`

Binding observado pelo próprio status do serviço:

- `127.0.0.1`

Leitura canônica:

- o MariaDB local está saudável no momento da validação
- o banco está preparado para atender a aplicação local e a rotina de dump
- o socket local é parte importante da camada de backup, porque a falha histórica registrada no log foi justamente de conexão ao socket

---

## Restore validado na prática

O restore foi validado no host real em **2026-03-18**, sem tocar a base produtiva.

### Dump usado no teste

```text
/opt/hsc/backups/mariadb/hsc_auth_2026-03-18T031502Z.sql.gz
```

### Estratégia usada

- inspeção prévia do dump com `gunzip -c`
- criação da base temporária:
  - `hsc_auth_restore_test`
- restore do dump nesta base temporária
- comparação de estrutura e volume básico com a base produtiva `hsc_auth`
- remoção da base temporária ao final do teste

### Resultado do restore

O restore concluiu sem erro.

### Estrutura restaurada

A base temporária restaurada apresentou **8 tabelas**:

- `admin_audit_log`
- `magic_links`
- `news`
- `profiles`
- `schema_meta`
- `seasons`
- `sessions`
- `users`

### Comparação com a produção

A estrutura da base temporária coincidiu com a produção.

A comparação básica de volume observada foi:

- `admin_audit_log` → `0`
- `magic_links` → `0`
- `news` → `0`
- `profiles` → `0`
- `schema_meta` → `0`
- `seasons` → `2`
- `sessions` → `0`
- `users` → `0`

Leitura canônica:

- o dump não apenas existe; ele restaura corretamente
- a cadeia `dump -> gunzip -> mysql` foi validada com sucesso no host real
- a pendência de “restore testado na prática” pode ser tratada como **fechada**
- o que continua não executado, por escolha correta de segurança, é apenas um restore destrutivo sobre a base produtiva

### Limpeza pós-teste

Após a validação, a base temporária foi removida:

- `DROP DATABASE IF EXISTS hsc_auth_restore_test`

Isso confirma que o teste foi concluído sem deixar artefato operacional residual.

---

## O que já está confirmado

Os pontos abaixo já podem ser tratados como fechados:

- existe script real de backup
- o script real está em `/opt/hsc/backup-mariadb.sh`
- a base-alvo do script é `hsc_auth`
- os dumps reais vivem em `/opt/hsc/backups/mariadb/`
- o log real vive em `/opt/hsc/backups/mariadb/backup.log`
- os dumps usam `.sql.gz`
- o prefixo observado é `hsc_auth_`
- a retenção exata no script é 14 dias
- o mecanismo exato de agendamento é o crontab do root
- a linha exata de cron é `15 3 * * * /opt/hsc/backup-mariadb.sh`
- o host usa timezone `Etc/UTC`
- o MariaDB local está ativo no host
- o restore foi validado com sucesso em base temporária no host atual
- a base temporária de teste foi removida após a validação

---

## O que ainda não está 100% fechado

Os pontos abaixo ainda podem ser refinados futuramente, mas já não bloqueiam o checkpoint:

- eventual endurecimento adicional do backup, como checksum
- eventual cópia externa/off-host dos dumps
- política operacional de retenção fora do host, se existir

Regra importante:

- este documento distingue o que foi comprovado do que ainda seria apenas evolução futura
- o restore local em base temporária já foi fechado com evidência prática suficiente

---

## Runbook de validação rápida da camada de backup

Este runbook deve ser usado:

- após mudança em MariaDB
- após mudança em deploy da Auth API
- quando houver suspeita de ausência de dump recente
- antes de executar restore
- quando houver suspeita de falta de espaço ou falha histórica de backup

### Objetivo

Confirmar rapidamente se a camada de backup do banco parece saudável.

### Validar presença do script

```bash
ls -lah /opt/hsc/backup-mariadb.sh
```

### Validar cron do root

```bash
sudo crontab -l
sudo sed -n '1,220p' /var/spool/cron/crontabs/root
```

### Validar diretório de dumps

```bash
ls -lah /opt/hsc/backups/mariadb/
```

### Validar dump mais recente

```bash
find /opt/hsc/backups/mariadb -maxdepth 1 -type f -name 'hsc_auth_*.sql.gz' | sort | tail -n 5
```

### Validar log

```bash
tail -n 50 /opt/hsc/backups/mariadb/backup.log
```

### Validar MariaDB

```bash
systemctl status mariadb --no-pager
```

### Critério de sucesso

A camada é considerada minimamente saudável quando:

- o script existe
- a entrada do cron do root existe
- o diretório de dumps existe
- há dumps recentes no diretório
- o log não indica falha persistente recente
- o MariaDB está ativo

---

## Runbook de inspeção de um dump antes do restore

Este runbook deve ser usado antes de qualquer restore.

### Objetivo

Inspecionar um dump comprimido sem ainda restaurá-lo.

### Ver arquivo alvo

Substitua pelo arquivo desejado.

```bash
ls -lah /opt/hsc/backups/mariadb/hsc_auth_2026-03-18T031502Z.sql.gz
```

### Ver cabeçalho do dump descomprimido

```bash
gunzip -c /opt/hsc/backups/mariadb/hsc_auth_2026-03-18T031502Z.sql.gz | head -n 50
```

### Procurar linhas úteis no dump

```bash
gunzip -c /opt/hsc/backups/mariadb/hsc_auth_2026-03-18T031502Z.sql.gz | grep -Ei 'create database|use |insert into|drop table' | head -n 50
```

Leitura canônica:

- a inspeção do dump deve preceder restore destrutivo
- isso ajuda a confirmar:
  - base alvo
  - estrutura do dump
  - presença de DDL e DML
  - compatibilidade geral do artefato com a expectativa operacional

---

## Runbook de restore seguro

Este runbook deve ser tratado como procedimento operacional seguro e conservador.

### Objetivo

Restaurar um dump de forma controlada, evitando sobrescrever a base viva sem validação.

### Regra de ouro

**não restaurar diretamente sobre a base produtiva sem confirmar o arquivo, a base-alvo e o plano de rollback**

### Estratégia preferida

1. inspecionar o dump
2. restaurar primeiro em base temporária
3. validar schema e volume
4. só então decidir por restore produtivo

### Exemplo de restore para base temporária

Crie uma base temporária de validação:

```bash
mysql -u root -p -e "CREATE DATABASE hsc_auth_restore_test;"
```

Restaure o dump nela:

```bash
gunzip -c /opt/hsc/backups/mariadb/hsc_auth_2026-03-18T031502Z.sql.gz | mysql -u root -p hsc_auth_restore_test
```

Valide presença de tabelas:

```bash
mysql -u root -p -e "SHOW TABLES FROM hsc_auth_restore_test;"
```

Valide volume básico:

```bash
mysql -u root -p -e "SELECT table_name, table_rows FROM information_schema.tables WHERE table_schema = 'hsc_auth_restore_test';"
```

Quando o teste terminar, remova a base temporária:

```bash
mysql -u root -p -e "DROP DATABASE IF EXISTS hsc_auth_restore_test;"
```

### Restore produtivo

Restore produtivo só deve acontecer depois de:

- confirmar o dump correto
- confirmar a base-alvo correta
- garantir janela operacional adequada
- garantir rollback ou snapshot anterior
- garantir que a Auth API seja parada se isso fizer parte do plano seguro

Exemplo genérico, apenas quando o alvo estiver 100% confirmado:

```bash
gunzip -c /opt/hsc/backups/mariadb/hsc_auth_2026-03-18T031502Z.sql.gz | mysql -u root -p hsc_auth
```

Observação importante:
- o nome da base produtiva está alinhado com o script como `hsc_auth`
- ainda assim, restore destrutivo continua exigindo validação operacional cuidadosa

---

## Relação entre restore e aplicação Node

O restore do banco deve ser lido em conjunto com a aplicação.

Isso significa:

- restore pode afetar imediatamente a Auth API
- inconsistência entre schema restaurado e versão da app pode quebrar endpoints
- restore não deve ser tratado como tarefa isolada do banco

Sequência prudente em operações sensíveis:

1. validar unit da app
2. validar backup escolhido
3. decidir se a app será parada
4. executar restore
5. validar MariaDB
6. validar `hsc-auth-api.service`
7. validar `/health` local e público

Comandos úteis:

```bash
systemctl status hsc-auth-api.service --no-pager
curl -I http://127.0.0.1:3000/health
curl -I https://auth-api.haxixesmokeclub.com/health
```

---

## Problemas comuns

### 1. Dump recente não aparece

Causas comuns:

- cron do root removido ou alterado
- script falhou
- problema de permissões
- problema no MariaDB

Impacto:
- falsa sensação de segurança
- risco operacional alto antes de deploy ou mudança de schema

---

### 2. Log mostra falha de socket

Causas comuns:

- MariaDB indisponível no momento do backup
- socket local não acessível
- banco reiniciando
- serviço parado

Impacto:
- dump do dia pode falhar
- retenção histórica pode mascarar ausência de backup recente

Evidência já reconciliada:
- houve pelo menos um erro histórico de `mysqldump` ao tentar conectar em `/run/mysqld/mysqld.sock`

---

### 3. Dump existe, mas restore quebra

Causas comuns:

- dump incompleto
- base-alvo errada
- incompatibilidade de schema
- restore sobre app viva sem coordenação
- permissões ou credenciais inadequadas

Impacto:
- downtime
- inconsistência de dados
- troubleshooting mais complexo

---

### 4. Restore em base produtiva sem validação prévia

Causas comuns:

- pressa operacional
- ausência de base temporária
- confiança excessiva no naming do arquivo

Impacto:
- sobrescrita indevida
- perda de estado atual
- incidente evitável

---

### 5. Confundir `/var/backups` com backup real da Auth API

Causas comuns:

- leitura superficial do host
- presença de backups do sistema em `/var/backups`

Impacto:
- operador olha o diretório errado
- camada real de dump do banco fica sem validação

Regra canônica:

- o diretório real da Auth API é `/opt/hsc/backups/mariadb/`
- `/var/backups` não é a fonte principal desta camada

---

## Invariantes operacionais

Os invariantes conhecidos desta camada incluem:

- o script real de backup vive em `/opt/hsc/backup-mariadb.sh`
- a base-alvo do script é `hsc_auth`
- o diretório real dos dumps vive em `/opt/hsc/backups/mariadb/`
- o log real da camada vive em `/opt/hsc/backups/mariadb/backup.log`
- o naming observado dos dumps usa prefixo `hsc_auth_`
- a extensão observada é `.sql.gz`
- a retenção configurada no script é 14 dias
- o mecanismo exato de agendamento é o crontab do root
- a linha exata do cron é `15 3 * * * /opt/hsc/backup-mariadb.sh`
- o host está em timezone `Etc/UTC`
- o MariaDB local está ativo via `mariadb.service`
- o socket local observado é `/run/mysqld/mysqld.sock`
- o restore em base temporária foi validado com sucesso no host atual
- `/var/backups` não é o diretório principal da camada de backup da Auth API

Esses invariantes ajudam a preservar a leitura correta da camada de backup atual.

---

## Limites deste documento

Este documento não detalha:

- credenciais reais de acesso ao banco
- cópia externa/off-host dos dumps, se existir
- integração futura com snapshot de volume
- verificação criptográfica dos dumps
- política futura de retenção externa

Esses tópicos pertencem a futuras reconciliações finas.

---

## Critério de pronto deste documento

Este documento pode ser considerado maduro quando:

- o diretório real dos dumps estiver explícito sem ambiguidade
- o script real de backup estiver corretamente posicionado
- o mecanismo exato de agendamento estiver formalizado
- a retenção real do script estiver explícita
- o log da camada estiver formalizado
- o restore em base temporária estiver validado na prática
- o runbook de validação estiver claro
- o restore seguro estiver descrito com cautela adequada
- ele puder ser usado como referência confiável de backup/restore da Auth API sem depender do master legado

---

## Última revisão

- Status: ativo
- Classificação: canônico
- Contexto: infraestrutura AWS Lightsail / backup and restore
- Última revisão: 2026-03-18