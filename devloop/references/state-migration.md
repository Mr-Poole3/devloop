# State Migration

Schema versioning and upgrade rules for `devloop/.state/<change-id>.yaml`.

## Current Schema

Version: `1`

```yaml
schema_version: 1
change_id: <string>
stage: <enum: intake|triaging|exploring|grilling|specifying|reviewing_plan|implementing|verifying|archiving|done>
risk_level: <enum: L0|L1|L2|L3>
fast_track: <bool>      # L2 fast track: merge requirement_understanding + plan_and_spec
hotfix: <bool>          # hotfix bypass: skip grilling/specifying, retroactive spec required
confirmed: [<enum: requirement_understanding|plan_and_spec|start_coding|final_archive>]
pending_confirmation: [<same enum>]
tasks:
  total: <int>
  completed: <int>
  current: <string|none>
consecutive_failures: <int>
metrics:
  stage_durations: <map<string, duration>>
  failure_counts: <map<string, int>>   # keys: test, typecheck, lint, verify
  confirmation_rejections: <int>
  spec_drifts: <int>
created_at: <ISO 8601>
last_updated: <ISO 8601>
```

## Required Fields

`schema_version`, `change_id`, `stage`, `risk_level`, `confirmed`, `consecutive_failures`, `created_at`, `last_updated` are always required.

`fast_track` and `hotfix` default to `false` if absent (treat as false on read).

`metrics` defaults to an empty map per key if absent.

## Migration Rules

### v0 → v1

Triggered when DevLoop reads a state file with no `schema_version` field, or reads the legacy single-file `devloop/.state.yaml`.

**Steps:**

1. Read the legacy `devloop/.state.yaml` (or a v0 per-change file).
2. Extract `current_change` as the change-id. If missing, prompt the user.
3. Compute `devloop/.state/<change-id>.yaml` as the target path.
   - If the target already exists, do not overwrite. Ask the user how to reconcile.
4. Write the new file with:
   - `schema_version: 1`
   - All preserved fields from v0 (`current_change` → `change_id`, `stage`, `risk_level`, `confirmed`, `pending_confirmation`, `tasks`, `consecutive_failures`, `last_updated`).
   - `fast_track: false`, `hotfix: false`.
   - `metrics: { stage_durations: {}, failure_counts: {}, confirmation_rejections: 0, spec_drifts: 0 }`.
   - `created_at`: copy from `last_updated` if absent.
5. Delete the legacy `devloop/.state.yaml`.
6. Report the migration in the intake stage output.

**Idempotency:** Re-running migration on an already-migrated file is a no-op. If `devloop/.state.yaml` does not exist and `devloop/.state/` does, skip migration.

**Failure:** If the v0 file is corrupt (unparseable YAML, missing `change_id` and no `current_change`), do not delete it. Move it to `devloop/.state/_corrupt-<timestamp>.yaml.bak` and report.

### Future Versions

When bumping `schema_version`:

1. Add a section `### v<n-1> → v<n>` below describing the transformation.
2. Migration must be backward-compatible: a v<n-1> file can always be upgraded in place.
3. Never silently drop fields. If a field is removed, log it in `metrics` or the delivery summary.
4. Bump the "Current Schema" version above.
5. Update [lifecycle.md](lifecycle.md) intake stage to call the migration check.

## Forward Compatibility

If DevLoop reads a state file whose `schema_version` is **higher** than it knows:

- Do not attempt migration.
- Do not modify the file.
- Output: "State file `<path>` has schema_version `<n>`, but this DevLoop only understands up to `<known>`. Upgrade DevLoop or hand-edit the file with caution."
- Halt intake for that change.

## Multi-Change Directory Layout

```text
devloop/.state/
├── add-dark-mode.yaml
├── fix-login-redirect.yaml
└── _corrupt-20260801T103000.yaml.bak   # quarantined v0 corrupt file
```

- One file per change. Filename MUST equal `<change_id>.yaml`.
- Intake scans the directory and lists all non-archive files.
- A change is "in progress" if its file exists and `stage != done`.
- Archiving a change: set `stage: done` and keep the file for one session, then delete on next intake. Alternatively move to `devloop/.state/_archive/` if the project wants history.
- Hotfix and parallel changes coexist: each has its own state file and lifecycle. Confirmation points apply per-change.

## .gitignore

`devloop/.state/` MUST be in `.gitignore`. Local state is not shared across machines.
