# Game Panel — Public Access and Ops Entrypoint

## Navegação rápida

- [Home da documentação](../README.md)
- [Game Panel](./README.md)
- [Master Index](../00-governance/99-master-index.md)

---
## Objetivo

Documentar a superfície pública e operacional atual de acesso ao AMP/Game Panel do ecossistema HSC, registrando a transição do acesso histórico via apex para o novo host operacional `ops.haxixesmokeclub.com`.

Este documento existe para registrar, de forma estável e auditável:

- como o AMP era exposto antes do hub da marca
- por que o acesso via apex deixou de funcionar
- qual é a entrada operacional canônica atual do painel
- como o Nginx publica o AMP para a nova entrada operacional
- como validar rapidamente o acesso ao painel
- quais limites e itens abertos ainda permanecem

---

## Escopo

Este documento cobre:

- a entrada pública atual do AMP/Game Panel
- a relação entre AMP e domínio público
- a recuperação do acesso operacional ao painel
- a decisão de usar `ops.haxixesmokeclub.com`
- a relação entre Game Panel, Hostinger hPanel e proxy local do AMP

Este documento não cobre em profundidade:

- runbooks completos do AMP
- operação de instâncias CS2
- DNS/TLS do host inteiro em toda sua extensão
- a homepage do Brand Hub

---

## Estado atual

O estado reconciliado do acesso público ao AMP é:

- o AMP não deve mais ser tratado como superfície do apex `haxixesmokeclub.com`
- a entrada operacional canônica atual do painel é `https://ops.haxixesmokeclub.com`
- o acesso via `ops` responde com HTTP 200 e entrega a interface do AMP/Game Panel
- o botão “Gerenciar painel” da Hostinger ainda pode apontar para o domínio antigo do painel, mas isso não bloqueia mais a operação

---

## Comportamento histórico anterior

Antes da publicação do Brand Hub no apex, o Nginx do host tratava o `location /` como proxy padrão para o AMP.

Isso implicava na prática que:

- `haxixesmokeclub.com` podia acabar abrindo o painel
- o painel dependia do domínio principal para ser acessado externamente
- marca pública e superfície operacional estavam acopladas no mesmo host público

Essa topologia foi considerada inadequada após a promoção do apex para hub da marca.

---

## Motivo da quebra operacional

Quando o root foi trocado para serving estático do Brand Hub, o `location /` do apex deixou de ser proxy do AMP e passou a servir o `index.html` do hub.

Resultado prático:

- o painel deixou de abrir pelo domínio principal
- o botão “Gerenciar painel” da Hostinger passou a cair no hub

Esse comportamento foi tratado como efeito colateral real da mudança de superfície.

---

## Decisão canônica

A decisão canônica atual é:

**o AMP/Game Panel deve ser acessado por `ops.haxixesmokeclub.com`.**

Essa decisão existe para:

- separar operação de marca
- impedir que o apex volte a depender do painel
- dar entrada pública estável ao AMP
- manter a homepage da marca intacta

---

## Topologia atual do acesso ao AMP

A topologia pública reconciliada é:

- DNS `ops` apontando para a VPS Hostinger
- bloco Nginx dedicado para `ops.haxixesmokeclub.com`
- proxy reverso do host `ops` para `http://127.0.0.1:8080`
- AMP respondendo atrás desse proxy
- TLS válido no subdomínio `ops`

---

## Relação entre hPanel e AMP

É importante distinguir:

### Hostinger hPanel

- painel da Hostinger
- gerencia VPS, billing, recursos e integração do produto Hostinger
- URL externa sob `https://hpanel.hostinger.com/...`

### AMP/Game Panel

- painel operacional do servidor de jogo
- gerencia instâncias, runtime e superfícies do lado jogo
- URL operacional canônica do checkpoint: `https://ops.haxixesmokeclub.com`

Regra canônica:

- hPanel e AMP não são a mesma superfície
- o hPanel não substitui a entrada operacional do AMP

---

## Sinais de validação do acesso correto

A entrada operacional do AMP é considerada válida quando:

- `curl -I https://ops.haxixesmokeclub.com` retorna HTTP 200
- o navegador abre a interface do AMP com menu lateral, instâncias e ações do Game Panel
- a sessão administrativa do AMP volta a funcionar normalmente pelo subdomínio `ops`

---

## Relação com outros documentos

Este arquivo é complementar a:

- `docs/02-game-panel/README.md`
- `docs/02-game-panel/game-panel-architecture-runtime.md`
- `docs/02-game-panel/game-panel-operational-runbooks.md`
- `docs/02-game-panel/game-panel-references-inventory.md`
- `docs/01-infra-hostinger/infra-hostinger-network-dns-tls.md`
- `docs/01-infra-hostinger/nginx-static-serving.md`
- `docs/03-portal-estatico/brand-hub-root-product-and-surface-decisions.md`
- `docs/03-portal-estatico/brand-hub-root-publishing-and-cutover-runtime.md`

---

## Open items explícitos

Itens ainda abertos nesta trilha:

- eventual atualização do destino interno do botão “Gerenciar painel” da Hostinger
- eventual documentação textual exata do bloco Nginx de `ops` em documento de serving estático do host
- eventual endurecimento adicional de segurança e políticas de acesso do subdomínio operacional

---

## Síntese canônica

A superfície operacional do painel está reconciliada assim:

- apex = hub da marca
- portal = `/portal/cs2/`
- AMP/Game Panel = `ops.haxixesmokeclub.com`

Essa separação deve ser preservada em futuras mudanças.
