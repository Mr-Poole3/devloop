# Routing Policy

Risk classification rules for the DevLoop workflow.

## Risk Levels

### L0 — No Code Behavior Change

**Examples:**
- Documentation updates
- Comment improvements
- Configuration typo fixes
- README edits

**Workflow:**
```text
Direct handling — no OpenSpec, no grilling, no state file.
```

**Confirmation points:** None.

---

### L1 — Single-Module Low-Risk Fix

**Criteria (ALL must be true):**
- Changes confined to a single module.
- No data model changes.
- No public API changes.
- No cross-service impact.
- No permission, payment, or user data involved.
- No migration needed.
- Low technical uncertainty.

**Examples:**
- Bug fix in a single component.
- UI adjustment in a single page.
- Refactor within one module (no interface change).

**Workflow:**
```text
diagnosing-bugs → tdd → code-review
```

**Confirmation points:**
1. Start fix (brief confirmation of what will be changed)
2. Complete (code-review result)

**OpenSpec:** Not required. If the bug reveals a missing spec, note it in the delivery summary for future L1→L2 upgrade.

---

### L2 — Medium Feature or Refactor

**Criteria (ANY triggers L2):**
- New feature within an existing module boundary.
- Refactor that changes internal structure but not public API.
- Changes to data model within a single service.
- New API endpoint that doesn't break existing clients.
- Performance optimization with observable behavior change.

**Risk escalation factors (ANY forces L3):**
- Data model changes
- Public API changes
- Permission, payment, privacy, or user data
- Data migration
- Cross-service or cross-module
- Backward compatibility requirements
- Rollback requirements
- High technical uncertainty
- Multi-person or multi-agent collaboration

**Workflow:**
```text
grill → OpenSpec propose → apply → verify → archive
```

**Confirmation points:**
1. Requirement understanding
2. Plan and spec
3. Final archive

**OpenSpec:** Required. Full change lifecycle.

---

### L3 — Cross-Module, High-Risk, or Architectural Change

**Criteria (ANY triggers L3):**
- Changes span multiple modules or services.
- Architectural change (new pattern, new infrastructure, new data store).
- Breaking API change.
- Security-sensitive (auth, permissions, payment, PII).
- Data migration required.
- Multi-person collaboration needed.
- Long-lived spec that must be maintained over time.

**Workflow:**
```text
grill-with-docs → explore → to-spec / to-tickets → OpenSpec → apply → verify → review → archive
```

**Confirmation points:**
1. Requirement understanding
2. Plan and spec
3. Start coding (explicit confirmation before first code modification)
4. Final archive

**OpenSpec:** Required. Full change lifecycle with design review.

**Additional:**
- `to-tickets` is used when the change can be decomposed into multiple independently deliverable units.
- Each ticket may map to its own OpenSpec change.
- `to-spec` is used when a standalone PRD is needed for stakeholder communication.

## Classification Process

```text
Receive requirement
  ↓
Check L0 criteria → match? → L0
  ↓ no
Check L1 criteria (ALL true) AND no L2/L3 escalation → match? → L1
  ↓ no
Check L2 criteria AND no L3 escalation → match? → L2
  ↓ no
L3
```

## Edge Cases

- **Unclear risk**: When in doubt, escalate to the higher level.
- **User override**: User can manually set a level. Skill should respect but note concerns.
- **Mid-flight escalation**: If an L1 fix reveals deeper issues, pause and re-classify.
- **Mid-flight de-escalation**: If an L3 task turns out simpler than expected, note in report but keep the current workflow to avoid losing artifacts.
