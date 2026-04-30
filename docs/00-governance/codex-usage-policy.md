# Política de Uso do Codex por Contexto — Projeto HSC

## Status

Este documento define como usar o Codex no desenvolvimento do Projeto HSC sem misturar responsabilidades de arquitetura, operação, deploy e implementação.

O objetivo é permitir que o Codex seja usado como **executor técnico local** em tarefas bem delimitadas, enquanto decisões de arquitetura, produto, segurança, deploy e operação continuam sob controle humano.

## Escopo

Esta política se aplica ao ecossistema HSC documentado em:

```text
docs/00-governance
docs/01-infra-hostinger
docs/02-game-panel
docs/03-portal-estatico
docs/04-infra-aws-lightsail
docs/05-backoffice-admin
docs/95-impl-log
docs/97-audit
docs/98-legacy
```

Ela também orienta o uso dos arquivos `AGENTS.md` nos repositórios de implementação, como:

```text
hsc-cs2-portal/AGENTS.md
hsc-cs2-portal/frontend/angular/AGENTS.md
```

## Princípio central

O Codex deve ser tratado como executor de desenvolvimento e automação local, não como decisor de arquitetura.

A separação recomendada é:

```text
ChatGPT / humano:
- arquitetura
- priorização
- tradeoffs
- produto
- UX estratégica
- deploy
- rollback
- cutover
- mudanças de API/ETL
- mudanças de Nginx/systemd
- decisões operacionais sensíveis

Codex:
- edição de código com escopo fechado
- refactors pequenos
- criação de componentes
- ajustes de CSS/HTML/TS
- testes locais
- build local
- correção de lint/build
- documentação localizada
- scripts auxiliares read-only
```

## Regras globais

O Codex não deve decidir sozinho:

```text
criar ou alterar contratos de API
alterar ETL em produção
alterar Nginx
alterar systemd
alterar DNS/TLS/firewall
executar cutover
executar deploy
executar rollback
reiniciar serviços sensíveis
mudar estrutura de repositórios
adicionar dependências sem aprovação
mexer no Game Panel real sem plano validado
```

Quando uma tarefa tocar algum desses pontos, o Codex deve parar e pedir decisão humana.

## Relação com AGENTS.md

Os arquivos `AGENTS.md` servem como contexto fixo para agentes de código dentro de um repositório.

Regras:

- cada repositório de implementação pode ter um `AGENTS.md` na raiz;
- subpastas com stack específica podem ter `AGENTS.md` próprio;
- instruções mais próximas do arquivo editado são mais específicas;
- prompts diretos da tarefa têm prioridade sobre instruções gerais;
- decisões estratégicas não devem ser empurradas para o Codex via `AGENTS.md`.

Exemplo atual no portal Angular:

```text
hsc-cs2-portal/AGENTS.md
hsc-cs2-portal/frontend/angular/AGENTS.md
```

Esses arquivos são adequados porque orientam o Codex dentro do codebase específico do portal.

## Matriz de uso por contexto

| Contexto | Uso de Codex | Ambiente local | Diretriz |
|---|---:|---:|---|
| `00-governance` | Baixo/médio | Não aplicável | Usar para documentação, índices, manutenção e reconciliação |
| `01-infra-hostinger` | Baixo/médio | Parcial/não real | Usar para diagnóstico e runbooks; não executar operação sensível sem aprovação |
| `02-game-panel` | Médio controlado | Não equivalente | Usar para diagnóstico e análise; não alterar servidor CS2/AMP livremente |
| `03-portal-estatico` | Alto | Sim/parcial | Usar para Angular, Static API v2, ETL local, docs e validações |
| `04-infra-aws-lightsail` | Médio/alto | Sim | Usar para Auth API local, migrations, testes e scripts; deploy por TAG exige aprovação |
| `05-backoffice-admin` | Alto | Sim | Usar para SPA admin, guards, serviços, telas e integrações locais |
| `95-impl-log` | Baixo | Não aplicável | Usar para síntese e organização histórica |
| `97-audit` | Baixo/médio | Não aplicável | Usar para transformar pendências em backlog/checklists |
| `98-legacy` | Baixo | Não aplicável | Usar apenas para extração/reconciliação controlada |

## Contexto 00 — Governance

### Papel

Governança documental do projeto HSC.

Inclui:

```text
índices
sistema de documentação
playbooks de manutenção
regras de navegação documental
política de preservação de legado
```

### Uso recomendado do Codex

Permitido:

```text
corrigir links
atualizar índices
criar documentos de política
organizar seções
padronizar headings
gerar checklists documentais
```

Não permitido sem aprovação:

```text
mudar a taxonomia dos contextos
promover documento legado para fonte canônica
remover documentos
renomear diretórios de contexto
reestruturar a governança
```

### Ambiente local

Não há ambiente de execução. O trabalho é documental.

## Contexto 01 — Infra Hostinger

### Papel

Infraestrutura da VPS Hostinger que sustenta:

```text
Nginx
Certbot/TLS
Docker host
systemd
webroots públicos
Static API
portal público
Game Panel host
```

### Uso recomendado do Codex

Permitido:

```text
gerar comandos read-only
montar checklists de auditoria
comparar outputs
produzir runbooks
documentar validações HTTP
validar permissões de forma read-only
```

Exemplos aceitáveis:

```text
curl -I
nginx -t apenas quando explicitamente pedido
ls/stat/find para auditoria
grep em configs/logs
```

Não permitido sem aprovação explícita:

```text
editar arquivos Nginx
rodar systemctl reload/restart
alterar firewall
alterar Certbot/TLS
alterar webroot público
executar deploy
executar rollback
```

### Ambiente local

Não existe equivalente local completo. A fonte de verdade operacional é a VPS.

## Contexto 02 — Game Panel

### Papel

Runtime do servidor CS2 via AMP/CubeCoders.

Inclui:

```text
MixHAXIXE01
CS2 server
MetaMod
CounterStrikeSharp
MatchZy
SimpleAdmin
plugins
configs operacionais
runbooks de troubleshooting
```

### Uso recomendado do Codex

Permitido:

```text
gerar comandos de diagnóstico
analisar logs/outputs colados
comparar configs
criar checklists de plugins
documentar hipóteses
sugerir plano de rollback
```

Não permitido sem aprovação:

```text
alterar plugins em produção
remover plugin
reiniciar instância
executar update
editar configs críticas
alterar AMP
alterar MatchZy em produção
```

### Ambiente local

Não há ambiente local equivalente documentado. Análise local de arquivos é possível, mas validação real depende da instância.

## Contexto 03 — Portal Estático

### Papel

Camada pública de dados e portal.

Inclui:

```text
MatchZy SQLite
ETL Bash
Static API v2
JSON materializado
portal legado
portal Angular cs2-next
conteúdo público
Nginx static serving
```

### Uso recomendado do Codex

Este é um dos melhores contextos para uso do Codex.

Permitido com escopo fechado:

```text
implementar componentes Angular
ajustar HTML/CSS/TS
criar DTOs e services
corrigir build/lint
refatorar páginas
melhorar UX localizada
ajustar scripts ETL localmente
criar validações de contrato
gerar fixtures
atualizar documentação contextual
```

Não permitido sem aprovação:

```text
alterar contratos da Static API v2
criar endpoints novos
mudar ETL de produção
publicar staging
publicar produção
alterar Nginx
executar cutover
```

### Ambiente local

Existe ambiente local útil.

Para Angular:

```bash
cd frontend/angular
npm run start:proxy
npm run build
```

Para ETL/Static API v2, a documentação registra reprodução local com ferramentas como:

```text
bash
git
sqlite3
jq
python3
curl
snapshot local do matchzy.db
```

## Contexto 04 — Infra AWS Lightsail

### Papel

Infraestrutura dinâmica da Auth API.

Inclui:

```text
Node/Express
MariaDB
Nginx reverse proxy
systemd
deploy por TAG
migrations
backup/restore
smoke tests
```

### Uso recomendado do Codex

Permitido:

```text
implementar endpoints localmente
ajustar migrations
criar smoke tests
refatorar serviços
corrigir validações
documentar contratos
gerar scripts auxiliares
```

Não permitido sem aprovação:

```text
deploy em produção
criar TAG de release
rodar migrations em produção
reiniciar serviço remoto
alterar Nginx
alterar banco remoto
executar rollback
```

### Ambiente local

Existe ambiente local documentado, com:

```text
.env.local
MariaDB local via Docker Compose
npm run db:migrate
smoke local
```

## Contexto 05 — Backoffice Admin

### Papel

SPA administrativa do HSC.

Inclui:

```text
login
magic link
auth callback
dashboard
guards
RBAC
news admin
users admin
contratos /admin
proxy local
integração com Auth API
```

### Uso recomendado do Codex

Este é um dos melhores contextos para Codex.

Permitido:

```text
criar telas
criar componentes
ajustar guards
implementar services
integrar contratos existentes
corrigir formulários
melhorar UX
criar testes/smokes locais
refatorar CSS/HTML/TS
```

Não permitido sem aprovação:

```text
alterar contratos da Auth API
alterar política de autenticação
mudar RBAC
mudar deploy
mudar proxy de produção
```

### Ambiente local

Existe ambiente local documentado.

Elementos conhecidos:

```text
proxy.conf.json
/auth
/admin
/content
/health
POST /auth/dev/bootstrap-session
Auth API local
Backoffice local
```

## Contexto 95 — Impl Log

### Papel

Histórico incremental de implementação.

### Uso recomendado do Codex

Permitido:

```text
resumir ciclos
adicionar entradas factuais
organizar datas
extrair decisões
criar índice de implementação
```

Não permitido:

```text
reescrever histórico como se fosse estado atual
apagar registros sem aprovação
promover log a fonte canônica sem reconciliação
```

### Ambiente local

Não aplicável.

## Contexto 97 — Audit

### Papel

Pendências, inconsistências, riscos e reconciliações.

### Uso recomendado do Codex

Permitido:

```text
converter achados em backlog
organizar pendências
criar checklists
cruzar docs
identificar lacunas
```

Não permitido:

```text
resolver arquitetura sozinho
remover pendências sem evidência
marcar item como fechado sem validação
```

### Ambiente local

Não aplicável.

## Contexto 98 — Legacy

### Papel

Acervo histórico e documentos antigos.

### Uso recomendado do Codex

Permitido:

```text
extrair trechos relevantes
comparar com docs canônicos
propor reconciliação
resumir diferenças
```

Não permitido:

```text
tratar legado como fonte canônica automaticamente
copiar decisões antigas sem validação
reescrever docs atuais com base apenas no legado
```

### Ambiente local

Não aplicável.

## Critérios para decidir se uma tarefa vai para Codex

Use Codex quando a tarefa tiver:

```text
escopo de arquivos claro
entrada e saída verificáveis
comando local de validação
baixo risco operacional
contrato já definido
decisão arquitetural já tomada
```

Não use Codex sozinho quando a tarefa envolver:

```text
ambiguidade de produto
arquitetura
deploy
rollback
infra remota
banco de produção
contratos públicos
segurança
cutover
```

## Prompt padrão para Codex

Use prompts curtos e fechados.

Modelo:

```text
Leia AGENTS.md e, se existir, o AGENTS.md mais específico da pasta.

Tarefa:
<descrição objetiva>

Escopo permitido:
<arquivos ou diretórios>

Não fazer:
- não alterar API
- não alterar deploy
- não alterar Nginx
- não adicionar dependências
- não mudar contratos
- não mexer fora do escopo

Validação:
- rode <comando>
- reporte git diff --stat
- reporte eventuais warnings/erros

Antes de editar:
explique o plano em até 5 bullets.
```

## Política para deploy

O Codex não deve executar deploy.

Ele pode auxiliar a preparar comandos, mas a execução deve ser conduzida por humano com runbook.

Para Angular `cs2-next`, o deploy seguro continua sendo:

```text
build local com baseHref
auditoria de browser/*
tar.gz
SHA256
upload para inbox remoto
extração em workdir
snapshot de staging
rsync --delete
validação HTTP interna
validação HTTP externa
validação visual
```

## Política para documentação

Codex pode criar ou atualizar documentação desde que:

```text
o arquivo-alvo esteja claro
a fonte da informação esteja clara
não reescreva documentos inteiros sem necessidade
não altere encoding/line endings em massa
rode git diff --check
```

Para docs transversais do projeto HSC, preferir o repositório:

```text
hsc-docs
```

Para docs locais do portal Angular, usar:

```text
hsc-cs2-portal/docs
```

## Estado recomendado

Manter esta política como documento canônico de governança.

Arquivos de contexto operacional específicos devem continuar nos repositórios de implementação via `AGENTS.md`.

## Próxima revisão

Revisar esta política quando:

```text
novos repositórios forem adicionados
novos ambientes locais forem formalizados
MCP passar a ser usado no projeto
o Codex ganhar permissões adicionais
houver automação de deploy
```
