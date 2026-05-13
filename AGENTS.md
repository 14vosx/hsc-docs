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

Codex must not make architecture, product, runtime, deployment, infrastructure, API, ETL, Game Panel, billing, authentication, security, or production-data decisions independently.

Documentation may record approved decisions and validated facts. Documentation must not invent decisions.

## Current HSC state to preserve

Current important project state:

```text
/portal/cs2
  current official/legacy portal path
  do not change unless the human explicitly starts a cutoff/rollback task

/portal/cs2-next
  active public canary for Angular CS2 Next
  contains the Player Bunker route

/player/*
  reverse-proxied by Hostinger Nginx to the Auth API on AWS
  used for Player Auth and Player Bunker authenticated data

Auth API AWS
  owns Admin Auth, Player Auth, session, Steam login, and Bunker gateway behavior

hsc-cs2-etl
  owns competitive stats materialization and Season/player artifacts

hsc-cs2-portal
  owns Angular presentation and canary UI

hsc-docs
  owns canonical documentation, decisions, runbooks, checkpoints, and planning
```

There has been no cutoff from `/portal/cs2-next` to `/portal/cs2`.

Do not write docs that imply the cutoff already happened.

## Player Bunker documentation boundary

The Player Bunker lives under:

```text
docs/06-player-bunker
```

This context covers:

```text
player-facing logged-in area
Bunker product/architecture plans
Player Auth architecture
Steam-first player identity
Admin Auth vs Player Auth separation
Bunker deploy/rollback plans
local/canary validation checkpoints
MVP2 Bunker enrichment plans
artifact and Auth API integration boundaries
```

Player Bunker documentation must preserve these boundaries:

```text
ETL owns competitive stats materialization
Auth API owns authenticated access and response shaping
Portal owns presentation
Docs records decisions and runbooks
Backoffice is admin/internal and not the Bunker
Brand Hub is brand/static and not the Bunker
```

Do not document Bunker as if it were part of Backoffice.

Do not document Bunker as if it were a Brand Hub route.

Do not document Bunker as if the Portal directly reads ETL artifacts.

## MVP2 Bunker documentation rules

The current approved next product direction is:

```text
MVP2 — Bunker Melhorado
```

Documentation may describe the approved MVP2 scope:

```text
enrich /player/bunker/summary
use existing data already available in the HSC ecosystem
keep Auth API as authenticated gateway
separate Season data from lifetime/competitive profile data
preserve /portal/cs2-next as canary
do not cutoff /portal/cs2
do not start billing
do not create a Bunker subdomain
do not migrate to Angular Material
do not create a new backend service
do not recalculate ranking or score outside the canonical owner
```

Do not expand MVP2 scope without explicit human approval.

Do not present future ideas as approved scope.

Future topics such as billing, entitlements, premium gates, public profile sharing, ranking eligibility, and subdomains must stay clearly marked as future/out-of-scope unless approved.

## Documentation is not deployment authorization

A runbook, checklist, or plan in this repository does not authorize live changes by itself.

Documentation may include operational steps, but Codex must not execute or recommend immediate production mutation unless the human explicitly starts an execution task.

Docs may cover:

```text
dry-run commands
read-only diagnostics
validation steps
rollback plans
go/no-go checklists
post-deploy evidence templates
```

Docs must not silently turn into:

```text
deploy
release
rollback
cutoff
systemd change
Nginx change
DNS/TLS change
production migration
production data mutation
webroot publication
```

## Source of truth rules

Treat this repository as the canonical documentation source for the HSC project.

Use the context structure to determine where information belongs.

Do not duplicate information across contexts unless explicitly requested.

Prefer linking to the canonical document over copying entire sections.

When information conflicts, do not silently choose one version. Identify the conflict and ask the human for a decision.

When recording status, distinguish clearly between:

```text
planned
implemented locally
validated locally
published to canary
published to production
deprecated
future/out-of-scope
```

Do not rewrite history as current canonical state without reconciliation.

## Evidence and safety rules

Allowed evidence in docs:

```text
HTTP status
sanitized command output
commit hashes
PR numbers
repo names
route names
artifact slug
artifact counts
non-secret filesystem paths
screenshots without secrets/cookies/tokens
safe JSON summaries
```

Forbidden evidence unless explicitly sanitized and approved:

```text
cookies
session tokens
token hashes
Steam API keys
DB credentials
.env values
private keys
authorization headers
raw callback query strings with sensitive values
personal data beyond what is already intentionally public in the project context
```

If in doubt, redact.

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
repository map
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
/player/* reverse proxy documentation
```

Allowed:

```text
document runbooks
document diagnostics
organize operational notes
update read-only validation procedures
record approved proxy/deploy findings
```

Do not prescribe live infrastructure changes unless the task explicitly asks for a runbook.

Do not imply a runbook has been executed unless evidence is provided.

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
portal publico
ETL outputs
MatchZy data flow
Angular cs2-next context when relevant
public JSON contracts
public canary and publication boundaries
```

Allowed:

```text
document portal behavior
document Static API contracts
document ETL-to-portal boundaries
document staging/deploy findings
document cs2-next canary state
```

Do not change API contract meaning without explicit approval.

Do not document `/portal/cs2-next` as official replacement for `/portal/cs2` until cutoff is explicitly approved and executed.

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
Player Auth runtime configuration when infrastructure-related
```

Allowed:

```text
document backend runbooks
document release procedures
document local/prod separation
document migration policy
document sanitized runtime config names
```

Do not convert runbook steps into automatic execution guidance without explicit approval.

Do not print secrets or production environment values.

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

Do not mix Backoffice Admin with Player Bunker scope.

### `docs/06-player-bunker`

Purpose:

```text
player-facing logged-in area
Bunker
Player Auth architecture
separation between Admin Auth and Player Auth
Steam as initial identity provider
Bunker artifact and Auth API gateway boundaries
MVP2 enrichment planning
canary validation checkpoints
deploy/rollback planning
```

Allowed:

```text
document Player Auth decisions
document Bunker architecture
document player-facing auth boundaries
document Steam identity assumptions
document canary validation
document MVP2 scope and constraints
link to related Auth API, Portal, ETL, and Static API docs
```

Do not change authentication, identity, entitlement, billing, product, or API contract decisions as a documentation assumption.

Do not document billing/subscription/premium gates as approved MVP2 work.

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
record unresolved questions
record inconsistencies
record reconciliation tasks
record known risks
```

Do not use audit docs as canonical architecture decisions until reconciled.

### `docs/98-legacy`

Purpose:

```text
legacy references
historical docs
deprecated notes
old plans
```

Allowed:

```text
preserve historical context
link legacy context when useful
```

Do not promote legacy behavior to current state without confirmation.

## Repository references

When documentation touches another repo, use the canonical repo names:

```text
hsc-docs
hsc-auth-api
hsc-cs2-etl
hsc-cs2-portal
hsc-backoffice-admin
hsc-brand-hub
```

Do not invent new repository responsibilities.

If a plan proposes a new repo/service, mark it as future/proposed and ask for approval.

## Git workflow

Work on a feature branch.

Before committing, verify:

```bash
git status --short
git diff --check
git diff --stat
```

Do not alter unrelated files.

Prefer focused commits.

Do not commit generated artifacts unless explicitly requested.

## Validation

For documentation changes, validate:

```text
target file path is correct
heading structure is coherent
links/routes/repo names are consistent
status labels are explicit
no secrets are included
git diff --check passes
```

If a documentation update references implementation status, require evidence from:

```text
sanitized user-provided output
repo history
approved prior docs
explicit human statement
```

Do not claim validation that was not actually performed.

## Stop and ask when

Stop and ask the human when documentation would decide or imply:

```text
cutoff from /portal/cs2-next to /portal/cs2
production deploy
rollback
Nginx/systemd/DNS/TLS change
database migration
API contract change
auth/session/cookie policy change
Admin Auth and Player Auth integration
billing/subscription/entitlement policy
new service/repository
ranking/scoring semantics
Season membership semantics
public/private data policy
LGPD/security policy
```
