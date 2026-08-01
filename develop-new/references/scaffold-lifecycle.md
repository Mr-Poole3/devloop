# Scaffold Lifecycle

Full state machine for the `develop-new` workflow: creating a brand-new project and wiring DevLoop into it.

## State Diagram

```text
                     ┌──────────┐
                     │  intake  │
                     └────┬─────┘
                          ▼
                    ┌───────────┐
                    │ grilling  │◄──────────────┐
                    └────┬──────┘               │  blueprint rejected
                         ▼                      │
                    ┌─────────────┐             │
                    │ blueprinting│─────────────┘
                    └──────┬──────┘
                           │ [CONFIRM blueprint]
                           ▼
                     ┌────────────┐
                     │ scaffolding│
                     └────┬───────┘
                          ▼
                    ┌──────────┐
                    │  wiring  │
                    └────┬─────┘
                         │ [CONFIRM wiring]
                         ▼
                       done
```

## Node / State Management

`develop-new` reuses the DevLoop state-file pattern exactly: one YAML file per in-progress operation under `devloop/.state/`, `schema_version: 1`, gitignored. The only difference is an additive `workflow` marker so `devloop` (which drives feature changes) and `develop-new` (which creates the project) never collide on the same state file.

```yaml
# devloop/.state/<project>-scaffold.yaml
schema_version: 1
workflow: project-creation        # additive marker; ignored by /develop's reader
change_id: <project>-scaffold
project_name: <name>
target_dir: <path>
risk_level: L2                    # kept for devloop reader compatibility (multi-module effort)
stage: scaffolding                # intake | grilling | blueprinting | scaffolding | wiring | done
tech_stack:
  language: TypeScript
  framework: null
  runtime: Node.js
  package_manager: npm
commands:
  test: npm test
  typecheck: npm run typecheck
  lint: npm run typecheck
  build: npm run build
modules:
  - name: app
    path: src/app/
    responsibility: "Application entry and wiring"
    has_spec: false
confirmed: [blueprint]
pending_confirmation: [wiring]
consecutive_failures: 0
metrics:
  stage_durations: { intake: 3s, grilling: 180s }
  failure_counts: { scaffold: 0, verify: 0 }
  confirmation_rejections: 0
created_at: "2026-08-01T10:00:00Z"
last_updated: "2026-08-01T10:40:00Z"
```

Rules:

- Filename is `<project>-scaffold.yaml`; a scaffold is in-progress when the file exists and `stage != done`.
- `workflow: project-creation` is **additive only**. Do not bump `schema_version`; do not rename or re-type any field `devloop` understands. The `devloop` reader must be able to parse the file and ignore the extra fields.
- On entering and exiting each stage, record `metrics.stage_durations[<stage>]`. On any failure, increment `metrics.failure_counts[<type>]`. On confirmation rejection, increment `metrics.confirmation_rejections`. On interrupt, save the handoff summary (see below).
- **3 consecutive failures → force pause.** Output a diagnostic report and request user intervention — do not keep retrying.
- A `devloop` feature change and a `develop-new` scaffold are **different workflows on different state files**. They do not interact.

## Stage Details

### 1. intake

**Entry:** User provides a requirement via `/develop-new <requirement>` or natural language.

**Actions:**

1. Scan `devloop/.state/*.yaml`:
   - If a file with `workflow: project-creation` and `stage != done` exists, list it and ask: **continue / abandon / start new**.
   - If a legacy `devloop/.state.yaml` (no `schema_version`) exists, do **not** migrate it here — that is `devloop`'s job. Just report its presence.
2. Resolve the **target directory**:
   - Default: a new directory under the current working directory named from the project name.
   - Ask the user to confirm the target path during grilling.
3. Validate the target directory:
   - **Empty** (no files, or only `.git`) → OK.
   - Already contains `devloop/` or source code → **reject**. Do not scaffold into it. Suggest `/devloop-setup` instead (it detects existing code).
   - Already exists and is a `devloop`-wired project → **reject**, recommend `/develop`.
4. If the target is not already a git repository, run `git init` (if the host supports it).
5. Create `devloop/.state/<project>-scaffold.yaml` with `stage: intake`, `workflow: project-creation`, `schema_version: 1`, empty `metrics`, and `last_updated` set.

**Exit:** Target validated. State file created. Ready to grill.

### 2. grilling

**Entry:** State file exists (`stage: grilling`).

**Actions:**

1. Use the `grilling` skill if available (from `mattpocock/skills`); otherwise follow the same rules inline: **ask questions one at a time, provide a recommended answer for each, walk down each branch of the decision tree**, and resolve dependencies between decisions one by one.
2. Cover the full checklist. See [grilling-checklist.md](grilling-checklist.md). At minimum:
   - Project goal and the problem it solves.
   - Primary users and their context.
   - Core domain terms (these become `CONTEXT.md`).
   - MVP scope — what is in, what is out.
   - Success criteria.
   - Tech stack preference (or accept the recommendation).
   - Deployment target / platform.
   - **Critical risk domains** (security, payment, privacy, user data): if any are in scope, they MUST be recorded in the blueprint and in `CONTEXT.md` under `## Out of Bounds` rules.
3. Record every decision in the state file as you go (a compact `decisions` list or in the blueprint draft).
4. If a question can be answered by the existing environment (e.g. "is X installed"), check the environment instead of asking.

**Exit:** All decision branches resolved. User and agent share understanding of what will be built and how.

**State file:** Update `stage: blueprinting`.

### 3. blueprinting

**Entry:** Grilling resolved.

**Actions:**

1. Produce `blueprint.md` at the project root. See [blueprint-spec.md](blueprint-spec.md) for the required structure. It captures:
   - Technical selection + rationale (why this stack for this problem).
   - Module division (planned `src/<module>/` directories with one-line responsibility).
   - Data model draft and API draft.
   - Detected critical domains and their guardrails.
   - Success criteria and out-of-scope.
2. Derive the planned module list and write it to the state file (`modules:`).
3. Present the blueprint for confirmation.

**Exit:** User approves the blueprint.

**Confirmation point — [blueprint]:**
Present the blueprint summary in one block:

```text
**Goal:** <project> — <one line>
**Users:** <audience>
**In scope:** <bullets>
**Out of scope:** <bullets>
**Tech stack:** <language / framework / runtime / package manager> — <why>
**Planned modules:** <module — responsibility>
**Critical domains detected:** <security / payment / data / privacy / none>
**Success criteria:** <bullets>
```

Ask: "Does this blueprint match what you want? Should I scaffold the project?"
- Approve → proceed to scaffolding.
- Reject → return to grilling (increment `confirmation_rejections`).

### 4. scaffolding

**Entry:** Blueprint approved.

**Actions:**

1. Scaffold the project skeleton using the official CLI for the chosen stack where one exists. See [stack-recipes.md](stack-recipes.md) for the command table. Never guess a CLI command — if it is not in the table, fall back.
2. **Fallback:** if no official CLI exists (or it is unavailable), copy the minimal TypeScript template from `templates/minimal-ts/` and rename `package.json`'s `name` to the project name.
3. Apply the blueprint's module division: create `src/<module>/index.ts` placeholder entry files for each planned module.
4. Install dependencies and run the skeleton's test + typecheck as a **skeleton health proof**:
   - `npm install && npm test && npm run typecheck` (or the stack-appropriate equivalent from stack-recipes.md).
   - If the health proof fails → diagnose, fix the scaffold, retry. 3 consecutive failures → force pause.
5. Do **not** implement features here. The skeleton must build and test green, then stop.

**Exit:** Skeleton exists, dependencies installed, health proof green.

**State file:** Update `stage: wiring`.

### 5. wiring

**Entry:** Skeleton healthy.

**Actions:**

1. Generate the full `devloop/` structure. See [wiring.md](wiring.md) for the exact checklist and the authoritative templates (from `devloop-setup/templates/`). The result must be indistinguishable from what `devloop-setup init` would produce.
2. `architecture-map.md` and `module-index.yaml` are filled from the **blueprint's planned modules** (not detected — nothing to detect yet), with `has_spec: false`. This keeps the Brownfield L1 on-demand modeling strategy intact: the first `/develop` change that touches a module deep-dives and builds its spec.
3. `CONTEXT.md` is filled with the grilling-derived domain terms and `## Out of Bounds` rules for any detected critical domains.
4. `.gitignore` gains `devloop/.state/` and legacy `devloop/.state.yaml` if not already present.
5. Run an equivalent of `/devloop-setup check` (wiring.md verification list) to confirm the wiring is valid.
6. Present the wiring summary for confirmation.

**Exit:** Wiring verified. User approves.

**Confirmation point — [wiring]:**
Present in one block:

```text
Created:
  <file tree, concise>
Tech stack: <language / framework / runtime / package manager>
Commands: test=…, typecheck=…, lint=…, build=…
Modules indexed: <n> (has_spec: false)
CONTEXT.md: <domain terms seeded> | <critical-domain guardrails>
Warnings:
  ⚠ OpenSpec CLI not found — L2/L3 changes will pause until installed (npm i -g @fission-ai/openspec@latest)
Next step: /develop <first feature>
```

Ask: "Ready to finalize the project scaffold?"
- Approve → set `stage: done`, output the delivery report.
- Reject → fix the reported issue, re-run the verification, re-present.

### 6. done

**Actions:**

1. Output the delivery report:

```text
Project <name> scaffolded and wired for DevLoop.

Tech stack: …
Commands: …
Modules: …
File tree: …

Warnings:
  ⚠ …

Next step:
  /develop <first feature>     → drive the first requirement through the full lifecycle
  /devloop-setup check         → verify wiring any time
```

2. Set `stage: done` in the state file. Keep the file for one session (post-create review), then delete on next `develop-new` intake (same policy as `devloop` archiving).

## Interruption Recovery

If the session is interrupted at any stage:

1. Save the state file with `stage: <current>` and `last_updated`.
2. Output a handoff summary:

```text
DevLoop project creation interrupted.
Resume with: /develop-new <same requirement>
Current stage: <stage>
Blueprint: <path or not yet created>
Created so far: <what exists>
Next action on resume: <first thing to do>
```

3. On resume, `develop-new` intake reads the state file and continues from `stage` — no re-grilling, no re-scaffolding.

## Backward Transitions

| From | To | Trigger |
|------|----|---------|
| blueprinting | grilling | Blueprint rejected — re-clarify decisions |
| scaffolding | blueprinting | Skeleton reveals a blueprint gap (e.g. missing module) — update blueprint, re-confirm |
| wiring | scaffolding | Wiring verification fails on a missing artifact — regenerate and re-verify |

## Host Compatibility

`develop-new` follows the same host-agnostic rule as `devloop`: **no workflow branch is gated on a host brand** (no `if claude`). Differences are handled by runtime capability detection:

| Capability | What It Enables | Fallback When Missing |
|------------|-----------------|------------------------|
| Sub-agent / parallel workers | Parallel scaffold verification and wiring checks | Serial execution of the same steps |
| Background tasks | Non-blocking install/health checks | Synchronous pipeline with explicit stage gates |
| Auto skill/command trigger | `/develop-new` invoked automatically on intake | User invokes `/develop-new` manually |
| Workspace filesystem coordination | Shared scaffold state across agents | Lock + single-writer state updates |

A capability is used only when detected at runtime; otherwise the serial fallback runs the identical steps.
