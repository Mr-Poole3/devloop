# Stack Recipes

Official scaffold CLI commands per tech stack, plus the fallback rules when no CLI exists or is unavailable.

## Official CLI Table

Prefer the official scaffold CLI for the chosen stack. Run it into the confirmed target directory. **Never invent a CLI command** — if the stack is not in this table, use the fallback.

| Stack | Official CLI | Notes |
|-------|--------------|-------|
| TypeScript / React (Vite) | `npm create vite@latest <dir> -- --template react-ts` | Also: `react`, `vue`, `vanilla-ts` templates |
| Vue (Vite) | `npm create vite@latest <dir> -- --template vue-ts` | |
| Next.js | `npx create-next-app@latest <dir> --typescript --eslint --app` | `--app` for App Router; `--no-*` flags per preference |
| Node / Express | `npm init -y` + install `express` | Manual; run health check after |
| Rust | `cargo new <dir>` | Built-in test: `cargo test` |
| Go | `go mod init <module>` | No scaffold command; manual layout + `go build ./...` |
| Python (FastAPI) | `uv init <dir> --package` | Requires `uv`; else `pip` + manual |
| Python (Django) | `django-admin startproject <name> <dir>` | |
| Python (general) | `uv init <dir>` | Or `python -m venv` + manual |
| Deno | `deno init <dir>` | Built-in: `deno test`, `deno check` |
| Bun | `bun init` | Creates minimal `index.ts` |
| Svelte | `npm create svelte@latest <dir>` | |
| Angular | `npx @angular/cli new <dir> --style=css` | |
| .NET | `dotnet new console` / `dotnet new web` | `dotnet test` |

## Fallback Rule

If the chosen stack has no official CLI (or the CLI is unavailable in the environment):

1. Copy the minimal TypeScript template from `develop-new/templates/minimal-ts/` into the target directory.
2. Rename `package.json`'s `name` field to the project name.
3. Proceed with scaffolding normally (create `src/<module>/` placeholders, health check).

The minimal template is the **only** in-repo scaffold. Do not hand-roll other stacks without an explicit user request.

## Scaffold Health Check

After scaffolding, prove the skeleton is healthy before wiring:

| Stack | Health command |
|-------|----------------|
| Node/TS (Vite, Next, minimal-ts) | `npm install && npm test && npm run typecheck` |
| Rust | `cargo test` |
| Go | `go build ./...` |
| Python (uv) | `uv sync && uv run pytest` (or stack-appropriate) |
| Deno | `deno check && deno test` |

If the health check fails: diagnose the scaffold, fix it, re-run. **3 consecutive failures → force pause** and report.

## Recording

After the health check, record the actual stack and commands into the scaffold state file:

```yaml
tech_stack:
  language: TypeScript
  framework: React
  runtime: Node.js
  package_manager: npm
commands:
  test: npm test
  typecheck: npm run typecheck
  lint: npm run typecheck
  build: npm run build
```

These feed `devloop/config.yaml` during wiring.
