# Routing

Risk classification and workflow selection for DevLoop.

## Classification Process

```text
Receive requirement
  ↓
Check L0 criteria → match? → L0
  ↓ no
Check L1 criteria (ALL true) AND no escalation factors → match? → L1
  ↓ no
Check L2 criteria AND no L3 escalation → match? → L2
  ↓ no
L3
```

## L0 — No Code Behavior Change

**Trigger:** The requirement does not change any runtime behavior.

**Examples:**
- Documentation updates
- Comment improvements
- Configuration typo fixes
- README edits
- Formatting changes

**Workflow:**
```text
Direct handling — no state file, no OpenSpec, no grilling.
```

**Confirmation:** None.

---

## L1 — Single-Module Low-Risk Fix

**Criteria (ALL must be true):**
- Changes confined to a single module.
- No data model changes.
- No public API changes.
- No cross-service impact.
- No permission, payment, or user data involved.
- No migration needed.
- Low technical uncertainty.

**Workflow:**
```text
1. Create .state.yaml (stage: implementing)
2. diagnosing-bugs (if bug fix) or direct implementation (if small feature)
3. tdd (red-green-refactor)
4. code-review
5. Clear .state.yaml
```

**Confirmation points:**
1. Brief start confirmation (what will be changed)
2. Complete (code-review result)

**Test coverage:**
- At least 1 regression test covering the fixed bug scenario.
- If the fix touches security, data, payment, or privacy domains, dedicated tests required.

**OpenSpec:** Not required. If the fix reveals a missing spec, note in delivery summary.

---

## L2 — Medium Feature or Refactor

**Criteria (ANY triggers L2):**
- New feature within an existing module boundary.
- Refactor that changes internal structure but not public API.
- Changes to data model within a single service.
- New API endpoint that doesn't break existing clients.
- Performance optimization with observable behavior change.

**Must NOT have any L3 escalation factors.**

**Workflow:**
```text
1. intake
2. triaging → L2
3. exploring (check module specs, build if missing)
4. grilling (using grilling skill)
5. specifying (/opsx:propose)
6. reviewing_plan [CONFIRM: requirement_understanding + plan_and_spec]
7. implementing (tdd, spec-first)
8. verifying (tests + /opsx:verify + code-review)
9. archiving [CONFIRM: final_archive] → /opsx:sync → /opsx:archive
10. done
```

**Test coverage:**
- All OpenSpec specs scenarios must have corresponding test coverage.
- Security, data, payment, and privacy domains must have dedicated tests.
- Tests verify external behavior, not internal implementation details.

**Confirmation points:**
1. Requirement understanding
2. Plan and spec
3. Final archive

---

## L3 — Cross-Module, High-Risk, or Architectural Change

**Escalation factors (ANY forces at least L2, potentially L3):**
- Data model changes
- Public API changes (breaking or non-breaking)
- Permission, payment, privacy, or user data
- Data migration
- Cross-service or cross-module impact
- Backward compatibility requirements
- Rollback requirements
- High technical uncertainty
- Multi-person or multi-agent collaboration

**L3-specific triggers:**
- Changes span multiple modules or services.
- Architectural change (new pattern, new infrastructure, new data store).
- Breaking API change.
- Security-sensitive (auth, permissions, payment, PII).
- Data migration required.
- Multi-person collaboration needed.
- Long-lived spec that must be maintained.

**Workflow:**
```text
1. intake
2. triaging → L3
3. exploring (/opsx:explore if needed, build module specs)
4. grilling (grill-with-docs — also updates CONTEXT.md and ADRs)
5. specifying:
   a. Optional: to-spec (if standalone PRD needed)
   b. Optional: to-tickets (if multiple deliverable units)
   c. /opsx:propose (create OpenSpec change)
6. reviewing_plan [CONFIRM: requirement_understanding + plan_and_spec]
7. implementing [CONFIRM: start_coding] (tdd, spec-first)
8. verifying (tests + /opsx:verify + code-review with design review)
9. archiving [CONFIRM: final_archive] → /opsx:sync → /opsx:archive
10. done
```

**Confirmation points:**
1. Requirement understanding
2. Plan and spec
3. Start coding (explicit pause before first code modification)
4. Final archive

**Test coverage:**
- All OpenSpec specs scenarios must have corresponding test coverage.
- Critical cross-module paths must have integration tests.
- Security, data, payment, and privacy domains must have dedicated tests.
- Tests verify external behavior, not internal implementation details.

**Additional:**
- `to-tickets` is used when the change can be decomposed into multiple independently deliverable units.
- Each ticket may map to its own OpenSpec change.
- `to-spec` is used when a standalone PRD is needed for stakeholder communication.

## Edge Cases

### Unclear Risk
When in doubt, escalate to the higher level. It's safer to over-spec than under-spec.

### User Override
User can manually set a level. Skill respects but notes concerns:
```text
"User specified L1, but this change touches the data model.
Proceeding as L1 per user request, but recommend L2."
```

### Mid-Flight Escalation
If an L1 fix reveals deeper issues:
1. Pause current work.
2. Re-classify.
3. If escalating to L2/L3, create OpenSpec change retroactively.
4. Resume at the appropriate stage.

### Mid-Flight De-escalation
If an L3 task turns out simpler:
1. Note in delivery summary.
2. Keep current workflow to avoid losing artifacts.
3. Do not tear down existing OpenSpec changes.
