# Project Detection

How to detect tech stack, commands, and module structure.

## Tech Stack Detection

Read these files in order. Stop at first match per category.

### Language and Runtime

| File | Language | Runtime |
|------|----------|---------|
| `package.json` | TypeScript / JavaScript | Node.js |
| `tsconfig.json` | TypeScript | Node.js / Bun / Deno |
| `pyproject.toml` | Python | Python |
| `requirements.txt` | Python | Python |
| `Cargo.toml` | Rust | Rust |
| `go.mod` | Go | Go |
| `pom.xml` | Java | JVM |
| `build.gradle` / `build.gradle.kts` | Java / Kotlin | JVM |
| `*.csproj` | C# | .NET |
| `Gemfile` | Ruby | Ruby |
| `composer.json` | PHP | PHP |

### Package Manager

| File / Lockfile | Package Manager |
|-----------------|-----------------|
| `pnpm-lock.yaml` | pnpm |
| `yarn.lock` | yarn |
| `package-lock.json` | npm |
| `bun.lockb` | bun |
| `uv.lock` | uv |
| `poetry.lock` | Poetry |
| `Cargo.lock` | Cargo |
| `go.sum` | Go modules |

### Framework Detection (from manifest content)

| Pattern in manifest | Framework |
|---------------------|-----------|
| `"react"` in dependencies | React |
| `"vue"` in dependencies | Vue |
| `"next"` in dependencies | Next.js |
| `"nuxt"` in dependencies | Nuxt |
| `"express"` in dependencies | Express |
| `"fastify"` in dependencies | Fastify |
| `"nestjs"` / `"@nestjs/core"` | NestJS |
| `"django"` in requirements | Django |
| `"fastapi"` in requirements | FastAPI |
| `"flask"` in requirements | Flask |

## Command Detection

### Test Command

Check in this order:
1. `scripts.test` in `package.json`
2. `[tool.pytest]` or `[tool.poetry.dependencies]` with pytest in `pyproject.toml`
3. `Makefile` target `test`
4. `Cargo.toml` (test is built-in: `cargo test`)
5. `go.mod` (test is built-in: `go test ./...`)
6. Fallback: ask user

### Lint Command

Check in this order:
1. `scripts.lint` in `package.json`
2. Presence of `.eslintrc*` / `eslint.config.*` → `eslint .`
3. `ruff` in `pyproject.toml` → `ruff check`
4. `clippy` in `Cargo.toml` → `cargo clippy`
5. Fallback: none detected

### Typecheck Command

Check in this order:
1. `scripts.typecheck` or `scripts.tsc` in `package.json`
2. `tsconfig.json` present → `tsc --noEmit`
3. `mypy` in `pyproject.toml` → `mypy .`
4. Fallback: none detected

### Build Command

Check in this order:
1. `scripts.build` in `package.json`
2. `Makefile` target `build`
3. `Cargo.toml` → `cargo build`
4. Fallback: none detected

### E2E Command

Check in this order:
1. `scripts.e2e` in `package.json`
2. Playwright config present → `npx playwright test`
3. Cypress config present → `npx cypress run`
4. Fallback: none detected

## Module Detection (L0 Architecture Map)

### What is a "module"

A module is a top-level directory under the source root that:
- Contains source code files.
- Has a clear responsibility.
- May depend on other modules.

### Source Root Detection

| Project Type | Source Root |
|--------------|-------------|
| Node.js / TypeScript | `src/`, `lib/`, `app/`, `server/`, `packages/*/src/` |
| Python | `src/`, package root directory |
| Rust | `src/` |
| Go | Root directory (Go packages) |
| Monorepo | Each `packages/*/` or `apps/*/` entry |

### Scan Process

1. Identify source root(s).
2. List immediate child directories.
3. For each directory:
   - Read entry files: `index.ts`, `main.ts`, `mod.rs`, `__init__.py`, `main.go`.
   - Extract exported functions, classes, and types from entry files.
   - Scan imports to detect dependencies on sibling modules.
   - Record one-line responsibility from README or first comment block.
4. Do NOT recurse more than 2 levels deep.
5. Do NOT read every file — only entry points and manifest.

### Module Index Entry Format

```yaml
modules:
  - name: auth
    path: src/auth/
    responsibility: "User authentication and session management"
    key_interfaces:
      - "AuthService.login()"
      - "SessionManager.create()"
    dependencies: [database, redis]
    has_spec: false
    last_scanned: "2026-08-01"
```

### Edge Cases

- **Empty project**: No modules found. Write empty module index and note in report.
- **Monorepo**: Treat each package/app as a top-level module. Scan each package's `src/` independently.
- **Flat structure**: If all code is in one directory, create a single module entry.
- **Config-heavy directories**: Skip directories that only contain config, assets, or scripts.

## CI/CD Detection

Read `.github/workflows/*.yml`, `.gitlab-ci.yml`, `Jenkinsfile`, or `circleci/config.yml`.

Record:
- CI platform.
- Key steps (test, lint, build, deploy).
- Whether spec verification could be added.

Do NOT modify CI config. Only note recommendations in the setup report.
