---
name: devloop-setup
description: Initialize, check, and repair a repository for the DevLoop development workflow — an integrated AI development loop combining requirement grilling, OpenSpec spec-driven changes, TDD, code review, and archival. Use when setting up DevLoop in a new or existing repository, checking workflow status, repairing incomplete configuration, or reconfiguring project conventions.
---

# DevLoop Setup

Initialize and configure a repository for the DevLoop development workflow.

## Quick Start

```text
User: /devloop-setup
AI: Checking project...
     ✓ Tech stack detected: TypeScript / Node.js
     ✓ Test command: pnpm test
     ✓ OpenSpec: not installed
     → Creating devloop/ directory structure...
     → Generating architecture map...
     → Writing config...
     DevLoop is ready. Run /develop <your requirement> to start.
```

## Modes

Identify the mode from the user's message. Default to `check` if unclear.

### init
First-time setup. Create the full `devloop/` directory structure, detect tech stack, generate architecture map, and write config. Never overwrite existing files — skip and report.

### check
Read-only inspection. Report current status without modifying anything.

### repair
Fill in missing pieces. Do not overwrite existing valid config.

### reconfigure
Update project conventions and rules. Confirm before overwriting.

## Process

### 1. Detect existing state

Read the following to determine what already exists:

- `devloop/config.yaml` — DevLoop config
- `devloop/openspec/config.yaml` — OpenSpec config
- `devloop/context/` — architecture map, module index, tech stack
- `openspec/` at project root — legacy OpenSpec installation
- `CONTEXT.md` at project root — legacy context file
- `.gitignore` — check if `devloop/.state.yaml` is ignored

Report findings before taking action.

### 2. Detect tech stack

Read project manifest files to identify language, framework, package manager, and commands:

- `package.json` / `pnpm-workspace.yaml` — Node.js
- `pyproject.toml` / `requirements.txt` — Python
- `Cargo.toml` — Rust
- `go.mod` — Go
- `Makefile` / `Dockerfile` / `docker-compose.yml`
- `.github/workflows/` — CI config

Record: build command, test command, lint command, typecheck command, e2e command.

### 3. Create directory structure

```text
devloop/
├── config.yaml
├── context/
│   ├── architecture-map.md
│   ├── tech-stack.md
│   ├── module-index.yaml
│   └── CONTEXT.md
├── decisions/
├── research/
├── tickets/
├── reports/
└── openspec/
    ├── config.yaml
    ├── specs/
    └── changes/
        └── archive/
```

Only create directories and files that don't exist. See [setup-checklist.md](references/setup-checklist.md).

### 4. Generate architecture map (L0)

Scan top-level source directories. For each module, record:

- Module name and path
- One-line responsibility
- Key entry points or interfaces
- Dependencies on other modules

Do NOT deep-scan implementations. Keep it fast. See [project-detection.md](references/project-detection.md).

### 5. Write configs

Write `devloop/config.yaml` with detected tech stack, risk routing, confirmation policy, and tracker settings.

Write `devloop/openspec/config.yaml` with project context and artifact rules. See templates for format.

If OpenSpec CLI is installed, run `openspec init` inside `devloop/openspec/` or configure paths accordingly. If not installed, write config manually and note the installation requirement.

### 6. Ensure .gitignore

Add the following to `.gitignore` if not present:

```text
devloop/.state.yaml
```

### 7. Initialize OpenSpec

If `devloop/openspec/` is empty and OpenSpec CLI is available:

```bash
cd devloop/openspec && openspec init
```

Then merge generated config with DevLoop rules.

If OpenSpec CLI is not available, create the minimum structure manually and remind the user to install it:

```bash
npm install -g @fission-ai/openspec@latest
```

### 8. Report results

Output a summary:

```text
DevLoop Setup Complete

Detected:
  Tech stack: TypeScript / React / Node.js
  Package manager: pnpm
  Unit test: pnpm test
  Typecheck: pnpm typecheck
  Lint: pnpm lint

Created:
  ✓ devloop/config.yaml
  ✓ devloop/context/architecture-map.md (12 modules indexed)
  ✓ devloop/context/module-index.yaml
  ✓ devloop/context/tech-stack.md
  ✓ devloop/context/CONTEXT.md (template)
  ✓ devloop/openspec/config.yaml
  ✓ devloop/decisions/ (empty)
  ✓ devloop/research/ (empty)
  ✓ devloop/tickets/ (empty)
  ✓ devloop/reports/ (empty)

Warnings:
  ⚠ OpenSpec CLI not found — install with: npm install -g @fission-ai/openspec@latest
  ⚠ CONTEXT.md is a template — fill in domain terms before first use

Next step:
  /develop <your requirement>
```

## Rules

- **Idempotent**: running twice must not destroy existing config.
- **Non-destructive**: never overwrite user-written content.
- **Conflict-aware**: if existing config conflicts with detected settings, pause and ask.
- **Fast**: L0 architecture map should complete in one pass, no deep analysis.
- **Honest**: report all warnings and missing tools.

## References

- [Setup Checklist](references/setup-checklist.md) — full file-by-file checklist
- [Project Detection](references/project-detection.md) — how to detect tech stack and modules
- [Routing Policy](references/routing-policy.md) — risk classification rules
- [Repair Rules](references/repair-rules.md) — what repair mode can and cannot do
