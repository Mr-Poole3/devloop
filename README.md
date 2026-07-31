# 🔄 DevLoop

> One requirement in. A verified, spec-driven, archived change out.

**[English](README.md)** | [中文](README.zh-CN.md) | [日本語](README.ja.md)

DevLoop is an integrated AI development workflow that combines [requirement grilling](https://github.com/mattpocock/skills), [OpenSpec](https://github.com/Fission-AI/OpenSpec) spec-driven changes, TDD, code review, and archival into a **single automated loop** 🚀

Instead of manually orchestrating multiple tools across the development lifecycle, you provide one requirement and DevLoop drives the entire process — clarifying, exploring, specifying, implementing, testing, verifying, and archiving.

---

## 🤔 Why DevLoop?

AI coding agents are powerful but unpredictable. Common failure modes:

| Failure | Cause | DevLoop's Fix |
|---------|-------|---------------|
| Agent builds the wrong thing | Misaligned understanding | 🔍 Grilling session before any code |
| Agent produces a ball of mud | No architecture discipline | 📐 Spec-first principle + module spec tracking |
| No one knows what was decided | Decisions live in chat history | 📄 OpenSpec artifacts as persistent truth |
| Specs drift from code | No verification step | ✅ Three-layer verification before archive |
| Small fixes get over-documented | One-size-fits-all process | ⚖️ Risk-based routing (L0–L3) |
| Work lost on interruption | Session-only state | 💾 `.state.yaml` for cross-session recovery |

---

## ⚙️ How It Works

```text
User: /develop I want to add dark mode support

DevLoop:
  1. Reads project context (config, architecture map, domain language)
  2. Classifies risk → L2 (medium feature)
  3. Explores affected modules, builds missing specs
  4. Grills the user to clarify every decision branch
  5. Creates OpenSpec change (proposal + specs + design + tasks)
  ── ⏸️ User confirms requirement understanding ──
  ── ⏸️ User confirms plan and spec ──
  6. Implements with TDD (red-green-refactor)
  7. Any spec gap? → Updates OpenSpec first, then code
  8. Runs tests, typecheck, lint
  9. Executes /opsx:verify (spec completeness, correctness, coherence)
 10. Executes code-review (standards + spec compliance)
  ── ⏸️ User confirms archive ──
 11. Syncs specs to formal registry
 12. Archives change
 13. Updates module index and architecture map
 14. Outputs delivery report 📦
```

---

## 🛠️ Two Skills

### `devloop-setup` — Initialize

Set up, check, and repair the DevLoop workflow in any repository.

```text
/devloop-setup              # check status (default)
/devloop-setup init         # first-time setup
/devloop-setup repair       # fix missing config
/devloop-setup reconfigure  # update settings
```

**What it does:**
- 🔎 Auto-detects tech stack (language, framework, package manager, test/lint/build commands)
- 🗺️ Scans source directories and builds an L0 architecture map
- 📁 Creates the `devloop/` directory structure
- ⚙️ Initializes OpenSpec configuration
- 📝 Generates config files with detected values
- 🔁 Idempotent — safe to run multiple times

### `devloop` — Develop

Drive a requirement through the full development loop.

```text
/develop I want to add dark mode support
/develop Fix the login redirect bug
/develop Refactor the payment module to event-driven architecture
/develop Add multi-tenant isolation across all services
```

**What it does:**
- 💾 Reads `.state.yaml` to resume interrupted work
- ⚖️ Classifies risk level (L0–L3) based on escalation factors
- 🔥 Grills the user to resolve every decision branch
- 📄 Creates OpenSpec change artifacts (proposal, specs, design, tasks)
- 🧪 Implements with TDD
- ✅ Verifies against specs
- 👁️ Reviews code quality
- 📦 Archives and syncs formal specs
- 📊 Outputs delivery report

---

## 🚀 Quick Start

### 1. Install DevLoop skills

**Option A — Install via `npx skills` (recommended)**

```bash
npx skills@latest add Mr-Poole3/devloop
```

The CLI will detect both `devloop-setup` and `devloop` skills and let you choose which agents to install them on (Claude Code, Codex, OpenCode, TRAE, Cursor, and 50+ more).

You can also install a specific skill:

```bash
npx skills@latest add Mr-Poole3/devloop/devloop-setup
npx skills@latest add Mr-Poole3/devloop/devloop
```

**Option B — Manual install**

Copy the `devloop/` and `devloop-setup/` skill directories into your agent's skill loading path.

### 2. Run setup in your project

```text
/devloop-setup init
```

This creates the `devloop/` directory, detects your tech stack, and generates all configuration.

### 3. Start developing

```text
/develop <your requirement>
```

That's it. DevLoop handles the rest. 🎉

---

## ⚖️ Risk Levels

DevLoop automatically classifies each requirement by risk level and selects the appropriate workflow weight.

| Level | Trigger | Workflow | Confirmations |
|-------|---------|----------|---------------|
| **L0** | No code behavior change (docs, comments, config typos) | Direct handling | None |
| **L1** | Single-module, low-risk fix | `diagnose → tdd → review` | Start, Complete |
| **L2** | Medium feature or refactor | `grill → OpenSpec → apply → verify → archive` | Requirement, Plan, Archive |
| **L3** | Cross-module, high-risk, architectural | `grill-with-docs → explore → spec/tickets → OpenSpec → apply → verify → review → archive` | Requirement, Plan, Start coding, Archive |

**🚨 Escalation factors** (any of these forces at least L2):

- Data model changes
- Public API changes
- Permission, payment, privacy, or user data
- Data migration
- Cross-service or cross-module impact
- Backward compatibility requirements
- Rollback requirements
- High technical uncertainty
- Multi-person or multi-agent collaboration

---

## 📁 Directory Structure

After `devloop-setup init`, your project gets a single `devloop/` directory:

```text
your-project/
├── devloop/                        # All DevLoop artifacts live here
│   ├── config.yaml                 # DevLoop total control config
│   ├── .state.yaml                 # Runtime state (gitignored)
│   ├── context/                    # Project context and architecture map
│   │   ├── architecture-map.md     # L0 module index (fast structural scan)
│   │   ├── tech-stack.md           # Detected language, framework, commands
│   │   ├── module-index.yaml       # Module spec tracking (has_spec flag)
│   │   └── CONTEXT.md              # Domain language and business context
│   ├── decisions/                  # Architecture Decision Records (ADRs)
│   ├── research/                   # Investigation notes and draft proposals
│   ├── tickets/                    # Task breakdown output (to-tickets)
│   ├── reports/                    # Verification reports and delivery summaries
│   └── openspec/                   # OpenSpec native directory
│       ├── config.yaml             # OpenSpec artifact rules and context
│       ├── specs/                  # Archived formal behavior specs
│       └── changes/                # In-flight changes
│           └── archive/            # Completed change history
├── src/                            # Your source code (untouched)
├── package.json                    # Your project manifest (untouched)
└── ...
```

**File management rules:**
1. Formal decisions never live in scratch — they migrate to `CONTEXT.md`, ADRs, or OpenSpec.
2. Each piece of information has exactly one primary file — others reference, not duplicate.
3. File lifecycle is clear — temporary files are cleanable, in-flight changes live in `changes/`, completed work is archived.

---

## 🔄 Lifecycle

DevLoop follows a 10-stage state machine. Stages are not locked phases — they're actions that can be revisited.

```text
intake          Receive requirement, load project context
triaging        Classify risk level (L0-L3)
exploring       Read architecture map, build missing module specs (L1 on-demand)
grilling        Clarify requirements (grill-me for L2, grill-with-docs for L3)
specifying      Create OpenSpec change (proposal, specs, design, tasks)
reviewing_plan  ⏸️ [CONFIRM] Present requirement + plan summary
implementing    ⏸️ [CONFIRM L3] Execute tasks with TDD, spec-first
verifying       Run tests, /opsx:verify, code-review
archiving       ⏸️ [CONFIRM] Sync specs, archive change, update module index
done            Output delivery report 📦
```

**Backward transitions are allowed:**
- `implementing → specifying`: Spec gap discovered
- `implementing → exploring`: Codebase structure mismatch
- `verifying → implementing`: Verification found code issues
- `verifying → specifying`: Verification found spec issues

---

## ⏸️ Confirmation Points

Even in high-automation mode, DevLoop pauses at key points to prevent catastrophic mistakes.

| Point | When | What's Presented |
|-------|------|------------------|
| **Requirement understanding** | After grilling, before specs | Goal, users, scope, success criteria, decisions, uncertainties |
| **Plan and spec** | After OpenSpec artifacts, before code | Proposal, specs, design, tasks, testing strategy, risks, rollback |
| **Start coding** (L3 only) | Before first code modification | Files to change, new files, test files |
| **Final archive** | After verification, before archive | Delivery summary, test results, verify results, warnings |

---

## 📐 Spec-First Principle

```text
Does this change affect external behavior or architecture?
  YES → Update OpenSpec first, then modify code
  NO  → Modify code directly, note in delivery summary
```

This prevents spec drift — the scenario where code and specs diverge over time, making `/opsx:verify` increasingly meaningless.

---

## 🏗️ Brownfield Strategy: Structured Index + On-Demand Modeling

DevLoop doesn't require a greenfield project. For existing codebases, it uses a three-layer approach:

```text
L0 — Architecture Map (at setup)
     Fast structural scan. Records modules, paths, responsibilities, dependencies.
     No behavior specs. No deep analysis. Minutes to complete.

L1 — On-Demand Module Specs (on first touch)
     When a change first touches a module with has_spec: false,
     DevLoop deep-dives into that module and builds a behavior spec.
     The spec is then reusable by all future changes.

L2 — Continuous Maintenance (on archive)
     Each archived change automatically syncs delta specs to the formal registry.
     Module index and architecture map are updated if structure changed.
```

This means your spec coverage grows naturally with actual development activity — no wasted upfront documentation. 🌱

---

## 💾 State Recovery

DevLoop maintains `devloop/.state.yaml` (gitignored) for cross-session recovery:

```yaml
current_change: add-dark-mode
stage: implementing
risk_level: L2
confirmed: [requirement_understanding, plan_and_spec]
pending_confirmation: []
tasks:
  total: 12
  completed: 7
  current: "2.3 Update theme toggle component"
consecutive_failures: 0
last_updated: "2026-08-01T10:30:00Z"
```

When you resume, DevLoop reads the state file and asks: **continue / start fresh / abandon?**

---

## 🛡️ Failure Recovery

| Error Type | Recovery Strategy |
|---|---|
| Test failure | Enter `diagnosing-bugs` skill — reproduce, minimise, hypothesise, instrument, fix, regression-test |
| Typecheck/Lint failure | Fix directly; if 3 consecutive failures, pause |
| OpenSpec CLI failure | Fix config or artifact format, retry; never bypass CLI |
| `/opsx:verify` inconsistency | Classify each issue, fix code or spec, re-verify |
| Codebase structure mismatch | Return to `exploring`, re-investigate |
| **3 consecutive failures** | **🚨 Force pause** — output diagnostic report, request user intervention |
| User interrupt | Save state, output handoff summary |
| Missing dependency | Pause and report, do not attempt alternatives |

---

## ✅ Three-Layer Completion Standard

A change is complete only when all three layers pass:

```text
Code Layer     🧪 Tests pass + Typecheck pass + Lint pass + Build pass
Spec Layer     📋 /opsx:verify: 0 criticals (warnings allowed)
Delivery Layer 📦 Tasks complete + Code review clean + Delivery summary generated
```

---

## 🖥️ Host Compatibility

DevLoop is designed to work across AI coding agents without depending on host-specific features.

| Host | Status | Notes |
|------|--------|-------|
| Claude Code | ✅ Full support | Sub-agent parallel code review |
| Codex | ✅ Full support | Serial code review |
| OpenCode | ✅ Full support | Serial code review |
| TRAE | ✅ Full support | Serial code review |
| Cursor | ✅ Full support | Serial code review |

**Degradation rules:**
- No sub-agent support → `code-review` runs serially instead of parallel
- No auto skill trigger → User manually invokes `/develop`
- No OpenSpec CLI → L0/L1 only; L2/L3 paused with install instructions

---

## 📂 Skill File Structure

```text
devloop-setup/
├── SKILL.md                              # Main instructions
├── references/
│   ├── setup-checklist.md                # File-by-file init/check/repair checklist
│   ├── project-detection.md              # Tech stack and module detection rules
│   ├── routing-policy.md                 # Risk classification (L0-L3) rules
│   └── repair-rules.md                   # What repair can and cannot do
└── templates/
    ├── devloop-config.yaml               # DevLoop total control config template
    ├── openspec-config.yaml              # OpenSpec artifact rules template
    ├── architecture-map.md               # L0 architecture map template
    ├── tech-stack.md                     # Tech stack record template
    ├── module-index.yaml                 # Module index with has_spec tracking
    ├── CONTEXT.md                        # Domain language template
    └── adr-template.md                   # Architecture Decision Record template

devloop/
├── SKILL.md                              # Main instructions (10-stage state machine)
├── references/
│   ├── lifecycle.md                      # Full state machine with backward transitions
│   ├── routing.md                        # Risk classification and workflow selection
│   ├── confirmation-points.md            # What to present at each checkpoint
│   ├── recovery.md                       # Failure, interruption, mid-flight change handling
│   └── completion-checklist.md           # Three-layer verification standard
└── templates/
    ├── requirement-summary.md            # Confirmation point 1 template
    ├── plan-summary.md                   # Confirmation point 2 template
    └── delivery-summary.md               # Confirmation point 4 template
```

---

## 📦 Dependencies

| Dependency | Required For | Install |
|------------|-------------|---------|
| [OpenSpec CLI](https://github.com/Fission-AI/OpenSpec) | L2/L3 full workflow | `npm install -g @fission-ai/openspec@latest` |
| [mattpocock/skills](https://github.com/mattpocock/skills) | Grilling, TDD, debugging, code review | `npx skills@latest add mattpocock/skills` |

DevLoop works without these for L0/L1 tasks. L2/L3 will pause and report missing dependencies.

---

## 💡 Key Principles

1. **Single entry point** — User only provides a requirement. No manual orchestration.
2. **Understand before executing** — Grilling happens before any code is written.
3. **OpenSpec is the truth** — Current change's proposal, specs, design, and tasks are the sole execution authority.
4. **Skills are capabilities** — Grilling, TDD, debugging, and review are composable tools, not a rigid framework.
5. **Risk determines weight** — Small fixes stay light; big changes get full treatment.
6. **Implementation allows rollback** — New information updates specs before code.
7. **Completion must be verifiable** — Three-layer verification: code, spec, delivery.

---

## 🏛️ Authority Hierarchy

```text
CONTEXT.md / ADR              → Long-term project facts and domain language
devloop/openspec/changes/     → Current change's sole execution truth
devloop/openspec/specs/       → Archived formal behavior specs
Issue Tracker                 → Team coordination, ownership, scheduling
Code & Tests                  → The actual implementation
```

When documents conflict: **current OpenSpec change wins**, but conflicts must be reported in the delivery summary.

---

## 🤝 Contributing

Contributions are welcome. For significant changes, please follow the DevLoop workflow itself:

1. Run `/devloop-setup init` in your fork
2. `/develop <your improvement>`
3. Ensure three-layer verification passes
4. Submit a PR with the delivery summary

---

## 📄 License

[MIT](LICENSE)
