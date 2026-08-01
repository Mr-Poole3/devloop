# Blueprint Spec

Structure and requirements for the `blueprint.md` that `develop-new` produces at the end of the blueprinting stage. It is the plan for the project's first version — a scaffold plus module division — not a behavior spec (behavior specs are created later by `/develop` via OpenSpec, on first touch).

## File

Written to `blueprint.md` at the project root (sibling of `devloop/` and `src/`). It stays as a living plan; `/develop` changes reference it during the first feature. Once the project has a `devloop/openspec/specs/` registry, the blueprint's role shrinks to historical context.

## Structure

```markdown
# Blueprint: <Project Name>

> Created by develop-new on <date>. Plan for the first version. Feature behavior is
> specified later via /develop + OpenSpec.

## Goal
<1-2 sentences: what the project does and why it exists>

## Users
<who it serves, their context>

## Scope
### In
<bulleted>
### Out
<bulleted>

## Success Criteria
<bulleted, measurable>

## Tech Stack
| Concern | Choice | Rationale |
|---------|--------|-----------|
| Language | <e.g. TypeScript> | <why> |
| Framework | <e.g. none / React / Fastify> | <why> |
| Runtime | <Node.js / Deno / ...> | <why> |
| Package manager | <npm / pnpm / cargo> | <why> |
| Test | <Vitest / cargo test> | <why> |

## Module Division
| Module | Path | Responsibility | Depends on |
|--------|------|----------------|------------|
| app | src/app/ | <one line> | — |
| auth | src/auth/ | <one line> | app |

## Data Model Draft
<core entities and rough relationships; "in-memory for MVP" is an acceptable answer>

## API Draft
<exposed surface; "none yet" is acceptable>

## Critical Domains & Guardrails
<security / payment / data / privacy, if any; for each: what is in scope and the
non-negotiable rule, e.g. "payment: no money math in the UI layer; use a provider API">

## Failure Modes
<what a failed first version looks like>

## Out of Bounds
<things the agent must NOT do without explicit permission — seeded into CONTEXT.md>
```

## Rules

1. **One-line responsibilities.** Module division entries must be single-line so they can be copied verbatim into `module-index.yaml` and `architecture-map.md`.
2. **Critical domains are mandatory.** If grilling surfaced security, payment, data, or privacy, the blueprint MUST name them and state a guardrail. This propagates to `CONTEXT.md` and gates dedicated tests later.
3. **No behavior specs.** The blueprint plans *structure and selection*, not feature behavior. Do not turn it into an OpenSpec proposal.
4. **Approved by confirmation point.** The blueprint is presented for [blueprint] confirmation before scaffolding. Rejection returns to grilling for the affected branch only.
5. **Sibling of devloop/.** `blueprint.md` lives at the project root, not inside `devloop/` — it is project planning, not DevLoop infrastructure.
