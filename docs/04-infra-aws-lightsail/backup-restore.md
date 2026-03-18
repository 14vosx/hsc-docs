# Backup and Restore

## Objetivo

Documentar a camada real de backup do MariaDB no contexto Infra AWS Lightsail do ecossistema HSC, registrando o diretório reconciliado dos dumps, o script real observado no host, o formato dos artefatos de backup, o estado atual da evidência de retenção e o procedimento seguro de restore com distinção clara entre o que já foi confirmado e o que ainda depende de validação prática.

Este documento existe para registrar, de forma estável e auditável:

- onde os dumps reais do banco estão sendo gravados
- qual script real de backup existe no host
- qual é o padrão de naming dos dumps observados
- qual log de backup já foi reconciliado
- como validar a existência e a saúde básica da camada de backup
- como executar um restore de forma controlada
- quais partes do fluxo já estão confirmadas e quais ainda precisam de validação prática

---

## Escopo

Este documento cobre:

- diretório real de dumps do MariaDB no Lightsail
- script real de backup observado no host
- arquivos de dump já encontrados
- log real da camada de backup
- estado observado do serviço MariaDB
- convenções reconciliadas de naming dos dumps
- validações operacionais mínimas da camada
- procedimento seguro de restore em nível operacional
- limitações e pendências ainda abertas dessa camada

Este documento não cobre em profundidade:

- modelagem do schema da Auth API
- deploy/release da aplicação Node
- configuração completa do Nginx
- tuning avançado do MariaDB
- política final de retenção, porque ainda não foi congelada a partir do script
- restore end-to-end já executado e certificado no host atual

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
- o MariaDB local está ativo no host
- o serviço reconciliado do banco é:
  - `mariadb.service`
- o banco está aceitando conexões locais via socket:
  - `/run/mysqld/mysqld.sock`

Também ficou reconciliado no host que:

- o diretório `/var/backups` não é o diretório real dos dumps da Auth API
- `/var/backups` contém backups de sistema/pacote, não os dumps principais do banco da Auth API
- a automação exata que dispara o script de backup ainda não foi completamente reconciliada por `systemd` ou `cron`
- a evidência atual sugere execução recorrente diária por volta de `03:15Z`, mas essa cadência ainda deve ser tratada como observação empírica, não como contrato formal congelado

Leitura canônica:

- o diretório real dos dumps já está fechado sem ambiguidade
- o fluxo de backup existe de forma material no host
- o ponto ainda não totalmente fechado é o mecanismo exato de agendamento e a política final de retenção extraída do script

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
- saída real de `systemctl status mariadb`
- presença do socket local:
  - `/run/mysqld/mysqld.sock`
- inventário real do filesystem no host

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

Leitura canônica:

- este script existe materialmente no host
- ele deve ser tratado como entrypoint real da rotina de backup do MariaDB
- a lógica completa de retenção e disparo ainda precisa de leitura direta do script para congelamento absoluto
- mesmo sem essa leitura linha a linha, já existe evidência suficiente para fechar o diretório real de destino e o naming dos dumps

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

Trecho operacional relevante já observado no host:

- o log contém linhas `Out: /opt/hsc/backups/mariadb/...`
- o log também registrou erro de conexão ao socket local em um momento anterior
- isso confirma que o log deve ser tratado como artefato importante de troubleshooting da camada

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
- o formato comprimido `.sql.gz` já está reconciliado
- o prefixo `hsc_auth_` sugere fortemente associação com a base da Auth API
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

## Cadência observada dos backups

Com base nos dumps observados e no log reconciliado, existe evidência empírica de uma cadência recorrente próxima de:

- um dump por dia
- aproximadamente às `03:15Z`

Leitura de precisão:

- esta cadência ainda deve ser tratada como observação reconciliada
- ela ainda não deve ser tratada como contrato formal definitivo até leitura completa do mecanismo de agendamento
- o host não mostrou, nesta rodada, unit `systemd` ou entrada `cron` explicitamente nomeada para este backup
- isso significa que o mecanismo de disparo ainda precisa de reconciliação adicional

---

## Retenção observada

O conjunto de arquivos observados mostra uma sequência contínua recente de dumps diários.

Também há indícios no `backup.log` de que arquivos mais antigos já foram manipulados ou removidos em algum momento.

Leitura honesta:

- existe evidência de retenção/rotação em prática
- a política exata de retenção ainda não foi congelada com base no conteúdo do script
- este documento não deve afirmar um número fechado de dias sem inspeção direta do `backup-mariadb.sh`

Regra canônica:

- política exata de retenção = ainda pendente de confirmação fina
- diretório real e naming dos dumps = já reconciliados

---

## O que já está confirmado

Os pontos abaixo já podem ser tratados como fechados:

- existe script real de backup
- o script real está em `/opt/hsc/backup-mariadb.sh`
- os dumps reais vivem em `/opt/hsc/backups/mariadb/`
- o log real vive em `/opt/hsc/backups/mariadb/backup.log`
- os dumps usam `.sql.gz`
- o prefixo observado é `hsc_auth_`
- o MariaDB local está ativo no host
- existe histórico recente suficiente de dumps para considerar a camada real, não hipotética

---

## O que ainda não está 100% fechado

Os pontos abaixo ainda precisam de validação adicional para congelamento absoluto:

- mecanismo exato de agendamento do backup
- política exata de retenção
- comando exato usado no script
- base exata restaurada pelo dump, embora o naming aponte fortemente para `hsc_auth`
- restore end-to-end já executado com validação prática no host atual

Regra importante:

- este documento deve distinguir claramente evidência observada de inferência operacional
- a confiabilidade aumenta muito quando essa distinção é preservada

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

### Restore produtivo

Restore produtivo só deve acontecer depois de:

- confirmar o dump correto
- confirmar a base-alvo correta
- garantir janela operacional adequada
- garantir rollback ou snapshot anterior
- garantir que a Auth API seja parada se isso fizer parte do plano seguro

Exemplo genérico, apenas quando o alvo estiver 100% confirmado:

```bash
gunzip -c /opt/hsc/backups/mariadb/hsc_auth_2026-03-18T031502Z.sql.gz | mysql -u root -p NOME_REAL_DA_BASE
```

Observação importante:
- o naming observado sugere fortemente que a base real seja `hsc_auth`
- ainda assim, o nome da base produtiva deve ser confirmado antes de restore destrutivo

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

- backup não executou
- script falhou
- problema de permissões
- problema no MariaDB
- agendamento real ainda desconhecido e interrompido

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
- o diretório real dos dumps vive em `/opt/hsc/backups/mariadb/`
- o log real da camada vive em `/opt/hsc/backups/mariadb/backup.log`
- o naming observado dos dumps usa prefixo `hsc_auth_`
- a extensão observada é `.sql.gz`
- o MariaDB local está ativo via `mariadb.service`
- o socket local observado é `/run/mysqld/mysqld.sock`
- `/var/backups` não é o diretório principal da camada de backup da Auth API
- a política exata de retenção ainda não está congelada sem leitura do script
- o mecanismo exato de agendamento ainda não está 100% reconciliado

Esses invariantes ajudam a preservar a leitura correta da camada de backup atual sem inventar certezas que ainda não foram confirmadas.

---

## Limites deste documento

Este documento não detalha:

- conteúdo completo linha a linha do `backup-mariadb.sh`
- credenciais reais de acesso ao banco
- restore end-to-end já executado com certificação formal
- política exata de retenção
- política exata de agendamento
- estratégia de snapshot do host além dos dumps SQL

Esses tópicos pertencem a futuras reconciliações finas.

---

## Critério de pronto deste documento

Este documento pode ser considerado maduro quando:

- o diretório real dos dumps estiver explícito sem ambiguidade
- o script real de backup estiver corretamente posicionado
- o log da camada estiver formalizado
- o runbook de validação estiver claro
- o restore seguro estiver descrito com cautela adequada
- a distinção entre fatos confirmados e pontos ainda pendentes estiver preservada
- ele puder ser usado como referência confiável de backup/restore da Auth API sem depender do master legado

---

## Última revisão

- Status: ativo
- Classificação: canônico
- Contexto: infraestrutura AWS Lightsail / backup and restore
- Última revisão: 2026-03-18