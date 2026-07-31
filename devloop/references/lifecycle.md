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
         ┌─────────┬────┴────────┬──────────┐
         │         │             │          │
         ▼         ▼             ▼          ▼
        L0        L1          L2/L3      hotfix
         │         │             │          │
         ▼         ▼             ▼          ▼
       done    diagnose      exploring  implementing
                  │              │        (skip grill/spec)
                  ▼              ▼             │
                 tdd         grilling          ▼
                  │              │          verifying
                  ▼              ▼             │
              code-review   specifying         ▼
                  │              │          archiving
                  ▼              ▼        (retroactive spec REQUIRED)
                done      reviewing_plan       │
                                  │            ▼
                                  ▼          done
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

**Metrics tracking (applies to every stage):** On entering and exiting each stage, record `metrics.stage_durations[<stage>]`. On any failure, increment `metrics.failure_counts[<type>]`. On confirmation rejection, increment `metrics.confirmation_rejections`. On spec drift detection, increment `metrics.spec_drifts`. These feed workflow self-optimization (e.g., stages with recurring failures get extra grilling attention).

### 1. intake

**Entry:** User provides a requirement via `/develop <requirement>` or natural language.

**Actions:**
1. Scan `devloop/.state/*.yaml` for in-progress changes.
2. If a legacy `devloop/.state.yaml` (no `schema_version`) exists, run the v0→v1 migration. See [state-migration.md](state-migration.md).
3. If one or more in-progress changes exist, list them and ask: continue an existing change / start a new parallel change / abandon one.
   - Parallel changes are allowed as long as their affected modules do not overlap (check `module-index.yaml`). If overlap is detected, warn and ask the user to pick one.
4. Read `devloop/config.yaml` for project settings.
5. Read `devloop/context/CONTEXT.md` for domain language.
6. Read `devloop/context/architecture-map.md` for module overview.

**Exit:** Project context loaded. Active change selected (existing or new). Ready to classify.

**State file:** Create `devloop/.state/<change-id>.yaml` (for new changes) or update the selected existing file. Set `stage: triaging`, `schema_version: 1`, `created_at` and `last_updated`. Initialize `metrics` to zeros.

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

**Exit:** Risk level assigned. `stage` updated in the active change's state file.

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
3. Evaluate fast-track eligibility (L2 only):
   - Single module? (check `module-index.yaml`)
   - No data model change? (check OpenSpec design.md)
   - Non-breaking API only? (check OpenSpec specs/)
   - If ALL true → set `fast_track: true` in the change's state file.
4. Present:
   - **Standard L2/L3**: requirement summary and plan summary separately.
     - **[CONFIRMATION POINT 1: requirement_understanding]**
     - **[CONFIRMATION POINT 2: plan_and_spec]**
   - **L2 fast track** (`fast_track: true`): combined summary in one block.
     - **[CONFIRMATION POINT 2.5: combined_plan]** — satisfies both 1 and 2.
   - **Hotfix**: skip this stage entirely (no plan to review). Go straight to implementing. Set `confirmed: [combined_plan]` nominally to satisfy downstream gates; the real gate is the retroactive spec at archiving.

**Exit:** User confirms plan (or combined plan). `confirmed` array updated in the active change's state file.

**State file:** Update `stage: implementing`.

---

### 7. implementing

**Entry:** Plan confirmed.

**Actions:**
1. **[L3 ONLY — CONFIRMATION POINT: start_coding]**
2. Read `tasks.md` from current OpenSpec change.
3. **Run critical-domain detection** over the planned changed files and OpenSpec artifacts. See [completion-checklist.md](completion-checklist.md#critical-domain-detection-heuristics). Record detected domains in the change's state file. Each detected domain requires dedicated tests before its tasks can be marked complete.
4. For each task, follow the TDD loop:
   a. **Red**: Write a failing test that covers the expected behavior.
      - Test file location: follow project convention (`tests/` or `src/__tests__/`).
      - Test file naming: `*.test.ts` (or language-appropriate suffix).
      - Test only external observable behavior, not internal implementation.
      - If the task touches a detected critical domain (security, data, payment, privacy), write dedicated tests for that domain.
   b. **Green**: Implement the minimum code to make the test pass.
   c. **Refactor**: Clean up code while keeping tests green.
   d. After completing each task, check it off in `tasks.md`.
   e. Update the active change's state file with current task and `metrics.stage_durations`/`failure_counts` as events occur.
5. **Spec-first principle:** If implementation reveals a spec gap:
   - Pause coding.
   - Update OpenSpec (`/opsx:update` or manual edit of specs/design/tasks).
   - Resume coding.
6. If test fails, enter `diagnosing-bugs` flow — do not blindly retry.
7. Track consecutive failures. If 3 in a row, pause and report.

**Exit:** All tasks in `tasks.md` completed.

**Test coverage by risk level:**
- **L1**: At least 1 regression test covering the bug scenario.
- **L2**: All OpenSpec specs scenarios have corresponding test coverage.
- **L3**: All scenarios + critical cross-module integration tests.
- **Critical domains** (security, data, payment, privacy): must have dedicated test coverage.

**State file:** Update `stage: verifying`.

---

### 8. verifying

**Entry:** All tasks completed.

**Actions:**
1. **Run full test suite** — `<test command from config.yaml>`
   - If tests fail → enter `diagnosing-bugs` flow.
   - Never modify test assertions to make them pass without verifying root cause.
   - Never disable or skip tests.
2. **Run typecheck** — `<typecheck command from config.yaml>`
3. **Run lint** — `<lint command from config.yaml>`
4. **Verify test coverage by risk level**:
   - Check L1: at least 1 regression test for the bug scenario.
   - Check L2/L3: every OpenSpec spec scenario has a corresponding test.
   - Check critical domains (security, data, payment, privacy): dedicated tests exist.
5. **Execute `/opsx:verify`** — checks completeness, correctness, coherence.
6. **Execute `code-review` skill** — checks standards and spec compliance.
   - Detect host capability (sub-agent / background tasks): if supported, run axes in parallel; otherwise run serially.

**Failure handling:**
- Test failure → `diagnosing-bugs` → fix → re-run tests.
- 3 consecutive test failures → **force pause**, output diagnostic report.
- If tests pass but `/opsx:verify` fails → fix spec gaps first.
- Tests cannot be skipped or bypassed — testing is a hard gate for archiving.

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
7. Clear the active change's state file (set `stage: done`). Keep the file for one session to allow post-archive review, then delete on next intake (or move to `devloop/.state/_archive/` if configured).

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
