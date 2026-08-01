# Wiring

How `develop-new` generates the `devloop/` structure for a freshly scaffolded project, and how it verifies the result. The output must be indistinguishable from what `devloop-setup init` produces, so `/devloop-setup check` passes and `/develop` works immediately.

## Authoritative Schemas

Do **not** re-derive file formats. The authoritative templates live in `devloop-setup/templates/` and are referenced here, not duplicated:

| Generated file | Authoritative template | Notes for develop-new |
|----------------|------------------------|-----------------------|
| `devloop/config.yaml` | `devloop-setup/templates/devloop-config.yaml` | Fill `tech_stack` and `commands` from the scaffold state file. Keep routing/escalation/hotfix/confirmation/failure/brownfield/host_capability as-is. |
| `devloop/openspec/config.yaml` | `devloop-setup/templates/openspec-config.yaml` | Fill the `context:` block's tech-stack/package-manager/testing placeholders from the chosen stack. Keep rules as-is. |
| `devloop/context/architecture-map.md` | `devloop-setup/templates/architecture-map.md` | `## Module Index` sections come from the **blueprint's planned modules**, not detection. |
| `devloop/context/tech-stack.md` | `devloop-setup/templates/tech-stack.md` | From the scaffold state file's `tech_stack` + `commands`. |
| `devloop/context/module-index.yaml` | `devloop-setup/templates/module-index.yaml` | Populate the example entries with the planned modules; every entry `has_spec: false`. |
| `devloop/context/CONTEXT.md` | `devloop-setup/templates/CONTEXT.md` | **Replace placeholders with grilling results** — this is a real fill, not a copy. See below. |
| `devloop/decisions/ADR-0001` | `devloop-setup/templates/adr-template.md` | Only if blueprinting recorded a significant architecture decision. |

## Directory Structure to Create

```text
<project>/
├── blueprint.md                    # from blueprinting stage
├── src/                            # from scaffolding stage
├── devloop/
│   ├── config.yaml
│   ├── .state/
│   │   └── <project>-scaffold.yaml # in-progress (later: devloop feature changes)
│   ├── context/
│   │   ├── architecture-map.md
│   │   ├── tech-stack.md
│   │   ├── module-index.yaml
│   │   └── CONTEXT.md
│   ├── decisions/                  # .gitkeep
│   ├── research/                   # .gitkeep
│   ├── tickets/                    # .gitkeep
│   ├── reports/                    # .gitkeep
│   └── openspec/
│       ├── config.yaml
│       ├── specs/
│       └── changes/
│           └── archive/
└── .gitignore                      # gains devloop/.state/ (+ legacy .state.yaml)
```

## CONTEXT.md Filling

This is the one file develop-new must **write from content**, not from a template:

- `## Project Purpose` — from the grilling goal statement.
- `## Core Domain Terms` — from grilling; fill the term/definition table.
- `## Module Glossary` — from the blueprint's module division (business role per module).
- `## Conventions` — from grilling (naming / testing / API).
- `## Out of Bounds` — the blueprint's out-of-bounds list, plus **critical-domain guardrails** (e.g. "payment: no money math in UI layer", "privacy: no PII in logs").

## module-index.yaml Filling

```yaml
modules:
  - name: app
    path: src/app/
    responsibility: "Application entry and wiring"
    key_interfaces: []
    dependencies: []
    has_spec: false
    last_scanned: "<today>"
```

Every planned module gets an entry with `has_spec: false`. This preserves DevLoop's Brownfield L1 on-demand strategy: the first `/develop` change that touches a module deep-dives and builds its behavior spec.

## .gitignore

Append if not present:

```text
devloop/.state/
devloop/.state.yaml
```

The minimal-ts template already ships both lines; other scaffolds may need the append.

## OpenSpec CLI

- If `openspec` CLI is available: run `openspec init` inside `devloop/openspec/`, then merge the generated config with DevLoop rules (same as devloop-setup step 7).
- If not available: create the minimum structure manually and record the warning in the wiring confirmation and delivery report. L2/L3 `/develop` changes will pause until it is installed (`npm install -g @fission-ai/openspec@latest`).

## Verification

Before the [wiring] confirmation point, run an equivalent of `/devloop-setup check`:

1. `devloop/config.yaml` exists and is valid YAML, contains: `version`, `tech_stack`, `commands`, `routing`, `tracker`.
2. `devloop/openspec/config.yaml` exists, contains: `schema`, `context`, `rules`.
3. `devloop/context/` has all four files; `architecture-map.md` and `tech-stack.md` non-empty; `CONTEXT.md` has no unresolved HTML placeholders (`<!-- ... -->`).
4. `devloop/context/module-index.yaml` is valid YAML and lists every planned module with `has_spec: false`.
5. `devloop/openspec/specs/`, `devloop/openspec/changes/`, `devloop/openspec/changes/archive/` exist.
6. `devloop/decisions/`, `research/`, `tickets/`, `reports/` exist (`.gitkeep`).
7. `.gitignore` contains `devloop/.state/` (and legacy `devloop/.state.yaml`).
8. `devloop/.state/` exists and contains the scaffold state file.
9. The scaffold's health proof still passes: `npm test && npm run typecheck` (or stack equivalent).

If any check fails → fix, re-verify, then present the [wiring] confirmation. Never present a partial wiring as approved.
