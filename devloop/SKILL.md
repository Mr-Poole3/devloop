---
name: devloop
description: Drive a software requirement from initial idea through clarification, spec-driven planning, implementation, testing, verification, and archival in a single automated loop. Use when the user provides a new feature, bug fix, refactor, or architectural change and wants the agent to drive the work end to end with minimal manual orchestration.
---

# DevLoop

The integrated development loop. One entry point, full lifecycle.

## Quick Start

```text
User: /develop I want to add dark mode support
AI: [reads project context]
    [classifies risk: L2]
    [starts grilling...]
```

## Prerequisites

- `devloop/config.yaml` must exist. If not, run `/devloop-setup` first.
- OpenSpec CLI recommended. If missing, DevLoop will pause and report.

## Entry Point

The user provides a requirement (either as an argument or in conversation). DevLoop takes over from there.

## Lifecycle

DevLoop follows a state machine. Each stage has entry conditions, actions, and exit conditions. See [lifecycle.md](references/lifecycle.md).

```text
intake → triaging → exploring → grilling → specifying
  → reviewing_plan → implementing → verifying → archiving → done
```

### Stage Summary

1. **intake** — Scan `devloop/.state/*.yaml` for in-progress changes. If any exist, list them and ask: continue one / start new. Read `devloop/config.yaml` and `devloop/context/` for project context.
2. **triaging** — Classify risk level (L0-L3). See [routing.md](references/routing.md).
3. **exploring** — Read architecture map. Check `module-index.yaml` for `has_spec`. If false, build module spec (L1 on-demand modeling). Deep-dive into affected modules.
4. **grilling** — L2: use `grilling` skill. L3: use `grill-with-docs` skill. Clarify until all decision branches resolved. Update `CONTEXT.md` and ADRs if L3.
5. **specifying** — Create OpenSpec change. L2: `/opsx:propose`. L3: `/opsx:propose` or step-by-step. Generate proposal, specs, design, tasks.
6. **reviewing_plan** — **[CONFIRMATION POINT]** Present requirement summary and plan. See [confirmation-points.md](references/confirmation-points.md).
7. **implementing** — **[CONFIRMATION POINT for L3]** Execute tasks. Use `tdd` skill. Spec-first principle: any behavior change updates OpenSpec first. See [recovery.md](references/recovery.md).
8. **verifying** — Run tests, typecheck, lint. Execute `/opsx:verify`. Execute `code-review`. See [completion-checklist.md](references/completion-checklist.md).
9. **archiving** — **[CONFIRMATION POINT]** Generate delivery summary. `/opsx:sync`. `/opsx:archive`. Update module index. Clear `.state.yaml`.
10. **done** — Output delivery report.

## L0 / L1 Shortcuts

- **L0**: Handle directly. No state file, no OpenSpec.
- **L1**: `diagnosing-bugs` → `tdd` → `code-review`. No OpenSpec required. Create a state file but skip OpenSpec stages.
- **Hotfix**: Urgent production fixes. Skip grilling/specifying/reviewing_plan, go straight to implementing with TDD. **Retroactive OpenSpec spec is a hard gate for archiving.** Triggered by `/develop --hotfix` or urgency keywords. See [routing.md](references/routing.md#hotfix-bypass).

## Spec-First Principle

```text
Does this change affect external behavior or architecture?
  YES → Update OpenSpec first, then code
  NO  → Change code directly, note in delivery summary
```

## Failure Recovery

- Test failure → `diagnosing-bugs` skill, no blind retries.
- 3 consecutive failures → pause, output diagnostic report.
- Design mismatch → pause implementing, return to `specifying`, update OpenSpec.
- User interrupt → save the active change's state file under `devloop/.state/`, output handoff summary.
- See [recovery.md](references/recovery.md) for full rules.

## State Management

DevLoop supports **multiple parallel changes**. Each change has its own state file under `devloop/.state/<change-id>.yaml`. DevLoop reads and writes the active change's state file at every stage transition. This enables cross-session recovery and concurrent work on unrelated changes.

```yaml
# devloop/.state/<change-id>.yaml
schema_version: 1
change_id: add-dark-mode
stage: implementing
risk_level: L2
fast_track: false        # L2 fast track active (merged confirmation points)
hotfix: false            # hotfix bypass active (retroactive spec required)
confirmed: [requirement_understanding, plan_and_spec]
pending_confirmation: []
tasks:
  total: 12
  completed: 7
  current: "2.3 Update theme toggle component"
consecutive_failures: 0
metrics:
  stage_durations: { intake: 2s, triaging: 5s, grilling: 120s }
  failure_counts: { test: 1, typecheck: 0, lint: 0, verify: 0 }
  confirmation_rejections: 0
  spec_drifts: 0
created_at: "2026-08-01T10:00:00Z"
last_updated: "2026-08-01T10:30:00Z"
```

### Schema Versioning & Migration

- `schema_version` is required. Current version: `1`.
- Pre-existing `devloop/.state.yaml` files (no `schema_version`) are treated as `schema_version: 0`.
- On first read of a v0 file, DevLoop migrates it: creates `devloop/.state/<current_change>.yaml` with `schema_version: 1` and the fields above, then deletes the old `devloop/.state.yaml`.
- Future schema bumps MUST ship with a migration rule in `references/state-migration.md`.
- If a state file's `schema_version` is newer than DevLoop understands, refuse to proceed and report.

## Authority Hierarchy

```text
CONTEXT.md / ADR          → Long-term project facts and domain language
devloop/openspec/changes/ → Current change's sole execution truth
devloop/openspec/specs/   → Archived formal behavior specs
Issue Tracker             → Team coordination, ownership, scheduling
Code & Tests              → The actual implementation
```

## Host Compatibility

DevLoop must work identically across supported AI coding agents. Do not implement workflow branches based on host names (e.g., `if claude`).

**Allowed runtime differences:**
- Capability detection → selects parallel or serial execution for review/verify stages.
- Trigger model → auto skill invocation when supported, manual invocation otherwise.
- Task concurrency → use host-supported concurrency primitives; degrade to serial if unsupported.

**Compatibility rules:**
- Each stage’s entry/exit contract remains constant across hosts.
- State file format and transitions remain constant across hosts.
- Verification standards remain constant across hosts.
- Parallel review is an optimization, not a correctness requirement.

## References

- [Lifecycle](references/lifecycle.md) — full state machine with entry/exit conditions
- [Routing](references/routing.md) — risk classification and workflow selection
- [Confirmation Points](references/confirmation-points.md) — what to present and when to pause
- [Recovery](references/recovery.md) — failure, interruption, and mid-flight change handling
- [Completion Checklist](references/completion-checklist.md) — three-layer verification standard
- [State Migration](references/state-migration.md) — `.state/` schema versioning and upgrade rules
