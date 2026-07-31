# Setup Checklist

File-by-file checklist for init, check, and repair modes.

## Directory Structure

```
devloop/                          # Root for all DevLoop artifacts
├── config.yaml                   # DevLoop total control config
├── .state.yaml                   # Runtime state (gitignored, not created by setup)
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
17. [ ] Run `openspec init` inside `devloop/openspec/` if CLI is available
18. [ ] Merge OpenSpec generated config with DevLoop rules
19. [ ] Add `devloop/.state.yaml` to `.gitignore`
20. [ ] Output setup report with warnings

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
11. [ ] Check if `.gitignore` contains `devloop/.state.yaml`
12. [ ] Check if test/lint/typecheck commands in config still work
13. [ ] Report status: ready / needs repair / needs init

## Repair Mode Checklist

1. [ ] Run Check Mode first
2. [ ] For each missing file or directory: create from template
3. [ ] For each invalid YAML: attempt to fix or flag for manual attention
4. [ ] For missing `.gitignore` entry: add it
5. [ ] For stale tech-stack.md: re-detect and update (confirm first)
6. [ ] For stale module-index.yaml: re-scan and update (confirm first)
7. [ ] For missing OpenSpec CLI: remind user to install
8. [ ] Never overwrite files that have user-written content
9. [ ] Report what was repaired and what needs manual attention

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
