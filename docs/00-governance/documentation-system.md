# Documentation System

## Objetivo

Definir o sistema oficial de documentação do ecossistema HSC.

Este documento existe para registrar, de forma estável e auditável:

- como a documentação do HSC é organizada
- quais são os contextos canônicos
- como novos documentos devem ser criados
- como manter a documentação alinhada ao runtime real
- como tratar mudanças, lacunas, conflitos e material legado
- como escrever documentação técnica útil, honesta e sustentável

---

## Escopo

Este documento cobre:

- organização por contexto
- estrutura do repositório documental
- regras de escrita e manutenção
- convenções de classificação dos documentos
- relação entre documentos canônicos, impl-logs, auditorias e legado
- tratamento de fatos confirmados, parcialmente confirmados e não confirmados
- regras de atualização e revisão

Este documento não cobre em profundidade:

- conteúdo técnico de um contexto específico
- runbooks operacionais de uma camada específica
- decisões arquiteturais individuais
- histórico completo de mudanças do ecossistema

Esses tópicos vivem nos contextos próprios, ADRs, impl-logs e auditorias.

---

## Princípio central

A documentação do HSC é organizada por **contexto**.

A unidade principal da documentação não é mais “um master grande que tenta falar de tudo”.

A unidade principal passa a ser:

- um contexto com fronteira clara
- um conjunto pequeno de documentos especializados
- um inventário explícito das suas evidências, paths, componentes e pendências
- uma relação clara com outros contextos do ecossistema

Regra canônica:

**cada documento deve existir para reduzir ambiguidade operacional real**

Se um arquivo não reduz ambiguidade, ele provavelmente não precisa existir naquele formato.

---

## Contextos canônicos iniciais

Os contextos canônicos iniciais do HSC são:

1. Infra Hostinger
2. Game Panel
3. Portal Estático
4. Infra AWS Lightsail

Eles correspondem às quatro áreas operacionais principais do ecossistema neste estágio.

---

## Estrutura oficial do repositório

A estrutura oficial do repositório documental é:

```text
docs/
├── 00-governance/
├── 01-infra-hostinger/
├── 02-game-panel/
├── 03-portal-estatico/
├── 04-infra-aws-lightsail/
├── 90-adr/
├── 95-impl-log/
├── 97-audit/
└── 98-legacy/
```

---

## Papel de cada diretório

## `00-governance/`

Contém os documentos que governam o sistema documental.

Exemplos:

- `README.md`
- `documentation-system.md`
- `99-master-index.md`

Função:
- explicar como a documentação funciona
- orientar navegação
- estabelecer regras de manutenção

---

## `01-infra-hostinger/`

Documenta a camada-base do lado Hostinger.

Função:
- Debian
- Nginx
- Docker host
- Certbot
- systemd
- filesystem e permissões
- observabilidade da infraestrutura base

---

## `02-game-panel/`

Documenta a operação do servidor CS2.

Função:
- AMP
- instância `MixHAXIXE01`
- MatchZy
- plugins
- configuração do servidor
- runbooks do lado jogo

---

## `03-portal-estatico/`

Documenta a camada pública estática do ecossistema.

Função:
- Static API v2
- ETL Bash
- fonte SQLite do MatchZy
- contratos JSON
- frontend estático
- publishing via Nginx

---

## `04-infra-aws-lightsail/`

Documenta a camada dinâmica da Auth API.

Função:
- host Lightsail
- Node.js via systemd
- MariaDB local
- deploy/release/rollback
- operações da Auth API
- observabilidade do backend dinâmico

---

## `90-adr/`

Reservado para Architecture Decision Records.

Função:
- registrar decisões arquiteturais relevantes
- preservar contexto de decisão
- evitar reabrir a mesma discussão sem histórico

---

## `95-impl-log/`

Reservado para impl-logs.

Função:
- registrar mudanças incrementais
- manter rastreabilidade sem poluir documentos canônicos
- permitir evolução contínua do sistema documental

---

## `97-audit/`

Reservado para auditorias e gap analysis.

Função:
- registrar divergências entre docs e runtime
- listar pendências
- consolidar verificações e inconsistências

---

## `98-legacy/`

Reservado para material legado.

Função:
- preservar documentação histórica
- evitar perda de contexto durante a migração
- manter apoio histórico sem deixar o legado governar o canônico

---

## Tipos de documento

Os tipos principais de documento no HSC são:

### Documento canônico

É o documento que representa a verdade oficial vigente de um contexto ou subtema.

Exemplos:
- `architecture-runtime.md`
- `network-dns-tls.md`
- `matchzy.md`
- `static-api-v2.md`

Critérios:
- deve ser legível sozinho
- deve ter escopo claro
- deve refletir o estado real reconciliado
- deve evitar improviso e ambiguidade

---

### Documento de inventário e referência

É o documento que concentra:

- fontes
- artefatos
- paths
- comandos
- pendências
- limites do contexto

Exemplo:
- `references-inventory.md`

Critério:
- serve como checklist de realidade do contexto

---

### Runbook operacional

É o documento orientado à ação repetível.

Exemplos:
- `operational-runbooks.md`
- `observability-troubleshooting.md`

Critério:
- deve servir para operar ou diagnosticar o sistema
- deve evitar floreio e abstração excessiva

---

### Documento de governança

Explica como o sistema documental funciona.

Exemplos:
- `documentation-system.md`
- `99-master-index.md`

---

### ADR

Registra decisão arquitetural.

Critério:
- não descreve o sistema inteiro
- descreve uma decisão específica, seu contexto e consequências

---

### Impl-log

Registra mudança incremental.

Critério:
- serve como trilha de evolução
- não substitui documento canônico
- ajuda futura reconciliação

---

### Documento legado

É material histórico preservado.

Critério:
- pode continuar útil
- não governa o canônico sem reconciliação explícita

---

## Ordem recomendada de documentação de um contexto

Quando um novo contexto for estruturado, a ordem recomendada é:

1. `README.md`
2. `architecture-runtime.md`
3. documentos estruturais centrais do contexto
4. `operational-runbooks.md`
5. `observability-troubleshooting.md`
6. `references-inventory.md`

Essa ordem existe porque:

- primeiro define a fronteira do contexto
- depois define sua topologia
- depois define subcamadas
- depois define operação
- depois define triagem
- por fim consolida o inventário

---

## Convenções de nome de arquivo

As convenções oficiais são:

- nomes em lowercase
- usar kebab-case
- nomes descritivos, sem abreviação desnecessária
- evitar nomes genéricos como `misc.md`, `notes.md`, `temp.md`

Exemplos corretos:

- `architecture-runtime.md`
- `network-dns-tls.md`
- `nginx-static-serving.md`
- `observability-troubleshooting.md`
- `references-inventory.md`

Exemplos ruins:

- `coisas.md`
- `geral.md`
- `misc.md`
- `novo-doc.md`

---

## Convenções de escrita

A documentação do HSC deve seguir estes princípios de escrita:

- escopo explícito
- linguagem direta
- foco técnico
- honestidade sobre incerteza
- separação clara entre confirmado e não confirmado
- evitar “achismos” ou preenchimento artificial de lacunas
- priorizar utilidade operacional sobre verbosidade ornamental

Cada documento deve, preferencialmente, conter:

- objetivo
- escopo
- estado atual
- source of truth / evidências
- relações com outros documentos
- corpo principal do tema
- limites do documento
- critério de pronto
- última revisão

---

## Regra de honestidade documental

A documentação do HSC não deve fingir certeza.

Quando algo não estiver validado, o documento deve dizer explicitamente:

- o que está confirmado
- o que está parcialmente confirmado
- o que ainda não está confirmado
- o que precisa ser verificado no ambiente real

Regra canônica:

**lacuna explícita é melhor do que precisão inventada**

---

## Como tratar fatos parcialmente confirmados

Esta seção define a regra editorial oficial para casos em que existe evidência relevante, mas ainda não evidência suficiente para promover um fato a “verdade canônica consolidada”.

### Quando um fato é parcialmente confirmado

Um fato deve ser tratado como **parcialmente confirmado** quando ocorrer algo como:

- existe evidência histórica, mas não validação recente
- existe um print, log ou conversa técnica antiga, mas não confirmação no runtime atual
- existe uso auxiliar de um componente, mas não confirmação de que ele é estrutural
- existe indício forte de topologia ou dependência, mas ainda falta fechar o ambiente real
- existe comportamento conhecido, mas ainda não foi reconciliado de ponta a ponta

### Como escrever nesses casos

O documento deve separar explicitamente:

- **o que está confirmado**
- **o que está parcialmente evidenciado**
- **o que não está confirmado**
- **qual é a leitura canônica atual**
- **quando o documento deve ser expandido**

### O que não fazer

Não fazer o seguinte:

- promover evidência parcial a arquitetura oficial
- escrever como “camada central” algo que pode ser apenas integração auxiliar
- completar lacunas com suposição só para o documento parecer completo
- misturar estado atual confirmado com hipótese futura

### Linguagem recomendada

Usar formulações como:

- “o estado atual reconciliado indica...”
- “há evidência histórica de...”
- “isso sugere que..., mas ainda não consolida...”
- “não há, nesta fase documental, confirmação suficiente para tratar X como camada central”
- “a leitura canônica atual deve ser...”

Evitar formulações como:

- “claramente é...”
- “com certeza roda assim...”
- “provavelmente é o padrão oficial...” sem evidência
- “vamos assumir que...”

### Padrão editorial recomendado

Quando o tema for importante, mas ainda parcialmente confirmado, o documento deve preferir uma estrutura como:

1. objetivo
2. estado atual
3. o que está confirmado
4. o que está parcialmente evidenciado
5. o que não está confirmado
6. leitura canônica atual
7. quando o documento deve ser expandido
8. validações úteis para próxima revisão

### Exemplo de aplicação

O arquivo `docs/02-game-panel/mariadb-runtime.md` passa a ser um **padrão editorial válido** para este tipo de caso.

Ele mostra como:

- reconhecer evidência histórica real
- não inventar centralidade arquitetural sem prova
- proteger a qualidade do repositório documental
- deixar uma trilha clara para futura expansão

Regra canônica:

**fato parcialmente confirmado deve ser documentado como parcialmente confirmado, não como verdade final e nem como silêncio total**

---

## Fonte de verdade

A ordem de prioridade da fonte de verdade deve ser:

1. ambiente real validado
2. documento canônico mais recente e reconciliado
3. impl-log relevante
4. documentação histórica útil
5. memória humana ou conversa antiga

Regra importante:

- se o runtime real contradiz documento antigo, o runtime validado prevalece
- a documentação deve ser atualizada para refletir isso

---

## Relação entre canônico, impl-log e legado

### Documento canônico

Responde:
- “como isso é oficialmente hoje?”

### Impl-log

Responde:
- “o que mudou, quando e por quê?”

### Legado

Responde:
- “como isso já foi entendido ou documentado antes?”

Essas três camadas não competem; elas se complementam.

---

## Quando criar um novo documento

Criar novo documento quando:

- o tema tiver fronteira clara
- houver complexidade suficiente para justificar separação
- a separação reduzir ambiguidade
- a manutenção futura ficar mais fácil com a divisão

Não criar novo documento quando:

- o assunto é pequeno demais
- ele caberia de forma limpa em um documento já existente
- a nova divisão só aumentaria fragmentação sem ganho real

---

## Quando atualizar um documento existente

Atualizar documento existente quando:

- houve mudança real no runtime
- o documento ficou desatualizado
- uma pendência foi resolvida
- um path, componente ou fluxo mudou
- uma lacuna importante foi confirmada no ambiente

Regra canônica:

- atualização deve privilegiar alinhamento com o runtime
- não deve ser feita só para “embelezar” o texto

---

## Como lidar com conflitos entre documentos

Quando dois documentos parecerem conflitar:

1. identificar qual é o documento canônico do tema
2. verificar a data da última revisão
3. verificar se há impl-log mais recente
4. verificar o runtime real
5. atualizar o documento correto
6. registrar a reconciliação se necessário

Regra importante:

- conflito documental não deve ser resolvido por preferência pessoal
- deve ser resolvido por evidência

---

## Como lidar com pendências

Pendências não devem ficar implícitas.

Quando houver pendência relevante, ela deve aparecer em local apropriado, como:

- `references-inventory.md`
- `97-audit/`
- documento específico do contexto
- impl-log, quando a pendência nasceu de mudança recente

Exemplos de pendência legítima:

- hostname público ainda não confirmado
- unit file ainda não validado no host
- path final de banco ainda não fixado
- inventário de plugins ainda não capturado no runtime atual

---

## Como lidar com material sensível

O repositório documental não deve conter:

- credenciais
- tokens
- secrets
- senhas
- strings de conexão sensíveis
- arquivos privados de acesso

Pode conter:

- paths
- nomes de serviços
- nomes de tabelas
- nomes de documentos
- fluxos operacionais
- arquitetura
- comandos de validação seguros

Regra canônica:

**documentar arquitetura e operação não é o mesmo que vazar segredo**

---

## Como manter a documentação viva

A documentação do HSC deve ser mantida por fluxo contínuo, não por reescritas gigantes e esporádicas.

Fluxo recomendado:

1. mudança acontece no runtime
2. impl-log registra a mudança relevante
3. documento canônico do contexto é revisado
4. inventário e índice são atualizados, se necessário
5. pendências remanescentes são registradas explicitamente

Esse fluxo reduz:

- perda de contexto
- dependência de memória
- reescrita traumática posterior
- acúmulo de drift documental

---

## Critério de qualidade documental

Um documento do HSC é considerado bom quando:

- tem escopo claro
- reduz ambiguidade real
- está alinhado ao runtime validado
- é honesto sobre lacunas
- aponta relações corretas com outros documentos
- pode ser usado por outra pessoa sem depender do autor original
- não tenta parecer “completo” às custas de inventar certeza

---

## Anti-padrões proibidos

Os principais anti-padrões documentais a evitar são:

- master único tentando explicar tudo
- copiar texto antigo sem reconciliação
- escrever topologia por memória
- esconder incerteza
- misturar backlog com documentação canônica
- misturar hipótese com estado atual
- inventar paths, hostnames ou serviços não validados
- deixar runbook crítico depender só de conversa informal

---

## Critério de manutenção deste documento

Este documento deve ser atualizado quando houver:

- mudança na estrutura do repositório documental
- abertura de novo contexto canônico
- mudança nas regras editoriais
- mudança na relação entre canônico, impl-log, audit e legacy
- necessidade de formalizar novo padrão editorial recorrente

---

## Critério de pronto deste documento

Este documento pode ser considerado saudável quando:

- explica claramente como o sistema documental do HSC funciona
- descreve a estrutura por contexto sem ambiguidade
- define regras de escrita e manutenção utilizáveis
- formaliza honestidade documental como padrão
- deixa explícito como tratar fatos parcialmente confirmados
- serve como referência operacional para manter o repositório coerente ao longo do tempo

---

## Última revisão

- Status: ativo
- Classificação: canônico
- Contexto: governança / sistema documental
- Última revisão: 2026-03-18