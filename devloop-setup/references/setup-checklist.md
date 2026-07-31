# Setup Checklist

File-by-file checklist for init, check, and repair modes.

## Directory Structure

```
devloop/                          # Root for all DevLoop artifacts
├── config.yaml                   # DevLoop total control config
├── .state/                       # Per-change state files (gitignored, multi-change parallel)
│   └── <change-id>.yaml          # One file per in-progress change (schema_version: 1)
├── context/                      # Project context and architecture map
│   ├── architecture-map.md       # Module list, entry points, dependencies
│   ├── tech-stack.md             # Tech stack, toolchain, test commands
│   ├── module-index.yaml         # Structured module index with has_spec flag
│   └── CONTEXT.md                # Domain language and business context
├── decisions/                    # ADR and decision records
├── research/                     # Investigation notes and draft proposals
├── tickets/                      # Task breakdown output
├── reports/                      # Verification reports and delivery summaries
└── openspec/                     # OpenSpec native directory
    ├── config.yaml               # OpenSpec artifact rules and context
    ├── specs/                    # Archived formal behavior specs
    └── changes/                  # In-flight changes
        └── archive/              # Completed change history
```

## Init Mode Checklist

1. [ ] Create `devloop/` root directory
2. [ ] Create `devloop/config.yaml` from template, filled with detected values
3. [ ] Create `devloop/context/` directory
4. [ ] Create `devloop/context/architecture-map.md` from scan results
5. [ ] Create `devloop/context/tech-stack.md` from detection results
6. [ ] Create `devloop/context/module-index.yaml` with all detected modules
7. [ ] Create `devloop/context/CONTEXT.md` from template
8. [ ] Create `devloop/decisions/` directory (empty, with `.gitkeep`)
9. [ ] Create `devloop/research/` directory (empty, with `.gitkeep`)
10. [ ] Create `devloop/tickets/` directory (empty, with `.gitkeep`)
11. [ ] Create `devloop/reports/` directory (empty, with `.gitkeep`)
12. [ ] Create `devloop/openspec/` directory
13. [ ] Create `devloop/openspec/config.yaml` from template
14. [ ] Create `devloop/openspec/specs/` directory
15. [ ] Create `devloop/openspec/changes/` directory
16. [ ] Create `devloop/openspec/changes/archive/` directory
17. [ ] Create `devloop/.state/` directory (empty, with `.gitkeep`)
18. [ ] Run `openspec init` inside `devloop/openspec/` if CLI is available
19. [ ] Merge OpenSpec generated config with DevLoop rules
20. [ ] Add `devloop/.state/` and legacy `devloop/.state.yaml` to `.gitignore`
21. [ ] If a legacy `devloop/.state.yaml` exists, leave it for DevLoop's v0→v1 migration (do not delete here)
22. [ ] Output setup report with warnings

## Check Mode Checklist

1. [ ] Verify `devloop/config.yaml` exists and is valid YAML
2. [ ] Verify `devloop/context/architecture-map.md` exists and is non-empty
3. [ ] Verify `devloop/context/module-index.yaml` exists and is valid YAML
4. [ ] Verify `devloop/context/tech-stack.md` exists and is non-empty
5. [ ] Verify `devloop/context/CONTEXT.md` exists
6. [ ] Verify `devloop/openspec/config.yaml` exists and is valid YAML
7. [ ] Verify `devloop/openspec/specs/` directory exists
8. [ ] Verify `devloop/openspec/changes/` directory exists
9. [ ] Verify `devloop/openspec/changes/archive/` directory exists
10. [ ] Check if OpenSpec CLI is installed and accessible
11. [ ] Check if `.gitignore` contains `devloop/.state/` (and legacy `devloop/.state.yaml`)
12. [ ] Verify `devloop/.state/` directory exists
13. [ ] Scan `devloop/.state/*.yaml`: report any in-progress changes (stage != done) and any state files with `schema_version` higher than supported
14. [ ] Check if test/lint/typecheck commands in config still work
15. [ ] Report status: ready / needs repair / needs init

## Repair Mode Checklist

1. [ ] Run Check Mode first
2. [ ] For each missing file or directory: create from template
3. [ ] For each invalid YAML: attempt to fix or flag for manual attention
4. [ ] For missing `.gitignore` entry: add `devloop/.state/` (and legacy `devloop/.state.yaml`)
5. [ ] For missing `devloop/.state/` directory: create it with `.gitkeep`
6. [ ] For legacy `devloop/.state.yaml` still present: leave it; DevLoop intake will migrate it (do not migrate during setup)
7. [ ] For stale tech-stack.md: re-detect and update (confirm first)
8. [ ] For stale module-index.yaml: re-scan and update (confirm first)
9. [ ] For missing OpenSpec CLI: remind user to install
10. [ ] Never overwrite files that have user-written content
11. [ ] Report what was repaired and what needs manual attention

## Reconfigure Mode Checklist

1. [ ] Read current `devloop/config.yaml`
2. [ ] Ask user which settings to change
3. [ ] Present new values for confirmation
4. [ ] Write updated config
5. [ ] If routing policy changed: update `devloop/config.yaml` routing section
6. [ ] If OpenSpec rules changed: update `devloop/openspec/config.yaml`
7. [ ] Report changes made

## Validation Rules

- YAML files must parse without errors
- `module-index.yaml` must have at least one module entry (unless project is empty)
- `config.yaml` must contain: version, tech_stack, commands, routing, tracker
- `openspec/config.yaml` must contain: schema, context, rules
- If any validation fails, report the specific error and do not mark as ready
