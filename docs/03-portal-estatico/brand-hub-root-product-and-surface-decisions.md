# Brand Hub Root — Product and Surface Decisions

## Navegação rápida

- [Home da documentação](../README.md)
- [Portal Estático](./README.md)
- [Master Index](../00-governance/99-master-index.md)

---
## Objetivo

Documentar de forma canônica, explícita e auditável todas as decisões de produto, superfície pública, linguagem de marca, roteamento e separação de responsabilidades que governam o Brand Hub Root do HSC em `haxixesmokeclub.com/`.

Este documento existe para registrar, de forma estável e auditável:

- por que o domínio apex deixou de redirecionar diretamente para o Portal CS2
- por que a superfície raiz passou a existir como hub da marca
- quais decisões se mostraram válidas, quais foram revistas e quais permanecem abertas
- como a marca HSC deve ser apresentada na camada pública principal
- como o root se relaciona com o Portal CS2, com News e com o AMP/Game Panel
- quais decisões de superfície foram tomadas para não misturar marca, operação e administração
- quais decisões de identidade visual já foram congeladas para o checkpoint atual

---

## Escopo

Este documento cobre:

- decisões de produto do root público
- decisões de arquitetura de superfície pública
- decisões de navegação e CTA do Brand Hub
- decisões de linguagem textual e posicionamento da homepage
- decisões sobre logo, assets e sistema visual do root
- decisões sobre separação entre apex, portal público e entrada operacional do AMP
- decisões sobre release e publicação do root

Este documento não cobre em profundidade:

- implementação detalhada de HTML/CSS da homepage
- contratos JSON do Portal CS2
- ETL Bash da Static API v2
- operação profunda do Nginx como infraestrutura geral
- runbooks do AMP/Game Panel
- Auth API, Backoffice Admin e superfícies dinâmicas

Esses assuntos vivem em documentos especializados deste mesmo contexto ou em outros contextos canônicos.

---

## Estado atual

O estado reconciliado atual das decisões desta trilha é:

- o apex `haxixesmokeclub.com/` deixou de ser tratado como redirect obrigatório para `/portal/cs2/`
- o root foi promovido a superfície pública própria, com papel de hub da marca
- o Portal CS2 permaneceu em `/portal/cs2/` como camada operacional pública
- a homepage v4 foi implementada e publicada como baseline funcional da nova superfície
- a percepção do resultado publicado foi avaliada como válida do ponto de vista estrutural, porém ainda próxima demais de uma leitura de “dashboard/app padrão”
- o sistema visual do root deixou de depender exclusivamente das artes detalhadas históricas e passou a usar o `HSC Master Digital Logo v1` como símbolo digital principal
- o AMP/Game Panel deixou de ser acessível pelo apex e passou a ser recuperado em `ops.haxixesmokeclub.com`

Leitura correta do checkpoint:

- a decisão estrutural do hub está mantida
- o problema de acesso operacional ao AMP foi resolvido por separação de superfície
- o refinamento visual do hub continua aberto
- a camada documental precisa registrar de forma total as decisões já fechadas antes de nova iteração de interface

---

## Decisão canônica central

A decisão central e já congelada deste workstream é:

**`haxixesmokeclub.com/` é o hub público da marca HSC.**

Essa decisão substitui a leitura operacional anterior em que o apex existia apenas como passagem para o Portal CS2.

Essa decisão implica:

- o root deve apresentar o HSC como clube, identidade e universo próprio
- o root não é mais uma superfície descartável ou transitória
- o root não deve ser tratado como dashboard do portal
- o root deve guiar o visitante para a camada operacional sem perder a camada cultural

---

## Problema anterior que foi explicitamente encerrado

A configuração anterior do apex produzia o seguinte comportamento:

- usuário acessava `haxixesmokeclub.com`
- domínio atuava como redirect direto para `/portal/cs2/`
- visitante caía imediatamente em uma experiência de produto/consulta sem contexto de marca

Essa configuração foi considerada insuficiente porque:

- anulava o domínio principal como hub da identidade HSC
- fazia a marca parecer apenas “um servidor com ranking”
- eliminava a possibilidade de narrativa, manifesto, visão e apresentação cultural
- misturava indevidamente marca e prova operacional como se fossem a mesma superfície

A partir deste checkpoint, essa leitura deve ser tratada como encerrada.

---

## Decisão de separação entre marca e operação

A separação correta entre as superfícies públicas agora é:

### `haxixesmokeclub.com/`

Função:

- hub da marca
- narrativa de entrada
- manifesto curto
- apresentação do clube
- ponte para superfícies públicas do ecossistema

### `/portal/cs2/`

Função:

- produto público operacional
- ranking, partidas, leitura competitiva e consulta
- prova viva do runtime atual do clube

### `/content/news/`

Função:

- conteúdo público same-origin
- extensão editorial leve do ecossistema

### `ops.haxixesmokeclub.com`

Função:

- entrada operacional do AMP/Game Panel
- superfície administrativa/operacional do lado jogo
- não é parte da experiência pública de marca

Regra canônica:

- hub e portal se complementam
- hub e AMP não se confundem
- operação administrativa não pertence ao apex público

---

## Decisão de não misturar superfície pública e superfície operacional do AMP

Durante a transição do apex para hub da marca, o acesso histórico ao AMP foi perdido porque o runtime antigo do Nginx usava o fallback do `location /` para encaminhar o domínio principal ao AMP via proxy local em `127.0.0.1:8080`.

Essa herança foi considerada inadequada para o novo estado do ecossistema.

A decisão canônica agora é:

**o AMP/Game Panel não deve mais depender do apex para ser acessado.**

Em vez disso, a entrada operacional do painel deve existir em:

- `ops.haxixesmokeclub.com`

Essa decisão existe para preservar simultaneamente:

- a autonomia do root como hub da marca
- a autonomia do AMP como superfície operacional do lado jogo
- a previsibilidade do acesso administrativo sem interferir na homepage pública

---

## Decisão sobre o subdomínio `ops`

O subdomínio `ops.haxixesmokeclub.com` foi aceito como nome canônico para a entrada do painel porque:

- é semanticamente claro
- separa operação de marca pública
- evita reutilizar o apex como host administrativo
- permite evolução futura sem conflitar com o hub

Leitura correta do `ops`:

- é host operacional
- não é host editorial
- não é host de marketing
- não é host do portal estático

---

## Decisão sobre o papel do Portal CS2 dentro da homepage

O Portal CS2 continua sendo parte central do ecossistema, mas sua posição narrativa dentro da homepage mudou.

Decisão canônica:

**o portal é a porta operacional do clube, não a definição total do HSC.**

Isso implica que a homepage deve:

- apontar claramente para o portal
- reconhecê-lo como superfície viva atual
- evitar que o portal absorva toda a narrativa do root

Essa decisão foi materializada nos blocos:

- hero com CTA para o portal
- bloco específico “A porta operacional do clube”
- CTA final voltado à entrada correta no portal

---

## Decisão sobre linguagem textual do hub

A homepage do hub deixou de priorizar linguagem institucional/corporativa e passou a adotar linguagem de clube, manifesto e identidade.

Frases canônicas já assumidas pela superfície:

- `Clube antes de servidor.`
- `Competição com identidade. Resenha com propósito.`
- `Não somos apenas um servidor. Somos um clube.`
- `Nem caos. Nem ego. Nem bagunça.`
- `No fim, o que fica é o grupo.`
- `Não queremos ser só um servidor.`

Regras dessa linguagem:

- mais cultural que empresarial
- mais afirmativa que explicativa
- mais editorial do que técnica
- mais orientada a pertencimento do que a feature checklist

---

## Decisão sobre o sistema visual do root

O sistema visual reconciliado para o hub é:

- dark premium como base
- ciano puxando para azul como acento principal
- tipografia de impacto para títulos
- uso controlado do ciano em CTA, logo e destaques
- blocos largos e superfície editorial em vez de layout excessivamente cardificado

Regra importante:

- o ciano assina a marca, mas não deve dominar a página inteira
- o root deve parecer clube digital premium, não template genérico de app

---

## Decisão sobre o logo master digital

A decisão sobre identidade digital do root foi:

**congelar o `HSC Master Digital Logo v1` como símbolo digital oficial do checkpoint atual.**

Motivação:

- as artes históricas detalhadas da marca são fortes como artwork, mas insuficientes como logo funcional de UI
- o hub e o portal precisavam de um símbolo limpo, escalável e utilizável em navbar, hero, footer, avatar e favicon

O sistema resultante ficou dividido em:

### Logo master digital

- escudo HSC simplificado
- função de UI, navegação e assinatura digital

### Artworks e emblemas históricos

- função de hero art, banner, merch, posters e identidade expandida

Regra canônica:

- UI usa o logo master digital
- artworks históricos não substituem o logo funcional do produto

---

## Decisão sobre variantes oficiais do logo

As variantes mínimas congeladas como oficiais são:

- `HSC-shield-primary-accent`
- `HSC-shield-mono-light`
- `HSC-shield-mono-dark`

Uso canônico do checkpoint:

### Navbar
- `mono-light`

### Hero
- `primary-accent`

### Footer
- `mono-light`

### Avatar / uso digital rápido
- `primary-accent`

### Favicon inicial
- `primary-accent` ou derivado dele

---

## Decisão sobre a homepage v4 como baseline, não como versão final estética

A homepage v4 foi aceita como baseline publicada porque resolveu os problemas estruturais centrais do root:

- o hub passou a existir
- a nova narrativa passou a existir
- o logo master digital foi incorporado
- a separação entre hub e portal foi materializada
- o AMP deixou de depender do apex

Ao mesmo tempo, a homepage v4 **não** foi tratada como acabamento visual definitivo.

Leitura canônica do estado visual:

- válida estruturalmente
- válida como baseline de implementação
- ainda aberta para refinamento de linguagem visual
- ainda passível de redução de “cara de dashboard/app padrão”

---

## Decisão sobre não transformar a próxima etapa em impl-log

Foi explicitamente decidido que o próximo movimento após o baseline publicado não seria continuar improvisando na UI, mas sim produzir documentação canônica total sobre:

- decisões de produto
- decisões de superfície
- decisões de identidade visual
- decisões de roteamento público
- implementações de runtime e publicação

Essa decisão existe para:

- reduzir dependência de memória informal
- interromper iteração visual sem contexto documental estável
- preservar auditabilidade real do workstream

---

## Decisões de publicação e rollout

As decisões de rollout adotadas para o root foram:

- publicação em diretório dedicado sob `/var/www/brand-hub/`
- uso de releases versionadas
- uso de symlink `current`
- validação local via preview HTTP antes do cutover
- cutover por troca de symlink, sem reescrever a topologia do portal e da v2

Essa estratégia foi escolhida porque:

- reduz risco de quebra abrupta
- facilita rollback
- separa o hub do portal e da API estática
- mantém o root com autonomia de release

---

## Open items explícitos

Os itens ainda abertos após essas decisões são:

- refinamento visual do hub além do baseline v4
- eventual ampliação da estrutura editorial interna do root
- eventual lockup textual adicional do logo master
- eventual favicon simplificado, caso necessário em resoluções menores
- eventual sincronização futura do botão “Gerenciar painel” da Hostinger com a nova entrada operacional `ops`

Regra importante:

- itens abertos não invalidam as decisões já congeladas
- itens abertos são evolução, não suspensão do checkpoint atual

---

## Síntese canônica

As decisões de produto e superfície já fechadas neste checkpoint são:

- o apex é o hub da marca
- o portal continua em `/portal/cs2/`
- o AMP/Game Panel agora vive em `ops.haxixesmokeclub.com`
- a homepage v4 é baseline publicada, não acabamento visual final
- o `HSC Master Digital Logo v1` é o símbolo digital oficial do checkpoint
- o root deve comunicar clube, identidade e código antes de empurrar operação
- a documentação canônica deve preceder a próxima rodada de refinamento visual
