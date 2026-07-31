# Recovery

Failure, interruption, and mid-flight change handling.

## Failure Types and Recovery

### Test Failure

```text
Test fails
  ↓
Enter diagnosing-bugs skill
  ↓
Reproduce → Minimise → Hypothesise → Instrument → Fix → Regression-test
  ↓
Is it a code bug?
  ├─ YES → Fix code, re-run test
  └─ NO → Is it a spec issue?
      ├─ YES → Pause implementing, return to specifying
      │        Update OpenSpec specs/design/tasks
      │        Return to implementing
      └─ NO → Is it an environment issue?
          ├─ YES → Report, ask user to fix environment
          └─ NO → Escalate to user
```

**Rule:** Never blindly retry the same fix. Never modify a test to make it pass without understanding why.

### Typecheck or Lint Failure

```text
Typecheck/Lint fails
  ↓
Read error messages
  ↓
Fix directly (these are usually mechanical)
  ↓
Re-run
  ↓
Fails again after fix?
  ├─ YES → increment consecutive_failures
  └─ NO → reset consecutive_failures to 0
```

**Rule:** If the same typecheck/lint error persists after 3 fix attempts, pause and report.

### OpenSpec CLI Command Failure

```text
openspec command fails
  ↓
Read error output
  ↓
Is it a config issue?
  ├─ YES → Fix config, retry
  └─ NO → Is it an artifact format issue?
      ├─ YES → Fix artifact (proposal.md, specs, design.md, tasks.md), retry
      └─ NO → Report error to user, do not bypass CLI
```

**Rule:** Never bypass OpenSpec CLI by hand-writing files that the CLI would generate. Never manually create change directories.

### /opsx:verify Finds Inconsistencies

```text
verify reports issues
  ↓
Classify each issue:
  - Completeness (missing implementation for a spec requirement)
  - Correctness (implementation doesn't match spec intent)
  - Coherence (design says X, code does Y)
  ↓
For each issue:
  ├─ Code is wrong → fix code, re-verify
  ├─ Spec is wrong → update spec, re-verify
  └─ Requirement changed → pause, ask user: update or new change?
  ↓
All criticals resolved?
  ├─ YES → proceed to archiving (warnings allowed)
  └─ NO → pause, report remaining criticals
```

### Codebase Structure Mismatch

```text
Implementation discovers code doesn't match expectations
  ↓
Pause implementing
  ↓
Return to exploring
  ↓
Re-investigate affected modules
  ↓
Update architecture-map.md and module-index.yaml if needed
  ↓
Return to specifying if spec needs update
  ↓
Return to implementing
```

### Consecutive Failure Limit

```text
Track consecutive_failures in .state.yaml

Any failure (test, typecheck, lint, verify) increments the counter.
Any success resets it to 0.

consecutive_failures >= 3
  ↓
FORCE PAUSE
  ↓
Output diagnostic report:
  - What was being attempted
  - What failed
  - Error messages
  - What was tried
  - Hypothesis for why it's stuck
  ↓
Ask user to intervene or provide guidance
```

**Rule:** This is a safety valve. Do not attempt a 4th retry. Do not try creative workarounds. Stop and report.

## Interruption Handling

### User Interrupts Mid-Stage

```text
User stops the conversation or says "stop" / "wait"
  ↓
Save current state to .state.yaml:
  - current_change
  - stage
  - risk_level
  - confirmed checkpoints
  - current task
  - consecutive_failures
  ↓
Output handoff summary:
  - Current change: <id>
  - Current stage: <stage>
  - Completed tasks: <n>/<n>
  - Current task: <description>
  - Known issues: <list>
  - Next step: <what to do when resuming>
  - User confirmations needed: <list>
```

### Session Resume

```text
/devloop is invoked
  ↓
Read .state.yaml
  ↓
Exists and stage != done?
  ├─ YES → Report:
  │         "You have an in-progress change: <id>
  │          Stage: <stage>
  │          Tasks: <completed>/<total>
  │          Current: <current task>
  │
  │          Continue / Start fresh / Abandon?"
  │
  ├─ Continue → Resume from saved stage
  ├─ Start fresh → Archive current (if any artifacts), create new
  └─ Abandon → Clear .state.yaml, optionally archive incomplete change
```

## Mid-Flight Requirement Change

### Same Intent, Refined Execution

```text
User wants to change approach or add edge cases
  ↓
Use /opsx:update
  ↓
Update proposal.md, specs/, design.md, tasks.md as needed
  ↓
Re-present at plan_and_spec confirmation point
  ↓
Continue implementing
```

### Intent Fundamentally Changed

```text
User's new requirement is different work
  ↓
Can current change be marked "done" standalone?
  ├─ YES → Archive current change, create new change
  └─ NO → Abandon current change (archive as incomplete or delete)
           Create new change
```

### Scope Exploded

```text
Change grew so much it's essentially different work
  ↓
Pause
  ↓
Assess: is this the same work?
  ├─ >50% overlap and same intent → /opsx:update
  └─ <50% overlap or different intent → new change
```

## Spec Drift Recovery

```text
During implementing, agent discovers spec and code have diverged
  ↓
Pause coding
  ↓
Identify drift:
  - Did code change without spec update?
  - Did spec change without code update?
  - Did understanding change?
  ↓
Fix the source of truth:
  - If spec is correct → fix code to match
  - If code is correct → update spec to match
  - If both are wrong → return to grilling, re-clarify
  ↓
Re-verify alignment
  ↓
Resume implementing
```

## Handoff Summary Format

When DevLoop cannot continue and must hand off:

```markdown
## DevLoop Handoff

**Change:** <change-id>
**Stage:** <stage>
**Risk level:** <L0/L1/L2/L3>
**Tasks:** <completed>/<total>
**Current task:** <description or "none">

### Confirmed checkpoints
- [x] requirement_understanding
- [x] plan_and_spec
- [ ] start_coding (L3 only)
- [ ] final_archive

### Blockers
1. <blocker 1>
2. <blocker 2>

### Next step
<what needs to happen to resume>

### User input needed
<specific questions or decisions needed from user>
```
