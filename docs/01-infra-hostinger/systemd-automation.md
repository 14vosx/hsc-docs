# systemd Automation

## Objetivo

Documentar a camada de automação do host Debian da Infra Hostinger, registrando o papel do systemd, dos services e dos timers que sustentam geração, publicação e manutenção operacional do lado game + portal do ecossistema HSC.

Este documento existe para registrar, de forma estável e auditável:

- como o systemd participa da operação do lado Hostinger
- quais tipos de services e timers são relevantes para essa camada
- como a automação do host se relaciona com ETL, publishing e checks operacionais
- quais paths e artefatos estruturais essa automação costuma tocar
- quais falhas são comuns quando a automação do host sai do estado esperado
- como validar rapidamente a saúde dessa camada

---

## Escopo

Este documento cobre:

- o papel do systemd na Infra Hostinger
- services e timers relevantes do host
- relação entre automação do host e Portal Estático
- relação entre automação do host e geração da Static API v2
- ordem operacional esperada em alto nível
- comandos de validação de systemd
- logs e diagnóstico em nível de host
- problemas comuns de automação no lado Hostinger

Este documento não cobre em profundidade:

- a implementação linha a linha de cada script do ETL
- a configuração textual completa de unit files específicos ainda não reconciliados no host
- a operação detalhada do AMP Instance Manager
- a operação detalhada da instância `MixHAXIXE01`
- a Auth API no AWS Lightsail
- a modelagem dos contratos JSON da v2

Esses tópicos vivem em documentos próprios dos contextos `02-game-panel`, `03-portal-estatico` e `04-infra-aws-lightsail`.

---

## Estado atual

O estado operacional conhecido da automação do lado Hostinger é:

- o host utiliza `systemd` como camada padrão de serviços e automação
- o lado portal/v2 depende de rotinas recorrentes do host
- existe geração de artefatos públicos da Static API v2
- existe geração de `health.json`
- a automação depende de scripts operacionais no host
- a automação depende de paths estáveis em `/opt/cs2-portal/`, `/usr/local/bin/` e `/var/www/api/cs2/v2/`
- a automação precisa coexistir com o substrate do lado game sem gerar concorrência indevida ou publicação parcial

Também já existe evidência reconciliada do uso de timers e execução recorrente de rotinas como:

- geração da v2
- geração de health
- rotinas correlatas de atualização do estado público

---

## Source of truth / evidências

As principais evidências deste documento, nesta fase de migração documental, são:

- documentação consolidada atual do ecossistema HSC
- blueprint técnico consolidado do HSC
- documentação reconciliada da Infra Hostinger
- documentação reconciliada do Portal Estático e da pipeline ETL
- referências já consolidadas a timers, geração da v2 e `health.json`

Enquanto a migração canônica do contexto não estiver concluída, essas fontes seguem sendo usadas como base de reconciliação do estado real.

---

## Relações com outros documentos

Este arquivo é complementar a:

- `docs/01-infra-hostinger/README.md`
- `docs/01-infra-hostinger/architecture-runtime.md`
- `docs/01-infra-hostinger/filesystem-paths-permissions.md`
- `docs/01-infra-hostinger/observability-troubleshooting.md`
- `docs/03-portal-estatico/etl-bash-pipeline.md`
- `docs/03-portal-estatico/operational-runbooks.md`
- `docs/03-portal-estatico/observability-troubleshooting.md`

Este documento descreve a automação do host via `systemd`.  
Ele não substitui os documentos de ETL, Portal Estático ou troubleshooting aprofundado do contexto da v2.

---

## Papel do systemd no contexto

No lado Hostinger, o `systemd` é a camada padrão de orquestração local do host.

Seu papel neste contexto inclui:

- iniciar e manter serviços do host
- agendar e disparar rotinas recorrentes via timers
- sustentar a execução previsível de scripts operacionais
- servir como base de observabilidade para automações críticas
- reduzir dependência de execução manual de rotinas do portal
- manter o lado público do portal e da v2 em estado mais previsível

Em termos arquiteturais, o `systemd` é o mecanismo que conecta:

- filesystem operacional
- scripts do host
- agendamento recorrente
- geração/publicação da camada pública

---

## Services relevantes

Os services relevantes desta camada são, em nível macro:

- services ligados à geração da v2
- services ligados à geração/atualização de `health.json`
- services auxiliares do host que participem da manutenção do estado público
- service do Nginx, como dependência indireta da entrega final
- eventuais services do host associados ao runtime da camada portal

Regra importante:

- este documento não congela nomes de units que ainda dependam de reconciliação fina no host
- ele formaliza o papel da camada de automação e as units já reconciliadas em alto nível

---

## Timers relevantes

Os timers relevantes desta camada são, em nível macro:

- timers de geração da Static API v2
- timers de geração/atualização de health
- timers auxiliares do host que mantenham a camada pública atualizada ou verificável

Há evidência reconciliada de rotinas do tipo:

- `gen-all-v2`
- `gen-health`

Esses nomes representam a intenção operacional já consolidada no ecossistema: gerar a v2 e manter o artefato de saúde atualizado.

Regra canônica:

- a existência de timer recorrente é parte do funcionamento saudável do contexto
- o host não deve depender exclusivamente de execução manual para manter a camada pública atualizada, salvo em contingência

---

## Relação com ETL e health generation

A automação do host existe para sustentar o ciclo público do Portal Estático.

Essa relação inclui, em nível macro:

- disparo da pipeline da v2
- atualização da árvore pública em `/var/www/api/cs2/v2/`
- atualização de `health.json`
- execução recorrente de checks ou rotinas auxiliares
- observabilidade do sucesso/falha da geração

Isso significa:

- se a automação falhar, a v2 tende a ficar stale
- se a automação estiver em concorrência incorreta, pode haver publicação inconsistente
- se a automação perder acesso a paths ou permissões, o problema aparece como falha pública, mesmo que o Nginx continue saudável

---

## Ordem operacional esperada

A ordem lógica esperada da automação do host é:

1. o host sobe com seus services básicos
2. o Nginx permanece disponível para serving
3. timers do host disparam rotinas conforme cronograma configurado
4. scripts operacionais leem a fonte de dados e/ou artefatos do contexto
5. a Static API v2 é atualizada
6. `health.json` é atualizado
7. o Nginx continua servindo a camada pública já renovada

Regra importante:

- a automação deve respeitar lock, statefiles lock, statefiles e escrita atômica
- systemd e timers não substituem a disciplina do pipeline; eles apenas a executam de forma recorrente

---

## Relação entre timers e filesystem

A camada de automação depende diretamente de paths estáveis.

Paths mais relevantes já reconciliados:

- `/opt/cs2-portal/`
- `/opt/cs2-portal/locks/`
- `/opt/cs2-portal/state/`
- `/opt/cs2-portal/sql/`
- `/usr/local/bin/`
- `/var/www/api/cs2/v2/`

Isso implica:

- um timer saudável pode falhar funcionalmente se os paths mudarem
- um service “executado” pode não publicar nada útil se o filesystem estiver incorreto
- troubleshooting de timer deve sempre considerar o estado dos diretórios e permissões

---

## Dependências operacionais da automação

A automação do host depende, no mínimo, de:

- systemd funcional no host
- timers habilitados e coerentes
- scripts operacionais executáveis
- paths operacionais estáveis
- permissões corretas de leitura e escrita
- lock global funcional
- source data disponível
- publishing público possível
- Nginx íntegro para servir o resultado final

Dependências cruzadas importantes:

- a automação depende do contexto `03-portal-estatico` estar coerente
- a automação pode depender da saúde indireta do lado game, já que a fonte de dados nasce lá
- a automação precisa respeitar a topologia do host, sem invadir a função do Nginx ou do AMP

---

## Invariantes operacionais

Os invariantes conhecidos desta camada incluem:

- a geração recorrente da v2 não deve depender apenas de execução manual
- timers e scripts do host devem operar sobre paths canônicos
- a automação não deve ignorar lock global
- o host não deve publicar artefatos parciais por causa de execução concorrente
- `health.json` deve refletir estado útil da camada pública
- a automação deve ser observável via `systemctl`, `list-timers` e logs
- a camada pública deve continuar íntegra mesmo entre ciclos de geração

Esses invariantes são parte do contrato operacional do lado Hostinger.

---

## Comandos de validação

Os comandos abaixo representam verificações típicas da camada de automação do host.

### Listar timers ativos

```bash
systemctl list-timers --all
```

### Ver status de um timer específico

Substitua pelo nome real reconciliado no host.

```bash
sudo systemctl status SEU_TIMER.timer
```

### Ver status de um service específico

Substitua pelo nome real reconciliado no host.

```bash
sudo systemctl status SEU_SERVICE.service
```

### Ver logs de uma unit

Substitua pelo nome real reconciliado no host.

```bash
sudo journalctl -u SEU_SERVICE.service -n 100 --no-pager
```

### Verificar timestamps da árvore pública da v2

```bash
ls -lah /var/www/api/cs2/v2/
```

### Verificar `health.json`

```bash
cat /var/www/api/cs2/v2/health.json
```

### Verificar lock global

```bash
ls -l /opt/cs2-portal/locks/
```

### Verificar statefiles

```bash
ls -lah /opt/cs2-portal/state/
```

### Validar integridade do Nginx após geração

```bash
sudo nginx -t
sudo systemctl status nginx
```

---

## Logs e diagnóstico

A fonte primária de diagnóstico da automação do host é:

- `systemctl`
- `journalctl`
- timestamps dos artefatos públicos
- inspeção dos diretórios operacionais

O padrão mínimo de diagnóstico deve observar:

- se o timer está habilitado e sendo disparado
- se o service correspondente executou com sucesso
- se houve erro no script operacional
- se o artefato esperado foi realmente atualizado
- se o problema é da automação, do pipeline, do filesystem ou do serving

Regra prática:

- timer “ativo” não significa resultado correto
- service “executado” não significa publicação íntegra
- a validação final precisa sempre tocar os outputs reais da camada pública

---

## Sinais de saúde da automação

Os sinais de saúde esperados incluem:

- timers visíveis e coerentes em `systemctl list-timers --all`
- services executando sem erro crítico recorrente
- atualização recente dos artefatos públicos da v2
- `health.json` coerente com a última geração esperada
- ausência de lock preso impedindo o ciclo
- ausência de statefiles claramente incoerentes
- ausência de erro recorrente em `journalctl` das units ligadas ao portal

A saúde da automação precisa ser inferida tanto pelas units quanto pelos efeitos visíveis no filesystem e no publishing.

---

## Problemas comuns

### 1. Timer existe, mas a v2 não atualiza

Causas comuns:

- service chamado pelo timer falha
- script operacional falha
- path ou permissão incorretos
- lock preso
- fonte de dados indisponível

Impacto:
- a camada pública fica stale
- o problema pode parecer “de cache” ou “de portal” quando a causa real é automação

---

### 2. Timer dispara concorrência indevida

Causas comuns:

- ausência ou quebra de lock
- reconfiguração ruim da cadência
- sobreposição de execução manual com automática

Impacto:
- artefato parcial
- state inconsistente
- troubleshooting confuso

---

### 3. Health atualiza, mas a v2 não

Causas comuns:

- unidade de health funcionando
- unidade da geração da v2 falhando
- checagens superficiais demais
- health não capturando o problema real

Impacto:
- falsa sensação de integridade do contexto

---

### 4. Service executa sem erro, mas não publica nada útil

Causas comuns:

- output indo para path errado
- write sem read público posterior
- statefile travado
- lógica do script pulando geração
- source data stale

Impacto:
- a automação parece saudável do ponto de vista do service
- a camada pública permanece antiga ou incompleta

---

### 5. Após reboot, a cadência esperada não volta

Causas comuns:

- timer não habilitado corretamente
- dependência de boot incompleta
- unit desabilitada
- estado do host alterado sem reconciliação

Impacto:
- o host volta “de pé”, mas deixa de manter a camada pública atualizada

---

## Sequência básica de diagnóstico

Quando houver suspeita de falha na automação do host, a sequência recomendada é:

1. listar timers e confirmar a presença dos relevantes
2. verificar status do timer suspeito
3. verificar status do service correspondente
4. ler `journalctl` da unit
5. verificar lock global
6. verificar statefiles
7. verificar timestamps dos artefatos públicos
8. validar `health.json`
9. validar um recurso público principal da v2

Essa sequência ajuda a responder rapidamente:

- o host disparou a rotina?
- a rotina executou?
- a rotina publicou?
- a publicação chegou ao público?

---

## Limites deste documento

Este documento não detalha:

- nomes finais de todas as units ainda não confirmadas no host
- conteúdo linha a linha dos unit files
- cronogramas exatos de cada timer sem reconciliação final
- troubleshooting detalhado de cada script do ETL
- recovery avançado de lock/state
- a lógica interna do AMP ou da instância CS2

Esses tópicos devem ser tratados em documentos complementares ou após validação adicional do ambiente real.

---

## Critério de pronto deste documento

Este documento pode ser considerado maduro quando:

- os timers e services reais do host estiverem confirmados sem ambiguidade
- a relação entre automação, ETL e outputs públicos estiver claramente reconciliada
- os comandos de validação refletirem a prática operacional real
- os riscos de timer saudável mas output ruim estiverem bem representados
- ele puder ser usado como referência da automação do host sem depender do master legado

---

## Última revisão

- Status: ativo
- Classificação: canônico
- Contexto: infraestrutura Hostinger / systemd automation
- Última revisão: 2026-03-18