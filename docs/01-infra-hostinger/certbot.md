# Certbot

## Objetivo

Documentar o papel do Certbot na Infra Hostinger do ecossistema HSC, registrando como a camada de certificados TLS participa da borda pública do lado game + portal.

Este documento existe para registrar, de forma estável e auditável:

- como o Certbot participa da exposição HTTPS do lado Hostinger
- como ele se relaciona com o Nginx
- como a renovação de certificados impacta o portal e a Static API v2
- quais validações mínimas ajudam a manter a camada íntegra
- quais falhas comuns podem surgir quando a gestão de certificados sai do estado esperado

---

## Escopo

Este documento cobre:

- o papel do Certbot na Infra Hostinger
- a relação entre Certbot e Nginx
- o modelo operacional de emissão e renovação de certificados
- a importância do TLS para o portal e para a Static API v2
- validações operacionais mínimas dessa camada
- riscos e problemas comuns ligados a certificados

Este documento não cobre em profundidade:

- configuração textual completa do Nginx
- DNS em detalhe
- troubleshooting aprofundado do host
- publishing detalhado do portal e da v2
- Auth API no Lightsail
- configuração de certificados de outras camadas do ecossistema

Esses tópicos vivem em documentos próprios deste contexto ou em outros contextos canônicos.

---

## Estado atual

O estado operacional conhecido da camada TLS do lado Hostinger é:

- a borda pública do portal e da Static API v2 usa HTTPS
- o Nginx é o edge público da camada
- o Certbot participa da emissão e renovação dos certificados
- a disponibilidade pública saudável do portal depende de TLS válido
- o filesystem e o Nginx precisam permanecer coerentes com a estratégia de certificado vigente
- falhas de certificado degradam a experiência pública mesmo quando o serving estático segue íntegro no host

No desenho atual do HSC:

- Hostinger sustenta a metade game + portal
- o Certbot é uma peça da confiabilidade pública dessa metade

---

## Source of truth / evidências

As principais evidências deste documento, nesta fase de migração documental, são:

- documentação consolidada atual do ecossistema HSC
- blueprint técnico consolidado do HSC
- documentação reconciliada da Infra Hostinger
- reconciliação do uso de Nginx + Certbot na borda pública
- reconciliação da exposição HTTPS do lado Hostinger

Enquanto a migração canônica do contexto não estiver concluída, essas fontes seguem sendo usadas como base de reconciliação do estado real.

---

## Relações com outros documentos

Este arquivo é complementar a:

- `docs/01-infra-hostinger/README.md`
- `docs/01-infra-hostinger/architecture-runtime.md`
- `docs/01-infra-hostinger/network-dns-tls.md`
- `docs/01-infra-hostinger/nginx-static-serving.md`
- `docs/01-infra-hostinger/observability-troubleshooting.md`

Este documento descreve a camada de certificados do lado Hostinger.  
Ele não substitui os documentos de DNS/TLS, serving estático ou troubleshooting geral da infraestrutura.

---

## Papel do Certbot no contexto

No lado Hostinger, o Certbot existe para sustentar a camada HTTPS da borda pública.

Seu papel inclui:

- emissão de certificados TLS
- renovação periódica dos certificados
- integração operacional com o Nginx
- preservação da confiança pública do domínio do portal
- manutenção da disponibilidade segura do portal e da Static API v2

O Certbot não é responsável por:

- servir o portal
- servir a v2
- corrigir paths públicos
- substituir Nginx
- definir DNS
- corrigir problemas lógicos do ETL ou do frontend

---

## Relação entre Certbot e Nginx

A relação estrutural conhecida é:

- o Nginx é o edge público
- o Certbot sustenta os certificados usados por esse edge
- o portal e a v2 dependem dessa dupla para exposição pública saudável em HTTPS

Isso significa:

- se o Nginx estiver íntegro, mas o certificado estiver inválido, a experiência pública continua degradada
- se o Certbot estiver saudável, mas o Nginx estiver incorreto, o portal continua indisponível
- TLS saudável depende de coerência entre:
  - domínio
  - certificado
  - Nginx
  - borda pública do host

---

## Modelo de emissão e renovação

O modelo operacional esperado desta camada é:

- o domínio público do lado Hostinger possui certificado válido
- a emissão/renovação é tratada pelo Certbot
- a camada TLS precisa permanecer compatível com a configuração real do Nginx
- a renovação não deve quebrar a borda pública

Regra importante:

- mudança de hostname, alias ou vhost do lado Hostinger pode impactar diretamente a emissão e renovação
- qualquer alteração relevante nessa borda deve considerar impacto na camada de certificado

---

## Dependências da camada TLS

A camada TLS depende, no mínimo, de:

- DNS coerente com o host Hostinger
- Nginx funcional
- Certbot funcional
- acesso correto à configuração de borda
- paths de certificado consistentes com a configuração do edge
- exposição pública operacional nas portas esperadas
- renovação ocorrendo sem drift entre domínio real e certificado emitido

Dependências cruzadas importantes:

- o portal depende do certificado válido para experiência pública íntegra
- a v2 depende do mesmo edge
- troubleshooting de “portal fora” pode ser, na verdade, troubleshooting de TLS

---

## Impacto operacional do TLS

Quando a camada TLS está saudável:

- o portal responde em HTTPS
- a Static API v2 responde em HTTPS
- o browser estabelece conexão segura sem alertas
- o consumo público do contexto Hostinger é previsível

Quando a camada TLS falha:

- o usuário vê alertas de certificado ou bloqueio
- integrações públicas podem falhar
- o portal pode parecer indisponível mesmo com arquivos íntegros e Nginx ativo
- troubleshooting precisa distinguir falha de certificado de falha de serving

---

## Sinais de saúde da camada

Os sinais de saúde esperados incluem:

- domínio público resolvendo corretamente
- HTTPS estabelecido sem erro de certificado
- portal público acessível em HTTPS
- artefatos da v2 acessíveis em HTTPS
- Nginx saudável
- ausência de expiração ou mismatch óbvio de hostname/certificado

A saúde do Certbot não deve ser inferida apenas pelo fato de “HTTPS responder”; é necessário que o certificado seja válido e coerente com o hostname público.

---

## Validação operacional

As validações mínimas da camada de certificado devem incluir:

### Validar acesso público HTTPS ao portal

Substitua pelo domínio público vigente.

```bash
curl -I https://SEU_DOMINIO/
```

### Validar acesso público HTTPS à v2

Substitua pelo path público real da v2.

```bash
curl -I https://SEU_DOMINIO/SEU_PATH_PUBLICO_DA_V2/health.json
```

### Validar sintaxe do Nginx antes de concluir que o problema é só de certificado

```bash
sudo nginx -t
```

### Validar status do Nginx

```bash
sudo systemctl status nginx
```

### Validar o Certbot instalado

```bash
certbot --version
```

### Verificar certificados conhecidos

```bash
sudo certbot certificates
```

Esses comandos ajudam a separar:

- problema de certificado
- problema de Nginx
- problema de DNS
- problema de publishing

---

## Rotina operacional

A rotina saudável desta camada pressupõe:

- certificados válidos
- renovação ocorrendo sem intervenção frequente
- Nginx apto a continuar servindo o portal e a v2 após renovações
- validação periódica mínima em cenários de mudança relevante de domínio, borda ou vhost

Regra prática:

- não esperar a falha pública do browser como primeiro indicador de problema
- sempre validar a camada TLS após mudanças relevantes na borda do lado Hostinger

---

## Falhas comuns

### 1. Certificado expirado

Causas comuns:

- renovação não executou
- rotina de renovação falhou
- mudança na borda que invalidou o fluxo esperado

Impacto:
- alertas no navegador
- indisponibilidade operacional do portal para o usuário final

---

### 2. Certificado válido, mas hostname incorreto

Causas comuns:

- mudança de domínio ou alias sem ajuste correspondente
- certificado emitido para hostname diferente
- drift entre DNS e configuração real da borda

Impacto:
- mismatch de certificado
- falha de confiança mesmo com HTTPS “de pé”

---

### 3. Certbot aparentemente saudável, mas portal segue quebrado

Causas comuns:

- problema real está no Nginx
- problema real está no publishing
- domínio resolve, certificado existe, mas serving público continua incorreto

Impacto:
- troubleshooting falso concentrado em certificado, quando a causa está em outra camada

---

### 4. Após alteração de vhost, HTTPS falha

Causas comuns:

- configuração do Nginx não compatível com os arquivos/caminhos esperados
- mudança não coordenada entre borda e certificado
- reload/restart incompleto ou incorreto

Impacto:
- edge público quebra
- portal e v2 ficam indisponíveis em HTTPS

---

### 5. Portal responde em HTTP, mas HTTPS falha

Causas comuns:

- problema na camada de certificado
- problema de vhost TLS
- problema de redirecionamento ou bind da borda segura

Impacto:
- a camada pública perde a forma operacional esperada de acesso

---

## Riscos e cuidados

Os principais riscos desta camada incluem:

- expiração silenciosa de certificado
- drift entre domínio público e certificado ativo
- alteração de borda sem considerar impacto na renovação
- troubleshooting incompleto que ignora Certbot ao investigar indisponibilidade pública
- tratar TLS como “acabamento”, e não como parte do runtime público real

Cuidados permanentes:

- validar periodicamente a borda HTTPS
- validar `certbot certificates` quando houver suspeita
- correlacionar domínio, certificado e Nginx sempre em conjunto
- não assumir que serving estático íntegro significa experiência pública saudável

---

## Limites deste documento

Este documento não detalha:

- arquivo completo da configuração do Certbot
- timers/units exatos de renovação sem reconciliação fina no host
- detalhes de todos os caminhos internos de certificados
- configuração textual completa do Nginx para TLS
- troubleshooting aprofundado de DNS
- políticas avançadas de segurança HTTP

Esses tópicos devem ser tratados em documentos complementares ou após validação adicional do ambiente real.

---

## Critério de pronto deste documento

Este documento pode ser considerado maduro quando:

- a relação entre Certbot, Nginx e o domínio público do lado Hostinger estiver reconciliada sem ambiguidade
- a rotina de emissão/renovação estiver suficientemente compreendida no ambiente real
- os comandos de validação refletirem a prática operacional real
- os riscos de expiração, mismatch e quebra pós-alteração de borda estiverem bem representados
- ele puder ser usado como referência da camada TLS do lado Hostinger sem depender do master legado

---

## Última revisão

- Status: ativo
- Classificação: canônico
- Contexto: infraestrutura Hostinger / Certbot
- Última revisão: 2026-03-18