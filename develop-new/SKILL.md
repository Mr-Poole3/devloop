---
name: develop-new
description: Create a brand-new software project from scratch and wire it into the DevLoop workflow — requirement grilling, a confirmed blueprint, official scaffold CLI (or minimal template fallback), and a verified devloop/ structure. Use when the user wants to start a new project from nothing, rather than modify an existing codebase.
---

# Develop New

Create a brand-new project and wire it into the DevLoop workflow.

## Quick Start

```text
User: /develop-new I want to build a blog platform
AI: [target: ./blog-platform — empty, OK]
    [state: devloop/.state/blog-platform-scaffold.yaml (stage: intake)]
    [starts grilling...]
    → blueprint confirmed → scaffold via create-vite → wiring → verified
    Project blog-platform is ready.
    Next: /develop <your first feature>
```

## Prerequisites

- A target directory that is **empty** (or only `.git`). Existing code is handled by `/devloop-setup`, not here.
- OpenSpec CLI recommended but not required at creation time. If missing, L2/L3 feature changes later will pause with install instructions.

## Entry Point

The user provides a requirement via `/develop-new <requirement>` or in conversation. `develop-new` creates the project and wires DevLoop; it does **not** implement the requirement's features. Feature work is driven later by `/develop <feature>`.

## Lifecycle

`develop-new` follows a 6-stage state machine. Stages are actions that can be revisited, not locked phases.

```text
intake → grilling → blueprinting → scaffolding → wiring → done
```

See [scaffold-lifecycle.md](references/scaffold-lifecycle.md) for full stage details, node/state management, and recovery.

### 1. intake

Resolve the target directory and validate it (empty or only `.git`; reject existing `devloop/` or source code with a pointer to `/devloop-setup`). Create the scaffold state file `devloop/.state/<project>-scaffold.yaml` (`workflow: project-creation`, `stage: intake`). If an in-progress scaffold exists, offer: continue / abandon / start new.

### 2. grilling

Clarify the project with the user. Ask **one question at a time, with a recommended answer for each**, walking down each branch of the decision tree. Cover goal, users, scope, tech stack, deployment, critical risk domains (security / payment / data / privacy), module division, and conventions. See [grilling-checklist.md](references/grilling-checklist.md).

### 3. blueprinting

Produce `blueprint.md` at the project root: tech selection + rationale, module division, data/API draft, critical-domain guardrails, success criteria. See [blueprint-spec.md](references/blueprint-spec.md).

**⏸️ [CONFIRMATION POINT: blueprint]** — present goal / users / scope / tech stack / planned modules / critical domains / success criteria. Rejection returns to grilling for the affected branch.

### 4. scaffolding

Scaffold the skeleton with the official CLI for the chosen stack (see [stack-recipes.md](references/stack-recipes.md)); fall back to the `templates/minimal-ts/` template when no official CLI exists. Create `src/<module>/` placeholders per the blueprint. Run install + test + typecheck as a **skeleton health proof** — the skeleton must build and test green. Do not implement features.

### 5. wiring

Generate the full `devloop/` structure (config, context, decisions, research, tickets, reports, openspec, `.state`). `architecture-map.md` and `module-index.yaml` are filled from the blueprint's planned modules (`has_spec: false`), preserving the Brownfield L1 on-demand strategy. `CONTEXT.md` is filled from grilling results. `.gitignore` gains `devloop/.state/`. Run an equivalent of `/devloop-setup check` to verify. See [wiring.md](references/wiring.md).

**⏸️ [CONFIRMATION POINT: wiring]** — present created file tree / tech stack / commands / modules indexed / warnings (e.g. OpenSpec CLI missing) / suggested first `/develop` feature. Rejection → fix, re-verify, re-present.

### 6. done

Output the delivery report: tech stack, commands, modules, file tree, warnings, and next steps (`/develop <first feature>`, `/devloop-setup check`). Set `stage: done` in the state file.

## Node / State Management

- One state file per scaffold: `devloop/.state/<project>-scaffold.yaml`, `schema_version: 1`, **additive** `workflow: project-creation` marker.
- The `devloop` reader must parse the file and ignore the extra fields; never bump `schema_version` or rename shared fields.
- Each stage transition updates `stage`, `last_updated`, and `metrics.stage_durations`; failures increment `metrics.failure_counts`; confirmation rejections increment `metrics.confirmation_rejections`.
- **3 consecutive failures → force pause** with a diagnostic report.
- Interruption → save state + handoff summary; resume via `/develop-new <same requirement>` continues from `stage`.

A `develop-new` scaffold and a `devloop` feature change are different workflows on different state files — they do not collide.

## Confirmation Points

| Point | When | What's Presented |
|-------|------|------------------|
| **Blueprint** | After blueprinting, before scaffolding | Goal, users, scope, tech stack + rationale, planned modules, critical domains, success criteria |
| **Wiring** | After wiring, before done | Created file tree, commands, modules indexed, CONTEXT terms, warnings, next step |

Never auto-approve a confirmation point. If rejected, fix and re-present rather than proceeding.

## Host Compatibility

`develop-new` must work identically across AI coding agents. No workflow branch is gated on a host brand (no `if claude`); differences are handled by runtime capability detection: sub-agents / background tasks enable parallel checks and degrade to serial execution of the same steps; auto skill trigger degrades to manual `/develop-new` invocation. See [scaffold-lifecycle.md](references/scaffold-lifecycle.md#host-compatibility).

## Rules

- **Greenfield only.** Existing codebase → point to `/devloop-setup`. Existing `devloop/` project → point to `/develop`.
- **Blueprint before scaffold.** Never scaffold without an approved blueprint.
- **Skeleton health is a gate.** Install + test + typecheck must pass before wiring.
- **No feature implementation.** The scaffold is structural; behavior is driven later by `/develop`.
- **Wiring must pass `/devloop-setup check`.** The output is indistinguishable from `devloop-setup init`.
- **Host-agnostic.** Capability detection, never host-name branching.
- **Idempotent state.** Re-running continues the in-progress scaffold from its `stage`; never re-grills or re-scaffolds.

## References

- [Scaffold Lifecycle](references/scaffold-lifecycle.md) — 6-stage state machine, node management, recovery
- [Grilling Checklist](references/grilling-checklist.md) — new-project clarification coverage
- [Stack Recipes](references/stack-recipes.md) — official CLI commands + fallback rules
- [Blueprint Spec](references/blueprint-spec.md) — blueprint.md structure and rules
- [Wiring](references/wiring.md) — devloop/ generation checklist + verification
