# Repair Rules

What repair mode can and cannot do.

## Can Repair

### Missing Directories
- Create any missing directory under `devloop/`.
- Add `.gitkeep` to keep empty directories in git.

### Missing Files
- Create missing files from templates.
- Fill template with detected values where possible.

### Invalid YAML
- Attempt to parse `config.yaml` and `module-index.yaml`.
- If parse fails, report the line and error.
- Do NOT auto-fix invalid YAML blindly — show the user the error and suggest a fix.
- If the user confirms, apply the suggested fix.

### Stale Detection
- Re-run tech stack detection if `tech-stack.md` is older than the last `package.json` / `pyproject.toml` modification.
- Re-run module scan if `module-index.yaml` is older than the last source file modification.
- Always confirm before overwriting.

### Missing .gitignore Entry
- Add `devloop/.state/` and legacy `devloop/.state.yaml` to `.gitignore` if missing.
- Do not modify other `.gitignore` entries.

### Missing OpenSpec Structure
- Create `devloop/openspec/specs/`, `devloop/openspec/changes/`, `devloop/openspec/changes/archive/` if missing.
- Create `devloop/openspec/config.yaml` from template if missing.

## Cannot Repair

### User-Written Content
- Never overwrite `CONTEXT.md` if it contains user-written content (not a template).
- Never overwrite `architecture-map.md` if it has been manually edited.
- Never overwrite `config.yaml` if it contains custom routing rules.
- Never overwrite ADR files in `decisions/`.

### OpenSpec Changes
- Never modify, delete, or move files under `devloop/openspec/changes/`.
- These are active change artifacts managed by the `devloop` skill and OpenSpec CLI.

### Source Code
- Never modify source code files.
- Never modify test files.
- Never modify CI configuration.

### Dependencies
- Never install, upgrade, or remove dependencies.
- Only report missing tools (e.g., OpenSpec CLI).

## Conflict Resolution

When existing config conflicts with detected settings:

```text
1. Report the conflict:
   "config.yaml has test_command: 'npm test'
    but package.json suggests: 'pnpm test'"

2. Ask user which to keep.

3. Update only if user confirms.
```

## Recovery from Partial Setup

If setup was interrupted:

```text
1. Run check mode.
2. Identify what was created and what wasn't.
3. Create only what's missing.
4. Do not recreate or overwrite existing files.
5. Report what was completed and what was added.
```
