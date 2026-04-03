# Hermes/Pi Dev Team — Portable Setup Guide

## What This Is

A portable AI dev team that can be dropped into any project directory. Hermes orchestrates, Pi codes, Beads tracks work. You manage via Telegram or CLI.

## Architecture

```
~/.hermes/                          # Global Hermes config (already exists)
  config.yaml                       # MCP servers, model config, memory
  mcp-servers/pi-agent/             # Pi MCP server (already installed)
  skills/dev-team/                  # Dev team skills (orchestration brain)
    work-loop/SKILL.md              # Main loop: bd ready → Pi → land the plane
    vibe-loop/SKILL.md              # Full pipeline: idea → research → stories → code
    escalation-handler/SKILL.md     # Routes failures to resolution paths
    telegram-dispatch/SKILL.md      # Parses Telegram commands into work
    model-tier-classifier/SKILL.md  # Assigns model tier per story complexity

~/.pi/agents/                       # Global Pi agent definitions
  tdd-coder.md                      # TDD implementation agent
  quinn-validator.md                # Full suite validation agent (read-only)
  failure-classifier.md             # Failure analysis agent (read-only)

YOUR PROJECT/                       # Per-project (what YOU create)
  AGENTS.md                         # Project conventions Pi reads on startup
  .beads/*.db                       # Beads database (bd init --prefix YourProject)
  docs/stories/*.md                 # Story files with AC + tasks + dev notes
  src/__tests__/                    # TDD test files (written BEFORE implementation)
```

## Setup a New Project for Hermes/Pi

### 1. Initialize Beads (if not already)

```bash
cd /path/to/your/project
bd init --prefix YourProject
```

### 2. Create AGENTS.md

This is the ONLY file Hermes/Pi need per-project. It tells Pi:
- What the tech stack is
- Where files live
- What conventions to follow
- What the current work is

Template:

```markdown
# YourProject — Agent Context

## What This Project Is
[One paragraph — what does this app do?]

## Architecture
- **Backend:** [framework, language, patterns]
- **Frontend:** [framework, components]
- **Database:** [type, access pattern — ORM or raw SQL?]
- **Testing:** [framework, location, patterns]
- **Task tracking:** Beads (bd CLI, prefix: YourProject)

## Conventions
- **Services:** [naming pattern, location]
- **Tests:** [framework, mocking approach, file location]
- **Migrations:** [naming pattern, location, next available number]
- **API routes:** [pattern, auth requirements]

## Project Structure
[Tree showing key directories Pi will touch]

## Current Work
- Stories: [path to story files]
- Tests: [path to test files]
- Architecture: [path to architecture doc, if any]
```

### 3. Create Stories as Beads Issues

#### From Story Files (recommended)

Write story markdown files with the BMAD story template, then file them as Beads issues:

```bash
# Single issue
bd create "Story-1.1: Feature Name" \
  --type feature \
  --priority P0 \
  -l lane-a,epic-1 \
  -d "story_file=docs/stories/1.1.feature-name.md | test_file=src/__tests__/feature.test.ts | budget_usd=2.00"

# Batch from markdown
bd create --file docs/stories/beads-import.md
```

#### Batch Import Format

Create a markdown file with this structure:

```markdown
# Epic 1: Your Epic Name

## Story-1.1: First Story Title
- type: feature
- priority: P0
- labels: epic-1, lane-a
- notes: story_file=docs/stories/1.1.first-story.md | test_file=src/__tests__/first.test.ts | budget_usd=2.00

Description of what this story implements.

## Story-1.2: Second Story Title
- type: feature
- priority: P1
- labels: epic-1, lane-a
- deps: Story-1.1
- notes: story_file=docs/stories/1.2.second-story.md | test_file=TBD | budget_usd=2.00

Description of what this story implements. Depends on 1.1.
```

Then import: `bd create --file docs/stories/beads-import.md`

After import, fix priorities and labels (batch import may not parse all fields):

```bash
bd update <id> --priority P0 --type feature --set-labels epic-1,lane-a
```

### 4. Write TDD Test Files FIRST

Hermes/Pi use TDD — tests exist before implementation. The pre-flight check (work-loop Step 3) requires `test_file` to exist on disk.

Write test files that:
- Import services/components that DON'T EXIST YET
- Define expected behavior via assertions
- Use the project's existing test patterns (mocking, assertions, structure)
- Will FAIL with "Failed to resolve import" until Pi implements the code

### 5. Set Up Parallel Lanes (optional)

If you have stories that can run in parallel, document file conflicts:

```markdown
# Parallel Execution Map

## Lanes
- Lane A: Epic 1 stories (sequential within lane)
- Lane B: Epic 2 stories (independent from Lane A)

## Gate Dependencies
- Story-1.1 must complete before any lane starts
- Story-2.1 gates Lane B

## File Conflict Matrix
| Story A | Story B | Shared File | Resolution |
|---------|---------|-------------|------------|
| 1.3 | 2.1 | SharedService.ts | 1.3 first |

## Migration Number Allocation
| Range | Lane | Stories |
|-------|------|---------|
| 001-010 | A | Epic 1 |
| 011-020 | B | Epic 2 |
```

### 6. Launch Hermes

```bash
cd /path/to/your/project

# All ready stories — loops until done
hermes chat -s dev-team/work-loop --yolo \
  -q "Run bd ready --json. Execute ALL ready stories. Keep looping until bd ready returns zero."

# Single story
hermes chat -s dev-team/work-loop --yolo \
  -q "Execute story <beads-id>."

# Via Telegram (if gateway running)
# Send: "run ready" or "<beads-id>"
```

## How the Work Loop Runs

```
Hermes starts
  ↓
Step 1a: STACK DETECT → auto-detect test runner, build tools from package.json
  → writes <!-- BEGIN STACK DETECT --> block to AGENTS.md
  → returns {test_cmd}, {test_single_cmd} for all subsequent steps
  → greenfield (no package.json)? → skip, use CI=true npm test fallback
  ↓
Step 1b: HEALTH CHECK → run lint + tsc
  → errors? → invoke health-fix skill (progress-based, web research, learns from fixes)
  → clean? → continue
  ↓
Step 2: QUEUE → bd ready --json → find unblocked stories
  → Hermes IS Bob's agent — "Bob Banks"/"bob" assignees are YOUR claims
  ↓
Per story:
  3. CLAIM     → bd update <id> --claim
  4. VALIDATE  → story_file exists? test_file exists?
  5. CONTEXT   → read story + checkpoints + failed approaches
  6. DISPATCH  → pi -q "..." --yolo (CLI, not MCP — process isolation)
     ↓
     Pi (tdd-coder):
       - Reads AGENTS.md for conventions + stack info
       - Implements code to make tests pass
       - Runs story's test file with {test_single_cmd} (auto-detected, NOT hardcoded)
       - Iterates until PASS (progress-based, no arbitrary limit)
     ↓
  7. EVALUATE  → Pi says PASS?
                  → CROSS-CHECK: Hermes re-runs {test_single_cmd} independently
                  → Cross-check PASS? Continue. FAIL? Send Pi back with correct runner.
                  → Pi FAIL? Retry with escalation chain:
                  different approach → Opus → web research → decompose
                  → Deep Research & Rearchitect (autonomous, no human dead end)
  8. LAND      → targeted regression (closed stories' test files only, NOT full suite)
                → git commit → bd close → git push → report
  9. EPIC CHECK → all stories in epic closed? → targeted epic test suite
  9a. BLOCKER REVISIT → any blocked stories from this session?
                → re-run tests (side-effect fixes may have resolved them)
                → still failing? → Opus → Deep Research & Rearchitect
  9b. DEEP RESEARCH → root cause archaeology, web search (GitHub issues, SO,
                changelogs), challenge assumptions, alternative architecture,
                prototype in isolation, apply via Opus
  10. LOOP     → bd ready again → next story → repeat until done
```

**Key design decisions:**
- **Stack-detect before everything** — auto-detects test runner from package.json/configs. Pi never guesses. CRA/CRACO projects get `CI=true` prefix to prevent watch mode hangs.
- **Pi via CLI** — each story is a fresh `pi -q` process. If Pi crashes, Hermes is unaffected.
- **Cross-check Pi's PASS** — Hermes independently re-runs tests after Pi claims success. Catches wrong-runner results.
- **Targeted regression** — only closed stories' test files, not full suite. Prevents 30+ minute timeouts on large codebases.
- **Blocker revisit** — after all ready stories done, retries blocked stories. Side-effect fixes from other stories may have resolved blockers.
- **No human dead ends** — Deep Research & Rearchitect is the final autonomous tier. Bob is not a developer — never escalate code issues to him.
- **Quinn is a hard gate** — Quinn adversarial review (Phase 10c) runs ONCE after all stories complete, not per-story. It's mandatory — vibe-loop cannot declare completion without it. Three parallel layers: Blind Hunter, Edge Case Hunter, Acceptance Auditor. P0/P1 findings are auto-filed as beads and fixed before push.
- **Progress-based retries** — no "3 strikes" limit. Keeps going while making progress. Escalates (model upgrade, web research, Deep Research) when stalled.
- **Claim identity** — Hermes runs as Bob's git identity. "Bob Banks" assignees are Hermes's own claims.

## Failure Handling

Progress-based detection (no arbitrary iteration limits):

```
Pi stalled (same test count for 2 retries)
  ↓
Escalation chain:
  1. Try different approach
  2. Escalate model: Sonnet → Opus
  3. Web search the error → feed results to Pi
  4. Decompose: split by file, attack individually
  5. Deep research: official docs, changelogs, known issues
  6. Deep Research & Rearchitect: root cause archaeology, targeted web search
     (GitHub issues, Stack Overflow, changelogs, migration guides), challenge
     every assumption, propose 2-3 alternative architectures, prototype in
     isolation, apply proven fix via Opus
  7. P0 blocker Beads issue with ALL accumulated research, prototype results,
     and failed approaches — tagged needs-deep-research-round-2 so next
     session continues from findings (not from scratch)
```

```
failure-classifier (after all strategies exhausted):
  STORY_AMBIGUITY    → route to BMAD for story rewrite
  MISSING_DEPENDENCY → create prerequisite Beads issue
  TEST_MISMATCH      → file Beads issue (do NOT edit tests)
  HARD_PROBLEM       → Deep Research & Rearchitect (NOT Bob — he's not a developer)
  INFRA              → auto-fix attempt → Deep Research & Rearchitect (NOT Bob)
```

**No human dead ends.** Every path resolves autonomously. When a blocker survives Deep Research, it's filed with full context so the next session's Deep Research continues from accumulated knowledge, not from zero.

## Budget Controls

| Tier | Scope | Default | What Happens on Exceed |
|------|-------|---------|----------------------|
| Per-story | Single Pi session | $2.00 | Pi session aborted, escalated |
| Per-work-loop | All stories in one run | $10.00 | Stop picking new stories |
| Daily | All activity | $5.00 | Telegram alert |

Set via environment variables:
```bash
export STORY_BUDGET_USD=2.00
export WORK_LOOP_BUDGET_USD=10.00
```

## Graduated Autonomy

| Phase | Behavior | When to Use |
|-------|----------|-------------|
| `manual` | Telegram approval before every commit | New project, building trust |
| `smart` | Auto-land familiar patterns, ask on novel | After 10+ successful stories |
| `off` | Auto-land everything, notify only | Mature project, high confidence |

Set: `export APPROVALS_MODE=manual`

**Recommended for autonomous runs:** Set `APPROVALS_MODE: "off"` in `~/.hermes/config.yaml` so `--yolo` sessions auto-land without waiting for Telegram approval. Override per-run with `APPROVALS_MODE=manual hermes chat ...` when you want gated commits.

## Required Config (in `~/.hermes/config.yaml`)

These settings were discovered through production debugging and are required for reliable work-loop execution:

```yaml
APPROVALS_MODE: "off"          # Auto-land in --yolo mode (no Telegram gate)
terminal:
  timeout: 1800                # 30 min safety net for zombie processes
                               # (retry loop handles all other cases)
```

**Why `timeout: 1800`?** Pi sessions need 10-20 min per story. The default 180s killed every run. The work-loop's progress-based retry handles stalled sessions — the timeout is only a safety net for truly hung processes.

**Why `APPROVALS_MODE: "off"`?** `--yolo` bypasses Hermes's tool approval prompts but NOT the work-loop's skill-level approval gate. Without this, Hermes waits for Telegram approval that never comes in CLI mode.

## Telegram Commands (when gateway running)

| Message | Action |
|---------|--------|
| `<beads-id>` | Execute that specific story |
| `<id1> <id2>` | Parallel execution of both |
| `run ready` | Execute all ready stories |
| `status` | Report all in-progress stories |
| `stop <id>` | Abort and checkpoint |

## Troubleshooting

### Pi can't find project files
Make sure you launched Hermes from the project directory: `cd /path/to/project && hermes chat ...`

### Tests fail with "Failed to resolve import"
This is CORRECT for TDD — the service doesn't exist yet. Pi needs to create it.

### Hermes doesn't pick up stories
Run `bd ready` manually. If no issues appear, check:
- Are issues status=open? (`bd list --status=open`)
- Do dependencies block them? (`bd show <id>`)
- Is the label filter matching? (`bd ready -l your-label`)

### Pi writes code but tests still fail
The test defines the contract. Pi should NOT modify tests. If the test is wrong, file a Beads issue — don't have Pi edit the test file.

### Hermes stops after one story
The work-loop should keep looping. If it exits after one story, restart with explicit loop instruction: `"Execute ALL ready stories. Keep looping until bd ready returns zero."`

### Full test suite shows failures from other stories
This is expected. Tests for stories that haven't been implemented yet WILL fail. Hermes only treats failures in CLOSED stories' tests as regressions. Open story failures are ignored.

### Pi crashes during a story
Pi runs as a CLI child process. If it crashes, Hermes is unaffected — it can retry or escalate. No MCP worker thread issues.

## Two Modes: Work Loop vs Vibe Loop

The dev team has two entry points:

| Mode | Skill | What It Does | When to Use |
|------|-------|--------------|-------------|
| **Vibe Loop** | `dev-team/vibe-loop` | THE loop — handles greenfield AND brownfield | **Default for everything.** Single entry point. |
| **Work Loop** | `dev-team/work-loop` | ⚠ DEPRECATED — internal engine only | Called by vibe-loop as Phase 10. Do NOT invoke directly — it skips Quinn review and phases 10b-13. |

Vibe-loop is the successor to work-loop. It wraps work-loop as its Phase 10 execution engine while adding discovery, project immersion, and validation phases around it.

**Vibe Loop pipeline (15 phases):**
```
GREENFIELD:                              BROWNFIELD:
Phase 0:  Analyst (research/validate)    Phase 0:  AUTO-SKIPPED
Phase 1:  Capture idea statement         Phase 1:  Capture task description
Phase 2:  Project scaffold               Phase 2:  Deep Project Immersion
                                                   (read project-context.md, deep pattern
                                                    scan, inject rules into AGENTS.md)
Phase 3:  Product brief                  Phase 3:  Feature brief (lean)
Phase 4:  Full PRD                       Phase 4:  Feature spec (lean)
Phase 5:  Architecture design            Phase 5:  Integration architecture ("how does this fit?")
Phase 6:  Epics & stories breakdown      Phase 6:  (same)
Phase 7:  Story specs + TDD tests        Phase 7:  Stories with embedded Coding Rules
Phase 8:  File Beads issues              Phase 8:  (same)
Phase 9:  Checkpoint & handoff           Phase 9:  (same)
Phase 10: Work-loop execution            Phase 10: (same)
                                         Phase 10b: Pattern Capture (update project-context.md)
                                         Phase 10c: Quinn Adversarial Review (MANDATORY HARD GATE)
                                                    3 parallel layers → triage → file beads → fix P0/P1
Phase 11: E2E validation                 Phase 11: (same + integration check)
Phase 12: Deploy to Railway              Phase 12: (same)
Phase 13: Completion report              Phase 13: (same)
```

**Brownfield — the key difference:** Before writing any code, vibe-loop becomes **intimate** with the project. Phase 2 reads actual service files, route files, test files to understand HOW the project works — not just what files exist. It produces a reuse-first Pattern Playbook and injects critical rules into AGENTS.md so Pi follows them. Only creates new code when no existing pattern can be reused.

**Four ways to use vibe-loop:**

```bash
# 1. Greenfield — you have an idea, research and build it
hermes chat -s dev-team/vibe-loop --yolo \
  -q "Build a meal prep timer with React and Express"

# 2. Greenfield — go idea hunting
hermes chat -s dev-team/vibe-loop \
  -q "Find me an app idea in the fitness space"

# 3. Brownfield — add a feature to existing project
cd /media/bob/I/AI_Projects/Crispi-app
hermes chat -s dev-team/vibe-loop --yolo --model anthropic/claude-sonnet-4-6 \
  -q "Add social sharing to Crispi"

# 4. Brownfield — infra/migration work on existing project
cd /media/bob/I/AI_Projects/Crispi-app
hermes chat -s dev-team/vibe-loop --yolo --model anthropic/claude-sonnet-4-6 \
  -q "Migrate Crispi from Lambda to Railway containers"
```

**Brownfield auto-detection:** If `package.json` + `AGENTS.md` + `.beads/` exist in the working directory, vibe-loop auto-detects brownfield and skips the analyst phase. No env var needed.

**Redundancy detection:** If you've already done some phases manually (wrote a PRD with John, ran the architect), vibe-loop detects existing artifacts and asks whether to use them or regenerate. In `--yolo` mode it auto-skips redundant phases.

### Resuming from Existing Artifacts

Vibe-loop looks for artifacts at specific `_output/` paths. If your existing docs are elsewhere (e.g., `_bmad-output/planning-artifacts/prd.md` from BMAD), Hermes won't find them and will regenerate.

**Two options to resume from where you left off:**

```bash
# Option 1: Tell Hermes where artifacts are in the prompt
hermes chat -s dev-team/vibe-loop --yolo \
  -q "PRD is at _bmad-output/planning-artifacts/prd.md. \
      Architecture is at _bmad-output/planning-artifacts/architecture.md. \
      Skip to story creation (Phase 7) and continue from there."

# Option 2: Symlink artifacts to expected paths
mkdir -p _output
ln -s ../_bmad-output/planning-artifacts/prd.md _output/prd.md
ln -s ../_bmad-output/planning-artifacts/architecture.md _output/architecture.md
hermes chat -s dev-team/vibe-loop --yolo -q "Build feature X"
```

**Option 1 is recommended** — explicit is better than implicit. Hermes reads the prompt, finds the files, and picks up from the right phase.

**Expected artifact paths and their phases:**

| Phase | Expected path | What it is |
|-------|--------------|------------|
| 0 | `_output/analyst-report.md` | Market research |
| 1 | `_output/idea-statement.md` | Idea capture |
| 3 | `_output/product-brief.md` or `_output/feature-brief.md` | Brief |
| 4 | `_output/prd.md` or `_output/feature-spec.md` | PRD / spec |
| 5 | `_output/architecture.md` | Architecture design |
| 6 | `_output/epics.md` | Epic breakdown |
| 7 | `docs/stories/*.md` + test files | Story specs + TDD tests |
| 8 | Beads issues with matching labels | Filed work items |

```bash
# Work loop — execute existing stories only
# IMPORTANT: Use --model to prevent smart routing from using a cheap model
# that runs out of context mid-session
hermes chat -s dev-team/work-loop --yolo --model anthropic/claude-sonnet-4-6
```

## Launching the Dev Team

The dev team is a set of skills loaded on top of Hermes via `-s`. Without a skill flag, `hermes chat` is just a vanilla agent.

### Skill Architecture

```
~/.hermes/skills/dev-team/
  ├── work-loop/             ← Execute pre-written stories (load with -s)
  ├── vibe-loop/             ← Full idea-to-code pipeline (load with -s)
  ├── health-fix/            ← Progress-aware error resolution + solution learning
  ├── escalation-handler/    ← Called internally when stories fail
  ├── telegram-dispatch/     ← Called internally for Telegram commands
  └── model-tier-classifier/ ← Called internally to pick model tier
```

You load either `work-loop` or `vibe-loop` — the other 4 are internal support skills called by the main loops.

**Health-fix** deserves special mention — it fixes lint/TS/build errors using progress-based monitoring (no arbitrary iteration limits), escalates from Sonnet → Opus → web research → decomposition, and **learns from every fix**. Solutions are saved as reusable skills so the same error is instantly resolved next time.

### Single Project

```bash
# Interactive — you talk to it, approve commits
cd /path/to/your/project
hermes chat -s dev-team/work-loop --model anthropic/claude-sonnet-4-6

# Fully autonomous — no approval prompts
hermes chat -s dev-team/work-loop --yolo --model anthropic/claude-sonnet-4-6

# Autonomous with a specific starting command
hermes chat -s dev-team/work-loop --yolo --model anthropic/claude-sonnet-4-6 \
  -q "Run bd ready --json. Execute ALL ready stories. Keep looping until bd ready returns zero."

# Isolated in a git worktree (for parallel agents on same repo)
hermes chat -s dev-team/work-loop --yolo --model anthropic/claude-sonnet-4-6 --worktree
```

**IMPORTANT:** Always use `--model anthropic/claude-sonnet-4-6` for work-loop sessions. Without it, smart model routing may select a cheap model (Gemini Flash) that runs out of context mid-session and exits after 1-2 stories.

### Multiple Projects in Parallel

Each project gets its own terminal and Hermes instance. No conflicts — separate repos, separate Beads DBs, separate file systems.

```bash
# Terminal 1
cd /media/bob/I/AI_Projects/Crispi-app
hermes chat -s dev-team/work-loop --yolo --model anthropic/claude-sonnet-4-6

# Terminal 2
cd /media/bob/I/AI_Projects/FlowInCash
hermes chat -s dev-team/work-loop --yolo

# Terminal 3
cd /media/bob/I/AI_Projects/LivingApp-Platform
hermes chat -s dev-team/work-loop --yolo --model anthropic/claude-sonnet-4-6
```

**Constraints for parallel runs:**
- **API rate limits** — multiple Pi sessions hit your provider concurrently
- **Cost** — each instance burns its own budget independently (3 teams = 3x spend)
- **Machine resources** — CPU/memory for concurrent sessions + test runners
- **Telegram routing** — if all report to the same chat, use project prefixes to tell them apart

### Parallel Tasks Within One Project

Use `--worktree` flag — Hermes auto-creates an isolated git worktree. No manual setup:

```bash
# Task 1: runs on main (or creates its own branch)
hermes chat -s dev-team/vibe-loop --yolo \
  -q "Build chat widget for Crispi"

# Task 2: runs in isolated worktree (parallel, no conflicts)
hermes chat -s dev-team/vibe-loop --yolo --worktree \
  -q "Add push notifications to Crispi PWA"
```

Each `--worktree` instance gets its own branch and working directory. Merge branches back to main when done.

**Manual worktrees** for lane-based parallelism (label-filtered):

```bash
git worktree add .worktrees/lane-b -b lane-b
git worktree add .worktrees/lane-c -b lane-c

cd .worktrees/lane-b && hermes chat -s dev-team/work-loop --yolo \
  -q "Read AGENTS.md. Run bd ready -l lane-b --json. Execute ready stories."

cd .worktrees/lane-c && hermes chat -s dev-team/work-loop --yolo \
  -q "Read AGENTS.md. Run bd ready -l lane-c --json. Execute ready stories."
```

Conflicts are typically small (route registrations, index files).

## File Reference

| File | Location | Purpose |
|------|----------|---------|
| Hermes config | `~/.hermes/config.yaml` | Global MCP, model, memory config |
| Pi CLI | `pi` (global install) | Coding agent — invoked as child process, not MCP |
| tdd-coder | `~/.pi/agents/tdd-coder.md` | Pi agent: implements code to make tests pass |
| failure-classifier | `~/.pi/agents/failure-classifier.md` | Pi agent: diagnoses failures |
| Quinn review | `bmad-code-review` skill | MANDATORY after every vibe-loop implementation (Phase 10c hard gate) |
| work-loop | `~/.hermes/skills/dev-team/work-loop/SKILL.md` | Hermes skill: story execution |
| vibe-loop | `~/.hermes/skills/dev-team/vibe-loop/SKILL.md` | Hermes skill: idea-to-code pipeline |
| stack-detect | `~/.hermes/skills/dev-team/stack-detect/SKILL.md` | Hermes skill: auto-detect project stack, write to AGENTS.md |
| health-fix | `~/.hermes/skills/dev-team/health-fix/SKILL.md` | Hermes skill: error resolution + learning |
| escalation-handler | `~/.hermes/skills/dev-team/escalation-handler/SKILL.md` | Hermes skill: failure routing (no human dead ends) |
| telegram-dispatch | `~/.hermes/skills/dev-team/telegram-dispatch/SKILL.md` | Hermes skill: Telegram commands |
| model-tier-classifier | `~/.hermes/skills/dev-team/model-tier-classifier/SKILL.md` | Hermes skill: model selection |
| AGENTS.md | `<project>/AGENTS.md` | Per-project context for Pi (stack block auto-generated by stack-detect) |
| Story files | `<project>/docs/stories/*.md` | Story specs |
| Beads DB | `<project>/.beads/*.db` | Issue tracking |
