---
name: health-fix
description: Progress-aware error resolution with web research escalation and solution learning. Fixes lint/TS/build errors without arbitrary iteration limits. Called by work-loop and vibe-loop.
version: 1.0.0
metadata:
  hermes:
    tags: [error-fixing, health-check, progress-monitoring, self-healing]
    related_skills: [dev-team/work-loop, dev-team/vibe-loop, dev-team/escalation-handler]
---

# Health Fix — Progress-Aware Error Resolution

Fixes codebase errors (lint, TypeScript, build) using progress-based monitoring, model escalation, web research, and solution learning. No arbitrary iteration limits — continues while making progress, escalates intelligently when stalled.

## Trigger

Called by other skills when errors block progress:
- **work-loop** Step 1: pre-flight health check before story work
- **vibe-loop** Phase 11: E2E validation failures
- **Any skill** that encounters blocking errors

## Input

The calling skill provides:
- `{error_source}`: what produced the errors (e.g., "eslint", "tsc", "npm run build", "npm test")
- `{error_list}`: the actual error output
- `{scope}`: "all" (fix everything) or "blocking-only" (fix only what prevents commit)
- `{project_root}`: working directory

## Process

### 1. Check Solution Library + Learned Patterns First

Before trying to fix anything, check two knowledge sources:

**A. Learned debugging patterns** — read `dev-team/learned-fixes` skill:
- Does the error match a known pattern? (e.g., "same error after 3+ fixes" = wrong diagnosis)
- Apply the pattern's recommended approach instead of brute-forcing

**B. Previously solved errors** — search installed skills:
```
Search installed skills for error message patterns:
  hermes skills search "{error_message_fragment}"
```

If a matching skill exists (e.g., `fix-sqlite3-elf-header`):
- Apply the documented solution
- Re-run the error check
- If fixed → done, return PASS
- If not fixed → the skill is stale, proceed to fix loop

### 2. Classify Errors

Parse `{error_list}` and classify each error:

| Classification | Criteria | Action |
|---------------|----------|--------|
| **BLOCKING** | Fails pre-commit hook, prevents `git commit` | Must fix now |
| **WARNING** | ESLint warns but doesn't error | Skip — don't fix unless `{scope}` is "all" |
| **ENVIRONMENT** | Missing native module, DB connection, wrong Node version | Cannot fix with code changes — log and skip |

If `{scope}` is "blocking-only", filter to BLOCKING errors only.

### 3. Initialize Tracking

```
round = 0
error_count_initial = count(BLOCKING errors)
error_count_previous = error_count_initial
stall_count = 0
current_model = default (Sonnet)
tool_call_hashes = []       # last 5 tool call hashes
files_edited = {}           # file path → edit count
approaches_tried = []       # descriptions of each approach
research_findings = []      # accumulated web search results
```

Create Beads issue for tracking:
```bash
bd create "chore: fix {error_source} errors ({error_count_initial} errors)" --type chore --priority 0
bd update {id} --claim
```

### 4. Fix Loop

**Each round:**

**4a. Dispatch to Pi fixer:**
```
"Fix these {error_source} errors. Do NOT touch test files. Only fix the listed errors.
Errors: {remaining_error_list}
Previously tried (do NOT repeat): {approaches_tried}
Research context (if available): {research_findings}"
```

**4b. Re-run error check.** Count errors: `{error_count_now}`

**4c. Progress detection:**

| Signal | Detection Method | Action |
|--------|-----------------|--------|
| **PROGRESS** | `error_count_now < error_count_previous` | Reset `stall_count` to 0. Log approach as successful. Continue. |
| **STALLED** | `error_count_now == error_count_previous` for 2 consecutive rounds | Increment `stall_count`. See escalation chain. |
| **REGRESSING** | `error_count_now > error_count_previous` | Revert Pi's changes (`git checkout -- .`). Log approach as failed. Try different approach. |
| **LOOP** | Hash of last 3 tool calls matches a previous 3-call sequence | Stop current approach. Force completely different strategy. |
| **THRASH** | Same file edited 3+ times across rounds | Stop editing. Read the ENTIRE file top-to-bottom. Trace the full execution path. The bug is in the logic flow, not the line being edited. Check: array indices, capture group numbers, variable naming, data handoff between functions. See `dev-team/learned-fixes` for known patterns. |
| **SAME ERROR** | Exact same error message after 3+ genuinely different fix attempts | The error message is MISLEADING. The bug is in a different part of the code than what the message suggests. Stop fixing what the error points to. Trace the full code path from input to output. |

**4d. Track tool calls:**
Hash each tool call: `hash(tool_name + file_path + operation_type)`. Store last 5. If any 3-call subsequence repeats, flag LOOP.

**4e. Track file edits:**
Increment `files_edited[path]` on each edit. If any file reaches 3, flag THRASH.

**4f. Self-assessment (every 3 rounds):**
Ask the model:
> "Round {N}. Started with {initial} errors, now at {current}. Approaches tried: {list}. Are you making progress or repeating yourself? What would you try differently?"

Use the response to guide the next approach.

**4g. Update tracking:**
```
error_count_previous = error_count_now
round += 1
```

### 5. Escalation Chain

When stalled, escalate through these levels in order:

| Level | Trigger | Action |
|-------|---------|--------|
| **1. Different approach** | Stall count 1 | Log "Stalled." Have Pi try a fundamentally different fix strategy. |
| **2. Escalate model** | Stall count 2 | Switch from Sonnet → Opus. Reset stall count. |
| **3. Web research** | Stall count 3 (on Opus) | Search the exact error message. Queries: `"{error_message}" fix`, `"{error_message}" {framework} github issue`, `"{error_message}" stackoverflow`. Feed top 3 results to Pi as context. |
| **4. Decompose** | Stall count 4 (on Opus + research) | Split remaining errors by file. Attack each file as a separate mini-session with research context. |
| **5. Deep research** | Still stuck after decompose | Search official docs, changelog, migration guides. Check if error is a known issue with specific version. Feed to Pi for final attempt. |
| **6. Deep Research & Rearchitect** | Still stuck after deep research | Invoke work-loop Step 9b: root cause archaeology, targeted web research, challenge assumptions, propose alternative architectures, prototype in isolation, apply via Opus. This is the FINAL autonomous resolution path. |
| **7. P0 blocker** | Deep Research & Rearchitect failed | File each remaining error as its own P0 Beads issue with ALL accumulated research findings, prototype results, and failed approaches. Tag `needs-deep-research-round-2` for next session. Do NOT expect a human to fix this — next session's Deep Research continues from where this one left off. |

### 6. P0 Blocker Filing

When all strategies are exhausted for an error, file it with maximum context:

```bash
bd create "blocker: {error_message_summary}" \
  --type bug \
  --priority 0 \
  --description "$(cat <<'ISSUE'
## Error
{exact error message with file and line}

## Approaches Tried
{numbered list of every approach and why it failed}

## Research Findings
{web search results that were relevant}
{library docs consulted}

## Environment
{node version, framework versions, OS}

## Suggested Next Steps
{model's best guess at what might work}
ISSUE
)" \
  --set-labels blocker,tech-debt
```

Tag as `blocker` so `bd ready` surfaces it as highest priority next session.

### 7. Learn from Solutions

**After successfully fixing an error**, check if the fix was non-obvious (required research, model escalation, or 3+ rounds):

If yes, save the solution as a reusable skill:

```
skill_manage create:
  name: fix-{error-slug}
  category: dev-team/learned-fixes
  content: |
    # Fix: {error summary}
    
    ## Error Pattern
    {the error message or pattern that triggers this fix}
    
    ## Root Cause
    {why this error happens}
    
    ## Solution
    {the exact fix that worked, with code examples}
    
    ## Context
    {framework, version, conditions where this applies}
    
    ## Learned
    {date} — discovered during {project} health fix
```

This builds a growing library of solutions. Next time any project hits the same error pattern, Step 1 finds it instantly — no research needed.

**If the fix revealed a structural debugging lesson** (e.g., "the error message was misleading," "the bug was in a different file than expected," "the real issue was an off-by-one index"), also update `dev-team/learned-fixes/SKILL.md` via `skill_manage patch` to add the pattern. These meta-lessons help avoid burning iterations on wrong diagnoses in future sessions.

**After filing a P0 blocker**, also save what was learned:

```
skill_manage create:
  name: unsolved-{error-slug}
  category: dev-team/unsolved
  content: |
    # Unsolved: {error summary}
    
    ## Error Pattern
    {the error message}
    
    ## Approaches That Failed
    {list}
    
    ## Research Findings
    {what was found}
    
    ## Why It's Hard
    {model's analysis of why this resists fixing}
```

Even failed attempts become knowledge — the next session won't waste time repeating them.

### 8. Completion

**All errors fixed:**
```bash
git add -A
git commit -m "chore: fix {error_source} errors ({error_count_initial} resolved)"
git push || { git pull --rebase && git push; }
bd close {id} --reason "All errors fixed in {round} rounds"
```

Return `PASS` to calling skill.

**Some errors remain as P0 blockers:**
```bash
# Commit what was fixed
git add -A
git commit --no-verify -m "chore: partial fix — {fixed_count}/{error_count_initial} {error_source} errors resolved"
git push || { git pull --rebase && git push; }
bd close {id} --reason "Partial fix. {remaining} errors filed as P0 blockers."
```

Return `PARTIAL` to calling skill with list of blocker issue IDs.

**No errors were fixable:**
Return `FAIL` to calling skill with list of blocker issue IDs.

## Dependencies

- **Pi subagents:** `tdd-coder.md` (dispatched for fix work)
- **Hermes toolsets:** `web` (for error research)
- **Hermes tools:** `skill_manage` (for saving learned solutions)
- **External:** `bd` CLI, `git`, `eslint`, `tsc`
