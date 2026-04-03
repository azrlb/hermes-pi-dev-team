# Model Tier Classifier

Analyzes task descriptions and story metadata to classify complexity and recommend the appropriate AI model tier. Used by work-loop and story-implementer to set the model for Pi subagents.

## Trigger

- Called by work-loop before invoking Pi subagents
- Called by story-implementer in Step 3
- Telegram: `classify {story_id}` or `what tier is {story_id}`

## The 5-Tier Model

| Tier | Model | Cost/MTok (in/out) | Use When |
|------|-------|-------------------|----------|
| 1 | google/gemini-2.5-flash | $0.15/$0.60 | Simple: utility functions, config changes, single-file edits |
| 2 | google/gemini-2.5-flash | $0.15/$0.60 | Standard-simple: CRUD endpoints, straightforward services |
| 3 | anthropic/claude-sonnet-4-6 | $3.00/$15.00 | Standard: multi-file features, business logic, API integration |
| 4 | anthropic/claude-opus-4-6 | $15.00/$75.00 | Complex: architectural changes, concurrency, multi-service |
| 5 | Human (Bob) | N/A | Beyond AI: requires business decisions, creative direction, external negotiations |

## Classification Rules

### Tier 1-2: Simple (Gemini Flash)
Indicators:
- Story touches 1-2 files
- No dependencies on other stories
- Acceptance criteria are mechanical (add field, rename, format change)
- Estimated < 50 lines of code
- Pattern exists in codebase (brownfield_scan finds matches)
- Story title contains: "add", "rename", "format", "config", "utility", "helper"

### Tier 3: Standard (Claude Sonnet) — DEFAULT
Indicators:
- Story touches 2-5 files
- Has acceptance criteria with logic (if/then, validation, error handling)
- Needs to integrate with existing services
- Estimated 50-200 lines of code
- Most stories fall here

### Tier 4: Complex (Claude Opus)
Indicators:
- Story touches 5+ files
- Involves concurrency, race conditions, state machines
- Architectural decisions needed
- Security-sensitive (auth, encryption, access control)
- Performance-critical (optimization, caching strategies)
- Story title contains: "refactor", "architecture", "security", "performance", "migration"

### Tier 5: Human Required
Indicators:
- Requires business decision (pricing, feature direction, UX)
- External dependency (API key, vendor contract, legal review)
- Story has been escalated 2+ times already
- Blocker type: HARD_PROBLEM with no suggested_action

## Steps

### 1. Read Story Metadata

From Beads issue or story file:
- `title` — keyword scan
- `touches` — file count
- `acceptance_criteria` — complexity of each AC
- `depends_on` — dependency chain depth
- `context_files` — breadth of context needed
- Prior `failed_approaches` — if retrying, may need higher tier

### 2. Apply Rules

Score each indicator. Default to Tier 3 if ambiguous.

**Automatic overrides:**
- Story has 3 failed attempts → bump up one tier
- Story marked `expected_outcome: fail` → Tier 3 minimum (trap stories need reasoning)
- `dev_mode: interactive` → Tier 5 (human session)

### 3. Return Classification

```json
{
  "tier": 3,
  "model": "anthropic/claude-sonnet-4-6",
  "confidence": 0.85,
  "reasoning": "Multi-file feature with validation logic, standard complexity",
  "cost_estimate_usd": 0.50
}
```

### 4. Log

Audit trail: action `tier_classified`, target `{story_id}`, detail with tier and reasoning.

## Dependencies

- story_read Pi tool (or parse story directly)
- brownfield_scan Pi tool (check for existing patterns)
- Beads CLI for story metadata
- platform.db for audit logging
