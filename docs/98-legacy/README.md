# Legacy — README

## Objetivo

Esta pasta existe para preservar o material histórico do ecossistema HSC sem permitir que ele continue governando a navegação principal ou a verdade canônica do sistema documental novo.

Ela existe para registrar, de forma estável e auditável:

- como os documentos antigos devem ser preservados
- como distinguir material legado de material canônico
- como usar documentos históricos de forma útil sem reintroduzir ambiguidade
- como mover masters antigos e arquivos amplos para um lugar correto dentro do repositório
- como manter memória histórica sem deixar o passado dominar o presente

---

## Estado atual

O ecossistema HSC já possui uma base documental canônica nova estruturada por contexto.

Os 4 contextos canônicos iniciais já estão estruturados:

1. Infra Hostinger
2. Game Panel
3. Portal Estático
4. Infra AWS Lightsail

Isso muda o papel da documentação antiga.

A documentação histórica continua útil para:

- reconciliação
- memória arquitetural
- comparação de evolução
- recuperação de decisões antigas
- auditoria de drift

Mas ela deixa de ser, por padrão:

- índice principal do repositório
- ponto de entrada oficial
- verdade canônica automática
- navegação viva do ecossistema

---

## Regra canônica desta pasta

A regra oficial é:

**documento legado pode ser preservado, consultado e citado, mas não governa o sistema documental novo sem reconciliação explícita**

Em termos práticos:

- arquivo antigo útil continua útil
- arquivo antigo importante continua importante
- arquivo antigo não vira verdade oficial só por existir
- material histórico precisa ser lido à luz do runtime validado e dos documentos canônicos atuais

---

## O que é considerado legado

Material legado é todo documento que se encaixa em um ou mais dos casos abaixo:

- master amplo antigo
- blueprint histórico já superado como navegação principal
- documentação consolidada de fase anterior
- snapshot técnico antigo útil apenas como apoio
- documentação que ainda contém informação válida, mas está fora do novo modelo por contexto
- arquivo `_old`
- documento histórico de transição que não deve mais ser tratado como canônico vivo

Exemplos típicos:

- antigos `HSC_Master_*`
- antigos documentos amplos de Auth API
- documentos antigos de infraestrutura consolidados em um único arquivo
- versões históricas que ainda ajudam a reconciliar paths, decisões ou topologias

---

## O que não é legado

Nem todo documento antigo é automaticamente “lixo” e nem todo documento novo é automaticamente “canônico”.

Não é legado:

- documento canônico atual por contexto
- impl-log atual
- auditoria atual
- documento novo já reconciliado com o runtime real
- documento que já foi migrado formalmente para a nova estrutura e continua ativo

Regra importante:

- “antigo” é critério temporal
- “legado” é critério de papel documental

---

## Função desta pasta

A função da pasta `98-legacy/` é:

- preservar histórico útil
- reduzir perda de contexto
- evitar apagar trilha arquitetural importante
- permitir comparação entre modelo antigo e modelo novo
- proteger o sistema atual de continuar dependendo da navegação antiga

Ela existe para resolver um problema específico:

- o repositório precisa lembrar o passado
- sem continuar sendo governado por ele

---

## Como usar material legado corretamente

Material legado deve ser usado para:

- reconciliar informação histórica
- localizar detalhes que ainda não foram migrados
- confirmar mudança de arquitetura ao longo do tempo
- comparar o que mudou entre versões do ecossistema
- apoiar auditoria quando houver conflito entre documentos

Material legado não deve ser usado para:

- sobrescrever documento canônico sem validação
- guiar operação diária por padrão
- servir como índice principal do repositório
- justificar arquitetura atual só porque “estava escrito antes”

---

## Ordem de prioridade da verdade documental

Quando houver dúvida entre legado e documentação atual, a ordem oficial de prioridade é:

1. ambiente real validado
2. documento canônico atual do contexto correto
3. impl-log relevante
4. auditoria relevante
5. documento legado útil

Regra canônica:

**legado ajuda a reconciliar; não ajuda a governar sozinho**

---

## Como os antigos masters devem ser tratados

Os antigos masters devem ser tratados como:

- documentos históricos relevantes
- memória de estrutura anterior
- fonte de reconciliação
- apoio de migração

Eles não devem mais ser tratados como:

- ponto principal de navegação
- índice oficial
- verdade única do ecossistema
- substituto do índice mestre novo

Isso vale especialmente para arquivos amplos que antes tentavam explicar várias camadas ao mesmo tempo.

---

## Estratégia oficial de preservação

A preservação do legado deve seguir estas regras:

### 1. preservar nome reconhecível quando útil

Se o nome do documento já é conhecido no histórico do projeto, preservar ajuda na rastreabilidade.

### 2. não editar legado como se fosse canônico novo

Se o documento é legado, ele deve permanecer identificado como histórico.  
Não deve receber manutenção contínua como se fosse contexto vivo.

### 3. adicionar contexto ao redor do legado, não reescrever tudo

Quando necessário, criar:

- README de pasta
- nota de migração
- impl-log
- auditoria

Mas evitar “ressuscitar” o legado como se fosse a documentação principal.

### 4. preferir referência cruzada a duplicação

Se um documento legado ainda contém algo útil, o caminho preferível é:

- citá-lo como apoio histórico
- migrar o que importa para o contexto correto
- não duplicar conteúdo sem necessidade

---

## Formas recomendadas de organizar o legado

Dentro de `98-legacy/`, a organização recomendada é:

- preservar masters antigos relevantes
- preservar blueprints históricos
- preservar snapshots técnicos antigos
- preservar documentação de transição ainda útil
- agrupar por tema ou fase quando isso reduzir ambiguidade

Exemplos de agrupamento aceitável:

- `98-legacy/masters/`
- `98-legacy/auth-api/`
- `98-legacy/infra/`
- `98-legacy/portal/`
- `98-legacy/season-v1/`

Regra importante:

- a organização do legado deve ajudar busca histórica
- não deve imitar a navegação viva do canônico novo

---

## O que fazer quando um documento legado ainda tem informação importante

Quando um documento legado ainda tiver informação importante, o fluxo recomendado é:

1. identificar qual contexto canônico atual é o dono do assunto
2. validar a informação contra o runtime real, quando necessário
3. migrar a parte útil para o documento canônico correto
4. registrar a reconciliação em impl-log ou auditoria, se fizer sentido
5. manter o documento legado preservado como histórico

Regra importante:

- informação útil deve migrar
- documento legado não precisa desaparecer
- mas também não deve continuar como fonte viva principal

---

## O que fazer quando legado e canônico conflitam

Quando houver conflito entre um documento legado e um documento canônico atual, a sequência oficial é:

1. identificar qual contexto atual é o dono do tema
2. validar o runtime real
3. verificar se existe impl-log mais recente
4. corrigir o documento canônico, se necessário
5. registrar a divergência em auditoria, se relevante
6. manter o legado como histórico, não como árbitro final

Regra canônica:

**conflito com legado se resolve por evidência, não por nostalgia documental**

---

## Sinais de uso incorreto do legado

Os sinais mais comuns de uso incorreto do legado são:

- alguém começa a navegar primeiro por um master antigo
- uma decisão atual é justificada só porque estava em um documento histórico
- um path antigo é tratado como vigente sem validação
- um hostname antigo volta a circular como se fosse oficial
- um documento novo é ignorado em favor de um consolidado antigo
- o time volta a depender de “aquele arquivo grande que tinha tudo”

Esses sinais indicam regressão do sistema documental.

---

## Relação entre legado e auditoria

A pasta `98-legacy/` se relaciona com `97-audit/` da seguinte forma:

- legado preserva material histórico
- auditoria registra divergência entre passado e presente quando isso importa

Exemplos:

- legado mostra topologia antiga
- auditoria registra que a topologia atual já mudou

Essa separação evita que o legado precise ser editado continuamente só para avisar que “não vale mais”.

---

## Relação entre legado e impl-log

A pasta `98-legacy/` se relaciona com `95-impl-log/` da seguinte forma:

- legado guarda o material antigo
- impl-log registra quando e como o sistema mudou

Exemplo:

- um antigo master diz que Auth API estava em Hostinger
- o impl-log registra a migração estrutural do sistema novo
- o contexto canônico atual mostra Lightsail como runtime atual

Assim, o repositório preserva:

- memória
- mudança
- estado atual

sem misturar papéis.

---

## Regras editoriais para notas de legado

Quando necessário, um documento legado pode receber uma nota curta no topo ou na pasta ao redor dele, deixando claro algo como:

- que se trata de material histórico
- que não é navegação principal
- qual contexto novo substitui sua função
- em quais casos ainda vale consultá-lo

Regra importante:

- a nota deve orientar
- não precisa reescrever todo o documento histórico

---

## Estratégia recomendada de migração do legado

A migração do legado para `98-legacy/` deve ser feita em ondas pequenas e controladas.

Fluxo recomendado:

1. identificar documento histórico relevante
2. decidir se ele:
   - já pode ir para legado
   - ainda precisa ser usado na reconciliação por curto prazo
3. mover ou copiar para `98-legacy/` com rastreabilidade adequada
4. atualizar índice ou auditoria, se necessário
5. evitar refazer o documento inteiro sem necessidade real

Regra importante:

- o objetivo não é “limpar rápido”
- o objetivo é preservar bem sem continuar confundindo navegação

---

## O que não fazer com esta pasta

Não usar `98-legacy/` para:

- jogar documentos aleatórios sem critério
- armazenar rascunhos atuais
- esconder documento canônico mal mantido
- misturar backlog, nota pessoal e histórico técnico no mesmo lugar
- reabrir o modelo antigo de documentação ampla como se fosse padrão vivo

`98-legacy/` não é:

- lixeira
- pasta de drafts
- pasta de “depois a gente vê”

Ela é um repositório histórico com função precisa.

---

## Critério de manutenção desta pasta

Esta pasta deve ser revisada quando houver:

- novo lote de documentos históricos a preservar
- conclusão de nova onda de migração do material antigo
- necessidade de registrar que determinado documento antigo deixou oficialmente de ser navegação viva
- necessidade de ajustar a estratégia de organização histórica

---

## Critério de pronto desta pasta

A pasta `98-legacy/` pode ser considerada saudável quando:

- os documentos históricos relevantes puderem ser preservados sem ambiguidade
- o material legado não competir com a navegação canônica
- qualquer pessoa conseguir entender que legado é apoio histórico, não governo atual
- o sistema novo puder evoluir sem voltar a depender dos antigos masters amplos

---

## Última revisão

- Status: ativo
- Classificação: canônico
- Contexto: legado / porta de entrada
- Última revisão: 2026-03-18