# Brand Hub Root — Publishing and Cutover Runtime

## Navegação rápida

- [Home da documentação](../README.md)
- [Portal Estático](./README.md)
- [Master Index](../00-governance/99-master-index.md)

---
## Objetivo

Documentar a implementação real de publicação, release, preview, cutover, rollback e inventário operacional do Brand Hub Root do HSC.

Este documento existe para registrar, de forma estável e auditável:

- a topologia de diretórios do Brand Hub Root
- o modelo de releases versionadas e symlink `current`
- o papel do `index.html` como baseline publicada do root
- como os assets do logo são publicados e consumidos
- como o cutover do root é realizado sem quebrar o Portal CS2
- como o rollback deve ser executado
- quais foram os efeitos colaterais operacionais observados na transição

---

## Escopo

Este documento cobre:

- a publicação do root sob `/var/www/brand-hub/`
- releases versionadas do hub
- o symlink `current`
- assets do root e do logo
- validação por preview local e smoke test HTTP
- rollback do root por troca de symlink
- relação entre o root e a configuração Nginx da borda pública

Este documento não cobre em profundidade:

- o Nginx como substrate completo do host
- publicação do Portal CS2 em `/portal/cs2/`
- ETL e Static API v2
- operação do AMP/Game Panel além do necessário para explicar a separação de superfície

---

## Estado atual

O runtime reconciliado atual do root é:

- o Brand Hub Root existe sob `/var/www/brand-hub/`
- a publicação é feita por releases versionadas
- a release viva é selecionada por symlink `current`
- o root público do domínio apex usa essa árvore como source of truth de arquivos estáticos
- os assets do logo foram incorporados na árvore do root
- a homepage v4 foi publicada a partir desse modelo

---

## Topologia de publicação

A topologia de publicação do root, neste checkpoint, é:

```text
/var/www/brand-hub/
├── current -> /var/www/brand-hub/releases/<release>
└── releases/
    ├── 2026-03-27-root-mvp/
    ├── 2026-03-27-root-v2/
    └── 2026-03-27-root-v4/
```

Leitura correta:

- `current` é o ponteiro vivo do root
- cada release é isolada
- o cutover ocorre trocando o symlink
- o rollback pode ocorrer retornando o symlink para a release anterior

---

## Release baseline relevante do checkpoint

A release importante do checkpoint atual é:

- `2026-03-27-root-v4`

Ela consolidou:

- homepage v4 com nova narrativa
- uso do `HSC Master Digital Logo v1`
- topologia do hub como baseline publicada

---

## Inventário mínimo de arquivos esperados na release

Cada release viva do root deve conter pelo menos:

```text
index.html
robots.txt
sitemap.xml
img/logo/svg/HSC-shield-primary-accent.svg
img/logo/svg/HSC-shield-mono-light.svg
img/logo/svg/HSC-shield-mono-dark.svg
```

Regra canônica:

- sem esses arquivos, a release não deve ser tratada como baseline pronta para cutover

---

## Modelo de atualização do `index.html`

O `index.html` do root é, neste estágio, a peça central da superfície pública.

Ele materializa:

- navbar
- hero
- manifesto
- código do clube
- bloco do Portal CS2
- bloco de comunidade
- bloco de visão
- CTA final
- footer

A homepage v4 é estática e não depende de backend dinâmico.

---

## Modelo de assets do logo

Os assets do logo usados pelo root são servidos como arquivos estáticos dentro da própria árvore da release.

Paths canônicos esperados:

- `/img/logo/svg/HSC-shield-primary-accent.svg`
- `/img/logo/svg/HSC-shield-mono-light.svg`
- `/img/logo/svg/HSC-shield-mono-dark.svg`

Esses arquivos são consumidos pelo HTML da homepage para:

- navbar
- hero
- footer

---

## Modelo de preview local antes do cutover

O preview local adotado no workstream foi baseado em servidor HTTP temporário apontando diretamente para a árvore da release.

Objetivo do preview:

- validar integridade do `index.html`
- validar paths dos logos
- validar copy principal antes do cutover
- evitar trocar o `current` sem smoke prévio

Esse modelo é deliberadamente simples e compatível com a natureza estática da superfície.

---

## Modelo de cutover

O cutover do root é feito por:

1. preparar a release nova em `/var/www/brand-hub/releases/<release>`
2. validar a release localmente
3. trocar o symlink `current`
4. executar smoke test público

Esse modelo foi escolhido porque:

- é atômico o suficiente para a superfície estática
- reduz acoplamento com outras camadas
- preserva rollback simples

---

## Modelo de rollback

O rollback do root é feito por:

- retornar o symlink `current` para a release anterior

Leitura correta:

- a estratégia de rollback do root é baseada em filesystem e symlink
- não depende de rebuild complexo nem de republishing integral do portal

---

## Relação entre root e configuração Nginx

A configuração reconciliada do Nginx para o apex substituiu o fallback histórico do AMP por serving direto da árvore do Brand Hub.

Mudanças estruturais relevantes dessa decisão:

- `root /var/www/brand-hub/current;`
- `index index.html;`
- `location = / { try_files /index.html =404; }`
- `location / { try_files $uri =404; }`

Leitura correta:

- o apex deixou de ser proxy padrão para AMP
- o root passou a servir arquivos do Brand Hub diretamente

---

## Efeito colateral operacional observado

A mudança do apex para Brand Hub produziu um efeito colateral imediato:

- o acesso histórico ao AMP/Game Panel pelo domínio principal deixou de existir

Isso aconteceu porque a configuração antiga do root usava fallback proxy para `127.0.0.1:8080`, enquanto a configuração nova passou a servir a homepage estática do hub.

Esse efeito colateral foi tratado como bug operacional real do rollout e precisou ser fechado no mesmo checkpoint.

---

## Correção do acesso ao AMP

A correção adotada foi:

- criar o subdomínio operacional `ops.haxixesmokeclub.com`
- publicar um novo bloco Nginx específico para esse host
- restaurar o proxy do AMP via `127.0.0.1:8080`
- emitir/validar TLS para `ops`

Leitura correta:

- a separação de superfície resolveu o conflito
- o hub permaneceu no apex
- o painel voltou a existir em host próprio

---

## Relação com o botão “Gerenciar painel” da Hostinger

Durante o checkpoint, foi observado que o botão “Gerenciar painel” da Hostinger ainda abria o domínio antigo associado à superfície pública, o que passou a cair no hub.

Leitura canônica atual:

- o acesso operacional urgente ao painel foi recuperado via `ops.haxixesmokeclub.com`
- a sincronização interna do botão da Hostinger com o novo host operacional é assunto separado e não bloqueia a operação

---

## Comandos canônicos de validação rápida

Exemplos de validação esperada para o root:

```bash
readlink -f /var/www/brand-hub/current
curl -I https://haxixesmokeclub.com/
curl -I https://haxixesmokeclub.com/portal/cs2/
curl -I https://haxixesmokeclub.com/img/logo/svg/HSC-shield-mono-light.svg
curl -s https://haxixesmokeclub.com/ | grep -E "Clube antes de servidor|A porta operacional do clube"
```

Exemplos de validação esperada para o `ops`:

```bash
curl -I https://ops.haxixesmokeclub.com
```

---

## Open items explícitos

Os itens ainda abertos nesta trilha são:

- eventual refinamento do layout da homepage além do baseline v4
- eventual padronização adicional da pasta de assets do hub
- eventual automatização futura do release do root
- eventual sincronização final do botão “Gerenciar painel” com a nova entrada `ops`

---

## Síntese canônica

A implementação de publicação do Brand Hub Root já está reconciliada com estas decisões:

- release versionada
- `current` como ponteiro vivo
- hub servido diretamente pelo apex
- portal preservado em `/portal/cs2/`
- AMP preservado em `ops.haxixesmokeclub.com`
- cutover por troca de symlink
- rollback simples por troca de symlink
