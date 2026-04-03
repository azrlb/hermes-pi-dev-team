---
name: vibe-loop
description: "THE dev loop — handles both greenfield (analyst → discovery → build) and brownfield (deep project immersion → lean spec → build). Replaces work-loop as the single entry point for all Hermes dev work."
version: 2.0.0
metadata:
  hermes:
    tags: [autonomous, vibe-coding, idea-to-code, pipeline, brownfield, greenfield]
    related_skills: [dev-team/work-loop, dev-team/escalation-handler]
---

# Vibe Loop — The One Loop

Autonomous pipeline for both greenfield and brownfield development. Single entry point for all Hermes dev work.

- **Greenfield**: Full pipeline — analyst research → idea validation → scaffold → discovery → stories → TDD tests → implementation → deploy
- **Brownfield**: Deep project immersion → lean feature spec → stories that reuse existing patterns → implementation → validation

The key difference: brownfield mode becomes **intimate** with the project before writing a single line. It reads project context, understands working patterns, and only creates new code when no existing pattern can be reused.

## Trigger

- **CLI (greenfield):** `hermes chat -s dev-team/vibe-loop --yolo -q "Build a meal prep timer with React and Express"`
- **CLI (brownfield):** `hermes chat -s dev-team/vibe-loop --yolo -q "Add social sharing to Crispi"`
- **Interactive:** `hermes chat -s dev-team/vibe-loop` then describe the idea or task
- **Telegram:** Send `/vibe Build a ...` or `/vibe Add feature X to ProjectY`

## Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `VIBE_SKIP_ANALYST` | `false` | Force-skip market research even for greenfield (brownfield auto-skips regardless) |
| `VIBE_AUTO_SCAFFOLD` | `true` | Auto-create project structure for greenfield |
| `HERMES_MAX_PARALLEL` | `1` | Max concurrent Pi subagent sessions in Phase 10 |

## Critical Rules

These rules apply to ALL phases, especially during implementation (Phase 10). No exceptions.

### Brownfield-First Development

**NEVER default to greenfield approaches when working in an existing project.**

Before writing ANY new code, Pi and all subagents must:
1. Check `_output/pattern-scan.md` (generated in Phase 2 for brownfield) for existing patterns
2. Search the codebase for similar implementations (`grep`, `glob` for related services/components)
3. Reuse existing utilities, helpers, shared code, and conventions
4. Only write new code when no existing pattern applies

When reusing patterns: adapt, don't copy-paste. Follow the existing code's style exactly — same naming, same error handling, same response format.

### No Unauthorized Refactoring

**NEVER refactor, rename, reorganize, or "improve" existing code without explicit user permission.**

- Do NOT clean up code adjacent to what you're changing
- Do NOT add docstrings, comments, or type annotations to code you didn't write
- Do NOT rename existing variables, functions, or files for consistency
- Do NOT restructure existing directories or move files around
- Do NOT upgrade dependencies or change build configs unless the story requires it

If existing code needs refactoring to support the new feature, **file a separate Beads issue** describing the refactor and why it's needed. Let the user decide.

### Additive-Only Changes

New features should be **additive** — extend the existing system, don't reshape it:
- Add new routes alongside existing ones (don't reorganize the router)
- Add new services following the existing service pattern (don't introduce new patterns)
- Add new tests matching the existing test structure (don't restructure the test suite)
- Add new database migrations (don't modify existing ones)

### Enforcing Rules in Subagents

Pi subagents (`tdd-coder`, `quinn-validator`) do NOT read this SKILL.md, project-context.md, or pattern-scan.md. Project knowledge reaches Pi through TWO channels:

**Channel 1 — AGENTS.md (always read by Pi):**
- immersion (Phase 2, brownfield) injects a `<!-- BEGIN PROJECT RULES -->` block into AGENTS.md
- Contains: Critical Don't-Miss Rules, Anti-Patterns, Security Non-Negotiables
- Pi reads this automatically on every invocation

**Channel 2 — Story file Dev Notes (read per-story):**
- story-specs (Phase 7a) embeds a `## Coding Rules` section into every brownfield story's Dev Notes
- Contains: concrete file paths to follow, services to reuse, story-specific do-NOTs
- The work-loop's Build Context step (dev / Phase 10, step 5) reads the story file — rules flow to Pi automatically

Both channels are required. AGENTS.md provides project-wide rules; story Dev Notes provide task-specific guidance with concrete file paths.

---

## Phase Name Reference

Every phase has both a number (for backward compat) and a BMAD agent name (for human readability). Both work in prompts — "start at dev" and "start at phase 10" mean the same thing.

| Number | BMAD Name | Agent Role | Greenfield | Brownfield |
|--------|-----------|-----------|-----------|-----------|
| Phase 0 | `analyst` | Mary — market research & validation | runs | auto-skipped |
| Phase 1 | `brief-capture` | PM — captures idea/task statement | runs | runs |
| Phase 2 | `immersion` | Architect — deep project immersion + pattern scan | scaffold | brownfield deep dive |
| Phase 3 | `product-brief` | PM — writes brief | product-brief | feature-brief (lean) |
| Phase 4 | `prd` | PM — writes spec | full PRD | feature spec (lean) |
| Phase 5 | `architecture` | Architect — designs solution | full architecture | integration architecture |
| Phase 6 | `epics` | SM — breaks into epics & stories | runs | runs |
| Phase 7a | `story-specs` | SM — writes story specs with AC, tasks, dev notes | runs | runs + Coding Rules injection |
| Phase 7b | `tdd` | QA — writes failing TDD test files from specs | runs | runs |
| Phase 8 | `beads-filing` | SM — files beads issues with deps | runs | runs |
| Phase 9 | `checkpoint` | SM — checkpoint & handoff to dev | runs | runs |
| Phase 10 | `dev` | Pi — codes to make tests pass (work-loop) | runs | runs |
| Phase 10b | `pattern-capture` | Brownfield-enforcer — updates project-context.md | skipped | runs |
| Phase 10c | `quinn-review` | QA — adversarial review (MANDATORY HARD GATE) | runs | runs |
| Phase 11 | `e2e-validation` | QA — end-to-end validation | runs | runs + integration check |
| Phase 12 | `deploy` | DevOps — deploy to Railway | runs | preview deploy |
| Phase 13 | `report` | Tech-writer — completion report | runs | runs |

**Prompt examples using BMAD names:**
- `hermes chat -s dev-team/vibe-loop --yolo -q "Add X to Crispi. Start at dev."`
- `hermes chat -s dev-team/vibe-loop --yolo -q "Build X. Start at tdd."`
- `hermes chat -s dev-team/vibe-loop --yolo -q "Build X. Start at architecture."`
- `hermes chat -s dev-team/vibe-loop --yolo -q "Run quinn-review on current branch."`

---

## Pipeline

### Completion Gate — HARD REQUIREMENT

**You are NOT done until Phase 10c / quinn-review has run.** This is a hard gate, not a suggestion. Implementation alone (Phase 10) is NOT completion. The pipeline minimum is:

**dev (Phase 10) → pattern-capture (Phase 10b) → quinn-review (Phase 10c)**

If you are about to output a final summary or declare the task complete, STOP and check:
1. Did you run `git diff main...HEAD` and invoke the three Quinn review layers?
2. Did you file findings as beads issues and fix P0/P1s?
3. Did you output the Quinn findings report?

If the answer to ANY of these is NO, you are not done. Continue to quinn-review (Phase 10c) before responding.

**This gate applies in ALL modes:** interactive, `-q` single-query, `--yolo`, and Telegram. No exceptions.

### Yolo Mode — No Human Gates

**⚠ CRITICAL RULE — READ THIS BEFORE EVERY PHASE TRANSITION:**

**When invoked with `--yolo`, the entire pipeline runs end-to-end with ZERO human gates.** Do NOT pause between phases to ask "want to continue?" or "ready to proceed?" or present a summary and wait. Do NOT end your turn with a question. Do NOT output a summary and stop. After completing any phase, IMMEDIATELY call your next tool to begin the next phase. The only acceptable halts in yolo mode are:
- Idea hunting selection (Phase 0 — user must pick which idea to build)
- Unrecoverable errors (tool missing, git push failed after retry)
- Secrets needed for deployment (Phase 12 — never auto-fill API keys)

**VIOLATION EXAMPLES (never do these in --yolo):**
- ❌ "Want me to start the work-loop now, or review the stories first?"
- ❌ "Ready to kick off Phase 10 whenever you say go."
- ❌ "Discovery complete. [summary]. What would you like to do next?"
- ✅ "Discovery complete. Starting Phase 10..." [immediately call work-loop]

Everything else flows continuously: discovery → implementation → validation → deploy. If you are about to output text without a tool call after it, and you are NOT at an acceptable halt condition above, you are violating this rule.

### Phase Transition Rules

Before starting any phase (except Phase 0/1), validate the previous phase's output:
- Check the output file exists on disk and is non-empty
- If the file exists but is under 50 bytes, treat it as corrupted — re-run the previous phase
- Log phase transitions to `_output/pipeline.log`: `[timestamp] Phase N → Phase N+1 | artifacts: {list}`

This prevents cascading failures from partial writes or crashes mid-phase.

At every phase transition:
1. Append to `_output/pipeline.log`:
   ```
   [{ISO timestamp}] Phase {N} complete → Phase {N+1} | artifacts: {comma-separated file list}
   ```
2. Send a progress notification (Telegram if available, otherwise log to stdout):
   ```
   Vibe Loop [{project}] — Phase {N} complete: {phase title}
   Next: Phase {N+1} — {next phase title}
   Elapsed: {time since start}
   ```

**`--yolo` passthrough:** When vibe-loop is invoked with `--yolo`, Phase 10 inherits this flag — work-loop runs fully autonomously (no approval prompts for commits/pushes).

### Redundancy Detection

Before each phase, check if its output artifacts already exist from a previous run or manual creation. If they do:

| Situation | Action |
|-----------|--------|
| `_output/hunt-results.md` exists when analyst (Phase 0) hunt starts | Ask: "Hunt results already exist ({N} candidates). **Pick from existing** / **Re-run hunt**?" In `--yolo` mode: present existing results for selection (still halts for pick). |
| `_output/analyst-report.md` exists when analyst (Phase 0) starts | Ask: "Analyst report already exists. **Use existing** / **Re-run research** / **Skip to brief-capture?**" |
| `_output/idea-statement.md` exists when brief-capture (Phase 1) starts | Ask: "Idea statement already exists. **Use existing** / **Revise**?" |
| `package.json` + `AGENTS.md` + `.beads/` exist when immersion (Phase 2) starts | Auto-detect brownfield — skip scaffold |
| `_output/product-brief.md` or `_output/feature-brief.md` exists when product-brief (Phase 3) starts | Ask: "Brief already exists. **Use existing** / **Regenerate**?" |
| `_output/prd.md` or `_output/feature-spec.md` exists when prd (Phase 4) starts | Ask: "Spec already exists. **Use existing** / **Regenerate**?" |
| `_output/architecture.md` exists when architecture (Phase 5) starts | Ask: "Architecture doc already exists. **Use existing** / **Regenerate**?" |
| `_output/epics.md` exists when epics (Phase 6) starts | Ask: "Epics already exist. **Use existing** / **Regenerate**?" |
| Story spec files exist when story-specs (Phase 7a) starts | Ask: "Story specs already exist ({N} stories). **Use existing** / **Regenerate all** / **Generate missing only**?" |
| Test files exist when tdd (Phase 7b) starts | Ask: "TDD test files already exist ({M} tests). **Use existing** / **Regenerate all** / **Generate missing only**?" |
| Beads issues with matching labels exist when beads-filing (Phase 8) starts | Ask: "Found {N} existing Beads issues with label '{prefix}'. **Use existing** / **Wipe and reimport**?" |
| Discovery commit exists in git log when checkpoint (Phase 9) starts | Auto-skip commit (already pushed). Proceed to dev (Phase 10). |
| `bd ready` returns issues when dev (Phase 10) starts | Auto-resume work-loop from where it left off. No prompt needed. |

**In `--yolo` mode:** Auto-select "Use existing" for all redundancy prompts — never regenerate unless the artifact is missing or corrupted. This keeps autonomous runs fast when resuming.

**Relevance check:** When reusing an existing artifact, read its first 200 characters and compare against the current idea statement. If the content appears to be about a completely different product (different name, different domain), warn: "Existing {artifact} appears to be for a different project. **Use anyway** / **Regenerate**?" In `--yolo` mode, regenerate if relevance check fails.

**Key principle:** Never redo work that's already done. If the user already wrote a PRD by hand or with a BMAD agent, the vibe loop should pick it up and keep going.

### Phase 0 / analyst — Research, Validate & Brainstorm (Greenfield Only)

Act as Mary (strategic business analyst). This phase has two modes depending on what the user provides.

**Skip conditions (checked in order):**
1. **Brownfield detected** (package.json + AGENTS.md or .beads/ exist in working directory) → auto-skip Phase 0 entirely. Log: `"Brownfield detected — skipping analyst, entering deep project immersion (Phase 2)."` Jump to Phase 1.
2. **`VIBE_SKIP_ANALYST` is `true`** → skip directly to Phase 1 (manual override for greenfield).

Brownfield projects do not need market research or idea validation — the project already exists. The analyst front-end is exclusively for greenfield where you're deciding WHAT to build.

**Mode detection:**
- User provided a specific idea (e.g., "Build a meal prep timer") → **Validation Mode** (Step 0a)
- User asked to find ideas (e.g., "find me an app idea", "research what's trending", "what should I build?") → **Idea Hunting Mode** (Step 0-hunt)
- User specified sources (e.g., "look on Reddit for pain points", "check Twitter for complaints about X") → **Targeted Hunting Mode** (Step 0-hunt with source filter)

---

#### Step 0-hunt: Idea Hunting (when no idea is provided)

Go foraging for product ideas using web search. The goal is to find real pain points that people are expressing *right now*.

**Step 0-hunt-a: Source Scanning**

Search these sources (or user-specified sources) for unmet needs and pain points:

1. **Reddit** — Search subreddits like r/SideProject, r/startups, r/Entrepreneur, r/webdev, r/AppIdeas, and domain-specific subs. Look for:
   - "I wish there was an app that..."
   - "Why doesn't X exist?"
   - "I'd pay for something that..."
   - Complaints about existing tools
2. **Twitter/X** — Search for frustration signals: "why is [tool] so bad", "there has to be a better way", "[category] apps suck"
3. **Hacker News** — Search "Ask HN" and "Show HN" for problems people are solving and gaps in existing solutions
4. **Product Hunt** — Check trending and recently launched products for categories with high engagement but low satisfaction
5. **App Store / Play Store reviews** — Search 1-2 star reviews of popular apps in a category for recurring complaints
6. **Google Trends** — Check rising search terms in tech, productivity, health, finance

**Source and domain filtering:**
- If user specified a **domain** (e.g., "fitness space") → search all sources but filter results to that domain
- If user specified **sources** (e.g., "look on Reddit") → only search the named sources, any domain
- If user specified both (e.g., "check Reddit for fitness app ideas") → filter both source and domain

**Step 0-hunt-b: Idea Extraction**

From the scan, extract 5-7 candidate ideas. For each:
- **Pain point** — What's the problem? (quote real user complaints when possible)
- **Existing solutions** — What do people use today? Why is it insufficient?
- **Idea sketch** — 1-sentence product concept that addresses the gap
- **Signal strength** — How many people are expressing this pain? (rough: isolated / recurring / widespread)

**Step 0-hunt-c: Scoring & Ranking**

Score each candidate on 3 dimensions (1-5 each):
1. **Problem severity** (1 = nice-to-have, 3 = real annoyance, 5 = hair-on-fire pain)
2. **Solution differentiation** (1 = exact clone exists, 3 = better UX/angle, 5 = novel approach)
3. **Feasibility** (1 = requires massive infra/data, 3 = moderate effort, 5 = weekend-hackable MVP)

Rank by total score. Present the **top 3** to the user:

```
Vibe Loop — Idea Hunting Results

1. [Score: 13/15] {Idea Name}
   Pain: {1-sentence pain point with source}
   Concept: {1-sentence product idea}
   Why it wins: {differentiation angle}

2. [Score: 11/15] {Idea Name}
   ...

3. [Score: 10/15] {Idea Name}
   ...

Which idea should we build? (1/2/3/none/refine/more)
```

Write the full candidate list and scores to `_output/hunt-results.md` before presenting to the user. This enables resumption if the pipeline crashes after hunting but before selection.

**HALT and wait for user selection.** If running via Telegram and no response after 24 hours, send a reminder: "Still waiting for your idea pick. Reply 1/2/3 to continue."

- If user picks one → set it as the idea, proceed to Step 0a (validation) with the chosen idea
- If user says "refine" → ask what to focus on, re-run hunt with narrower scope
- If user says "none" → ask what domain/direction to explore, re-run hunt
- If user says "more" → present remaining candidates (4-N, however many were extracted)

After selection, flow continues into validation (Step 0a) with the chosen idea, then into Phase 1.

---

**Step 0a: Market Landscape Scan** (Validation Mode — runs when an idea is provided or selected from hunting)

Using web search, research:
1. **Existing solutions** — What already exists in this space? Name 3-5 competitors or alternatives.
2. **Market size signals** — Is this a growing market? Look for recent funding, user growth, or trend data.
3. **Underserved gaps** — What do existing solutions do poorly? What are users complaining about? Check app store reviews, Reddit, forums.

Produce a competitive snapshot: for each competitor, note name, pricing model, key strength, key weakness.

**Step 0b: Idea Validation**

Score the idea across 3 dimensions (1-5 each):
1. **Problem severity** — How painful is the problem? (1 = nice-to-have, 5 = hair-on-fire)
2. **Solution differentiation** — Does this idea do something meaningfully different? (1 = clone, 5 = novel approach)
3. **Feasibility** — Can a solo dev or small team build an MVP? (1 = requires massive infra, 5 = weekend hackable)

**Validation gate:**
- Total score >= 9: Proceed with confidence
- Total score 6-8: Proceed but note risks in the brief
- Total score < 6: Telegram alert "Idea scored {score}/15 — weak validation. Proceeding with caution." Flag specific concerns.

Do NOT halt on a low score — build it anyway, but carry the risks forward so the PRD can scope defensively.

**Step 0c: Brainstorm Expansion**

Before locking the idea, push past the obvious:
1. Generate 10-15 feature ideas beyond what the user described — think adjacent use cases, monetization angles, viral hooks, power-user features
2. Identify 3 "what if" scenarios that could change the product direction (e.g., "what if this became a marketplace?", "what if mobile-first?")
3. Pick the 3-5 strongest brainstormed features and flag them as "consider for MVP" or "defer to v2"

**Step 0d: Analyst Report**

Compile everything into `_output/analyst-report.md`:

```markdown
# Analyst Report: {Idea Name}

## Market Landscape
{Competitive snapshot — 3-5 competitors with strengths/weaknesses}

## Validation Score: {N}/15
- Problem severity: {score}/5 — {rationale}
- Solution differentiation: {score}/5 — {rationale}
- Feasibility: {score}/5 — {rationale}

## Key Risks
{Bullet list of concerns from validation}

## Brainstorm Highlights
### Consider for MVP
{3-5 strongest feature ideas from brainstorm}

### Defer to V2
{Remaining interesting ideas}

## Recommendation
{1-2 sentence go/caution/pivot recommendation with reasoning}
```

**Output:** `_output/analyst-report.md`

---

### Phase 1 / brief-capture — Capture Idea

Parse the user's input query combined with the analyst report (if Phase 0 ran) as the product idea.

**If Phase 0 ran:** Incorporate the validation score, key risks, and MVP-candidate features from the analyst report into the idea statement. The analyst report becomes input context for all downstream phases.

**If the idea is clear** (names a product, describes core functionality, implies a tech stack):
- Produce a 1-paragraph idea statement summarizing: what it is, who it's for, what it does, and the implied tech stack.
- If analyst report exists, append: key differentiator (from validation), top risk, and MVP-candidate features from brainstorm.
- Proceed to Phase 2.

**If the idea is too vague** (e.g., "build something cool"):
- Ask up to 3 clarifying questions in a single message. Focus on:
  1. What does it DO? (core user action)
  2. Who is it FOR? (target user)
  3. Any tech preferences? (framework, language, platform)
- After one round of answers, produce the idea statement and proceed.
- If still unclear after clarification, send Telegram: "Vibe loop halted — idea too vague after clarification." and stop.

**Output:** `_output/idea-statement.md` — 1 paragraph + tech stack note + analyst context (if available).

---

### Phase 2 / immersion — Project Scaffold (Greenfield) / Deep Project Immersion (Brownfield)

Detect whether this is greenfield (new project) or brownfield (existing project).

**Pre-flight:** Verify required tools are installed before proceeding:
```bash
command -v bd >/dev/null 2>&1 || { echo "ERROR: bd (beads) CLI not found. Install it first."; exit 1; }
command -v git >/dev/null 2>&1 || { echo "ERROR: git not found."; exit 1; }
command -v npm >/dev/null 2>&1 || { echo "ERROR: npm not found."; exit 1; }
```
If any tool is missing, Telegram alert and halt.

**Brownfield detection:** Check if current working directory has:
- A `package.json` or equivalent project manifest
- An `AGENTS.md` file
- A `.beads/` directory
- Verify the project is the intended target: check `package.json` name field or git remote URL if available

**If brownfield (existing project) — Deep Project Immersion:**

The goal is to know this project as well as a senior dev who's been on it for months BEFORE writing any new code. This phase is the foundation everything else builds on.

**Step 2a. Create a feature branch:**
Derive slug from idea: lowercase, replace spaces/symbols with hyphens, strip non-alphanumeric, max 50 chars.
```bash
git checkout -b vibe/{idea-slug}
# e.g., "Add Social Sharing!" → vibe/add-social-sharing
```
All commits go on this branch. User merges to main when satisfied.

**Step 2b. Load project context:**

Read ALL project context docs (in this order — each builds on the previous):
1. `AGENTS.md` — stack, conventions, Pi instructions, stack-detect block
2. `CLAUDE.md` — project rules, beads workflow, landing protocol
3. `_bmad-output/project-context.md` — implementation rules, gotchas, anti-patterns:
   - If exists and < 7 days old → use as-is
   - If exists but stale (check `date:` in frontmatter) → regenerate via `bmad-generate-project-context`
   - If missing → generate via `bmad-generate-project-context`
4. `package.json` — deps, scripts, versions
5. Existing docs in `_bmad-output/` (prior PRDs, architecture docs, pattern scans)
6. `docs/` directory — any architecture docs, story templates, QA docs

Extract and carry forward into all subsequent phases:
- **Critical Don't-Miss Rules** (from project-context.md)
- **Anti-Patterns** (from project-context.md)
- **Security Non-Negotiables** (from project-context.md)
- **Stack-detect commands** (from AGENTS.md `<!-- BEGIN STACK DETECT -->` block)

**Step 2c. Deep pattern scan — understand HOW this project works:**

This is NOT a surface grep. Read actual source files to understand the project's working patterns:

1. **Services** — Read 2-3 representative service files end-to-end (e.g., the most recently modified ones in `src/server/services/`). Extract:
   - Class structure and constructor pattern
   - Method signatures and return types
   - Error handling approach
   - Database access pattern (Knex calls, query style)
   - Export pattern (singleton instance, static methods, etc.)

2. **Routes** — Read 2-3 representative route files in `src/server/routes/`. Extract:
   - Middleware chain (auth, validation, rate limiting)
   - Request/response patterns
   - Error response format
   - Route registration pattern

3. **Tests** — Read 2-3 representative test files in `src/server/__tests__/`. Extract:
   - Mock setup (what's mocked globally vs per-test)
   - Assertion style and patterns
   - Test organization (describe/it nesting)
   - Setup/teardown patterns

4. **Middleware** — Understand the middleware stack order and what each piece does (read `src/server/index.ts` route registration)

5. **Database** — Read 1-2 migration files and the DB config. Understand:
   - Migration naming convention
   - Table/column naming (snake_case)
   - Knex query patterns used

6. **Task-specific scan** — Search for existing code that touches the same domain as the current task:
   - `grep` and `glob` for related service names, route paths, config patterns
   - Check if similar work has been done before (e.g., for a Railway migration: search for Docker, deployment, health check, env config)
   - Identify files that will need modification vs files to create fresh

**Step 2d. Produce the Pattern Playbook (`_output/pattern-scan.md`):**

This is a **reuse-first playbook** — structured so downstream phases know exactly what to reuse and how:

```markdown
# Pattern Playbook — {Project Name}

## How This Project Works
{3-5 sentences capturing the essence: stack, architecture, patterns, key gotchas}

## Reusable Patterns (MUST use these — do not reinvent)

| Pattern | Example File | Structure | How to Reuse |
|---------|-------------|-----------|--------------|
| Service class | {actual path} | {class Name, singleton export, constructor pattern} | Copy structure, adapt methods |
| Route handler | {actual path} | {router, middleware chain, response format} | Copy chain, add new routes |
| Jest test | {actual path} | {mock setup, describe/it structure} | Copy mock boilerplate, adapt assertions |
| DB migration | {actual path} | {up/down, column naming} | Copy structure, new table/columns |
| Middleware | {actual path} | {pattern description} | Import and chain, don't recreate |

## Existing Code Relevant to This Task
- {file path} — {what it does, how the new work should interact with or extend it}
- {file path} — {reusable utility/helper that applies}

## Critical Rules (verbatim from project-context.md)
{Paste the "Critical Don't-Miss Rules" section — route registration order, security requirements, DB gotchas, build/deploy rules}

## Anti-Patterns (NEVER do these)
{Paste the "Anti-Patterns to Avoid" section verbatim}

## Task-Specific Findings
{What the domain-specific search found — existing implementations, related services, files to modify}
```

If the scan finds no reusable patterns (bare project with minimal code):
```markdown
# Pattern Playbook — {Project Name}
No existing patterns found. This is effectively a greenfield codebase within a brownfield project structure.
New code should establish conventions that future features will follow.
```

**Step 2e. Inject project rules into AGENTS.md:**

Add a `<!-- BEGIN PROJECT RULES -->` block to AGENTS.md (same injection pattern as stack-detect). This ensures Pi reads the rules automatically since Pi always reads AGENTS.md:

```markdown
<!-- BEGIN PROJECT RULES -->
## Critical Project Rules (auto-injected from project-context.md)

> Auto-generated by vibe-loop Phase 2. Do not edit manually.
> Source: _bmad-output/project-context.md
> Last updated: {date}

### Route Registration Order
{from project-context.md}

### Security — Non-Negotiable
{from project-context.md}

### Database Gotchas
{from project-context.md}

### Anti-Patterns
{from project-context.md}

### Build & Deploy Rules
{from project-context.md}
<!-- END PROJECT RULES -->
```

If a `<!-- BEGIN PROJECT RULES -->` block already exists, replace it. Never duplicate.

**Step 2f.** Skip scaffold creation. Proceed to Phase 3.

**If greenfield (new project) and `VIBE_AUTO_SCAFFOLD` is true:**
1. Create project directory if not in one: `mkdir -p {project-name} && cd {project-name}`
2. Initialize git: `git init`
3. Initialize project based on tech stack from idea statement:
   - **Node/TypeScript:** `npm init -y`, install deps (Express, React, etc.), `tsconfig.json` with strict mode
   - **Python:** `python -m venv venv`, `pip install` framework (Flask, FastAPI, Django), create `requirements.txt`
   - **Go:** `go mod init {module}`, create `main.go` skeleton
   - **Other:** Use the language's standard project init, install the framework from the idea statement
4. Install test framework matching the stack:
   - Node: `npm install --save-dev jest @types/jest ts-jest`
   - Python: `pip install pytest`
   - Go: built-in `testing` package (no install needed)
7. Initialize Beads: `bd init --prefix {ProjectName}`
8. Create `AGENTS.md` with:
   ```markdown
   # {ProjectName} — Agent Context

   ## What This Project Is
   {idea statement}

   ## Architecture
   - **Backend:** {from idea}
   - **Frontend:** {from idea}
   - **Database:** {from idea or TBD}
   - **Testing:** Jest (unit/integration)
   - **Task tracking:** Beads (bd CLI, prefix: {ProjectName})

   ## Conventions
   - Services: singleton pattern, `class FooService { } export const fooService = new FooService()`
   - Tests: Jest, `__tests__/` dirs colocated with source
   - Routes: `const router = express.Router()` → `export default router`
   - Response format: `{ success: boolean, data?: T, error?: string }`

   ## Current Work
   - Stories: `docs/stories/*.md`
   - Tests: `src/**/__tests__/*.test.ts`
   ```
9. **Scaffold verification** — confirm the project actually builds before moving on:
   - Node: `npm run build` or `npx tsc --noEmit` (should exit 0)
   - Python: `python -c "import {main_module}"` or `pytest --collect-only`
   - Go: `go build ./...`
   - If build fails: diagnose, fix scaffold, retry once. If still failing, Telegram alert with error and halt.
10. Initial commit: `git add -A && git commit -m "feat: project scaffold"`

**Output:** Working, verified project directory with git, package.json, AGENTS.md, Beads initialized.

---

### Phase 3 / product-brief — Product Brief (Greenfield) / Feature Brief (Brownfield)

**Greenfield → Product Brief.** Act as a product manager. Full product brief.
**Brownfield → Feature Brief.** Lean scope focused on what's being added/changed.

#### Greenfield Process:
1. Read the idea statement from Phase 1
2. If `_output/analyst-report.md` exists, read it — incorporate competitive landscape, validation risks, and brainstormed features
3. Produce `_output/product-brief.md` containing:
   - **Problem Statement** — What pain does this solve? (2-3 sentences)
   - **Target User** — Who benefits? (1 persona)
   - **Core Value Proposition** — Why would someone use this? (1 sentence)
   - **Key Features** — 5-8 features that define the MVP
   - **Success Metrics** — 3-5 measurable outcomes
   - **Out of Scope** — What this is NOT (prevent scope creep)
   - **Competitive Context** — Key competitors and our differentiation (from analyst report if available)
   - **Key Risks** — Carried forward from analyst validation (if available)
4. Produce `_output/product-brief-distillate.md` — dense, token-efficient version for downstream phases

#### Brownfield Process:
1. Read the idea statement from Phase 1
2. Read `_output/pattern-scan.md` (produced in Phase 2) — understand what already exists
3. Produce `_output/feature-brief.md` — a lean feature brief focused on the CHANGE:
   - **What's Changing** — 1-2 sentences describing the addition/modification/migration
   - **Why** — Business or technical motivation
   - **What Exists Today** — Current state (from pattern scan and project context)
   - **What's Being Added/Modified** — Concrete list of new capabilities
   - **Integration Points** — Which existing services, routes, configs are touched
   - **Out of Scope** — What NOT to change (prevent scope creep into refactoring)
   - **Risks** — What could break, rollback strategy
4. Produce `_output/feature-brief-distillate.md` — dense version for downstream phases:
   - Bullet-pointed requirements
   - Existing code to reuse (file paths from pattern scan)
   - Integration constraints
   - Open questions for architecture phase

**Output:** `_output/product-brief.md` (greenfield) or `_output/feature-brief.md` (brownfield) + corresponding distillate

---

### Phase 4 / prd — Full PRD (Greenfield) / Feature Spec (Brownfield)

**Greenfield → Full PRD.** Complete product requirements document.
**Brownfield → Feature Spec.** Focused on the change, integration points, and what to reuse.

#### Greenfield Process:
1. Read product brief + distillate
2. If `_output/analyst-report.md` exists, read it for competitive context and risks
3. Generate `_output/prd.md` with these sections:
   - **Vision Statement** — 1 paragraph
   - **Executive Summary** — What, why, who, how
   - **Success Metrics** — Measurable KPIs with targets
   - **User Journeys** — 3-5 core flows as step-by-step narratives
   - **Domain Model** — Key entities, relationships, data shapes
   - **Functional Requirements** — Numbered list (FR1, FR2, ...) with Given/When/Then
   - **Non-Functional Requirements** — Performance, security, scalability (NFR1, NFR2, ...)
   - **MVP Scope** — What ships first vs. what's deferred
   - **Assumptions & Risks** — What could go wrong
4. Track steps in frontmatter: `stepsCompleted: [...]`

#### Brownfield Process:
1. Read feature brief + distillate
2. Read `_output/pattern-scan.md` — carry forward reuse requirements
3. Generate `_output/feature-spec.md` — lean and focused:
   - **Change Summary** — What this feature/migration/fix does (1 paragraph)
   - **Functional Requirements** — Numbered (FR1, FR2, ...) with Given/When/Then. ONLY requirements for the NEW work — do not re-document existing functionality.
   - **Non-Functional Requirements** — Only NFRs relevant to this change (e.g., "health check must respond < 200ms")
   - **Integration Map** — Which existing files/services/routes are touched, and how:
     | Existing File | Change Type | What Changes |
     |--------------|-------------|--------------|
     | {path} | Modify | {description} |
     | {path} | Extend | {description} |
     | {new path} | Create | {description} |
   - **Reuse Inventory** — Existing code that MUST be reused (from pattern scan):
     | What | File | Why Reuse |
     |------|------|-----------|
     | {service/pattern} | {path} | {it already does X} |
   - **Migration/Rollback Plan** — If applicable (infra changes, DB migrations, deployment changes)
   - **Assumptions & Risks** — What could break in the existing system

**Quality gate:** Every functional requirement must be testable. Every modified file must be justified (why touch it?).

**Output:** `_output/prd.md` (greenfield) or `_output/feature-spec.md` (brownfield)

---

### Phase 5 / architecture — Full Architecture (Greenfield) / Integration Architecture (Brownfield)

**Greenfield → Full architecture.** Design the system from scratch.
**Brownfield → Integration architecture.** "How does this fit?" — only document what's NEW.

#### Greenfield Process:
1. Read PRD
2. Generate `_output/architecture.md` with:
   - **Tech Stack Decisions** — Framework, language, database, hosting (with rationale)
   - **Project Structure** — Directory tree showing where code lives
   - **API Design** — Route patterns, auth approach, response format
   - **Data Model** — Tables/collections with fields, relationships, indexes
   - **Key Patterns** — Service layer, error handling, middleware, state management
   - **Testing Strategy** — What gets unit tested vs. integration tested vs. E2E
   - **Implementation Notes** — Patterns agents must follow for consistency
   - Each decision should reference the PRD requirement it serves
   - Keep under 500 lines — concise enough for Pi to consume as context

#### Brownfield Process:

The existing architecture is already defined in project-context.md and AGENTS.md. This phase only documents NEW decisions required by the feature/migration.

1. Read feature spec (`_output/feature-spec.md`)
2. Read pattern playbook (`_output/pattern-scan.md`) — this is the existing architecture
3. Read project-context.md — these are the rules that constrain all decisions
4. Read AGENTS.md — existing stack and conventions

**Constraint: The existing stack is NOT up for debate.** Do not propose new frameworks, ORMs, state management libraries, or testing tools. Work within what exists. If something truly cannot be done with the existing stack, document it as a risk and propose the minimal addition.

5. Generate `_output/architecture.md` with ONLY:
   - **New Architectural Decisions** — Only decisions this work requires that don't already exist. For each:
     - What the decision is
     - Why it's needed (reference FR from feature spec)
     - How it fits with existing patterns (reference pattern playbook)
   - **Existing Patterns Being Reused** — Explicit list:
     | Pattern | Source File | How It's Used in This Work |
     |---------|------------|---------------------------|
     | {pattern} | {path} | {how this work reuses it} |
   - **New Files to Create** — With justification for each (why can't an existing file handle this?)
   - **Existing Files to Modify** — What changes and why
   - **Data Model Changes** — New tables/columns only. Reference existing migration patterns.
   - **Testing Approach** — Follow existing test patterns. Reference specific test files as templates.
   - **Deployment/Infrastructure Changes** — If applicable (new env vars, new services, new configs)
   - Keep under 300 lines — brownfield architecture docs should be SHORT because most decisions are already made

**Output:** `_output/architecture.md`

---

### Phase 6 / epics — Epics & Stories

Consume PRD + architecture. Break into implementable work.

**Process:**
1. Read PRD (extract all FRs and NFRs)
2. Read architecture (extract implementation patterns)
3. Generate `_output/epics.md` with:
   - **Requirements Inventory** — All FRs and NFRs listed
   - **FR Coverage Map** — Which epic covers which FR
   - **Epic List** — 3-8 epics, each with:
     - Description (1-2 sentences)
     - FRs covered
     - NFRs addressed
   - **Dependency Chain** — ASCII diagram showing epic ordering
   - **Stories per Epic** — Each story with:
     - User story format: As a {persona}, I want {action}, So that {value}
     - Acceptance criteria in Given/When/Then format
     - FR/NFR coverage tags
   - **Parallel Lane Map** — Which stories can run concurrently

**Quality gate:**
- Every FR maps to at least one story
- Every story has testable acceptance criteria
- No circular dependencies

**Output:** `_output/epics.md`

---

### Phase 7a / story-specs — Story Spec Files

**Agent: SM (Scrum Master / Story Writer)**

For each story in epics.md, the SM writes a story spec file. No test files yet — those come in Phase 7b.

**Story spec format** (in `docs/stories/{slug}.md`):
```markdown
# Story {X.Y}: {Title}

**Epic:** {N} - {Epic Name}
**Story:** {X.Y}
**Status:** Ready for Development
**Created:** {today's date}
**Priority:** {P0|P1|P2}

---

## Story
**As a** {persona}, **I want** {action}, **So that** {value}.

## Prerequisites
{What must exist first — reference other stories}

## Acceptance Criteria
{Checkbox list with Given/When/Then from epics.md}

## Tasks
{Implementation tasks with hour estimates}

## Dev Notes
- **Test file:** {path to test file}
- **Key files to modify:** {list}
- **Patterns to follow:** {reference architecture.md patterns}
```

**Brownfield MANDATORY addition — Coding Rules in Dev Notes:**

Pi does NOT read project-context.md or pattern-scan.md. The ONLY way project knowledge reaches Pi is through the story file (which Pi reads via AGENTS.md context + story content). Therefore, for brownfield projects, EVERY story's Dev Notes section MUST include a `## Coding Rules` block with concrete, actionable rules specific to that story:

```markdown
## Coding Rules (auto-injected from project context)

### Patterns to Follow
- Service pattern: copy structure from {actual file path from pattern-scan.md}
- Route pattern: copy middleware chain from {actual file path}
- Test pattern: copy mock setup from {actual file path}

### Reuse These (do NOT recreate)
- {service/utility name} at {file path} — {what it does, why to reuse it}
- {helper/config} at {file path} — {what it does}

### Do NOT
- {relevant anti-patterns from project-context.md for this story's domain}
- {e.g., "Do NOT use raw SQL — use Knex query builder (db('table').where(...))"}
- {e.g., "Do NOT create a new ORM — use existing Knex at src/server/config/database.ts"}

### Route Registration (if adding routes)
- {middleware stack order from project-context.md}
- {specific registration rules, e.g., "Mount BEFORE recipeRoutes"}

### Security (if adding endpoints)
- {auth requirements — authenticateToken middleware}
- {CSRF rules}
- {validation requirements — express-validator or Joi}

### Database (if touching DB)
- {Knex patterns — db('table').where(...)}
- {Column naming — snake_case}
- {Migration naming convention}

### Testing
- {Mock setup — what's globally mocked (DB, Redis) via setup.ts}
- {Test file location convention}
- {Naming convention}
```

Only include sections relevant to the story. A story that doesn't touch routes doesn't need the Route Registration section. A story that doesn't touch the DB doesn't need the Database section. But the Patterns/Reuse/Do-NOT sections are ALWAYS required for brownfield.

This is how project intimacy flows to Pi — through the story file, with concrete file paths, not vague references.

**Post-write validation (7a):** After generating all story files, verify each story has a corresponding story file. Log missing specs to `_output/phase7a-validation.log`. Proceed to Phase 7b.

**Output (7a):** N story files in `docs/stories/`

---

### Phase 7b / tdd — TDD Test Files

**Agent: QA (Quinn / Test Writer)**

QA reads each story spec from Phase 7a and writes failing TDD test files that define the contract. Code does not exist yet — tests will fail until dev (Phase 10) implements it. This is the correct starting state.

**TDD test format** — use the test framework and language chosen in Phase 5 / architecture. Example for TypeScript/Jest:
```typescript
/**
 * Epic {N}: {Epic Name}
 * Story {X.Y}: {Title}
 * TDD Tests — these define the contract. Code does not exist yet.
 */

// Imports will fail until Pi implements the code
import { ServiceName } from '../../path/to/service';

describe('ServiceName', () => {
  // Tests derived from acceptance criteria
  // Import things that DON'T EXIST YET — correct for TDD
});
```

For Python/pytest:
```python
# Epic {N}: {Epic Name} — Story {X.Y}: {Title}
# TDD Tests — code does not exist yet.

from app.services.service_name import ServiceName  # Will fail until implemented

def test_service_does_thing():
    ...
```

Adapt the format to match whatever stack Phase 5 selected.

**Rules:**
- Test file location follows project conventions from AGENTS.md and architecture.md
- Tests are practical — test the contract, not implementation details
- Each acceptance criterion becomes at least one test
- Tests import modules that don't exist yet (Pi will create them)

**Post-write validation (7b):** After generating all test files, verify each story spec has a corresponding test file (except doc-only stories like architecture spikes). If any test file is missing, regenerate it before proceeding. Log mismatches to `_output/phase7b-validation.log`.

**Output (7b):** N test files in `src/**/__tests__/`

---

### Phase 8 / beads-filing — File Beads Issues

Convert stories into Beads issues with full dependency tracking.

**Process:**
1. Generate `docs/stories/beads-import.md` in Beads batch import format:
   ```markdown
   # Epic 1: {Name}

   ## Story-1.1: {Title}
   - type: feature
   - priority: P0
   - labels: epic-1, lane-a, {project-prefix}
   - notes: story_file=docs/stories/{slug}.md | test_file=src/server/__tests__/{dir}/{name}.test.ts

   {Description}
   ```
2. Import: `bd create --file docs/stories/beads-import.md`
3. Fix metadata that batch import may miss:
   - Set correct priorities: `bd update {id} --priority {N} --type feature --set-labels {labels}`
4. Wire dependencies using `bd dep add {blocked} {blocker}` for:
   - Sequential stories within an epic
   - Cross-epic gate dependencies
   - Parallel lane independence
5. Verify with `bd ready` — only the first story in the dependency chain should be ready

**Output:** Beads issues with priorities, labels, and dependency chain.

---

### Phase 9 / checkpoint — Checkpoint & Handoff

Discovery is complete. Prepare for implementation.

**Process:**
1. Count total stories, total test files, total Beads issues
2. Verify all test files exist on disk (pre-flight for work-loop)
3. Verify all story files exist on disk
4. Commit discovery artifacts:
   ```bash
   git add _output/ docs/stories/
   # Add test files for any language (handles spaces in paths)
   find src -name "*.test.*" -o -name "*_test.*" -o -name "test_*" | xargs -d '\n' git add 2>/dev/null
   git commit -m "feat: discovery complete — {N} stories, {M} test files"
   bd sync
   git push || { git pull --rebase && git push; }
   ```
   If push fails after retry, Telegram alert "Vibe loop Phase 9: git push failed after rebase. Manual intervention needed." and halt.
5. Log to Telegram (if available):
   ```
   Vibe Loop — Discovery Complete
   Project: {name}
   Epics: {count}
   Stories: {count}
   Test files: {count}
   Beads issues: {count}
   Starting implementation...
   ```
6. **Immediately transition to dev (Phase 10).** Do NOT pause, ask for confirmation, or wait for user input. In `--yolo` mode the pipeline is continuous — discovery flows directly into implementation with zero human gates. Even in interactive mode, proceed unless the user explicitly said to stop after discovery. **⚠ RE-READ "Yolo Mode — No Human Gates" section NOW before outputting anything. If your next action is text without a tool call, you are about to violate the yolo rule.**

---

### Phase 10 / dev — Implementation Execution

**Pre-check:** Before starting, verify the work-loop skill exists:
```bash
ls ~/.hermes/skills/dev-team/work-loop/SKILL.md
```
If missing: Telegram alert "Vibe loop dev (Phase 10) halted — dev-team/work-loop skill not found. Install it and re-run." Then halt. Do NOT attempt to implement stories without the work-loop orchestration.

**Hands-off rule:** During Phase 10, Hermes orchestrates but NEVER edits source or test files directly. All code changes flow through Pi subagents. If tests fail due to environment issues, classify as INFRA and escalate — do not edit tests to work around the problem.

Execute the standard dev-team work-loop. This phase follows the EXACT same steps as the `dev-team/work-loop` skill:

1. **Health Check:** Run lint + typecheck. If pre-existing errors found, enter Health Fix Loop (progress-based, no arbitrary limits — fixes errors while making progress, escalates model if stalled, decomposes if stuck)
2. **Check Queue:** `bd ready --json` — pick up to `HERMES_MAX_PARALLEL` issues
3. **Claim:** `bd update {id} --claim`
4. **Pre-Flight:** Validate story_file and test_file exist on disk
5. **Build Context:** Read story + test + checkpoints + failed approaches
6. **Parallel Safety:** Check file overlap between concurrent stories
7. **Invoke Pi via CLI:** Run Pi as child process (not MCP) — process isolation means Pi crash never kills Hermes
8. **Evaluate:** Progress-aware monitoring — continue while making progress, escalate model if stalled, detect loops via tool call hashing, detect thrash via file edit counts
9. **Land the Plane:** git commit → bd close → git push → discover new issues → report
10. **Loop:** Back to step 2 until no ready issues remain

When all stories are closed (or only escalated stories remain), proceed to Phase 10b.

If stories are blocked on escalations, proceed to Phase 10b anyway with what's complete — partial apps can still be validated and deployed.

**⚠ COMPLETION GATE:** Do NOT declare the task done, output a final summary, or exit after dev (Phase 10). You MUST continue through pattern-capture (10b) → quinn-review (10c) → e2e-validation (11). quinn-review is mandatory — skipping it is a bug in your execution, not a valid shortcut. This applies even in `-q` mode.

### Phase 10b / pattern-capture — Pattern Capture (Brownfield Only)

**Skip if greenfield.** Greenfield projects establish patterns — brownfield projects need to record new patterns so the NEXT session inherits them.

After work-loop execution completes, scan what was created during this session:

1. **Identify new files created:**
   ```bash
   git diff --name-only --diff-filter=A HEAD~{N}..HEAD
   ```
   (where N = number of commits from this session)

2. **Classify new patterns:**
   - New service files → extract the pattern (class structure, export style)
   - New route files → extract the pattern (middleware chain, registration)
   - New test files → extract the pattern (mock setup, assertion style)
   - New config/infra files → extract the pattern (env vars, Docker, deployment)

3. **Update project-context.md** if new patterns were established:
   - Append new rules to the relevant section (e.g., "Railway health check at `/health` must return 200 with `{ status: 'ok' }`")
   - Update the `date:` field in frontmatter
   - Do NOT rewrite existing rules — only append

4. **Update pattern-scan.md** with new reusable patterns:
   - Add new entries to the "Reusable Patterns" table
   - Add new "Existing Code Relevant to This Task" entries

5. If NO new patterns were established (only extended existing ones):
   - Update project-context.md `date:` field only (so staleness check stays current)

6. Commit the updated docs:
   ```bash
   git add _bmad-output/project-context.md _output/pattern-scan.md
   git commit -m "chore: update project context after vibe-loop session"
   ```

---

### Phase 10c / quinn-review — Quinn Adversarial Review (MANDATORY — HARD GATE)

**This is NOT optional. This is a HARD GATE — the pipeline cannot complete without it.** If you are in `-q` mode and about to return your response: you are not done until this phase runs. Pi's code passes tests, but tests only validate what was anticipated. Quinn catches what wasn't — omissions, dead code, composition errors, security issues, spec deviations. Tests and adversarial review are complementary; neither replaces the other.

**Trigger:** All stories in Phase 10 are closed (or only escalated stories remain). Runs ONCE after the full implementation, not per-story.

**Why mandatory:** Pi writes both tests and code. When the same agent writes both, they share the same blind spots. The test suite validates "does what's here work?" — Quinn asks "what's missing? what breaks at scale? what doesn't match the spec?" These are fundamentally different questions.

**Process:**

**Step 1 — Collect the diff:**
```bash
git diff main...HEAD --stat
git diff main...HEAD
```
This captures everything the vibe-loop session produced.

**Step 2 — Collect spec context:**
Gather all story files from `docs/stories/` that were part of this session (match by beads labels or epic tag). These provide the acceptance criteria Quinn reviews against.

**Step 3 — Invoke `bmad-code-review` skill with three parallel review layers:**

Hermes invokes the code review skill (or runs the review layers directly if the skill is unavailable):

- **Blind Hunter** — receives diff ONLY. No project context, no spec. Finds bugs, security issues, logic errors purely from the code. Invoke via `bmad-review-adversarial-general`.
- **Edge Case Hunter** — receives diff + project read access. Walks every branching path and boundary condition. Reports unhandled edge cases only. Invoke via `bmad-review-edge-case-hunter`.
- **Acceptance Auditor** — receives diff + story spec files. Checks: AC violations, spec deviations, missing implementation, contradictions. Reviews against every AC in every story file.

**Step 4 — Triage findings:**

Classify each finding:
- **Critical/High** → file as P0 beads issue tagged `quinn-review`
- **Medium** → file as P1 beads issue tagged `quinn-review`
- **Low/Defer** → file as P2 beads issue tagged `quinn-review`
- **Dismiss** → drop (false positive, pre-existing, handled elsewhere)

**Step 5 — Fix loop:**

If P0 or P1 findings were filed:
1. Run `bd ready --json` — Quinn review issues are now in the queue
2. For each `quinn-review` issue:
   - Claim it
   - Invoke Pi to fix (include the finding detail, file path, and what's wrong in Pi's prompt)
   - Verify the fix (re-run relevant tests + spot-check the specific finding)
   - Land the fix (commit, close, push)
3. Loop until all `quinn-review` P0/P1 issues are closed

P2 findings remain open for future sessions — they're real but not blocking.

**Step 6 — Report:**
```
Quinn Adversarial Review — Complete
Findings: {total} ({critical} critical, {high} high, {medium} medium, {low} low)
Fixed this session: {fixed_count}
Deferred (P2): {deferred_count}
Dismissed: {dismissed_count}
```

**Why not per-story:** Running three adversarial reviewers per story is expensive and catches less — many composition errors only appear when multiple stories interact. Running once after the full implementation sees the complete picture, like we just demonstrated with the Railway migration review.

---

### Phase 11 / e2e-validation — End-to-End Validation

All stories are implemented and adversarially reviewed. Now verify the whole app works as a system, not just individual stories.

**Process:**

1. **Full test suite + build check:**
   ```bash
   {test_cmd} 2>&1  # Use test_cmd from stack-detect (e.g., CI=true npx jest). NEVER bare npm test.
   npm run build 2>&1
   ```
   If failures, invoke the `dev-team/health-fix` skill with:
   - `error_source`: "test-suite+build"
   - `error_list`: the failure output
   - `scope`: "all"
   
   Health-fix handles progress-based fixing, research, and learning. 
   - If health-fix returns `PASS` → all clear, proceed
   - If health-fix returns `PARTIAL` → classify remaining: **critical** (auth, data integrity, payment) → halt. **Non-critical** → proceed to deployment.
   - If unsure whether a failure is critical, treat it as critical.

2. **Build verification:**
   ```bash
   npm run build  # or equivalent for the stack
   ```
   The app must compile/build without errors. Build warnings are acceptable.

3. **Smoke test (if server app):**
   - Check if external services are needed (DB, Redis, APIs). If docker-compose exists, run `docker-compose up -d` first. If external deps are unavailable and required, skip smoke test with log: "Smoke test skipped — external services unavailable."
   - Start the server in the background
   - If health endpoint exists: hit it, expect 200 OK
   - Independently: hit 2-3 core API endpoints with test data, expect valid responses
   - Stop the server (and docker-compose if started)
   Each check is independent — a missing health endpoint does not skip core endpoint checks.

4. **Integration check (brownfield only):**
   - Verify the new feature doesn't break existing features
   - Run the full existing test suite (not just new tests)
   - Check for import errors, circular dependencies, or missing modules

5. **Report:**
   ```
   Vibe Loop — E2E Validation
   Test suite: {passed}/{total} ({failures} failures)
   Build: {pass/fail}
   Smoke test: {pass/skip/fail}
   Integration: {pass/skip/fail}
   Ready for deployment: {yes/no}
   ```

**Output:** `_output/e2e-validation.md` with full results.

---

### Phase 12 / deploy — Deploy to Railway

Deploy the completed app. This phase adapts based on greenfield vs brownfield.

**Skip condition:** If the project has no Railway configuration (`railway.toml`, `Dockerfile.railway`, or Railway CLI not installed), skip deployment and report: "No Railway config found. Manual deployment needed."

**Greenfield deployment:**

1. **Verify Railway CLI is available:**
   ```bash
   command -v railway >/dev/null 2>&1
   ```
   If missing, skip with Telegram: "Railway CLI not installed. Run `npm i -g @railway/cli && railway login` then re-run deploy (Phase 12)."

2. **Check for existing Railway project:**
   ```bash
   railway status 2>/dev/null
   ```
   If a project already exists, use it (don't create a duplicate). If not, run `railway init`.

3. **Provision database FIRST (if needed):**
   - If architecture calls for PostgreSQL: `railway add --plugin postgresql`
   - Wait for DB to be provisioned
   - Run migrations against Railway DB
   - Verify migrations complete
   Database must be live before the app deploys — the app will crash on startup without it.

4. **Configure environment variables:**
   - Read required env vars from `src/server/config/env.ts` or equivalent
   - **CRITICAL: distinguish config from secrets:**
     - Config values (PORT, LOG_LEVEL, NODE_ENV): can use `.env.example` defaults
     - Secrets (API keys, JWT_SECRET, DB passwords, Stripe keys): NEVER use defaults. HALT and prompt user for each secret, even in `--yolo` mode.
   - Use `railway variables set KEY=VALUE` to configure

5. **Deploy:**
   ```bash
   railway up
   ```

6. **Post-deploy verification:**
   - Wait 30s for service to start
   - Hit the deployed health endpoint: expect 200 OK
   - Hit 2-3 core API endpoints with test data
   - If health check fails: retry after 60s. If still failing, Telegram alert with deployment URL and logs.

**Brownfield deployment:**

1. **Check if Railway project exists:** `railway status`
2. **Deploy from feature branch:**
   ```bash
   railway up
   ```
   Railway deploys the current branch. The feature branch goes live as a preview/staging deployment.
3. **Post-deploy verification:** same as greenfield
4. **Do NOT merge to main automatically** — the user decides when to merge and promote to production

**Deployment report:**
```
Vibe Loop — Deployed
URL: {railway deployment URL}
Health: {pass/fail}
Database: {provisioned/existing/none}
Branch: {branch name}
Status: {live/preview}
```

**Output:** `_output/deployment.md` with URL, status, and any manual steps remaining.

---

### Phase 13 / report — Completion Report

**Final report:**
```
Vibe Loop — Complete
Project: {name}
Stories completed: {N}/{total}
Stories escalated: {M}
Quinn review: {findings_total} findings ({fixed} fixed, {deferred} deferred)
E2E validation: {pass/partial/fail}
Deployment: {deployed to {URL} / skipped / failed}
Stories per epic: {epic breakdown}
Duration: {time}
Branch: {branch name} (merge to main when ready)
```

If stories remain open (escalated or blocked), log remaining work:
```
bd list --status=open -l {project-prefix} --json
```
And report via Telegram what's left.

---

## Error Handling

### analyst (Phase 0) Errors

| Error | Action |
|-------|--------|
| Web search tool not configured | Skip analyst, proceed to brief-capture (Phase 1) with warning in idea statement |
| Web search rate-limited or network error | Retry with 30s backoff, up to 3 attempts. If still failing, skip analyst with warning |
| Market research returns no results | Proceed with empty competitive snapshot, note "no data found" in analyst report |
| Idea hunt finds zero candidates | Telegram: "Hunt found no viable ideas for '{domain}'. Try a different domain." HALT for user redirect |
| All hunt candidates score below 6 | Present them anyway with warning: "All candidates scored low. Pick one to explore or refine the search." |
| Validation score < 6 | Proceed anyway — carry risks forward, do NOT halt |

### Discovery Phase Errors (brief-capture through checkpoint, Phases 1-9)

| Error | Action |
|-------|--------|
| Idea too vague after clarification | Telegram alert, halt pipeline |
| Scaffold fails (npm, git) | Telegram with diagnostic, halt |
| Brief/PRD/Architecture generation fails | Checkpoint to `_output/`, retry once, then Telegram + halt |
| Story/test file write fails | Retry, then Telegram + halt |
| Beads import partially fails | Count existing issues (`bd list -l {prefix} --json`), delete duplicates (`bd close {dup} --reason "duplicate from partial import"`), retry remaining with individual `bd create` |
| Beads import fully fails | Retry with individual `bd create` per story, then Telegram + halt |
### dev (Phase 10) Errors

Standard work-loop error handling applies:
- Pi crashes: read `.result` marker file as fallback
- `bd` CLI fails: log error, skip issue, continue
- Git push fails: retry with `git pull --rebase`, then Telegram alert
- Story fails 3 attempts: failure-classifier → escalation-handler

### e2e-validation / deploy Errors (Phases 11-12)

| Error | Action |
|-------|--------|
| Test suite has failures | File Beads issues per failure, attempt one fix round, proceed anyway |
| Build fails | Diagnose error, attempt fix, retry. If still failing, Telegram alert + halt |
| Railway CLI not installed | Skip Phase 12, report "manual deployment needed" |
| `railway up` fails | Telegram with error output, halt. User must fix Railway config |
| Health check fails post-deploy | Retry after 60s. If still failing, Telegram with URL + logs |
| Database migration fails on Railway | Telegram with migration error, halt. Do NOT proceed with broken DB |

### Resumption

If the pipeline is interrupted (crash or manual stop):
1. Check `_output/` for existing artifacts — resume from the last incomplete phase
2. Check Beads for filed issues — if issues exist, skip to dev (Phase 10)
3. If mid-implementation, work-loop handles its own resumption via checkpoints
4. If `_output/e2e-validation.md` exists, skip to deploy (Phase 12)
5. If `_output/deployment.md` exists, skip to report (Phase 13)

---

## Dependencies

- **Tools:** `bd` CLI, `git`, `npm/npx`, file operations, terminal, web search (for Phase 0)
- **Pi subagents:** `tdd-coder.md`, `quinn-validator.md`, `failure-classifier.md` (in `~/.pi/agents/`)
- **Pi extensions:** `beads-checkpoint.ts`, `budget-enforcer.ts`, `auto-committer.ts`
- **Hermes skills:** `dev-team/work-loop` (Phase 10), `dev-team/health-fix` (Phase 11 error resolution), `dev-team/escalation-handler` (failure routing)
- **BMAD skills:** `bmad-generate-project-context` (Phase 2 brownfield context generation)
- **Optional:** Railway CLI (Phase 12 deployment), Telegram gateway (notifications), `model-tier-classifier` (model selection)
