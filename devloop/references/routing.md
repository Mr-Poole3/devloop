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
- New API endpoint that doesn't break existing clients (**non-breaking API change**).
- Performance optimization with observable behavior change.

**Must NOT have any L3 escalation factors.**

**API change boundary (disambiguation):**
- **Non-breaking** (stays L2): adds an optional field/param, adds a new endpoint, adds a new response field, widens an accepted input range. Existing clients keep working without modification.
- **Breaking** (forces L3): removes or renames a field/endpoint, makes an optional field required, narrows an accepted input range, changes a response field's type/semantics, changes status code semantics.

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

### L2 Fast Track

When ALL of the following are true, the L2 change qualifies for **fast track** (set `fast_track: true` in the change's state file):

- **Single module only** — changes confined to one module (verify against `module-index.yaml`).
- **No data model change** — no schema, migration, entity, or persistence-layer changes.
- **Non-breaking API only** — see disambiguation above.

Fast track **merges** the first two confirmation points into a single **combined_plan** checkpoint at the end of `reviewing_plan`. The user reviews the requirement summary and the OpenSpec plan in one pause, then approves both together. The `final_archive` confirmation point is unchanged.

```text
Standard L2:  [requirement_understanding] → [plan_and_spec] → ... → [final_archive]
Fast track:   [combined_plan]                                  → ... → [final_archive]
```

**If fast track is active but the user rejects `combined_plan`, fall back to standard L2**: split into separate `requirement_understanding` and `plan_and_spec` checkpoints, return to grilling to resolve the rejected part.

**Fast track is forfeited** (revert to standard L2) if at any point during implementing:
- A second module becomes affected.
- A data model change is discovered.
- A breaking API change is introduced.

Record the forfeiture in the delivery summary under "Spec-first deviations".

---

## L3 — Cross-Module, High-Risk, or Architectural Change

**Escalation factors (ANY forces at least L2, potentially L3):**
- Data model changes
- **Breaking** public API changes (non-breaking API changes stay L2 — see disambiguation above)
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
- **Breaking API change** (removes/renames fields or endpoints, makes optional required, narrows input, changes response type/semantics, changes status code semantics).
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

### Hotfix Bypass

For urgent production fixes where the full L2/L3 ceremony would block recovery. Hotfix bypass is opt-in — it is NEVER auto-applied.

**Triggers (any one, with user confirmation):**
- User invokes `/develop --hotfix <requirement>` explicitly.
- User's request contains urgency keywords: "production down", "P0", "critical bug", "hotfix", "outage", "sev1".
- User confirms hotfix mode when DevLoop asks ("This looks urgent — treat as hotfix? Skip grilling/specifying, fix now, backfill spec after?").

**Workflow:**
```text
1. intake
2. triaging → hotfix (set hotfix: true, risk_level: L1-equivalent)
3. SKIP exploring, grilling, specifying, reviewing_plan
4. implementing (TDD still required — no skipping tests)
     - Brief start confirmation: "Fixing <symptom> by <approach>. OK?"
     - Run critical-domain detection (still required — hotfixes often touch auth/payment/data)
5. verifying (full test suite, typecheck, lint — no skipping)
6. archiving:
     a. **MANDATORY retroactive spec**: create OpenSpec change describing what was fixed
        - If the fix is too small to warrant a full change, write a minimal spec in devloop/openspec/specs/<module>/hotfix-<date>.md
     b. If retroactive spec creation fails or user refuses: BLOCK archive, escalate
     c. [CONFIRMATION POINT: final_archive with retroactive spec attached]
7. done — delivery summary MUST note hotfix status and any pending follow-up
```

**Hard rules:**
- Tests are NEVER skipped, even in hotfix. If the project has no test for the broken path, write one as part of the fix (this IS the regression test).
- `consecutive_failures` still applies. 3 failures = pause.
- Critical-domain detection still runs. A hotfix touching auth/payment/data without dedicated tests is blocked.
- The retroactive spec is a hard gate for archiving. No exceptions. If the user wants to archive without a spec, they must explicitly override and the deviation is recorded in `devloop/reports/` and surfaced in the next non-hotfix change's intake.

**Hotfix de-escalation:** If during implementing the agent discovers the issue is not actually urgent (e.g., the "production down" was a false alarm), pause and ask the user whether to continue as hotfix or convert to a normal L1/L2 change. Converting means creating proper OpenSpec artifacts and going through standard confirmation points.

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
