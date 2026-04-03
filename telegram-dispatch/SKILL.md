# Telegram Dispatch

Parses Telegram messages from Bob into work-loop triggers. The Beads issue IS the prompt — Bob sends an ID, Hermes reads everything it needs from Beads.

## Trigger

Incoming Telegram message from an authorized user (Bob).

## Command Parsing

Parse the incoming message and match against these patterns (case-insensitive):

| Pattern | Example | Action |
|---------|---------|--------|
| Single Beads ID | `LivingApp-S2.1` | Execute that specific issue via work-loop (chain mode) |
| Multiple Beads IDs | `LivingApp-S2.1 LivingApp-S2.3` | Execute all listed issues via work-loop (parallel mode, respecting HERMES_MAX_PARALLEL) |
| `run ready` | `run ready` | Run `bd ready --json`, execute all returned issues via work-loop |
| `status` | `status` | Report all in-progress issues with checkpoint summaries |
| `stop {id}` | `stop LivingApp-S2.1` | Abort the Pi session for that issue, checkpoint current state to Beads |
| `queue` | `queue` | Show all ready issues without executing: `bd ready` |
| `help` | `help` | Show this command table |

### ID Detection

A Beads ID matches the pattern: `LivingApp-{alphanumeric}` (e.g. `LivingApp-S2.1`, `LivingApp-815`, `LivingApp-0f4`).

Extract all IDs from the message. If the message contains only IDs (no other command words), treat it as an execute request.

## Command Handlers

### Execute (single or multiple IDs)

1. Validate each ID exists: `bd show {id} --json`
   - If ID not found: reply "Unknown issue: {id}"
   - If issue is already closed: reply "{id} is already closed"
   - If issue is blocked: reply "{id} is blocked by: {dep_ids}"
2. For valid, unblocked issues: invoke the **work-loop** skill with those specific IDs
3. Reply: "Starting {id}..." or "Starting {n} stories in parallel..."

### Run Ready

1. Run `bd ready --json`
2. If no issues ready: reply "No ready issues."
3. If issues found: reply "Found {n} ready issues: {ids}. Starting work-loop..."
4. Invoke the **work-loop** skill with all ready IDs

### Status

1. Run `bd list --status=in_progress --json`
2. For each in-progress issue:
   - Read metadata for latest checkpoint
   - Format: "{id}: {title} — {checkpoint.tests_passing} tests, approach: {checkpoint.approach}, cost: ${checkpoint.cost_usd}"
3. If no in-progress issues: reply "Nothing in progress."
4. Reply with formatted status table

### Stop

1. Parse the ID from `stop {id}`
2. Verify the issue is in_progress
3. Trigger checkpoint save (beads_checkpoint) for the active Pi session
4. Abort the Pi session
5. Update Beads: `bd update {id} --status=open --append-notes "Stopped by Bob via Telegram"`
6. Reply: "Stopped {id}. Progress checkpointed."

### Queue

1. Run `bd ready`
2. Format as a numbered list with priorities
3. Reply with the list (no execution)

### Help

Reply with the command table from the Command Parsing section above.

## Error Handling

- Unrecognized messages: reply "I didn't understand that. Type `help` for available commands."
- bd CLI failures: reply "Beads error: {error message}"
- Work-loop failures: the work-loop skill handles its own errors and reports via Telegram

## Security

- Only process messages from authorized users (Hermes TELEGRAM_ALLOWED_USERS)
- This is enforced by Hermes's built-in user authorization — this skill does not need to re-check

## Dependencies

- Hermes Telegram gateway (built-in)
- work-loop skill (for execution)
- Beads CLI (`bd`) for issue queries
