# Backoffice UI Material Foundation

## Navegação rápida

- [Backoffice Admin](./README.md)
- [References and Inventory](./backoffice-admin-references-inventory.md)
- [Master Index](../00-governance/99-master-index.md)

---

## Objetivo

Documentar a fundação transversal de UI Material do Backoffice Admin.

Esta fundação existe para:

- padronizar interações e feedbacks administrativos com Angular Material
- separar feedback transitório, feedback persistente, confirmação e input simples
- reduzir dependência de APIs nativas do navegador em fluxos administrativos
- criar uma base compartilhada para evolução visual futura sem tratar essa evolução como já concluída

Regra importante:

- este documento registra o estado funcional da fundação atual
- ele não documenta dark mode, redesign global ou design system completo como implementados

---

## Estado implementado

O estado conhecido da fundação UI Material é:

- Angular Material/CDK está presente no Backoffice
- `src/app/shared/ui` funciona como base compartilhada para componentes e serviços transversais de UI
- foram materializados componentes e serviços compartilhados para feedback, confirmação e input simples

Componentes e serviços compartilhados:

- `ConfirmationDialogComponent`
- `ConfirmationService`
- `InputDialogComponent`
- `InputDialogService`
- `PageFeedbackComponent`
- `UiFeedbackService`

Leitura correta:

- a fundação cobre padrões administrativos transversais já usados por áreas principais
- a fundação não significa migração total da aplicação para Angular Material
- a fundação não significa padronização visual completa de todas as telas

---

## Padrões de uso

### `UiFeedbackService`

Uso esperado:

- feedback transitório após ações administrativas
- mensagens de sucesso, erro ou informação
- encapsulamento do uso de `MatSnackBar`

Exemplos de uso:

- confirmação visual após criação ou atualização
- erro transitório após falha de ação administrativa
- mensagem informativa breve quando a ação não exige estado persistente na página

---

### `PageFeedbackComponent`

Uso esperado:

- feedback persistente de página
- estados de loading
- estados de erro
- empty states
- mensagens de contexto que devem permanecer visíveis junto ao conteúdo

Leitura correta:

- feedback persistente deve ficar próximo da superfície afetada
- ações transitórias não devem ser promovidas automaticamente a estado permanente de página

---

### `ConfirmationService`

Uso esperado:

- ações sensíveis que exigem confirmação explícita
- fluxos administrativos em que a confirmação deve ser padronizada
- substituição de `window.confirm` em novas telas e fluxos migrados

Exemplos de uso:

- publish/unpublish/remove em `news`
- activate/close em `seasons`
- alteração sensível de papel em `users`

---

### `InputDialogService`

Uso esperado:

- input administrativo simples
- fluxos pequenos que não justificam uma tela ou formulário dedicado
- substituição de `window.prompt` em novas telas e fluxos migrados

Exemplos de uso:

- renomear usuário
- alterar email de usuário

---

### APIs nativas do navegador

Novas telas e fluxos administrativos devem evitar:

- `alert(...)`
- `window.confirm`
- `window.prompt`

Leitura correta:

- a ausência dessas APIs é critério da fundação atual nos fluxos principais de `news`, `seasons` e `users`
- este documento não afirma que todos os módulos futuros já seguem esse padrão

---

## Aplicação por área

### Seasons

Estado implementado:

- activate/close usam confirmação padronizada e feedback transitório
- list/edit/form usam `PageFeedbackComponent` para feedback persistente
- SCSS de `seasons` usa tokens Material/CSS variables no lugar de cores hardcoded no escopo migrado

Lacunas fora desta fundação:

- ranking por season
- partidas associadas
- snapshot histórico
- integrações futuras de domínio

---

### News

Estado implementado:

- publish/unpublish/remove usam confirmação padronizada e feedback transitório
- list/edit/form usam `PageFeedbackComponent` para feedback persistente
- SCSS de `news` usa tokens Material/CSS variables no lugar de cores hardcoded no escopo migrado

Leitura correta:

- a fundação de UI não altera contratos de conteúdo, publish/unpublish ou remoção
- este documento registra apenas o padrão de interação e feedback da UI administrativa

---

### Users

Estado implementado:

- create/update usam feedback transitório padronizado
- changeRole usa confirmação padronizada
- rename/email usam input dialog padronizado
- fluxos principais migrados não dependem de `alert(...)`, `window.prompt` ou `window.confirm`

Lacuna conhecida:

- `users` ainda não teve padronização visual completa de layout, formulário e listagem

---

## Tokens e dark mode

Estado implementado:

- SCSS de `news` e `seasons` trocou cores hardcoded por tokens Material/CSS variables no escopo migrado
- SCSS de Core layout, Auth e Dashboard também trocou cores hardcoded por tokens Material/CSS variables no escopo auditado da fatia `#17`
- a auditoria restrita de Core/Auth/Dashboard/Users não apontou cores hardcoded residuais após a fatia `#17`
- essa troca reduz risco para uma evolução futura de tema

Estado não implementado:

- dark mode ainda não está implementado
- não há toggle de tema
- não há design system completo
- a migração total para Angular Material ainda não está concluída

Regra importante:

- a fundação prepara caminho para dark mode futuro
- ela não deve ser descrita como dark mode pronto

---

## Lacunas explícitas

As lacunas conhecidas desta etapa são:

- Users ainda não teve padronização visual completa de layout/form/list
- dark mode completo ainda não existe
- não há toggle de tema
- não há design system completo
- a migração total para Angular Material ainda não deve ser tratada como concluída
- o redesign global do Backoffice ainda não deve ser tratado como concluído
- validação visual futura ainda é necessária antes de ativar dark mode ou toggle de tema

Essas lacunas devem permanecer documentadas como dívida futura até haver evidência explícita de implementação.

---

## Critério de pronto da fundação atual

A fundação UI Material atual pode ser considerada pronta no escopo documentado quando:

- ações administrativas principais de `news`, `seasons` e `users` não dependem mais de `alert(...)`, `window.prompt` ou `window.confirm`
- feedback transitório de ações administrativas usa `UiFeedbackService`/`MatSnackBar`
- feedback persistente de `news` e `seasons` está centralizado com `PageFeedbackComponent`
- confirmações sensíveis de `news`, `seasons` e `users` usam `ConfirmationService`
- inputs administrativos simples já migrados usam `InputDialogService`
- SCSS de `news`, `seasons`, Core layout, Auth e Dashboard usa tokens Material/CSS variables nos escopos já auditados/migrados

Leitura correta:

- esse critério vale apenas para a fundação transversal descrita neste documento
- ele não fecha lacunas de tema, design system, layout global ou domínios administrativos futuros
