# HSC Docs Maintenance Playbook

## Objetivo

Este playbook define a rotina operacional de manutenção do repositório documental `hsc-docs`.

Ele consolida regras simples para:

- criação de novos documentos
- manutenção da navegação
- redução de ambiguidade nominal
- correção de nós soltos no Graph
- operação segura com Git + Obsidian na mesma base de arquivos

Este documento complementa o `documentation-system.md`.  
Ele não substitui as regras canônicas de governança documental.

---

## Navegação

### Entrada
- [Home da documentação](../README.md)
- [Governance — README](./README.md)
- [Documentation System](./documentation-system.md)
- [Master Index](./99-master-index.md)

### Relação com a governança
- [Documentation System](./documentation-system.md)
- [Governance — README](./README.md)

### Contextos canônicos
- [Infra Hostinger](../01-infra-hostinger/README.md)
- [Game Panel](../02-game-panel/README.md)
- [Portal Estático](../03-portal-estatico/README.md)
- [Infra AWS Lightsail](../04-infra-aws-lightsail/README.md)
- [Backoffice Admin](../05-backoffice-admin/README.md)

---

## 1. Princípio-base

O `hsc-docs` é simultaneamente:

- repositório Git canônico
- Vault do Obsidian
- documentação viva organizada por contexto

A verdade documental continua distribuída por contexto.  
O Obsidian é uma camada de navegação e leitura, não uma segunda fonte de verdade.

---

## 2. Regra de ouro para novos documentos

Todo documento importante novo deve nascer com navegação explícita.

### Estrutura mínima recomendada

Cada novo `.md` deve incluir, perto do topo:

- link para `docs/README.md`
- link para o `README.md` do contexto
- link para `docs/00-governance/99-master-index.md`
- 2 a 4 links para documentos tecnicamente adjacentes

### Intenção

Se um documento novo nascer sem isso, ele tende a virar nó solto no Graph e fica mais difícil de percorrer fora da árvore de pastas.

---

## 3. Regra de criação por contexto

Novos documentos devem ser criados sempre no contexto canônico correto.

### Referência rápida

- `01-infra-hostinger` → host base, Nginx, systemd, filesystem, TLS
- `02-game-panel` → operação do servidor, plugins, MatchZy, runtime do game panel
- `03-portal-estatico` → ETL, dados, Static API v2, frontend público, publicação
- `04-infra-aws-lightsail` → Auth API, runtime dinâmico, reverse proxy, Node, MariaDB local
- `05-backoffice-admin` → SPA administrativa, contratos, guards, consumo da Auth API

Evitar criar documentos “transversais” fora de contexto sem necessidade real.

---

## 4. Regra de navegação

Ao criar ou revisar um documento, considerar sempre três níveis de leitura:

### Entrada global
- `docs/README.md`

### Hub local
- `README.md` do contexto

### Fluxo técnico real
- entradas
- dependências
- saídas
- operação
- consumo

Os links devem refletir relações reais do sistema, e não apenas a árvore de diretórios.

---

## 5. Regra de nomenclatura

Usar nomes simples quando forem naturalmente únicos.

Usar prefixo de contexto quando o nome puder se repetir entre contextos.

### Exemplos recomendados

- `portal-estatico-frontend-structure.md`
- `infra-hostinger-network-dns-tls.md`
- `backoffice-admin-references-inventory.md`
- `game-panel-operational-runbooks.md`

### Exemplos a evitar, quando houver chance de repetição

- `frontend-structure.md`
- `references-inventory.md`
- `observability-troubleshooting.md`
- `operational-runbooks.md`

---

## 6. Exceção consciente: README.md

`README.md` continua sendo exceção válida.

### Motivos

- funciona bem no GitHub
- funciona bem como hub por pasta
- tem papel estrutural no repositório
- reduz atrito na navegação contextual

Não renomear `README.md` sem motivo muito forte.

---

## 7. Quando atualizar governança

Sempre revisar governança quando houver:

- criação de novo contexto canônico
- formalização de novo padrão recorrente de documentação
- mudança estrutural importante na navegação
- mudança que altere a forma de entrada, leitura ou travessia entre documentos

### Arquivos que podem precisar revisão

- `docs/README.md`
- `docs/00-governance/documentation-system.md`
- `docs/00-governance/99-master-index.md`

---

## 8. Fluxo padrão para editar documentação

Para mudanças pequenas ou médias, seguir o fluxo abaixo.

### Passo 1
Criar uma branch temática curta.

### Passo 2
Editar poucos arquivos por vez.

### Passo 3
Testar clique real no Obsidian.

### Passo 4
Validar no Graph quando a mudança afetar navegação ou nomenclatura.

### Passo 5
Commitar em blocos pequenos, claros e reversíveis.

---

## 9. Fluxo padrão para renomear arquivos

Nunca fazer renomeação em massa sem validação.

### Sequência obrigatória

1. renomear um arquivo ou uma família pequena
2. buscar referências antigas
3. corrigir links
4. testar clique real
5. validar `git status`
6. commitar

### Comando base de varredura

Sempre excluir `.obsidian` para evitar ruído do workspace local:

```bash
grep -RIn --exclude-dir=.obsidian "termo-antigo" docs
```

---

## 10. Regra para nós soltos no Graph

Se um documento estiver solto no Graph:

1. verificar se o `README.md` do contexto aponta para ele
2. verificar se o documento aponta de volta para o contexto
3. verificar se ele aponta para documentos tecnicamente adjacentes
4. validar novamente no Graph

### Regra importante

Não conectar documento solto com link arbitrário.
A conexão deve refletir relação arquitetural ou operacional real.

---

## 11. Regra para `.obsidian`

O diretório `docs/.obsidian/` é local ao Vault e não faz parte da documentação canônica.

### Regra operacional

* não versionar `docs/.obsidian/`
* manter `.gitignore` protegendo essa pasta
* ignorar ruído de `workspace.json` durante renomeações e validações

---

## 12. Critério de mudança boa

Uma mudança boa no `hsc-docs` faz pelo menos um destes:

* melhora navegação
* reduz ambiguidade nominal
* melhora contexto operacional
* conecta melhor runtime, dados, publicação e consumo
* reduz chance de documento órfão
* deixa a manutenção futura mais previsível

Se a mudança não melhora nada disso, provavelmente não vale o custo.

---

## 13. Convenção de mensagens de commit

Preferir mensagens curtas, temáticas e diretamente relacionadas ao efeito da mudança.

### Exemplos

* `docs: add obsidian navigation hubs and contextual links`
* `docs: formalize root home and navigation conventions`
* `docs: rename architecture runtime docs by context`
* `docs: rename references inventory docs by context`
* `docs: connect mariadb runtime to game panel context`

---

## 14. Quando parar

Parar quando:

* o Graph estiver legível o suficiente
* a navegação humana estiver funcionando
* o ganho marginal da próxima mudança for baixo
* a próxima alteração começar a gerar mais ruído do que valor

O objetivo não é um grafo perfeito.
O objetivo é uma documentação operacionalmente navegável, previsível e sustentável.

---

## 15. Ritual curto de manutenção

Antes de concluir uma mudança no `hsc-docs`, validar este checklist:

1. o documento está no contexto certo?
2. ele aponta para Home, contexto e índice?
3. ele aponta para documentos adjacentes reais?
4. o nome do arquivo está específico o suficiente?
5. os links principais foram testados com clique real?
6. o commit está pequeno, coerente e reversível?

---

## 16. Regra final

Melhor uma melhoria pequena, coerente e mergeável do que uma refatoração documental grande e confusa.

A manutenção do `hsc-docs` deve privilegiar:

* clareza
* contexto correto
* navegação útil
* baixo risco operacional
* evolução incremental
