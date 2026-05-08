# HSC Docs

Este repositório é a documentação canônica do ecossistema HSC.

Ele registra arquitetura, infraestrutura, runbooks, decisões, contratos públicos, histórico de implementação e regras de manutenção documental. Este repositório não é uma aplicação runtime e não deve ser tratado como produto deployável.

## Entrada principal

- [Home da documentação](./docs/README.md)
- [Mapa de repositórios HSC](./docs/00-governance/hsc-repositories-map.md)
- [Master Index](./docs/00-governance/99-master-index.md)

## Contextos principais

- `docs/00-governance`
- `docs/01-infra-hostinger`
- `docs/02-game-panel`
- `docs/03-portal-estatico`
- `docs/04-infra-aws-lightsail`
- `docs/05-backoffice-admin`
- `docs/95-impl-log`
- `docs/97-audit`
- `docs/98-legacy`

## Segurança

- não commitar segredos
- não registrar tokens, credenciais, cookies, chaves, SMTP, DB URLs ou valores reais de `.env`
- não tratar este repositório de documentação como runtime deployável
- não usar documentação genérica como substituto de runbooks com rollback e notas de segurança

## Fluxo básico

1. trabalhar em branch
2. editar Markdown no contexto correto
3. rodar `git diff --check`
4. abrir PR pequeno e focado
