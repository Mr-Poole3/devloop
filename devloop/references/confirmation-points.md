# Confirmation Points

What to present at each checkpoint and when to pause.

## Principle

DevLoop is highly automated, but pauses at key points to prevent catastrophic mistakes. At each confirmation point:

1. Present a clear, structured summary.
2. Wait for explicit user approval.
3. Do not proceed until approved.
4. If rejected, ask what to change and return to the appropriate stage.

## Confirmation Points by Risk Level

| Point | L0 | L1 | L2 | L2 fast track | L3 | Hotfix |
|------|----|----|----|----|-----|--------|
| Requirement understanding | — | — | ✓ | merged | ✓ | — |
| Plan and spec | — | — | ✓ | merged | ✓ | — |
| Combined plan | — | — | — | ✓ | — | — |
| Start coding | — | — | — | — | ✓ | ✓ (brief) |
| Final archive | — | ✓ (review result) | ✓ | ✓ | ✓ | ✓ (with retroactive spec) |

"L2 fast track" merges `requirement_understanding` + `plan_and_spec` into a single `combined_plan` checkpoint when single-module + no-data-model + non-breaking-API criteria are met (see [routing.md](routing.md#l2-fast-track)).

## Point 1: Requirement Understanding

**Stage:** End of grilling, before specifying.

**Present:**
```markdown
## Requirement Summary

**Goal:** <what we're building and why>
**Users:** <who benefits>
**In scope:** <what's included>
**Out of scope:** <what's explicitly excluded>
**Success criteria:** <how we know it's done>
**Key decisions made during grilling:**
1. <decision 1>
2. <decision 2>
**Remaining uncertainties:** <any unresolved items>
**Risk level:** <L2/L3> — <rationale>
```

**Ask:** "Does this match your understanding? Should I proceed to create the spec?"

**If rejected:** Return to grilling. Ask what was missed.

**If approved:** Record in the active change's state file: `confirmed: [requirement_understanding]`.

## Point 2: Plan and Spec

**Stage:** After OpenSpec artifacts generated, before implementing.

**Present:**
```markdown
## Plan Summary

**Change ID:** <openspec change id>
**Proposal:** <one paragraph summary from proposal.md>
**Specs:** <list of requirements and scenario count>
**Design:** <one paragraph summary from design.md>
  - Modules affected: <list>
  - New interfaces: <list>
  - Data changes: <summary>
  - Migration needed: <yes/no>
**Tasks:** <count> tasks in <count> groups
  - Group 1: <theme> (<n> tasks)
  - Group 2: <theme> (<n> tasks)
**Testing strategy:** <seam description, test types>
**Risks:** <identified risks>
**Rollback plan:** <how to undo if needed>
```

**Ask:** "Does this plan look right? Should I start implementing?"

**If rejected:** Update OpenSpec artifacts (`/opsx:update`). Re-present.

**If approved:** Record in the active change's state file: `confirmed: [requirement_understanding, plan_and_spec]`.

## Point 2.5: Combined Plan (L2 Fast Track Only)

**Stage:** End of `reviewing_plan`, when `fast_track: true` is set on the change.

**Present (in one combined block):**

```markdown
## Combined Plan (L2 Fast Track)

### Requirement summary
**Goal:** <what we're building and why>
**In scope:** <what's included>
**Out of scope:** <what's explicitly excluded>
**Success criteria:** <how we know it's done>
**Fast track rationale:** single module (<name>), no data model change, non-breaking API

### Plan summary
**Change ID:** <openspec change id>
**Specs:** <list of requirements and scenario count>
**Tasks:** <count> tasks in <count> groups
**Testing strategy:** <seam description, test types>
**Risks:** <identified risks>
```

**Ask:** "Does the requirement AND the plan look right? Should I start implementing? (Saying no returns to grilling to refine the requirement, then re-plans.)"

**If rejected:** Fall back to standard L2. Split into separate `requirement_understanding` and `plan_and_spec` checkpoints. Return to grilling to resolve the rejected part. Set `fast_track: false` in the state file only if the rejection reveals a fast-track criterion no longer holds (second module touched, data model change, breaking API). Otherwise keep `fast_track: true` and re-present `combined_plan` after refining.

**If approved:** Record in the change's state file: `confirmed: [combined_plan]`. The `combined_plan` confirmation satisfies both `requirement_understanding` and `plan_and_spec` for downstream stages.

## Point 3: Start Coding (L3 or Hotfix)

**Stage:** Just before first code modification in implementing.

**Present:**
```markdown
## Ready to Code

**Change:** <change id>
**First task:** <task description>
**Files to be modified:**
  - <file 1> — <what changes>
  - <file 2> — <what changes>
**New files:**
  - <file 1> — <purpose>
**Test files:**
  - <test file> — <what it tests>
```

**Ask:** "Ready to start coding?"

**If rejected:** Return to reviewing_plan. Ask what needs to change.

**If approved:** Record in the active change's state file: `confirmed: [..., start_coding]`.

## Point 4: Final Archive

**Stage:** After verification, before sync and archive.

**Present:**
```markdown
## Delivery Summary

**Change:** <change id>
**Risk level:** <L1/L2/L3>
**What was built:** <summary>
**Tasks completed:** <n>/<n>
**Tests:** <n> added, <n> modified — all passing
**Typecheck:** ✓ passing
**Lint:** ✓ passing
**OpenSpec verify:** ✓ no criticals (<n> warnings)
**Code review:** ✓ no major issues (<n> minor notes)
**Spec-first deviations:** <list of code changes made without spec update>
**Warnings:** <list>
**Remaining items:** <list>
**Files changed:** <n> files (<n> added, <n> modified, <n> deleted)
```

**Ask:** "Ready to sync specs and archive this change?"

**If rejected:** Return to implementing or verifying. Ask what needs fixing.

**If approved:** Execute `/opsx:sync`, `/opsx:archive`, set `stage: done` in the active change's state file.

## L1 Completion

L1 does not have full confirmation points, but should show:

```markdown
## Fix Complete

**Problem:** <what was wrong>
**Root cause:** <diagnosis>
**Fix:** <what was changed>
**Test:** <test added/modified>
**Code review:** <result>
**Spec gap noted:** <if the bug revealed missing spec>
```

**Ask:** "Does this look good?"

## Rules

- Never skip a confirmation point.
- Never auto-approve on behalf of the user.
- If the user says "just do everything", still pause at requirement understanding and final archive.
- Present summaries in the templates above. Do not improvise format.
- Keep summaries concise — link to full artifacts, don't reproduce them.
