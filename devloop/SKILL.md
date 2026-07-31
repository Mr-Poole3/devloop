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

1. **intake** — Read `.state.yaml` for pending work. Read `devloop/config.yaml` and `devloop/context/` for project context.
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
- **L1**: `diagnosing-bugs` → `tdd` → `code-review`. No OpenSpec required. Create `.state.yaml` but skip OpenSpec stages.

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
- User interrupt → save `.state.yaml`, output handoff summary.
- See [recovery.md](references/recovery.md) for full rules.

## State Management

DevLoop reads and writes `devloop/.state.yaml` at every stage transition. This enables cross-session recovery.

```yaml
current_change: add-dark-mode
stage: implementing
risk_level: L2
confirmed: [requirement_understanding, plan_and_spec]
pending_confirmation: []
tasks: { total: 12, completed: 7, current: "2.3 Update theme toggle component" }
consecutive_failures: 0
last_updated: "2026-08-01T10:30:00Z"
```

## Authority Hierarchy

```text
CONTEXT.md / ADR          → Long-term project facts and domain language
devloop/openspec/changes/ → Current change's sole execution truth
devloop/openspec/specs/   → Archived formal behavior specs
Issue Tracker             → Team coordination, ownership, scheduling
Code & Tests              → The actual implementation
```

## Host Compatibility

DevLoop works across Claude Code, Codex, OpenCode, TRAE, and Cursor. If the host supports sub-agents, `code-review` runs in parallel. Otherwise, it runs serially.

## References

- [Lifecycle](references/lifecycle.md) — full state machine with entry/exit conditions
- [Routing](references/routing.md) — risk classification and workflow selection
- [Confirmation Points](references/confirmation-points.md) — what to present and when to pause
- [Recovery](references/recovery.md) — failure, interruption, and mid-flight change handling
- [Completion Checklist](references/completion-checklist.md) — three-layer verification standard
