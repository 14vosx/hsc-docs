# Operational Runbooks

## Objetivo

Documentar os runbooks operacionais do contexto Backoffice Admin do HSC, registrando como validar, operar e diagnosticar a SPA administrativa em interação com a Auth API.

Este documento existe para registrar, de forma estável e auditável:

- os checks mínimos de operação do Backoffice
- os smoke tests funcionais da camada administrativa
- os procedimentos de validação após mudanças relevantes
- os sintomas operacionais mais prováveis do Backoffice
- os fluxos mínimos de troubleshooting funcional
- os limites entre problema de frontend, problema de contrato e problema de backend
- a relação entre a operação do Backoffice e os contextos canônicos adjacentes

---

## Escopo

Este documento cobre:

- runbooks do Backoffice como produto administrativo
- validações mínimas pós-release
- smoke tests do shell administrativo
- validação de auth, guards, RBAC e navegação
- validação funcional de `seasons`, `news` e `events`
- troubleshooting funcional do frontend administrativo
- critérios práticos para separar falha de UI, contrato e backend
- comportamento esperado diante de 401, 403, 404, 409 e falha sistêmica

Este documento não cobre em profundidade:

- deploy detalhado da Auth API
- operação de Nginx
- systemd
- MariaDB
- rollback de infraestrutura
- operação do Portal estático
- ETL
- Game Panel / AMP
- runbooks internos do backend dinâmico

Esses tópicos pertencem a outros contextos canônicos, especialmente o contexto da infraestrutura dinâmica.

---

## Estado atual

O estado operacional conhecido, nesta fase, é:

- o contexto do Backoffice está sendo formalizado agora
- o produto administrativo é uma SPA separada do Portal público
- a Auth API é a dependência dinâmica central do Backoffice
- o fluxo principal de autenticação deve ser session-first
- o backend continua sendo autoridade de autenticação, autorização e invariantes
- ações sensíveis devem respeitar fail-closed na camada dinâmica
- os domínios prioritários do admin são `seasons`, `news` e `events`
- o runbook do Backoffice deve focar em operação funcional da SPA, não em detalhes profundos de host

Regra importante:

- este documento não deve simular maturidade operacional que ainda não existe
- onde o runtime final ainda não estiver fechado, o runbook deve descrever procedimento-alvo, não inventar automação inexistente

---

## Relação com outros contextos

Este runbook depende diretamente de:

- `docs/05-backoffice-admin/architecture-runtime.md`
- `docs/05-backoffice-admin/frontend-structure.md`
- `docs/05-backoffice-admin/auth-rbac-and-guards.md`
- `docs/05-backoffice-admin/admin-api-contracts.md`

E se relaciona fortemente com:

- `docs/04-infra-aws-lightsail/README.md`
- `docs/04-infra-aws-lightsail/auth-api-operations.md`

Leitura importante:

- este documento valida o Backoffice como produto administrativo
- ele não substitui runbooks da Auth API
- quando o erro estiver abaixo da fronteira do frontend, o runbook deve orientar escalonamento para o contexto correto

---

## Princípios operacionais do Backoffice

A operação do Backoffice deve obedecer aos seguintes princípios:

- validar primeiro a superfície administrativa, depois o detalhe técnico
- distinguir claramente problema de sessão, permissão, contrato e backend
- não assumir que falha visual é sempre falha de frontend
- ações sensíveis só confirmam sucesso após resposta do backend
- smoke tests devem ser simples, repetíveis e de alto valor
- troubleshooting deve começar pelo fluxo do operador
- a UI deve refletir erros de forma útil
- o Backoffice não deve esconder falhas críticas com fallback silencioso

Princípio importante:

- troubleshooting do Backoffice deve preservar a separação entre:
  - problema de renderização/UI
  - problema de navegação/guard
  - problema de contrato HTTP
  - problema de autorização
  - problema de domínio/invariante
  - problema sistêmico da Auth API

---

## Objetivos operacionais mínimos do contexto

O Backoffice está operacionalmente saudável quando:

- a SPA carrega
- o shell administrativo renderiza
- a sessão do operador é resolvida de forma consistente
- as rotas protegidas obedecem os guards
- a navegação administrativa funciona
- os domínios prioritários conseguem listar dados
- formulários carregam e submetem de forma previsível
- 401 e 403 são tratados corretamente
- ações sensíveis não geram falso positivo visual
- erros de contrato ou backend ficam visíveis de forma útil

---

## Checklist mínimo pós-release

Após qualquer release relevante do Backoffice, executar pelo menos o seguinte checklist.

### 1. Carregamento da aplicação

Verificar:

- aplicação abre sem tela em branco
- componente raiz carrega
- layout base renderiza
- assets críticos carregam
- não há erro fatal imediato de bootstrap

Sinais de falha típicos:

- blank screen
- erro fatal no console
- erro de import/standalone
- rota inicial não resolve

---

### 2. Resolução de sessão

Verificar:

- a aplicação resolve o estado de sessão
- usuário autenticado entra no shell administrativo
- usuário sem sessão não entra em área protegida
- a store de sessão converge para estado consistente

Sinais de falha típicos:

- loading infinito de auth
- rota protegida oscilando entre estados
- usuário autenticado tratado como anônimo
- usuário sem sessão conseguindo ver shell indevido

---

### 3. Guards e navegação protegida

Verificar:

- rotas protegidas exigem autenticação
- rotas sensíveis respeitam papel mínimo
- `/403` aparece quando apropriado
- `/404` funciona corretamente
- itens de menu compatíveis com papel atual

Sinais de falha típicos:

- guard deixa passar rota indevida
- rota válida redireciona incorretamente
- menu mostra ações incompatíveis
- operador autenticado cai em loop de navegação

---

### 4. Listagens administrativas

Verificar pelo menos uma listagem por domínio disponível:

- `seasons`
- `news`
- `events` quando já existir no release

Checks mínimos:

- loading inicial aparece
- dados carregam
- empty state aparece se não houver dados
- erro aparece de forma explícita quando a leitura falha
- ação primária de criação aparece apenas quando compatível com o papel

---

### 5. Formulários administrativos

Verificar pelo menos um fluxo create/edit por domínio ativo no release.

Checks mínimos:

- formulário abre
- estado inicial carrega
- validação básica funciona
- submit bloqueia durante envio
- sucesso é refletido sem ambiguidade
- erro de domínio ou backend aparece de forma útil

---

### 6. Ações sensíveis

Quando o release incluir ações sensíveis, verificar explicitamente:

- publish/unpublish de `news`
- activate/close de `seasons`
- delete/cancel de `events`, quando aplicável

Checks mínimos:

- botão aparece só quando faz sentido
- ação exige confirmação
- UI não confirma antes do backend
- erro de permissão ou invariante aparece claramente
- tela/listagem reflete o estado final real após sucesso

---

## Smoke test funcional mínimo

O smoke test mínimo do Backoffice deve seguir uma ordem curta e repetível.

### Smoke 1 — Boot da SPA

Objetivo:

- validar que o Backoffice sobe e renderiza

Passos:

1. abrir a URL do Backoffice
2. confirmar que a aplicação carrega
3. confirmar ausência de erro fatal visível
4. confirmar que a rota inicial responde conforme o estado de sessão

Resultado esperado:

- aplicação funcional
- sem blank screen
- sem erro fatal de bootstrap

---

### Smoke 2 — Auth básica

Objetivo:

- validar o fluxo session-first

Passos:

1. acessar rota protegida sem sessão
2. validar bloqueio/redirecionamento
3. autenticar operador válido
4. reabrir a área protegida
5. confirmar resolução correta da identidade

Resultado esperado:

- sem sessão: acesso bloqueado
- com sessão válida: shell administrativo acessível

---

### Smoke 3 — Shell e menu

Objetivo:

- validar a moldura administrativa

Passos:

1. entrar no shell
2. validar sidebar
3. validar header
4. validar rota dashboard
5. navegar para pelo menos uma feature disponível

Resultado esperado:

- shell consistente
- navegação estável
- sem loop de rota

---

### Smoke 4 — Leitura administrativa

Objetivo:

- validar listagem

Passos:

1. abrir listagem de `seasons`
2. confirmar loading
3. confirmar renderização da lista ou empty state
4. repetir para `news` quando presente
5. repetir para `events` quando presente

Resultado esperado:

- leitura administrativa funcional
- estados visuais coerentes

---

### Smoke 5 — Mutação administrativa simples

Objetivo:

- validar create/edit básico

Passos:

1. abrir formulário de um domínio ativo
2. preencher payload válido
3. submeter
4. confirmar resposta visual
5. voltar à listagem
6. validar reflexo da mutação

Resultado esperado:

- write concluído com resposta clara
- UI coerente com estado persistido

---

### Smoke 6 — Ação sensível

Objetivo:

- validar fail-closed e confirmação de ação crítica

Passos:

1. selecionar ação sensível permitida ao papel
2. disparar confirmação
3. executar ação
4. observar resposta
5. validar estado final

Resultado esperado:

- sucesso só após backend confirmar
- erro de permissão/invariante claramente tratado
- sem optimistic update indevido

---

## Procedimento de validação por domínio

## Seasons

Validar, quando disponível no release:

- listagem administrativa
- create
- edit
- activate
- close

Sinais de saúde:

- existe visualização coerente do estado da season
- activate respeita a invariante de season única ativa
- close respeita transição válida
- erro de conflito é explícito

Sinais de falha:

- duas seasons aparentam ficar ativas
- ação crítica parece ter sucesso, mas a listagem não reflete
- erro de conflito aparece como erro genérico
- formulário permite fluxo incompatível com contrato

---

## News

Validar, quando disponível no release:

- listagem
- create
- edit
- publish
- unpublish
- delete

Sinais de saúde:

- drafts e publicados aparecem com clareza
- publish/unpublish alteram estado corretamente
- delete exige confirmação
- erros de slug, validação ou permissão ficam legíveis

Sinais de falha:

- item some visualmente sem confirmação real do backend
- publish parece funcionar, mas o estado não muda
- 403 vira erro genérico
- UI confirma write que falhou por fail-closed

---

## Events

Validar, quando disponível no release:

- listagem
- create
- edit
- delete ou cancelamento

Sinais de saúde:

- agenda administrativa visível
- dados básicos do evento aparecem de forma consistente
- mutação simples funciona
- separação entre gestão admin e presença pública permanece clara

Sinais de falha:

- contrato admin de leitura inexistente ou improvisado
- UI tenta tratar confirmação pública como fluxo admin
- delete/cancel sem confirmação explícita
- listagem/admin detail divergentes

---

## Tratamento operacional de 401

### Sintomas prováveis

- shell some e volta para login
- rota protegida redireciona inesperadamente
- ação retorna erro de autenticação
- refresh de página derruba acesso

### Diagnóstico inicial

Verificar:

- existe sessão válida?
- o endpoint de identidade/sessão está respondendo?
- a store de auth está convergindo para `unauthenticated` por motivo legítimo?
- o interceptor está reagindo corretamente?
- houve expiração real de sessão?

### Resposta esperada do produto

- limpar estado administrativo inconsistente
- redirecionar corretamente
- mostrar feedback claro quando apropriado

### Escalonamento

Se a UI está correta, mas a sessão falha indevidamente, escalar para:

- contrato de auth
- Auth API
- operação de sessão/backend

---

## Tratamento operacional de 403

### Sintomas prováveis

- rota abre parcialmente e ação falha
- botão some ou fica desabilitado
- ação retorna acesso negado
- usuário autenticado não consegue concluir operação específica

### Diagnóstico inicial

Verificar:

- o papel atual do operador está correto?
- o guard exigiu papel coerente?
- a UI está derivando permissão corretamente?
- o backend rejeitou por autorização real?
- houve mismatch entre permissão visual e permissão efetiva?

### Resposta esperada do produto

- mostrar estado de acesso negado ou erro contextual
- não mascarar como falha genérica
- não deixar a UI confirmar sucesso

### Escalonamento

Se a role visual parece correta, mas o backend diverge, revisar:

- contrato de identidade
- RBAC do backend
- mapeamento de papel/permite no frontend

---

## Tratamento operacional de 404

### Sintomas prováveis

- tela de edição não carrega
- item não existe mais
- slug/id inválido
- deep link antigo falha

### Diagnóstico inicial

Verificar:

- o identificador existe?
- a rota foi gerada corretamente?
- o recurso foi removido?
- a UI diferencia “não encontrado” de “sem permissão”?

### Resposta esperada do produto

- mensagem clara
- navegação de retorno previsível
- sem parecer erro sistêmico

---

## Tratamento operacional de 409 e conflitos de domínio

### Sintomas prováveis

- activate de season falha
- publish falha por estado inválido
- slug duplicado
- transição inválida de lifecycle

### Diagnóstico inicial

Verificar:

- o operador tentou ação incompatível com o estado real?
- a listagem estava atualizada?
- houve corrida entre operadores?
- o backend respondeu conflito de invariante de forma explícita?

### Resposta esperada do produto

- erro claro e específico
- tela continua consistente
- usuário entende que a operação foi negada por regra de domínio

### Regra importante

- conflito de domínio não é bug visual por definição
- pode ser comportamento correto da regra de negócio

---

## Tratamento operacional de 5xx e falha sistêmica

### Sintomas prováveis

- listagem não carrega
- mutação falha sem conclusão
- erro interno retorna em múltiplas telas
- ações sensíveis falham repetidamente

### Diagnóstico inicial

Verificar:

- a Auth API está saudável?
- o erro afeta leitura, escrita ou ambos?
- o problema é geral ou restrito a um domínio?
- a UI está apenas refletindo falha sistêmica?

### Resposta esperada do produto

- erro claro
- sem falso sucesso
- sem travamento silencioso
- possibilidade de retry controlado onde fizer sentido

### Escalonamento

Escalar para o contexto da Auth API e infraestrutura dinâmica.

---

## Checklist de troubleshooting rápido

Quando o Backoffice “parecer quebrado”, verificar nesta ordem:

1. a SPA carrega?
2. o shell renderiza?
3. a sessão resolve?
4. a rota protegida está correta?
5. o papel atual está correto?
6. a chamada HTTP chega?
7. a resposta é 401, 403, 404, 409 ou 5xx?
8. a UI está tratando essa resposta corretamente?
9. o problema é do domínio ou do runtime?
10. o erro precisa ser escalado para Auth API?

Essa ordem reduz troubleshooting confuso e evita culpar o frontend por todo sintoma.

---

## Separação prática entre tipo de falha

### Falha de frontend puro

Exemplos:

- componente não renderiza
- template quebra
- rota não carrega por erro de import
- signal não atualiza a tela corretamente

Indícios:

- erro no console
- sem request útil chegando ao backend
- falha reproduzível antes do contrato HTTP

### Falha de contrato

Exemplos:

- DTO mudou
- campo esperado sumiu
- endpoint responde shape inesperado
- listagem quebra por mismatch de modelo

Indícios:

- request acontece
- resposta chega
- UI não consegue mapear ou interpretar

### Falha de autorização

Exemplos:

- 403 em ação crítica
- usuário autenticado sem papel suficiente
- UI mais permissiva do que backend

Indícios:

- request válido
- backend recusa explicitamente

### Falha de domínio/invariante

Exemplos:

- season já ativa
- publish inválido
- slug duplicado

Indícios:

- request válido
- operador autenticado
- backend recusa por regra de negócio

### Falha sistêmica/backend

Exemplos:

- 5xx
- indisponibilidade da Auth API
- falha interna transacional
- fail-closed impedindo write

Indícios:

- múltiplas operações falhando
- sintomas cruzados entre features
- backend incapaz de concluir operação

---

## Logs e observabilidade esperada

Mesmo sem detalhar stack de observabilidade aqui, o Backoffice deve operar assumindo que existem ao menos três superfícies úteis de observação:

### 1. Console e Network do navegador

Útil para:

- falha de bootstrap
- erro de renderização
- request/response
- códigos HTTP
- shape de payload

### 2. Logs da Auth API

Útil para:

- falha de sessão
- falha de autorização
- erro interno
- fail-closed
- conflito de domínio com detalhe operacional

### 3. Documentação canônica

Útil para:

- confirmar contrato esperado
- confirmar separação de camadas
- evitar troubleshooting baseado em hipótese errada

---

## Regras para ações sensíveis

Sempre que o release incluir ações sensíveis, aplicar as seguintes regras operacionais:

- exigir confirmação explícita
- validar papel mínimo antes de expor a ação
- não confirmar visualmente antes do backend
- refletir erro de permissão com clareza
- refletir erro de invariante com clareza
- revisar estado final da tela/listagem após sucesso

Essas regras se aplicam especialmente a:

- publish/unpublish/delete de `news`
- activate/close de `seasons`
- delete/cancel de `events`

---

## Critérios de aceite operacionais do MVP

O MVP do Backoffice estará operacionalmente aceitável quando:

- shell administrativo estiver estável
- auth session-first estiver resolvida
- guards estiverem consistentes
- `seasons` estiver operável no mínimo
- `news` estiver operável no mínimo, quando incluído
- `events` estiver operável no mínimo, quando incluído
- 401/403 forem legíveis e previsíveis
- ações sensíveis não gerarem falso sucesso
- o troubleshooting básico conseguir separar UI, contrato e backend

---

## Estrutura de execução recomendada para validação manual

A validação manual do Backoffice deve ser executada em três camadas.

### Camada 1 — Estrutural

Validar:

- bootstrap
- shell
- rotas
- guards
- estado de sessão

### Camada 2 — Funcional

Validar:

- listagens
- formulários
- navegação entre telas
- feedback visual

### Camada 3 — Sensível

Validar:

- permissões por papel
- ações críticas
- conflitos de domínio
- tratamento de falhas do backend

Essa ordem reduz ruído operacional e aumenta confiança no release.

---

## Gaps operacionais esperados nesta fase

Os gaps mais prováveis do contexto, agora, são:

- hostname/URL final do Backoffice ainda não formalizado
- rotina final de deploy do frontend ainda não descrita aqui
- smoke automatizado ainda pode não existir
- contrato final de `events` ainda pode evoluir
- instrumentação e telemetria ainda podem estar em fase inicial

Esses gaps não invalidam o runbook.

Mas exigem que:

- a validação manual seja disciplinada
- a documentação seja atualizada conforme a maturidade real do produto

---

## Ordem de uso deste documento

A ordem prática de uso do runbook deve ser:

1. antes do release relevante
2. imediatamente após o release
3. durante troubleshooting funcional
4. ao validar regressões de auth, RBAC e domínio
5. ao revisar readiness do MVP administrativo

---

## Resumo executivo

O `operational-runbooks.md` do Backoffice Admin do HSC existe para garantir que a SPA administrativa possa ser validada e operada de forma disciplinada, sem confundir problema de UI com problema de backend e sem romper a separação de camadas do ecossistema.

A lógica operacional central é:

- validar shell
- validar sessão
- validar guards
- validar listagens e formulários
- validar ações sensíveis
- distinguir 401, 403, 404, 409 e 5xx
- escalar corretamente quando o problema ultrapassar a fronteira do frontend

Esse runbook trata o Backoffice como produto administrativo real, dependente da Auth API, e preparado para crescer com clareza operacional.