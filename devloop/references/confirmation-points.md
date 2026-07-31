# Confirmation Points

What to present at each checkpoint and when to pause.

## Principle

DevLoop is highly automated, but pauses at key points to prevent catastrophic mistakes. At each confirmation point:

1. Present a clear, structured summary.
2. Wait for explicit user approval.
3. Do not proceed until approved.
4. If rejected, ask what to change and return to the appropriate stage.

## Confirmation Points by Risk Level

| Point | L0 | L1 | L2 | L3 |
|------|----|----|----|-----|
| Requirement understanding | — | — | ✓ | ✓ |
| Plan and spec | — | — | ✓ | ✓ |
| Start coding | — | — | — | ✓ |
| Final archive | — | ✓ (review result) | ✓ | ✓ |

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

**If approved:** Record in `.state.yaml`: `confirmed: [requirement_understanding]`.

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

**If approved:** Record in `.state.yaml`: `confirmed: [requirement_understanding, plan_and_spec]`.

## Point 3: Start Coding (L3 Only)

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

**If approved:** Record in `.state.yaml`: `confirmed: [..., start_coding]`.

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

**If approved:** Execute `/opsx:sync`, `/opsx:archive`, clear `.state.yaml`.

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
