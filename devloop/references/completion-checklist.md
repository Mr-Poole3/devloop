# Completion Checklist

Three-layer verification standard for DevLoop delivery.

## Overview

```text
Code Layer → Spec Layer → Delivery Layer
```

A change is complete only when all three layers pass.

## Layer 1: Code Layer

All must pass. No exceptions.

### Tests

- [ ] Test command runs without errors: `<test command from config.yaml>`
- [ ] All existing tests still pass
- [ ] New tests added for new behavior
- [ ] Edge cases from specs are covered by tests
- [ ] No tests were disabled or deleted to make things pass

### Typecheck

- [ ] Typecheck command passes: `<typecheck command from config.yaml>`
- [ ] No new `any` types or type assertions introduced (unless justified)

### Lint

- [ ] Lint command passes: `<lint command from config.yaml>`
- [ ] No new linting rules disabled

### Build (if applicable)

- [ ] Build command succeeds: `<build command from config.yaml>`

**Failure handling:** Any code layer failure blocks progress. Fix before proceeding to spec layer.

---

## Layer 2: Spec Layer

Uses OpenSpec's `/opsx:verify` command.

### Completeness

- [ ] All tasks in `tasks.md` are checked off
- [ ] All requirements in `specs/` have corresponding code implementation
- [ ] All scenarios are covered by tests or manual verification
- [ ] No "TODO" or "FIXME" left in spec artifacts

### Correctness

- [ ] Implementation matches spec intent
- [ ] Edge cases from scenarios are handled in code
- [ ] Error states match spec definitions
- [ ] No behavior was added that isn't in the spec (scope creep)

### Coherence

- [ ] Design decisions in `design.md` are reflected in code structure
- [ ] Naming conventions are consistent with design
- [ ] Module boundaries match design
- [ ] No contradictions between proposal, specs, design, and tasks

**Result classification:**
- **Critical issues:** Must be resolved before archiving.
- **Warnings:** Allowed but must be listed in delivery summary.

**Failure handling:** Critical issues block archiving. Return to implementing or specifying.

---

## Layer 3: Delivery Layer

### Code Review

Execute `code-review` skill (or manual review if unavailable):

**Standards axis:**
- [ ] Code follows repo's coding standards
- [ ] No Fowler code smells introduced
- [ ] Naming is clear and consistent
- [ ] No dead code or unused imports
- [ ] Error handling follows project patterns

**Spec axis:**
- [ ] Implementation faithfully follows the originating spec/PRD
- [ ] No undocumented behavior changes
- [ ] All user stories from spec are addressed

**Result:** No major issues. Minor notes allowed and listed in delivery summary.

### Task Completion

- [ ] All tasks in `tasks.md` are marked complete
- [ ] No tasks were silently skipped
- [ ] Tasks that were removed are documented in delivery summary

### Delivery Summary Generated

- [ ] Delivery summary written to `devloop/reports/<change-id>-delivery.md`
- [ ] Summary includes: what was built, tests, verification, review, warnings, remaining items

### Spec-First Compliance

- [ ] All behavior changes were reflected in OpenSpec before code changes
- [ ] Any deviations are documented in delivery summary
- [ ] No spec drift remaining

---

## Archive Flow

Once all three layers pass:

```text
1. Generate delivery summary → devloop/reports/<change-id>-delivery.md
2. [CONFIRMATION POINT: final_archive]
3. Execute /opsx:sync
   - Merges delta specs into devloop/openspec/specs/
4. Execute /opsx:archive
   - Moves change to devloop/openspec/changes/archive/
5. Update devloop/context/module-index.yaml
   - Add new modules
   - Update key_interfaces for changed modules
   - Update last_scanned date
6. Update devloop/context/architecture-map.md if structure changed
7. Update devloop/context/CONTEXT.md if new domain terms emerged
8. Clear .state.yaml (set stage: done)
9. Output delivery report to user
```

## Delivery Summary Template

```markdown
# Delivery: <change-id>

**Date:** <YYYY-MM-DD>
**Risk level:** <L0/L1/L2/L3>
**Duration:** <sessions or stages>

## What was built

<2-3 paragraph summary from the user's perspective>

## Tasks

- Total: <n>
- Completed: <n>
- Skipped: <n> (with reasons)

## Tests

- Added: <n>
- Modified: <n>
- All passing: ✓

## Verification results

| Check | Result |
|-------|--------|
| Unit tests | ✓ |
| Typecheck | ✓ |
| Lint | ✓ |
| Build | ✓ |
| OpenSpec verify | ✓ (0 criticals, <n> warnings) |
| Code review | ✓ (0 major, <n> minor) |

## Warnings

1. <warning 1>
2. <warning 2>

## Spec-first deviations

<list of code changes made without prior spec update, or "None">

## Remaining items

1. <item 1>
2. <item 2>

## Files changed

- Added: <n>
- Modified: <n>
- Deleted: <n>

## Spec gap notes (L1 only)

<if this was an L1 fix that revealed missing specs, note them here for future L2 upgrade>
```

## Edge Cases

### Tests pass but verify fails
Priority: verify result. Tests passing doesn't mean the spec is satisfied. Fix spec gaps.

### Verify passes but code review fails
Priority: code review. Spec compliance doesn't guarantee code quality. Fix standards issues.

### Everything passes but user is unhappy
Priority: user. The user's satisfaction is the ultimate gate. Return to grilling if the requirement was misunderstood.

### Warnings from verify
Allowed to archive. Must be listed in delivery summary under "Warnings" and "Remaining items".

### No tests exist in the project
Code layer can only check typecheck and lint. Note this as a warning. Recommend setting up tests via `/devloop-setup reconfigure`.
