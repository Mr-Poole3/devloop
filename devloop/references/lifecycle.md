# Lifecycle

Full state machine for the DevLoop workflow.

## State Diagram

```text
                    ┌──────────┐
                    │  intake  │
                    └────┬─────┘
                         ▼
                   ┌───────────┐
                   │ triaging  │
                   └────┬──────┘
                        │
              ┌─────────┼─────────┐
              │         │         │
              ▼         ▼         ▼
             L0        L1      L2/L3
              │         │         │
              ▼         ▼         ▼
           done    diagnose   exploring
                      │           │
                      ▼           ▼
                     tdd       grilling
                      │           │
                      ▼           ▼
                  code-review  specifying
                      │           │
                      ▼           ▼
                    done    reviewing_plan
                                  │
                                  ▼
                            implementing ◄──────┐
                                  │              │
                                  │ spec drift?  │
                                  └──────────────┘
                                  │
                                  ▼
                              verifying
                                  │
                                  ▼
                              archiving
                                  │
                                  ▼
                                done
```

## Stage Details

### 1. intake

**Entry:** User provides a requirement via `/develop <requirement>` or natural language.

**Actions:**
1. Read `devloop/.state.yaml` if it exists.
2. If a previous change is in progress, report status and ask: continue / abandon / start new.
3. Read `devloop/config.yaml` for project settings.
4. Read `devloop/context/CONTEXT.md` for domain language.
5. Read `devloop/context/architecture-map.md` for module overview.

**Exit:** Project context loaded. Ready to classify.

**State file:** Create or update `.state.yaml` with `stage: triaging`.

---

### 2. triaging

**Entry:** Project context loaded.

**Actions:**
1. Analyze the requirement against routing rules. See [routing.md](routing.md).
2. Check escalation factors:
   - Data model changes?
   - Public API changes?
   - Permission, payment, privacy, user data?
   - Data migration?
   - Cross-service or cross-module?
   - Backward compatibility?
   - Rollback requirements?
   - High technical uncertainty?
   - Multi-person collaboration?
3. Assign risk level: L0, L1, L2, or L3.
4. Report classification to user with rationale.

**Exit:** Risk level assigned. `stage` updated in `.state.yaml`.

**L0 shortcut:** Handle directly, set `stage: done`, no further lifecycle stages.
**L1 shortcut:** Skip to implementing-equivalent flow (diagnose → tdd → review).
**L2/L3:** Continue to exploring.

---

### 3. exploring

**Entry:** Risk level is L2 or L3.

**Actions:**
1. Read `devloop/context/module-index.yaml`.
2. Identify which modules the requirement touches.
3. For each touched module, check `has_spec`:
   - `false` → Deep-dive into the module. Read source files, interfaces, tests. Generate a behavior spec in `devloop/openspec/specs/<module>/`. Update `has_spec: true` in module index.
   - `true` → Read existing spec.
4. For L3: Consider running `/opsx:explore` for open-ended investigation.
5. Record findings in `devloop/research/`.

**Exit:** Affected modules understood. Specs loaded or created.

**State file:** Update `stage: grilling`.

---

### 4. grilling

**Entry:** Modules understood.

**Actions:**
- **L2**: Use the `grilling` skill. Ask questions one at a time. Provide recommended answers. Resolve every branch of the decision tree.
- **L3**: Use `grill-with-docs` skill. Same grilling, but also:
  - Update `devloop/context/CONTEXT.md` with new domain terms.
  - Create ADRs in `devloop/decisions/` for significant architecture decisions.

**Grilling covers:**
- What problem are we solving?
- Who is affected?
- What is in scope? Out of scope?
- What are the success criteria?
- What are the failure modes?
- What are the edge cases?
- Are there migration, compatibility, or rollback concerns?
- What alternatives were considered and rejected?

**Exit:** All decision branches resolved. User and agent share understanding.

**State file:** Update `stage: specifying`.

---

### 5. specifying

**Entry:** Requirements clarified.

**Actions:**
1. Create OpenSpec change:
   ```bash
   /opsx:propose <change-name>
   ```
2. OpenSpec generates `proposal.md`, `specs/`, `design.md`, `tasks.md` in `devloop/openspec/changes/<change-id>/`.
3. Review generated artifacts for completeness:
   - Do requirements cover success and failure paths?
   - Are scenarios in Given/When/Then format?
   - Does design reference existing module boundaries?
   - Are tasks vertical slices (not horizontal layers)?
4. If artifacts are incomplete, use `/opsx:update` to refine.
5. For L3 with multiple deliverable units: use `to-tickets` to break into independent tickets. Each ticket may get its own OpenSpec change.
6. For L3 needing standalone PRD: use `to-spec` before OpenSpec.

**Exit:** OpenSpec artifacts complete and internally consistent.

**State file:** Update `stage: reviewing_plan`.

---

### 6. reviewing_plan

**Entry:** OpenSpec artifacts generated.

**Actions:**
1. Generate requirement summary from grilling results. See [confirmation-points.md](confirmation-points.md).
2. Generate plan summary from OpenSpec artifacts.
3. Present both to user.
4. **[CONFIRMATION POINT 1: requirement_understanding]**
5. **[CONFIRMATION POINT 2: plan_and_spec]**

**Exit:** User confirms plan. `confirmed` array updated in `.state.yaml`.

**State file:** Update `stage: implementing`.

---

### 7. implementing

**Entry:** Plan confirmed.

**Actions:**
1. **[L3 ONLY — CONFIRMATION POINT: start_coding]**
2. Read `tasks.md` from current OpenSpec change.
3. For each task:
   a. Use `tdd` skill: write failing test → implement → green → refactor.
   b. After completing each task, check it off in `tasks.md`.
   c. Update `.state.yaml` with current task.
4. **Spec-first principle:** If implementation reveals a spec gap:
   - Pause coding.
   - Update OpenSpec (`/opsx:update` or manual edit of specs/design/tasks).
   - Resume coding.
5. If test fails, enter `diagnosing-bugs` flow.
6. Track consecutive failures. If 3 in a row, pause and report.

**Exit:** All tasks in `tasks.md` completed.

**State file:** Update `stage: verifying`.

---

### 8. verifying

**Entry:** All tasks completed.

**Actions:**
1. Run test command from `config.yaml`.
2. Run typecheck command.
3. Run lint command.
4. Execute `/opsx:verify` — checks completeness, correctness, coherence.
5. Execute `code-review` skill — checks standards and spec compliance.
   - If host supports sub-agents: run both axes in parallel.
   - If not: run serially.

**Exit:** All checks pass (warnings allowed, criticals must be resolved).

**State file:** Update `stage: archiving`.

---

### 9. archiving

**Entry:** Verification passed.

**Actions:**
1. Generate delivery summary to `devloop/reports/<change-id>-delivery.md`. See [completion-checklist.md](completion-checklist.md).
2. **[CONFIRMATION POINT: final_archive]**
3. Execute `/opsx:sync` — merge delta specs into `devloop/openspec/specs/`.
4. Execute `/opsx:archive` — move change to `devloop/openspec/changes/archive/`.
5. Update `devloop/context/module-index.yaml`:
   - If new modules were added, add entries.
   - If module interfaces changed, update `key_interfaces`.
   - Update `last_scanned` date.
6. Update `devloop/context/architecture-map.md` if structure changed.
7. Clear `.state.yaml` or set `stage: done`.

**Exit:** Change archived. Specs synced. Context updated.

---

### 10. done

**Actions:**
1. Output delivery report:
   - What was built.
   - Tests added/modified.
   - Verification results.
   - Code review results.
   - Warnings and remaining items.
   - Files changed summary.
2. Suggest next steps if applicable.

## Backward Transitions

| From | To | Trigger |
|------|----|---------|
| implementing | specifying | Spec gap discovered, design mismatch |
| implementing | exploring | Codebase structure doesn't match expectations |
| verifying | implementing | Verification found code issues |
| verifying | specifying | Verification found spec issues |
| any | done | User abandons change |
