# AGENTS.md — HSC Docs

## Project context

This repository contains the canonical documentation for the HSC project.

Repository:

```text
hsc-docs
```

This repository is documentation-first. It is not a runtime application and should not be treated as a deployable product.

Canonical context folders:

```text
docs/00-governance
docs/01-infra-hostinger
docs/02-game-panel
docs/03-portal-estatico
docs/04-infra-aws-lightsail
docs/05-backoffice-admin
docs/06-player-bunker
docs/95-impl-log
docs/97-audit
docs/98-legacy
```

## Agent role

Codex should act as a documentation maintenance assistant.

Codex may edit Markdown files when the task is explicit and the target context is clear.

Codex must not make architecture, product, runtime, deployment, infrastructure, API, ETL, Game Panel, or security decisions independently.

## Source of truth rules

Treat this repository as the canonical documentation source for the HSC project.

Use the context structure to determine where information belongs.

Do not duplicate information across contexts unless explicitly requested.

Prefer linking to the canonical document over copying entire sections.

When information conflicts, do not silently choose one version. Identify the conflict and ask the human for a decision.

## Context boundaries

### `docs/00-governance`

Purpose:

```text
documentation governance
master index
maintenance playbook
documentation system
Codex usage policy
cross-context rules
```

Allowed:

```text
update indexes
maintain documentation policy
add cross-context guidance
fix navigation and links
document documentation workflow
```

Do not change governance taxonomy without explicit approval.

### `docs/01-infra-hostinger`

Purpose:

```text
Hostinger VPS
Nginx
TLS/Certbot
Docker host
systemd
webroots
Static API hosting
public portal hosting
```

Allowed:

```text
document runbooks
document diagnostics
organize operational notes
update read-only validation procedures
```

Do not prescribe live infrastructure changes unless the task explicitly asks for a runbook.

### `docs/02-game-panel`

Purpose:

```text
AMP/CubeCoders
CS2 server
MixHAXIXE01
MetaMod
CounterStrikeSharp
MatchZy
plugins
admin operations
troubleshooting
```

Allowed:

```text
document troubleshooting
document plugin state
organize operational findings
update runbooks
```

Do not infer plugin removal, server restart, update, or runtime action as safe without explicit validation.

### `docs/03-portal-estatico`

Purpose:

```text
Static API v2
portal público
ETL outputs
MatchZy data flow
Angular cs2-next context when relevant
public JSON contracts
```

Allowed:

```text
document portal behavior
document Static API contracts
document ETL-to-portal boundaries
document staging/deploy findings
```

Do not change API contract meaning without explicit approval.

### `docs/04-infra-aws-lightsail`

Purpose:

```text
Auth API infrastructure
Lightsail
Node.js service
MariaDB
Nginx reverse proxy
systemd
release/rollback
migrations
```

Allowed:

```text
document backend runbooks
document release procedures
document local/prod separation
document migration policy
```

Do not convert runbook steps into automatic execution guidance without explicit approval.

### `docs/05-backoffice-admin`

Purpose:

```text
Backoffice Admin SPA
Auth API integration
admin UX
guards
RBAC
news/users/admin features
local proxy and local validation
```

Allowed:

```text
document admin features
document local development flow
document Auth API integration assumptions
document feature status
```

Do not change RBAC/auth policy or product behavior as a documentation assumption.

### `docs/06-player-bunker`

Purpose:

```text
player-facing logged-in area
Bunker
Player Auth architecture
separation between Admin Auth and Player Auth
Steam as initial identity provider
```

Allowed:

```text
document Player Auth decisions
document Bunker architecture
document player-facing auth boundaries
document Steam identity assumptions
link to related Auth API, Portal, and Static API docs
```

Do not change authentication, identity, entitlement, billing, product, or API contract decisions as a documentation assumption.

### `docs/95-impl-log`

Purpose:

```text
implementation history
chronological records
execution logs
merge/deploy notes
```

Allowed:

```text
append factual implementation notes
summarize completed work
link logs to canonical docs
```

Do not rewrite history as current canonical state without reconciliation.

### `docs/97-audit`

Purpose:

```text
open questions
risks
inconsistencies
reconciliation backlog
```

Allowed:

```text
organize findings
turn findings into checklists
mark items with evidence
link to source docs
```

Do not close audit items without explicit evidence.

### `docs/98-legacy`

Purpose:

```text
legacy documentation
historical blueprints
old master docs
preserved prior context
```

Allowed:

```text
extract relevant history
compare old docs with current docs
propose reconciliation
```

Do not treat legacy documents as canonical automatically.

## Canonical vs local repository docs

This repository contains project-wide documentation.

Implementation repositories may contain local operational docs and `AGENTS.md` files, for example:

```text
hsc-cs2-portal/AGENTS.md
hsc-auth-api/AGENTS.md
hsc-backoffice-admin/AGENTS.md
hsc-cs2-etl/AGENTS.md
```

Do not assume those repositories are available from this workspace.

If a documentation change requires current code context from another repository, ask the human to provide the relevant file, diff, command output, or repository checkout.

## Relation to Codex usage policy

The canonical policy for Codex usage is:

```text
docs/00-governance/codex-usage-policy.md
```

Follow it when updating documentation related to agents, Codex, MCP, repo-local instructions, or development workflow.

Do not contradict this policy in repo-local docs.

## Allowed work

Codex may:

```text
create Markdown docs
edit Markdown docs
fix broken links
update indexes
add cross-references
summarize implementation logs
convert notes into structured docs
standardize headings
improve wording without changing meaning
add checklists
document validated command outputs
```

## Forbidden work without explicit approval

Do not:

```text
rename context directories
delete documents
move documents across contexts
rewrite the documentation architecture
promote legacy docs to canonical status
change architecture decisions
change product decisions
invent runtime state
invent command outputs
invent validation results
create deployment instructions without rollback/safety notes
copy secrets or sensitive operational values into docs
```

## Documentation quality rules

Prefer:

```text
clear context ownership
short sections
explicit assumptions
links to related docs
dates when documenting historical events
separation of current state vs historical notes
```

Avoid:

```text
duplicated long sections
ambiguous "current" claims without evidence
mixing legacy notes into canonical docs
large rewrites without reason
uncited operational claims
```

## Link and navigation rules

When adding a new document:

```text
place it in the correct context
link it from the context README when relevant
link it from the master index if it is important enough
add cross-links to related contexts when useful
```

Important governance docs:

```text
docs/00-governance/README.md
docs/00-governance/99-master-index.md
docs/00-governance/documentation-system.md
docs/00-governance/hsc-docs-maintenance-playbook.md
docs/00-governance/codex-usage-policy.md
```

## Validation

Before finalizing documentation changes, run:

```bash
git status --short
git diff --check
git diff --stat
```

If links are changed, inspect them manually.

Do not claim validation that was not run.

Always report:

```text
files changed
summary of changes
validation commands run
warnings/errors
```

## Git workflow

Work on a feature branch.

Prefer branch names such as:

```text
docs/<short-topic>
```

Before committing:

```bash
git status --short
git diff --check
git diff --stat
```

Use focused commits.

Do not alter unrelated files.

## Writing style

Use Portuguese for project documentation unless the surrounding document is already in English.

Preserve the language and tone of the target document.

Keep technical terms precise.

Do not over-polish logs or historical notes in a way that changes their meaning.

## Stop and ask when

Stop and ask the human when the task involves:

```text
architecture decisions
context taxonomy changes
canonical source conflicts
legacy-to-current reconciliation
deploy or rollback guidance
security-sensitive details
secrets
API contract changes
ETL contract changes
Game Panel operations
infrastructure operations
large-scale rewrite
```
