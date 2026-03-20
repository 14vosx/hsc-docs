# Nginx Static Serving

## Objetivo

Documentar o papel do Nginx como edge público e camada de serving estático da Infra Hostinger no ecossistema HSC.

Este documento existe para registrar, de forma estável e auditável:

- a função do Nginx na borda pública do lado Hostinger
- como o portal público é servido
- como a Static API v2 é servida
- como conteúdos públicos same-origin podem ser expostos, quando aplicável
- quais proteções de borda precisam existir
- quais sintomas e falhas são mais comuns na camada de serving estático

---

## Navegação

### Entrada
- [Home da documentação](../README.md)
- [Infra Hostinger](./README.md)
- [Master Index](../00-governance/99-master-index.md)

### Infraestrutura imediata
- [Architecture Runtime](./infra-hostinger-architecture-runtime.md)
- [Network, DNS and TLS](./network-dns-tls.md)
- [Systemd Automation](./systemd-automation.md)
- [Certbot](./certbot.md)

### Publicação e consumo
- [Portal Estático](../03-portal-estatico/README.md)
- [Static API v2](../03-portal-estatico/static-api-v2.md)
- [Nginx Publishing and Cache](../03-portal-estatico/nginx-publishing-cache.md)
- [Frontend Structure](../03-portal-estatico/portal-estatico-frontend-structure.md)

### Operação e suporte
- [Operational Runbooks](../03-portal-estatico/portal-estatico-operational-runbooks.md)
- [Observability and Troubleshooting](./infra-hostinger-observability-troubleshooting.md)
- [Documentation System](../00-governance/documentation-system.md)

---

## Escopo

Este documento cobre:

- o papel do Nginx na Infra Hostinger
- o modelo de serving estático do portal
- o modelo de serving estático da Static API v2
- aliases, roots e paths públicos relevantes
- headers e cache em nível operacional
- proteções de borda para arquivos e diretórios sensíveis
- validação operacional do serving público
- falhas comuns da camada de edge estático

Este documento não cobre em profundidade:

- DNS e política geral de TLS
- Certbot em detalhe
- configuração completa da VPS Hostinger
- pipeline ETL Bash
- contratos JSON da v2
- Auth API no Lightsail
- operação detalhada do Game Panel

Esses tópicos vivem em documentos próprios deste contexto ou em outros contextos canônicos.

---

## Estado atual

O estado operacional conhecido do Nginx no lado Hostinger é:

- o Nginx é a borda pública do portal e da Static API v2
- o portal é servido como conjunto de arquivos estáticos
- a v2 é servida como conjunto de arquivos JSON estáticos
- o serving depende de paths públicos canônicos no filesystem do host
- o Nginx atua sobre a metade game + portal do ecossistema
- o host pode expor conteúdo same-origin complementar, quando aplicável
- a árvore pública deve conter apenas artefatos finais já aptos para serving
- a árvore operacional do host não deve ser exposta como se fosse publicação pública

Esse desenho preserva a separação entre:

- borda pública
- diretórios públicos finais
- diretórios operacionais do ETL
- diretórios operacionais do lado AMP / CS2

---

## Source of truth / evidências

As principais evidências deste documento, nesta fase de migração documental, são:

- documentação consolidada atual do ecossistema HSC
- blueprint técnico consolidado do HSC
- documentação reconciliada da Infra Hostinger
- documentação reconciliada do Portal Estático e da Static API v2
- reconciliação dos paths públicos do portal e da v2
- reconciliação das regras de proteção de borda para arquivos não públicos

Enquanto a migração canônica do contexto não estiver concluída, essas fontes seguem sendo usadas como base de reconciliação do estado real.

---

## Relações com outros documentos

Este arquivo é complementar a:

- `docs/01-infra-hostinger/README.md`
- `docs/01-infra-hostinger/infra-hostinger-architecture-runtime.md`
- `docs/01-infra-hostinger/network-dns-tls.md`
- `docs/01-infra-hostinger/certbot.md`
- `docs/01-infra-hostinger/filesystem-paths-permissions.md`
- `docs/01-infra-hostinger/infra-hostinger-observability-troubleshooting.md`
- `docs/03-portal-estatico/nginx-publishing-cache.md`

Este documento descreve o serving estático do lado Hostinger.  
Ele não substitui os documentos de DNS/TLS, filesystem, Portal Estático ou troubleshooting geral do host.

---

## Papel do Nginx no contexto

O Nginx é o edge público do lado Hostinger.

Suas responsabilidades incluem:

- receber tráfego público HTTP/HTTPS do portal
- servir o frontend estático do portal
- servir os JSONs públicos da Static API v2
- servir conteúdo público same-origin, quando aplicável
- aplicar política de acesso e exposição adequada à árvore pública
- impedir exposição de artefatos internos, sensíveis ou operacionais
- sustentar cache e headers públicos coerentes com a camada estática

O Nginx não é responsável por:

- gerar os JSONs da v2
- calcular ranking ou agregados
- operar o runtime do jogo
- manter sessão ou backend dinâmico
- substituir a Auth API
- corrigir inconsistência lógica do ETL

---

## Modelo de serving estático

O modelo canônico do lado Hostinger é:

- cliente externo acessa o domínio público do portal
- o Nginx resolve o path solicitado
- o Nginx lê o artefato público correspondente no filesystem
- o Nginx serve o arquivo ao cliente
- o browser carrega o frontend e, depois, consome os JSONs públicos da v2

Esse desenho implica:

- o Nginx publica arquivos finais, não artefatos intermediários
- a árvore pública precisa estar íntegra no filesystem
- o frontend e a v2 dependem da mesma camada de edge estático
- falha no serving quebra o acesso público mesmo quando os artefatos existem no disco

---

## Publicação do Portal

O portal é servido a partir da árvore pública:

- `/var/www/portal/cs2/`

Essa árvore contém, em nível macro:

- HTML
- CSS
- JavaScript
- imagens e assets visuais
- demais artefatos públicos da interface

Regras operacionais desta publicação:

- o portal deve ser legível por `www-data`
- a árvore pública deve conter apenas artefatos finais
- mudanças em root/alias precisam ser tratadas como alteração relevante
- o portal não deve depender de backend dinâmico local para seu serving básico

---

## Publicação da Static API v2

A Static API v2 é servida a partir da árvore pública:

- `/var/www/api/cs2/v2/`

Essa árvore contém, em nível macro:

- `health.json`
- `ranking.json`
- `matches.json`
- `match/{id}.json`
- `maps.json`
- `map/{map}.json`
- `player/{steamid64}.json`
- `steam-cache/{steamid64}.json`

Regras operacionais desta publicação:

- apenas JSONs finais devem ser servidos
- artefatos temporários não devem aparecer como recursos públicos
- o Nginx deve servir a versão final já estabilizada pelo ETL
- o frontend deve encontrar os recursos nos paths públicos esperados

---

## Conteúdo same-origin de News

Quando o fluxo same-origin de conteúdo está ativo, o Nginx também publica essa camada complementar.

O objetivo desse serving é:

- expor conteúdo público sob o mesmo domínio do portal
- reduzir atrito no consumo pelo frontend
- manter previsibilidade de integração pública entre portal e conteúdo

Regra importante:

- esse serving complementar não altera a natureza estática da borda
- a borda do Hostinger continua servindo artefatos públicos finais
- a origem do conteúdo same-origin continua sendo preocupação separada do edge

---

## Aliases e roots relevantes

Os paths públicos mais relevantes já reconciliados neste contexto são:

- `/var/www/portal/cs2/`
- `/var/www/api/cs2/v2/`

Em termos de desenho de serving, o Nginx precisa manter coerência entre:

- domínio público
- root/alias efetivos
- estrutura real do filesystem
- paths esperados pelo frontend
- paths esperados da Static API v2

Regra operacional:

- mudança de root/alias pode quebrar portal e v2 mesmo sem qualquer mudança no ETL
- root/alias incorretos costumam gerar sintomas como:
  - `404`
  - assets ausentes
  - JSON público ausente
  - serving de diretório errado
  - exposição indevida de áreas operacionais

---

## Headers e cache básico

A camada de Nginx do lado Hostinger precisa equilibrar:

- performance
- previsibilidade
- simplicidade operacional
- facilidade de troubleshooting

Princípios esperados:

- assets do portal podem seguir política de cache coerente com a estratégia real de atualização
- JSONs da v2 devem seguir política de cache coerente com a cadência de regeneração
- cache não deve mascarar indefinidamente atualizações importantes
- headers públicos devem ser compatíveis com o consumo real em browser

Regra prática:

- cache excessivo pode fazer o portal parecer stale
- cache ausente pode gerar custo desnecessário e reduzir previsibilidade
- troubleshooting deve sempre distinguir:
  - arquivo servido errado
  - arquivo correto servido com cache antigo
  - arquivo local correto mas path público incorreto

---

## Proteções de borda

A camada de Nginx do lado Hostinger precisa bloquear exposição indevida de artefatos internos.

As proteções já reconciliadas em alto nível incluem:

### deny para `.db` e `.sqlite`

Objetivo:

- impedir exposição pública do `matchzy.db` e de quaisquer bancos SQLite

Motivo:

- a base de dados é artefato operacional interno
- o público deve consumir apenas os JSONs gerados

---

### deny para dotfiles

Objetivo:

- impedir exposição de arquivos ocultos e artefatos internos

Motivo:

- dotfiles podem conter estado, configuração ou material que não deve ser público

---

### proteção de diretórios operacionais

Objetivo:

- impedir que diretórios como:
  - `/opt/cs2-portal/`
  - `locks/`
  - `state/`
  - `sql/`
  - árvores internas do AMP
  sejam tratados como paths públicos

Motivo:

- a borda deve servir apenas publicação final, não infraestrutura interna do host

---

## Relação entre filesystem e serving

O serving estático depende diretamente da integridade do filesystem.

Isso implica:

- se os arquivos públicos não existirem, o Nginx servirá erro
- se existirem mas `www-data` não puder ler, o Nginx servirá erro
- se os paths públicos apontarem para a árvore errada, o Nginx servirá conteúdo incorreto ou nada
- se artefatos parciais forem publicados, o Nginx servirá estado quebrado

Regra importante:

- Nginx saudável não significa publicação saudável
- o filesystem precisa estar correto para que o edge funcione de fato

---

## Relação entre Nginx, ETL e frontend

A relação estrutural é:

- ETL gera artefatos públicos
- o filesystem recebe a publicação final
- o Nginx serve os artefatos
- o frontend consome o que foi servido

Isso significa:

- problema de ETL pode parecer problema de frontend
- problema de path/permissão pode parecer problema de Nginx
- problema de contrato JSON pode parecer problema do portal
- a borda precisa ser analisada em conjunto com geração e consumo

---

## Validação operacional

As validações mínimas do serving estático devem incluir:

### Validar sintaxe do Nginx

```bash
sudo nginx -t
```

### Validar status do serviço

```bash
sudo systemctl status nginx
```

### Validar resposta pública do portal

Substitua pelo domínio vigente.

```bash
curl -I https://SEU_DOMINIO/
```

### Validar resposta pública de um artefato da v2

Substitua pelo path público real da v2.

```bash
curl -I https://SEU_DOMINIO/SEU_PATH_PUBLICO_DA_V2/health.json
```

### Validar árvores públicas no host

```bash
ls -lah /var/www/portal/cs2/
ls -lah /var/www/api/cs2/v2/
```

### Validar leitura estrutural de JSON publicado

```bash
jq . /var/www/api/cs2/v2/health.json
```

Essas validações ajudam a separar:

- problema de borda
- problema de publishing
- problema de permissão
- problema de conteúdo/publicação do ETL

---

## Sinais de saúde do serving estático

Os sinais de saúde esperados incluem:

- `nginx -t` sem erro
- serviço Nginx ativo
- portal público respondendo corretamente
- JSONs públicos da v2 respondendo corretamente
- ausência de artefatos internos expostos
- leitura pública funcionando do ponto de vista do edge
- cache coerente com a expectativa operacional do contexto

A saúde do serving deve ser inferida tanto do lado do host quanto do lado do acesso público real.

---

## Problemas comuns

### 1. Portal responde, mas assets não carregam

Causas comuns:

- root/alias incorreto
- asset ausente
- permissão de leitura incorreta
- publishing parcial da árvore do portal

Impacto:

- interface quebra mesmo com edge “de pé”

---

### 2. JSON público retorna `404`

Causas comuns:

- artefato não foi publicado
- path público incorreto
- alias/root incorreto
- detalhe incremental não gerado

Impacto:

- páginas específicas do portal falham ao consumir a v2

---

### 3. Arquivo existe localmente, mas público falha

Causas comuns:

- `www-data` sem leitura
- traversal de diretório bloqueado
- Nginx apontando para caminho diferente do esperado
- cache/public path divergente do teste

Impacto:

- problema parece “de Nginx”, mas a raiz pode estar em path ou permissão

---

### 4. JSON servido está stale

Causas comuns:

- cache do browser
- política de cache inadequada
- ETL não atualizou artefato
- troubleshooting feito só por cliente já “sujo”

Impacto:

- estado público parece antigo mesmo com host aparentemente íntegro

---

### 5. Arquivo sensível exposto na borda

Causas comuns:

- ausência de deny rule
- root amplo demais
- path operacional confundido com path público
- configuração de serving permissiva demais

Impacto:

- risco operacional e de segurança
- quebra da separação entre público e interno

---

## Sequência básica de diagnóstico

Quando houver falha pública no lado Hostinger, a sequência recomendada é:

1. validar `nginx -t`
2. validar status do serviço Nginx
3. validar acesso público ao portal
4. validar acesso público à v2
5. validar árvore pública local
6. validar permissão de leitura dos artefatos
7. só depois aprofundar em ETL, cache ou frontend

Essa sequência ajuda a responder rapidamente:

- o edge está saudável?
- os paths públicos estão corretos?
- os arquivos existem?
- o problema é de serving, publicação ou consumo?

---

## Limites deste documento

Este documento não detalha:

- vhost completo do Nginx
- política completa de TLS
- política completa de cache por tipo de recurso
- pipeline ETL detalhada
- shape dos JSONs da v2
- operação do AMP e do runtime do jogo
- troubleshooting completo do host

Esses tópicos pertencem a documentos específicos.

---

## Critério de pronto deste documento

Este documento pode ser considerado maduro quando:

- os paths públicos estiverem confirmados sem ambiguidade no host real
- a relação entre root/alias, portal e v2 estiver reconciliada
- as proteções de borda estiverem formalizadas em nível suficiente
- os principais riscos de serving incorreto estiverem bem representados
- ele puder ser usado como referência do edge estático do lado Hostinger sem depender do master legado

---

## Última revisão

- Status: ativo
- Classificação: canônico
- Contexto: infraestrutura Hostinger / Nginx static serving
- Última revisão: 2026-03-18