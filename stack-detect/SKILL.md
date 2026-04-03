---
name: stack-detect
description: Auto-detect project software stack and write a machine-readable stack context block into AGENTS.md. Runs during work-loop health check so Pi always knows the correct test runner, build tool, and conventions.
version: 1.0.0
metadata:
  hermes:
    tags: [stack-detection, health-check, project-context, brownfield]
    related_skills: [dev-team/work-loop, dev-team/health-fix]
---

# Stack Detect — Auto-Detect Project Software Stack

Inspects the project's config files and package.json to determine the actual software stack, then writes (or updates) a `## Project Stack` section in AGENTS.md. This ensures Pi subagents always use the correct test runner, build commands, and conventions — no guessing.

## Trigger

Called by `dev-team/work-loop` Step 1 (Pre-Flight Health Check), **before** lint/TS checks. Runs once per work-loop session.

Also callable on-demand: `hermes chat -s dev-team/stack-detect`

## Input

- `{project_root}`: working directory (default: current directory)

## Process

### 0. Guard — Check Project Exists

```bash
test -f {project_root}/package.json
```

**If no `package.json` exists** (greenfield vibe-loop, pre-scaffold):
- Log: "stack-detect: no package.json found — skipping (greenfield project)"
- Do NOT write to AGENTS.md
- Return early with:

```json
{
  "status": "NO_STACK",
  "reason": "no package.json — project not yet scaffolded"
}
```

The calling skill (work-loop or vibe-loop) should treat `NO_STACK` as non-blocking and continue without stack context. Stack-detect will run again on the next work-loop invocation after the project is scaffolded.

---

### 1. Detect Package Manager

Check which lockfile exists:

| File | Package Manager | Install Command |
|------|----------------|-----------------|
| `pnpm-lock.yaml` | pnpm | `pnpm install` |
| `yarn.lock` | yarn | `yarn` |
| `package-lock.json` | npm | `npm install` |
| `bun.lockb` | bun | `bun install` |

### 2. Read package.json Scripts

```bash
cat {project_root}/package.json | jq '.scripts'
```

Extract the key commands:
- `test` script → **test command** (e.g., `craco test`, `jest`, `vitest`, `mocha`)
- `build` script → **build command**
- `start` / `dev` script → **dev server command**
- `lint` script → **lint command**

### 3. Detect Test Runner

Priority order — first match wins:

Priority order — first match wins:

| Check | Detection | Test Command |
|-------|-----------|--------------|
| `scripts.test` contains `jest` or `craco test` or `react-scripts test` | Jest (possibly via CRA/craco) | `CI=true npx jest` |
| `scripts.test` contains `vitest` | Vitest | `npx vitest run` |
| `scripts.test` contains `mocha` | Mocha | `npx mocha` |
| `vitest.config.*` exists | Vitest | `npx vitest run` |
| `jest.config.*` exists | Jest | `CI=true npx jest` |
| `.babelrc` or `babel.config.*` + `jest` in devDeps | Jest | `CI=true npx jest` |
| `karma.conf.*` exists | Karma | `npx karma start` |

### 3a. Detect Interactive Watch Mode (CRITICAL)

Some frameworks run tests in watch mode by default, which hangs in non-interactive environments (Pi subagents, CI). Detect and neutralize:

| Framework | Watch Mode Default? | Fix |
|-----------|-------------------|-----|
| Create React App (`react-scripts test`) | YES — watch mode by default | Prefix ALL test commands with `CI=true` |
| CRACO (`craco test`) | YES — inherits CRA behavior | Prefix ALL test commands with `CI=true` |
| Vitest | NO — `vitest run` is non-interactive | No fix needed (`vitest run` not `vitest`) |
| Jest (standalone) | NO — unless `--watch` flag | No fix needed |

**Rule:** If `react-scripts` OR `@craco/craco` is in dependencies, ALL test commands MUST be prefixed with `CI=true`. This includes the single-file command and the full-suite command. Without this prefix, the test process will hang indefinitely waiting for keyboard input.

Also detect:
- **Test file pattern**: check `jest.config` or `vitest.config` for `testMatch`/`include` patterns
- **Test environment**: check for `jsdom`, `happy-dom`, `node` in config
- **Setup files**: check for `setupFiles`, `setupFilesAfterFramework`, `globalSetup`

### 4. Detect Framework & Build Tool

| Check | Framework |
|-------|-----------|
| `react-scripts` in deps | Create React App |
| `@craco/craco` in deps | CRA + CRACO override |
| `next` in deps | Next.js |
| `vite` in deps | Vite |
| `webpack.config.*` exists | Webpack (manual) |
| `express` in deps | Express.js backend |
| `@aws-cdk` in deps | AWS CDK infra |

### 5. Detect Database & ORM

| Check | Database Tool |
|-------|---------------|
| `knex` in deps | Knex.js (check `knexfile` for migration path) |
| `prisma` in deps | Prisma (check `prisma/schema.prisma`) |
| `typeorm` in deps | TypeORM |
| `sequelize` in deps | Sequelize |
| `pg` in deps | PostgreSQL driver |
| `mysql2` in deps | MySQL driver |

For Knex: detect migration directory and naming convention by reading existing migrations.

### 6. Detect TypeScript Config

```bash
ls {project_root}/tsconfig*.json
```

Note which configs exist (e.g., `tsconfig.json`, `tsconfig.server.json`, `tsconfig.app.json`) and which is used for the build/typecheck.

### 7. Write Stack Block to AGENTS.md

Look for an existing `<!-- BEGIN STACK DETECT -->` / `<!-- END STACK DETECT -->` block in AGENTS.md. If found, replace it. If not, insert it **before** the first `## ` heading (so it's near the top).

**Template:**

```markdown
<!-- BEGIN STACK DETECT -->
## Project Stack (auto-detected by stack-detect)

> Auto-generated — do not edit manually. Re-run `stack-detect` to refresh.

| Category | Tool | Command |
|----------|------|---------|
| Package manager | {pkg_manager} | `{install_cmd}` |
| Test runner | {test_runner} | `{test_cmd}` |
| Test (single file) | {test_runner} | `{test_single_cmd} {file}` |
| Build | {build_tool} | `{build_cmd}` |
| Dev server | {dev_tool} | `{dev_cmd}` |
| Lint | {linter} | `{lint_cmd}` |
| TypeCheck | tsc | `{tsc_cmd}` |
| Database | {db_tool} | migrations in `{migration_path}` |
| Migration naming | {migration_pattern} | e.g. `YYYYMMDDHHMMSS_description.ts` |

### Key Conventions
- **Test file location:** `{test_pattern}`
- **Test environment:** `{test_env}` (jsdom/node/happy-dom)
- **Setup files:** `{setup_files}`
- **Config files:** `{tsconfig_list}`

### Pi Instructions
- Run single test file with: `{test_single_cmd} {file}`
- Run full suite with: `{full_test_cmd}`
- NEVER use `{wrong_test_runner}` — this project uses `{test_runner}`
- NEVER run bare `npm test` without CI=true if this is a CRA/CRACO project — it will hang in watch mode
- If CI=true is in the commands above, it is REQUIRED — do not omit it
<!-- END STACK DETECT -->
```

### 8. Verify

Read back the written block and confirm it parses correctly. Log the detected stack summary.

## Output

Returns a structured object to the calling skill:

```json
{
  "test_cmd": "CI=true npx jest",
  "test_single_cmd": "CI=true npx jest --testPathPattern",
  "build_cmd": "npm run build",
  "lint_cmd": "npx eslint src/**/*.ts src/**/*.tsx --quiet",
  "tsc_cmd": "npx tsc -p tsconfig.server.json --noEmit",
  "migration_path": "database/migrations/",
  "migration_pattern": "YYYYMMDDHHMMSS_description.js"
}
```

The work-loop uses `test_cmd` and `test_single_cmd` in Step 7 (Pi dispatch) instead of hardcoded runner names.

## Dependencies

- **External:** `jq` (for parsing package.json), `ls`, `cat`
- **Files read:** `package.json`, `*config*` files, existing migrations
- **Files written:** `AGENTS.md` (stack block only, between markers)
