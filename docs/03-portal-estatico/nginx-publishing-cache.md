# Nginx Publishing and Cache

## Objetivo

Documentar o papel do Nginx na publicação pública do Portal Estático e da Static API v2, registrando os paths servidos, as regras de exposição, o comportamento de cache e os cuidados operacionais da borda estática deste contexto.

Este documento existe para registrar, de forma estável e auditável:

- como o Nginx publica o portal público
- como o Nginx publica a Static API v2
- como o conteúdo same-origin é exposto quando aplicável
- quais artefatos podem e quais não podem ser expostos
- como cache e headers públicos devem ser tratados em alto nível
- quais sintomas e falhas são mais comuns na camada de publishing

---

## Escopo

Este documento cobre:

- o papel do Nginx no contexto do Portal Estático
- publicação do frontend público
- publicação da Static API v2
- paths públicos relevantes
- cache e headers em nível operacional
- espelho same-origin de conteúdo, quando aplicável
- proteções de borda para arquivos sensíveis
- validação e falhas comuns da camada de publishing

Este documento não cobre em profundidade:

- configuração completa da VPS Hostinger
- unit files e timers do host
- o pipeline ETL script por script
- contratos JSON campo a campo
- troubleshooting aprofundado de browser
- Auth API no Lightsail
- operação do Game Panel

Esses tópicos vivem em documentos próprios deste contexto ou em outros contextos canônicos.

---

## Estado atual

O estado operacional conhecido da camada de publishing deste contexto é:

- o Nginx serve o portal público estático
- o Nginx serve a Static API v2 a partir de diretórios públicos dedicados
- o portal e a API estática são entregues como arquivos
- a publicação pública depende da integridade dos paths de saída do ETL
- há necessidade de proteger artefatos que não devem ser expostos
- o contexto pode publicar conteúdo same-origin de News, quando aplicável
- o comportamento de cache deve preservar boa experiência pública sem mascarar completamente atualizações importantes

O Nginx é a camada final de entrega pública do contexto, mas não é responsável por gerar os dados nem por definir sozinho os contratos consumidos pelo frontend.

---

## Source of truth / evidências

As principais evidências deste documento, nesta fase de migração documental, são:

- documentação consolidada atual do ecossistema HSC
- blueprint técnico consolidado do HSC
- documentação reconciliada do portal e da Static API v2
- reconciliação dos paths públicos de publicação
- reconciliação das regras de proteção de borda para arquivos não públicos
- registros de espelho same-origin de conteúdo e publishing do portal

Enquanto a migração canônica do contexto não estiver concluída, essas fontes seguem sendo usadas como base de reconciliação do estado real.

---

## Relações com outros documentos

Este arquivo é complementar a:

- `docs/03-portal-estatico/README.md`
- `docs/03-portal-estatico/architecture-runtime.md`
- `docs/03-portal-estatico/static-api-v2.md`
- `docs/03-portal-estatico/etl-bash-pipeline.md`
- `docs/03-portal-estatico/frontend-structure.md`
- `docs/03-portal-estatico/operational-runbooks.md`
- `docs/03-portal-estatico/observability-troubleshooting.md`
- `docs/01-infra-hostinger/nginx-static-serving.md`

Este documento descreve a borda de publicação do portal e da API estática.  
Ele não substitui os documentos de ETL, frontend, infraestrutura base ou troubleshooting geral.

---

## Papel do Nginx no contexto

No contexto do Portal Estático, o Nginx é a camada pública de entrega.

Suas responsabilidades incluem:

- servir o frontend estático
- servir os JSONs da Static API v2
- servir conteúdo same-origin complementar, quando aplicável
- isolar a camada pública de publicação dos diretórios operacionais internos
- bloquear exposição de artefatos que não devem ser públicos
- aplicar política coerente de cache e headers para assets e dados públicos

O Nginx não é responsável por:

- gerar JSONs
- recalcular ranking
- interpretar schema do banco
- operar MatchZy
- substituir o ETL
- substituir a Auth API

---

## Paths servidos

Os paths públicos principais já reconciliados neste contexto incluem:

### Publicação do portal

- `/var/www/portal/cs2/`

Função:
- servir HTML, CSS, JavaScript e assets públicos do frontend

### Publicação da Static API v2

- `/var/www/api/cs2/v2/`

Função:
- servir os arquivos JSON públicos da v2

### Conteúdo same-origin de News, quando aplicável

- publishing público sob paths de conteúdo same-origin do portal

Função:
- expor conteúdo público integrado ao domínio do portal
- simplificar consumo no frontend quando esse fluxo estiver ativo

Regra importante:
- os diretórios públicos devem conter apenas artefatos finais já aptos a serem servidos
- diretórios operacionais, temporários ou sensíveis não devem ser expostos diretamente

---

## Publicação do Portal

A publicação do portal assume que o frontend já está pronto para serving estático.

A camada de publishing do portal deve:

- servir páginas públicas
- servir módulos JavaScript e estilos
- servir assets visuais
- manter previsibilidade de paths públicos
- não depender de backend dinâmico local para renderização pública básica

Em termos operacionais, isso significa:

- o browser deve conseguir carregar a aplicação pública apenas a partir do que o Nginx entrega
- qualquer falha no publishing de assets quebra a experiência pública, independentemente da saúde do ETL

---

## Publicação da Static API v2

A publicação da v2 assume que o ETL já gerou e validou seus artefatos.

A camada de publishing da v2 deve:

- expor os JSONs nos paths públicos corretos
- não servir arquivos parciais
- não servir artefatos temporários como se fossem finais
- manter compatibilidade entre o que o frontend espera e o que está publicado
- preservar legibilidade pública dos recursos conhecidos da v2

Exemplos de artefatos públicos relevantes:

- `health.json`
- `ranking.json`
- `matches.json`
- `match/{id}.json`
- `maps.json`
- `map/{map}.json`
- `player/{steamid64}.json`
- `steam-cache/{steamid64}.json`

---

## Espelho same-origin de News

Quando o fluxo same-origin de News está ativo, o Nginx também participa da publicação dessa camada complementar.

O papel do publishing same-origin é:

- expor conteúdo público sob o mesmo domínio do portal
- reduzir atrito de consumo pelo frontend
- manter previsibilidade de integração pública entre portal e conteúdo

Regra importante:
- esse publishing não altera a natureza estática do portal
- ele continua sendo publishing público de arquivo/artefato entregue pela borda
- a origem de geração ou espelhamento do conteúdo continua sendo uma preocupação separada da borda de entrega

---

## Cache e headers

A camada de publishing do portal precisa equilibrar:

- performance pública
- previsibilidade de atualização
- simplicidade operacional
- compatibilidade com consumo no browser

Princípios esperados de cache:

- assets estáticos do frontend podem tolerar política de cache mais favorável, desde que compatível com sua estratégia real de atualização
- dados da Static API v2 devem usar política coerente com sua cadência de regeneração
- cache não deve mascarar indefinidamente mudanças importantes da camada pública
- o comportamento deve ser suficientemente previsível para troubleshooting

Regra operacional:
- cache agressivo demais pode fazer o portal parecer “parado” mesmo com ETL saudável
- cache inexistente pode aumentar custo e ruído sem benefício real

---

## Distinção entre cache de assets e cache de dados

É importante distinguir:

### Assets do frontend

Exemplos:
- CSS
- JavaScript
- imagens
- HTML público

### Dados da v2

Exemplos:
- `ranking.json`
- `matches.json`
- `player/{steamid64}.json`

Esses dois grupos têm comportamento operacional diferente.

Em termos de troubleshooting:

- problema em asset geralmente quebra a interface
- problema em JSON ou cache de JSON geralmente faz a interface carregar com dados stale, vazios ou inconsistentes

---

## Proteções de borda

A camada de Nginx do contexto precisa bloquear exposição indevida de artefatos sensíveis ou internos.

As proteções já reconciliadas em alto nível incluem:

### deny para `.db` e `.sqlite`

Objetivo:
- impedir exposição pública do banco SQLite e artefatos semelhantes

Motivo:
- o `matchzy.db` é fonte interna da cadeia de geração, não recurso público

---

### deny para dotfiles

Objetivo:
- impedir exposição de arquivos ocultos e artefatos operacionais internos

Motivo:
- dotfiles frequentemente contêm configuração, estado ou material que não deve ser público

---

### proteção de paths internos/temporários

Objetivo:
- impedir que diretórios intermediários ou operacionais sejam expostos como se fossem publicação final

Motivo:
- a camada pública deve servir apenas artefatos finais e deliberadamente públicos

Regra canônica:
- o Nginx do contexto do portal deve publicar arquivos públicos, não a árvore operacional inteira do host

---

## Relação entre ETL e publishing

A borda de publishing depende diretamente da qualidade do ETL.

Isso implica:

- se o ETL falhar, o Nginx continuará servindo o último estado publicado
- se o ETL publicar arquivo inválido, o Nginx servirá arquivo inválido
- se a escrita atômica for quebrada, o Nginx pode servir JSON parcial
- se as permissões estiverem erradas, o ETL pode gerar sem conseguir publicar, ou o Nginx pode não conseguir ler

Regra importante:
- Nginx saudável não significa pipeline saudável
- pipeline saudável sem Nginx funcional também não gera publicação útil ao público

---

## Relação entre publishing e frontend

A camada de publishing precisa permanecer coerente com o frontend.

Isso significa:

- os paths públicos precisam continuar estáveis
- os recursos que o frontend busca precisam estar acessíveis
- o cache não pode criar estado público enganoso
- o portal não deve depender de paths “implícitos” ou não documentados

Qualquer mudança de path público deve ser tratada como breaking change operacional para o frontend.

---

## Validação operacional

As validações mínimas desta camada incluem:

### Validar sintaxe do Nginx

```bash
sudo nginx -t
```

### Validar status do serviço Nginx

```bash
sudo systemctl status nginx
```

### Validar publicação do portal

Substitua pela URL pública vigente do portal.

```bash
curl -I https://SEU_DOMINIO/
```

### Validar publicação de artefato da v2

Substitua pela URL pública vigente da v2.

```bash
curl -I https://SEU_DOMINIO/SEU_PATH_PUBLICO_DA_V2/health.json
```

### Validar conteúdo público principal

Exemplos representativos:

```bash
curl -sS https://SEU_DOMINIO/SEU_PATH_PUBLICO_DA_V2/health.json
curl -sS https://SEU_DOMINIO/SEU_PATH_PUBLICO_DA_V2/ranking.json
curl -sS https://SEU_DOMINIO/SEU_PATH_PUBLICO_DA_V2/matches.json
```

### Validar diretórios públicos no host

```bash
ls -lah /var/www/portal/cs2/
ls -lah /var/www/api/cs2/v2/
```

Essas validações ajudam a separar:

- problema de publishing
- problema de ETL
- problema de frontend
- problema de path/permissão

---

## Sinais de saúde da camada de publishing

Os sinais de saúde esperados incluem:

- Nginx ativo
- `nginx -t` sem erro
- diretórios públicos presentes
- assets do portal acessíveis publicamente
- JSONs da v2 acessíveis publicamente
- ausência de artefatos sensíveis expostos
- cache coerente com a experiência pública esperada
- frontend consumindo os recursos publicados sem erro estrutural

A saúde da camada de publishing deve ser inferida tanto pelo lado do host quanto pelo lado do acesso público real.

---

## Problemas comuns

### 1. Portal carrega parcialmente ou sem assets

Causas comuns:

- path público do portal incorreto
- asset ausente
- cache incoerente
- Nginx servindo root/alias incorreto

Impacto:
- interface quebra mesmo que a v2 esteja íntegra

---

### 2. JSON público retorna 404

Causas comuns:

- ETL não publicou o artefato
- path público incorreto
- alias/root do Nginx incorreto
- artefato incremental não gerado

Impacto:
- páginas específicas do portal deixam de funcionar

---

### 3. JSON público existe no disco, mas não serve corretamente

Causas comuns:

- problema de permissão
- Nginx apontando para caminho errado
- path público divergente do esperado pelo frontend

Impacto:
- host parece íntegro, mas a publicação pública falha

---

### 4. Cache serve versão antiga dos dados

Causas comuns:

- política de cache inadequada
- browser segurando artefato stale
- CDN intermediária, se existir no futuro
- troubleshooting feito apenas por uma única origem de teste

Impacto:
- ETL publica dado novo, mas o usuário continua vendo estado antigo

---

### 5. Artefato sensível exposto indevidamente

Causas comuns:

- proteção de borda ausente
- alias mal configurado
- root amplo demais
- arquivo operacional ou banco colocado em área pública

Impacto:
- risco operacional e de segurança
- violação da separação entre interno e público

---

### 6. Portal público funciona, mas same-origin de News quebra

Causas comuns:

- publishing específico do espelho falhou
- path same-origin incorreto
- conteúdo não foi espelhado corretamente
- publicação parcial de uma camada complementar

Impacto:
- portal principal parece saudável
- conteúdo integrado específico falha

---

## Riscos e cuidados

Os principais riscos desta camada incluem:

- serving de diretório errado
- exposição indevida de arquivos internos
- cache enganando troubleshooting
- frontend depender de paths não formalizados
- JSON válido no disco mas não publicamente acessível
- publishing parcial passar despercebido
- diferença entre saúde do Nginx e saúde real da experiência pública

Cuidados permanentes:

- validar `nginx -t` antes de mudanças
- manter separação clara entre diretórios públicos e operacionais
- testar acesso público real, não apenas existência do arquivo no host
- revisar deny rules e proteção de arquivos sensíveis
- tratar mudança de path público como mudança relevante

---

## Limites deste documento

Este documento não detalha:

- configuração textual completa do vhost
- estratégia de cache linha a linha
- headers específicos completos
- troubleshooting aprofundado do browser
- a lógica de geração dos arquivos publicados
- a automação do host que chama o ETL

Esses detalhes devem ser tratados em documentos complementares ou impl-logs, quando necessário.

---

## Critério de pronto deste documento

Este documento pode ser considerado maduro quando:

- os paths públicos servidos estiverem confirmados sem ambiguidade
- a relação entre portal, v2 e same-origin de conteúdo estiver clara
- as proteções de borda estiverem formalizadas em nível suficiente
- o comportamento geral de cache estiver descrito de forma útil
- os problemas comuns de publishing estiverem bem representados
- ele puder ser usado como referência da entrega pública do portal sem depender do master legado

---

## Última revisão

- Status: ativo
- Classificação: canônico
- Contexto: portal estático / Nginx publishing and cache
- Última revisão: 2026-03-18