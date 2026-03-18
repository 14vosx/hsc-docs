# Node Runtime via systemd

## Objetivo

Documentar a execução da Auth API em produção como serviço Node.js gerenciado pelo systemd no contexto AWS Lightsail.

Este documento existe para registrar, de forma estável e auditável:

- o nome oficial do serviço da aplicação
- o modo de execução do processo Node.js
- o usuário operacional do serviço
- os paths críticos do runtime
- a relação entre `systemd`, diretório da aplicação, arquivo `.env` e processo Node.js
- os comandos essenciais de validação e diagnóstico do serviço

---

## Escopo

Este documento cobre:

- o serviço `hsc-auth-api`
- o unit file do serviço
- usuário de execução
- diretório de trabalho
- arquivo de ambiente
- comando de inicialização
- política de reinício
- dependências de boot
- comandos operacionais básicos de `systemctl` e `journalctl`
- sintomas e falhas comuns do runtime da aplicação

Este documento não cobre em profundidade:

- configuração detalhada do Nginx
- DNS e TLS
- contratos HTTP da Auth API
- fluxo completo de deploy e rollback
- troubleshooting de banco em profundidade
- modelagem funcional da aplicação

Esses assuntos vivem em documentos próprios do contexto.

---

## Estado atual

O estado operacional conhecido do runtime Node.js da Auth API é:

- nome do serviço: `hsc-auth-api`
- gerenciador de serviço: `systemd`
- usuário de execução: `hscadmin`
- diretório de trabalho: `/opt/hsc/hsc-auth-api`
- arquivo de ambiente: `/opt/hsc/hsc-auth-api/.env`
- comando de execução: `/usr/bin/node index.js`
- política de reinício: `Restart=always`
- dependências observadas: `network.target` e `mariadb.service`
- health local esperado: `http://127.0.0.1:3000/health`

A Auth API roda como processo persistente no host Lightsail e não deve ser exposta diretamente ao público sem o edge Nginx.

---

## Source of truth / evidências

As evidências principais para este documento, nesta fase de migração documental, são:

- manual operacional da Auth API em AWS Lightsail
- documentação consolidada do ecossistema HSC
- blueprint técnico consolidado
- fluxos documentados de deploy e operação
- observações reconciliadas sobre unit file, working directory, `.env`, restart policy e health local

Enquanto a migração canônica do contexto não estiver 100% concluída, essas fontes continuam servindo como base de reconciliação.

---

## Relações com outros documentos

Este arquivo é complementar a:

- `docs/04-infra-aws-lightsail/README.md`
- `docs/04-infra-aws-lightsail/architecture-runtime.md`
- `docs/04-infra-aws-lightsail/nginx-reverse-proxy.md`
- `docs/04-infra-aws-lightsail/mariadb-local.md`
- `docs/04-infra-aws-lightsail/deploy-release-rollback.md`
- `docs/04-infra-aws-lightsail/auth-api-operations.md`
- `docs/04-infra-aws-lightsail/observability-troubleshooting.md`

Este documento descreve o runtime da aplicação sob `systemd`.  
Ele não substitui os documentos de edge, banco, deploy ou operação funcional.

---

## Papel do systemd neste contexto

No contexto AWS Lightsail, o `systemd` é o mecanismo oficial de gerenciamento do processo de aplicação da Auth API.

Suas responsabilidades são:

- iniciar a aplicação no boot
- manter o processo supervisionado
- reiniciar a aplicação em caso de falha
- expor status operacional consistente
- centralizar logs via `journalctl`
- padronizar a interação operacional com o serviço

Esse modelo reduz dependência de execução manual e fornece previsibilidade para operação, deploy e troubleshooting.

---

## Serviço oficial

O serviço oficial da aplicação é:

- `hsc-auth-api`

Esse nome deve ser tratado como referência canônica para:

- start
- stop
- restart
- status
- leitura de logs
- troubleshooting do runtime

---

## Unit file oficial

O unit file operacional conhecido deste contexto é:

- `/etc/systemd/system/hsc-auth-api.service`

Esse arquivo define o comportamento de execução da aplicação no host.

O conteúdo efetivo do unit file deve permanecer alinhado com os seguintes pontos operacionais conhecidos:

- usuário de execução: `hscadmin`
- diretório de trabalho: `/opt/hsc/hsc-auth-api`
- arquivo de ambiente: `/opt/hsc/hsc-auth-api/.env`
- comando de start: `/usr/bin/node index.js`
- reinício automático: `Restart=always`
- dependência de boot: rede e MariaDB disponíveis antes da subida completa do serviço

---

## Usuário operacional

O usuário operacional conhecido do serviço é:

- `hscadmin`

O serviço deve rodar sob esse usuário para manter consistência entre:

- permissões do diretório da aplicação
- acesso ao arquivo `.env`
- execução dos fluxos de deploy
- ownership esperado dos artefatos operacionais do contexto

Qualquer mudança de usuário operacional deve ser tratada como mudança estrutural e refletida na documentação canônica.

---

## Working directory

O diretório de trabalho oficial da aplicação é:

- `/opt/hsc/hsc-auth-api`

Este path é central para o funcionamento do serviço, pois dele dependem:

- leitura do `index.js`
- leitura de `package.json` e dependências instaladas
- resolução de paths relativos da aplicação
- execução de `npm ci` durante deploy
- alinhamento entre working tree, release ativa e restart do serviço

Se o serviço apontar para um diretório incorreto, o runtime real deixa de refletir a release esperada.

---

## Environment file

O arquivo de ambiente oficial conhecido é:

- `/opt/hsc/hsc-auth-api/.env`

Esse arquivo concentra a configuração operacional do runtime, incluindo, quando aplicável:

- conexão com banco
- configurações de CORS
- chaves administrativas
- demais variáveis necessárias para o funcionamento da Auth API

Regras importantes:

- o `.env` é um artefato sensível
- seu conteúdo não deve ser replicado em documentação canônica
- sua existência, papel e localização devem ser documentados
- falhas nesse arquivo costumam impactar subida da aplicação, banco, CORS e autenticação administrativa

---

## ExecStart

O comando de execução conhecido do serviço é:

- `/usr/bin/node index.js`

Esse modelo indica um runtime direto de Node.js executando o entrypoint principal da aplicação.

Implicações operacionais:

- o serviço depende da presença de `node` no path informado
- o entrypoint `index.js` precisa existir no working directory correto
- qualquer mudança de entrypoint deve ser refletida no unit file e na documentação

---

## Restart policy

A política conhecida de reinício do serviço é:

- `Restart=always`

Essa política existe para:

- reduzir indisponibilidade após falha de processo
- preservar a natureza persistente do backend
- tornar o runtime mais resiliente a falhas transitórias

Importante:
- reinício automático melhora disponibilidade
- mas também pode mascarar falhas repetitivas se o operador não consultar os logs
- por isso, `journalctl` continua sendo parte obrigatória do diagnóstico

---

## Dependências de boot

As dependências de boot observadas para o serviço incluem:

- `network.target`
- `mariadb.service`

Esse desenho busca garantir que:

- a rede local do host esteja funcional
- o banco MariaDB esteja disponível antes da aplicação depender dele

Se essas dependências estiverem incorretas, o serviço pode:

- subir cedo demais
- falhar em conexão com banco
- entrar em loop de restart
- apresentar health instável após reboot do host

---

## Porta e health local

O health local conhecido da aplicação é:

- `http://127.0.0.1:3000/health`

Isso indica que:

- a aplicação escuta localmente em porta interna
- o Nginx faz o papel de edge público
- o health local pode ser usado para isolar falhas entre:
  - edge Nginx
  - aplicação Node.js
  - dependências internas

Esse endpoint é importante para troubleshooting porque permite testar a aplicação sem depender da borda pública.

---

## Fluxo operacional do serviço

O fluxo operacional esperado deste runtime é:

1. o host sobe
2. `systemd` carrega a unit do serviço
3. após rede e MariaDB disponíveis, o serviço é iniciado
4. o processo Node.js sobe a aplicação
5. a aplicação responde localmente
6. o Nginx encaminha tráfego público para esse processo
7. em caso de falha do processo, o `systemd` tenta reiniciar automaticamente

Esse fluxo define o comportamento básico de disponibilidade da Auth API.

---

## Comandos operacionais

Os comandos operacionais básicos deste contexto devem usar sempre o nome canônico do serviço.

### Ver status do serviço

```bash
sudo systemctl status hsc-auth-api
```

### Iniciar o serviço

```bash
sudo systemctl start hsc-auth-api
```

### Parar o serviço

```bash
sudo systemctl stop hsc-auth-api
```

### Reiniciar o serviço

```bash
sudo systemctl restart hsc-auth-api
```

### Recarregar units após alteração do unit file

```bash
sudo systemctl daemon-reload
```

### Habilitar serviço no boot

```bash
sudo systemctl enable hsc-auth-api
```

### Ver logs do serviço

```bash
sudo journalctl -u hsc-auth-api -f
```

### Ver health local da aplicação

```bash
curl -sS http://127.0.0.1:3000/health
```

---

## Logs e diagnóstico

A fonte primária de diagnóstico do serviço é:

- `journalctl`

O comando operacional principal de acompanhamento é:

```bash
sudo journalctl -u hsc-auth-api -f
```

Esse log deve ser a primeira fonte de leitura quando houver sintomas como:

- aplicação não sobe
- restart em loop
- erro de conexão com banco
- falha de leitura do `.env`
- módulo ausente
- porta interna indisponível
- comportamento inconsistente após deploy

O `status` do serviço é útil para triagem rápida, mas o diagnóstico real tende a depender do journal.

---

## Sinais de saúde do serviço

Os sinais de saúde esperados deste runtime incluem:

- `systemctl status hsc-auth-api` indicando serviço ativo
- `journalctl` sem erros repetitivos críticos
- resposta válida em `http://127.0.0.1:3000/health`
- upstream funcionando por trás do Nginx
- aplicação conseguindo acessar o MariaDB local quando necessário

Um serviço “ativo” no `systemd` não é suficiente por si só.  
O health local e o comportamento dos endpoints continuam sendo parte do diagnóstico.

---

## Problemas comuns

### 1. Serviço não sobe

Causas comuns:

- unit file incorreta
- working directory errado
- `index.js` ausente
- `node` indisponível no path esperado
- `.env` inválido
- dependência crítica da aplicação quebrada

Sinais típicos:

- `systemctl status` mostra falha
- `journalctl` mostra erro de start

---

### 2. Serviço sobe e cai em loop

Causas comuns:

- erro interno na inicialização da aplicação
- falha de conexão com banco
- exceção fatal no bootstrap
- configuração inconsistente
- dependência ausente

Sinais típicos:

- `Restart=always` provoca tentativas repetidas
- `journalctl` mostra padrão repetido de crash e restart

---

### 3. Health local falha

Causas comuns:

- aplicação não escutando na porta esperada
- serviço não ativo
- bootstrap incompleto
- crash logo após start

Sinais típicos:

- `curl http://127.0.0.1:3000/health` falha
- edge público pode responder erro de upstream

---

### 4. Serviço sobe, mas app não acessa banco

Causas comuns:

- MariaDB indisponível
- credenciais erradas no `.env`
- bind local incorreto
- schema ou database esperado ausente

Sinais típicos:

- health parcial ou endpoints degradados
- logs com erro de conexão ou autenticação no banco

---

### 5. Após deploy, serviço não reflete a versão esperada

Causas comuns:

- working directory divergente
- deploy incompleto
- restart não executado corretamente
- TAG/worktree não alinhadas com o runtime real

Sinais típicos:

- código do diretório e comportamento observado divergem
- endpoints não refletem a mudança esperada

---

## Sequência básica de diagnóstico

Quando houver incidente no runtime da aplicação, a sequência recomendada é:

1. verificar status do serviço
2. verificar logs via `journalctl`
3. testar health local em `127.0.0.1:3000`
4. validar working directory e integridade do `.env`
5. validar disponibilidade do MariaDB
6. confirmar se a release esperada está realmente presente no diretório da aplicação
7. só depois investigar edge Nginx e comportamento público

Essa sequência ajuda a distinguir rapidamente se o problema está em:

- processo Node.js
- configuração local
- banco
- deploy
- ou camada de proxy

---

## Limites deste documento

Este documento não detalha:

- configuração textual do unit file linha a linha
- reverse proxy do Nginx
- DNS e TLS
- política completa de CORS
- fluxo completo de deploy e rollback
- backup do banco
- modelo funcional da Auth API

Esses assuntos são documentados em arquivos próprios do contexto.

---

## Critério de pronto deste documento

Este documento pode ser considerado maduro quando:

- os parâmetros principais do unit file estiverem confirmados no host real
- o nome do serviço, usuário, working directory e `.env` estiverem validados
- a política de restart e dependências de boot estiverem consistentes com o runtime real
- os comandos operacionais refletirem exatamente a prática de produção
- ele puder ser usado como runbook de serviço sem depender do master legado

---

## Última revisão

- Status: ativo
- Classificação: canônico
- Contexto: infraestrutura AWS Lightsail / runtime Node.js via systemd
- Última revisão: 2026-03-18