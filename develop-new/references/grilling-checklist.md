# New-Project Grilling Checklist

What `develop-new` must resolve with the user before producing a blueprint.

## Method

- Ask questions **one at a time**. Never dump a survey.
- Provide a **recommended answer** with every question.
- Walk down each branch of the decision tree; resolve dependencies between decisions one by one (e.g. framework choice before folder structure).
- If a question is answerable from the environment (tools installed, existing conventions), check instead of asking.
- Record each settled decision into the scaffold state file as you go.

## Coverage Checklist

Resolve every branch below. A branch is *resolved* when the user has made an explicit choice (or accepted the recommendation).

### 1. Problem & Goal

- What problem does this project solve, and for whom?
- What is the one-line goal statement?
- Is this an MVP, a prototype, or a production system?

### 2. Users & Context

- Who are the primary users?
- What environment will they be in (internal, public, mobile, desktop)?
- Any multi-tenant / multi-org isolation needed?

### 3. Scope

- What is in the MVP? What is explicitly out of scope for now?
- What are the success criteria (measurable)?
- What would a failed first version look like (failure modes)?

### 4. Tech Stack

- Language and runtime preference, or accept the recommendation.
- Framework (web/CLI/library), or "none".
- Package manager.
- Test framework and test directory convention.
- Typecheck / lint requirements (strict TS? linter now or later?).
- **Host note:** some hosts (Claude Code, Codex, OpenCode, TRAE, Cursor) support sub-agents / background tasks. `develop-new` never branches on host name — capability is detected at runtime and the same serial fallback runs everywhere.

### 5. Deployment Target

- Where will this run (Node server, static host, desktop, library, CI)?
- Does the environment impose constraints (Node version, no native deps, offline)?

### 6. Critical Risk Domains

Check each — if present, the domain MUST be recorded in the blueprint and in `CONTEXT.md`:

- **Security** — auth, permissions, secrets, input handling.
- **Payment** — money movement, billing, financial data.
- **Data / Privacy** — PII, personal data, data retention, compliance (GDPR etc.).
- If any domain is hit: the first `/develop` change that touches it gets dedicated tests, per DevLoop's critical-domain rules. Note this in the blueprint so it is never a surprise.

### 7. Module Division

- What are the natural module boundaries (e.g. `auth`, `posts`, `billing`)?
- For each planned module: one-line responsibility, and its expected dependencies on sibling modules.
- These become `src/<module>/` directories and the entries in `module-index.yaml`.

### 8. Data & API Draft

- What core entities exist and their rough relationships?
- Any API surface to expose (REST / RPC / CLI)?
- Does the MVP need persistence, or is in-memory enough?

### 9. Conventions

- Naming conventions (PascalCase files vs kebab-case, etc.).
- Test naming (`*.test.ts`, `*.spec.ts`).
- Commit / PR conventions if the team has them.

## Exit Condition

All decision branches resolved; the user and the agent share an understanding of **what will be built, how, and with what guardrails**. Only then produce the blueprint. If a new decision emerges during blueprinting or scaffolding, return to grilling for that branch rather than assuming.
