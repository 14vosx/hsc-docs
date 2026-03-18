# Backup and Restore

## Objetivo

Documentar a rotina operacional de backup e restore do contexto AWS Lightsail da Auth API, com foco na persistência local em MariaDB e nos cuidados necessários para preservar recuperabilidade, previsibilidade e rastreabilidade.

Este documento existe para registrar, de forma estável e auditável:

- o que é considerado artefato crítico de backup neste contexto
- como o backup do banco é tratado operacionalmente
- qual a rotina conhecida de execução e retenção
- como validar se os backups estão sendo gerados
- quais cuidados devem existir antes de qualquer restore
- quais limites e riscos operacionais existem no processo de recuperação

---

## Escopo

Este documento cobre:

- artefatos críticos de backup do contexto
- backup do MariaDB local
- rotina conhecida de backup
- retenção conhecida
- verificação operacional do backup
- princípios de restore
- riscos de restore no contexto da Auth API

Este documento não cobre em profundidade:

- backup completo do host em nível de infraestrutura
- dump detalhado de cada tabela
- procedimentos avançados de recuperação ponto no tempo
- backup de secrets em sistemas externos
- fluxo completo de deploy e rollback de aplicação
- troubleshooting geral do host

Esses tópicos vivem em documentos complementares ou ainda dependem de formalização adicional.

---

## Estado atual

O estado operacional conhecido deste contexto indica:

- MariaDB local como persistência da Auth API
- existência de script operacional de backup em `/opt/hsc/backup-mariadb.sh`
- rotina diária de execução
- retenção operacional conhecida de 14 dias
- foco principal do backup na preservação do banco `hsc_auth`

Também é conhecido que:

- o banco roda localmente no host Lightsail
- o restore precisa ser tratado com cautela, pois rollback de aplicação não equivale a rollback de banco
- a recuperabilidade do contexto depende não apenas da existência do dump, mas também de sua consistência e utilizabilidade real

---

## Source of truth / evidências

As principais evidências deste documento, nesta fase de migração documental, são:

- manual operacional da Auth API em AWS Lightsail
- documentação consolidada do ecossistema HSC
- reconciliação documental do script `/opt/hsc/backup-mariadb.sh`
- reconciliação documental da rotina diária de backup
- impl-logs e snapshots técnicos ligados ao schema da Auth API

Enquanto a migração canônica do contexto não estiver concluída, essas fontes seguem servindo como base de reconciliação.

---

## Relações com outros documentos

Este arquivo é complementar a:

- `docs/04-infra-aws-lightsail/README.md`
- `docs/04-infra-aws-lightsail/architecture-runtime.md`
- `docs/04-infra-aws-lightsail/mariadb-local.md`
- `docs/04-infra-aws-lightsail/deploy-release-rollback.md`
- `docs/04-infra-aws-lightsail/observability-troubleshooting.md`
- `docs/95-impl-log/`

Este documento trata da preservação e recuperação de persistência.  
Ele não substitui deploy/rollback de código nem troubleshooting geral do runtime.

---

## Artefatos cobertos

Os artefatos críticos cobertos por este documento são, prioritariamente:

- banco MariaDB local da Auth API
- database operacional `hsc_auth`
- dumps gerados pela rotina de backup do banco

Este documento não assume, nesta fase, uma política canônica já consolidada para backup versionado de:

- working directory da aplicação
- `.env`
- certificados
- arquivos completos do host
- logs históricos

Esses itens podem até existir em práticas operacionais paralelas, mas não devem ser afirmados como parte do backup canônico sem validação explícita.

---

## Papel do backup neste contexto

O backup existe para preservar a recuperabilidade da camada dinâmica do HSC.

No contexto AWS Lightsail, isso é especialmente importante porque:

- a Auth API depende de persistência local
- sessões, auditoria, conteúdo e estruturas de domínio vivem no banco
- falha de banco ou erro destrutivo pode comprometer operação administrativa e conteúdo
- rollback de aplicação não reverte dados persistidos

Portanto, o backup do banco é parte essencial da segurança operacional do contexto.

---

## Backup do MariaDB

O elemento principal de backup conhecido deste contexto é o MariaDB local.

Estado operacional conhecido:

- database principal: `hsc_auth`
- estratégia: dump operacional automatizado via script
- script conhecido: `/opt/hsc/backup-mariadb.sh`

O objetivo desse backup é gerar cópia recuperável do estado do banco, suficiente para:

- contingência
- recuperação após incidente
- validação de integridade estrutural
- apoio operacional em situações de erro destrutivo

---

## Script operacional conhecido

O script operacional conhecido para backup do banco é:

- `/opt/hsc/backup-mariadb.sh`

Este script deve ser tratado como artefato operacional do contexto.

Regras documentais importantes:

- o caminho do script é canônico até revisão explícita
- o conteúdo do script não precisa ser colado integralmente na documentação canônica
- qualquer alteração relevante no comportamento do script deve ser refletida no impl-log e, se necessário, neste documento

---

## Rotina e agendamento

A rotina operacional conhecida para o backup do banco é:

- execução diária
- horário reconciliado: `03:15 UTC`

Esse agendamento existe para:

- garantir cadência mínima de preservação
- reduzir risco de janelas longas sem backup
- tornar o processo previsível e auditável

Se o agendamento real mudar no host, este documento deve ser atualizado.

---

## Retenção

A retenção operacional conhecida é:

- 14 dias

Objetivo da retenção:

- manter janela recente de recuperação
- limitar acúmulo excessivo de artefatos
- preservar histórico curto suficiente para contingência operacional básica

Limite conhecido:
- retenção curta não substitui estratégia mais ampla de continuidade
- mas atende a uma postura prática e simples de operação, compatível com o desenho atual do contexto

---

## Princípios de backup saudável

Um backup saudável, neste contexto, não é apenas “arquivo gerado”.

Ele precisa ser, idealmente:

- recente
- íntegro
- utilizável
- coerente com o banco atual
- localizado em path conhecido
- verificável sem ambiguidade

Regra operacional:
- backup não validado é melhor do que ausência total de backup, mas não deve ser tratado como garantia plena de recuperação

---

## Validação de backup

As validações mínimas esperadas de backup incluem:

- confirmar que a rotina está executando
- confirmar que os artefatos recentes existem
- confirmar que o tamanho/estrutura dos dumps parecem coerentes
- confirmar que a retenção está funcionando como esperado
- registrar falha de geração como incidente operacional relevante

Sem validação mínima, o contexto corre o risco de “achar” que possui backup quando, na prática, possui apenas automação quebrada.

---

## Comandos de validação

Como o path final de saída do dump não foi fixado aqui como canônico reconciliado, a validação deve começar pelo script e pelo resultado operacional real observado no host.

### Verificar existência do script

```bash
ls -l /opt/hsc/backup-mariadb.sh
```

### Inspecionar a rotina, quando necessário

```bash
sudo cat /opt/hsc/backup-mariadb.sh
```

### Procurar artefatos recentes de backup

Substitua `SEU_DIRETORIO_DE_BACKUP` pelo diretório real validado no host.

```bash
ls -lah SEU_DIRETORIO_DE_BACKUP
```

### Verificar cron ou rotina equivalente, quando aplicável

A forma exata depende da configuração real do host.  
Quando necessário, validar a agenda efetivamente usada no sistema.

### Validar serviço MariaDB antes de inferir qualquer problema de backup

```bash
sudo systemctl status mariadb
```

---

## Restore

O restore deve ser tratado como operação sensível e controlada.

Objetivo do restore:

- recuperar estado persistido após incidente
- restaurar banco a partir de dump conhecido
- reduzir perda de dados dentro da janela disponível de retenção

Regra importante:
- restore não deve ser executado de forma impulsiva
- sempre considerar compatibilidade entre:
  - dump restaurado
  - release atual da aplicação
  - schema esperado pela versão em produção
  - necessidade de rollback ou não do código

---

## Princípios de restore seguro

Antes de qualquer restore, validar:

- qual incidente ocorreu
- qual banco será restaurado
- qual dump será usado
- qual é a data/hora do dump
- se a aplicação atual é compatível com o estado restaurado
- se o host atual está saudável o suficiente para receber a restauração
- se o estado atual precisa ser preservado antes do restore

Regra de ouro:

**restore de banco é operação destrutiva ou potencialmente substitutiva, então nunca deve ser tratado como simples “reexecução de script”**

---

## Estratégia mínima antes de restaurar

Antes de restaurar, a prática recomendada é:

1. identificar o escopo real do incidente
2. congelar mudanças concorrentes no contexto
3. registrar o estado atual
4. gerar snapshot/dump do estado presente antes de sobrescrever qualquer coisa
5. selecionar o dump de restore explicitamente
6. validar compatibilidade com a release ativa
7. restaurar de forma controlada
8. validar app, banco e rotas funcionais após a operação

Mesmo quando o restore for necessário, preservar o estado imediatamente anterior continua sendo prática importante de segurança operacional.

---

## Validação pós-restore

Depois de qualquer restore, validar no mínimo:

- serviço MariaDB ativo
- aplicação `hsc-auth-api` ativa
- health local respondendo
- health público respondendo
- rotas públicas críticas respondendo
- fluxos administrativos mínimos coerentes
- ausência de erro crítico recorrente no `journalctl`

Comandos típicos:

```bash
sudo systemctl status mariadb
sudo systemctl status hsc-auth-api
curl -sS http://127.0.0.1:3000/health
curl -sS https://SEU_DOMINIO/health
curl -sS https://SEU_DOMINIO/content/news
curl -sS https://SEU_DOMINIO/content/seasons
```

---

## Restore e compatibilidade com aplicação

Restore de banco deve sempre ser interpretado junto com a release ativa da aplicação.

Risco central:

- restaurar banco para estado incompatível com a versão atual do código

Exemplos de incompatibilidade:

- tabela esperada pela aplicação não existe no dump restaurado
- coluna usada pela release atual não existe
- estado restaurado pressupõe versão mais antiga da aplicação
- auditoria ou sessão ficam estruturalmente inconsistentes

Quando houver dúvida relevante de compatibilidade:

- tratar restore junto com análise de release/rollback
- consultar impl-log estrutural correspondente
- evitar restore cego

---

## Restore e auditoria administrativa

Como o contexto usa trilha de auditoria administrativa, restore também pode impactar:

- histórico de ações administrativas
- consistência temporal de mutações
- rastreabilidade de writes recentes

Isso não impede restore, mas exige consciência operacional.

Regra prática:
- restore de banco em ambiente vivo deve considerar que parte do histórico administrativo recente pode ser revertida junto com os dados

---

## Problemas comuns

### 1. Backup existe, mas não é utilizável

Causas comuns:

- dump incompleto
- rotina quebrada há dias
- arquivo corrompido
- path de backup errado ou não documentado corretamente

Impacto:
- falsa sensação de segurança
- restore inviável no momento crítico

---

### 2. Backup recente, mas schema incompatível com a release atual

Causas comuns:

- aplicação evoluiu mais que o dump
- mudança estrutural não reconciliada
- tentativa de restore sem considerar compatibilidade de release

Impacto:
- aplicação volta a falhar após restore
- troubleshooting se torna mais confuso

---

### 3. Restore é feito sem preservar o estado atual

Causas comuns:

- pressa em incidente
- ausência de procedimento explícito
- suposição de que o estado atual “já está perdido”

Impacto:
- perda da chance de recuperar parcialmente o estado presente
- perda de material útil para auditoria e pós-mortem

---

### 4. Rotina diária deixou de rodar

Causas comuns:

- agendamento quebrado
- script alterado
- permissões incorretas
- problema de espaço ou ambiente

Impacto:
- aumento silencioso da janela sem backup válido

---

### 5. Restore resolve banco, mas app continua falhando

Causas comuns:

- problema real estava também na release ativa
- `.env` inconsistente
- Nginx ou runtime do serviço degradado
- incompatibilidade entre restore e aplicação

Impacto:
- recuperação parcial, mas não funcional

---

## Riscos e cuidados

Os principais riscos desta camada incluem:

- confiar em backup não validado
- tratar restore como procedimento trivial
- restaurar dump incompatível com a release ativa
- esquecer de preservar o estado atual antes de restaurar
- assumir que rollback de aplicação resolve problema de banco
- manter retenção funcionando “por suposição”, sem conferência

Cuidados permanentes:

- validar periodicamente a existência dos dumps
- registrar mudanças relevantes no fluxo de backup
- tratar restore como operação controlada
- sempre correlacionar banco e release da aplicação
- manter este documento alinhado ao runtime real

---

## Lacunas atuais

As lacunas conhecidas ou prováveis deste contexto incluem:

- ausência, nesta fase documental, de um passo a passo canônico completo de restore já reconciliado com o host real
- ausência, até aqui, de formalização canônica do diretório exato de armazenamento dos dumps
- dependência de validação futura para transformar restore em runbook totalmente operacional e detalhado

Regra editorial importante:
- registrar a lacuna explicitamente é melhor do que inventar procedimento “completo” sem lastro operacional

---

## Limites deste documento

Este documento não detalha:

- script completo de backup
- credenciais
- diretório definitivo dos dumps sem validação adicional
- restore completo comando a comando já certificado no host
- políticas de backup de todo o servidor
- snapshots de filesystem ou imagem da VM

Esses pontos podem ser formalizados futuramente, após reconciliação adicional com o runtime real.

---

## Critério de pronto deste documento

Este documento pode ser considerado maduro quando:

- o script de backup e seu agendamento estiverem confirmados no host real
- o diretório real dos dumps estiver documentado sem ambiguidade
- a retenção estiver validada
- a rotina mínima de verificação estiver consolidada
- o restore tiver um procedimento reconciliado com o ambiente real
- ele puder ser usado como base confiável de preservação e recuperação do contexto sem depender do master legado

---

## Última revisão

- Status: ativo
- Classificação: canônico
- Contexto: infraestrutura AWS Lightsail / backup and restore
- Última revisão: 2026-03-18