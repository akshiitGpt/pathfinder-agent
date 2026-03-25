---
description: Scan all Linear teams for Todo issues assigned to the user and process each one through the full Pathfinder pipeline.
---

# /intake

Usage: `/intake`

## What it does
1. Fetches all Linear teams accessible to the agent via `linearis teams list`
2. For each team, searches for issues in "Todo" status assigned to the user via `linearis issues search "" --status "Todo" --assignee <USER_EMAIL>`
3. Filters out already-processed issues (checks `memory/processed-issues.json`)
4. For each unprocessed issue, runs the full pipeline:
   - Classify (bug/feature)
   - Generate RCA or TRD
   - Identify code changes
   - Assemble plan
   - Post to Linear via `linearis comments create`
   - Transition to In Progress via `linearis issues update`
5. Reports summary of processed issues

## Example
```
/intake

Pathfinder — Scanning all Linear teams...

Found 3 Todo issues assigned to you across 2 teams:
  RP-400: "Fix conversation pagination timeout" (Bug)
  RP-401: "Add message reactions" (Feature)
  DS-150: "Support batch inference" (Improvement)

Processing RP-400...
  → Classified: bug
  → RCA generated
  → Code changes: agent-gateway (2 files), communication-service (1 file)
  → Plan posted to Linear
  → Transitioned to In Progress

Processing RP-401...
  → Classified: feature
  → TRD generated
  → Code changes: agent-gateway (3 files), communication-service (4 files), proto-definitions (1 file)
  → Plan posted to Linear
  → Transitioned to In Progress

Processing DS-150...
  → Classified: feature
  → TRD generated
  → Code changes: ai-proxy (2 files)
  → Plan posted to Linear
  → Transitioned to In Progress

Intake complete — 3/3 issues processed
```
