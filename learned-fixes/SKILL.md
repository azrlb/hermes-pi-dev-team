---
name: learned-fixes
description: Accumulated debugging patterns learned from real failures. Consulted by health-fix before attempting fixes.
version: 1.0.0
metadata:
  hermes:
    tags: [debugging, patterns, learned, knowledge-base]
    related_skills: [dev-team/health-fix, dev-team/work-loop]
---

# Learned Debugging Patterns

These patterns were learned from real failures during development. Health-fix and work-loop should consult these BEFORE attempting to fix an error — the solution may already be known.

## Pattern: Same Error After Multiple Fix Attempts

**Signal:** The exact same test failure message appears after 3+ different fix attempts.

**Wrong approach:** Keep modifying the code that seems related to the error message.

**Right approach:** The error message is misleading. The bug is likely in a DIFFERENT part of the code than what the error message points to. Stop. Read the full code path from test input to error output. Trace the actual execution flow, not what the error message suggests.

**Example:** `recipeImportRecovery.ts` — test expected `strategyUsed: MICRODATA` but got `MANUAL_FALLBACK`. 60 iterations were spent rewriting the microdata regex pattern. The actual bug was `match[1]` (tag name) instead of `match[2]` (content) — a capture group index error, not a regex pattern error. The regex was fine all along.

**Rule:** If the same error persists after 3 genuinely different regex/pattern fixes, the bug is NOT in the pattern. Check array indices, capture group numbers, variable names, and data flow.

---

## Pattern: Regex Returns Correct Match But Code Reads Wrong Group

**Signal:** Regex test tools show the pattern matches correctly, but the code still fails.

**Check:** Are you reading `match[1]` when you should be reading `match[2]`? Count the capture groups in the regex:
- `match[0]` = full match
- `match[1]` = first `(...)` group
- `match[2]` = second `(...)` group

**Example:** `/<(tag).*?>(content)<\/\1>/` has TWO groups. Group 1 is the tag name, Group 2 is the content. Reading `match[1]` gives you "div", not the HTML content.

---

## Pattern: Test Expects Value But Gets Undefined

**Signal:** `expect(obj.field).toBe(something)` fails because `obj.field` is `undefined`.

**Check in order:**
1. Is the field optional (`field?:`) in the interface? If yes, the code path may not set it.
2. Is the field being passed through all function calls? Check every handoff — it only takes one missing parameter to lose it.
3. Is the field name spelled exactly the same everywhere? (`userContext` vs `user_context` vs `context`)

**Example:** `userContext` was `undefined` in escalation calls because `handleNovelError()` didn't include it in the object passed to `notifyEscalation()`. The interface allowed it (optional `?`), the caller provided it, but one function in the chain didn't forward it.

---

## Pattern: Same File Patched 3+ Times Without Progress

**Signal:** health-fix THRASH detection fires — same file edited 3+ times.

**Right approach:** Stop patching. Read the ENTIRE file from top to bottom. Understand the full control flow. The bug is in the logic flow, not in the line you keep editing. Write out the execution path as comments before making another change.
