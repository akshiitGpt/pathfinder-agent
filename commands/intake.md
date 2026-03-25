---
description: Scan all Jira boards for To-Do tickets and process each one through the full Pathfinder pipeline.
---

# /intake

Usage: `/intake`

## What it does
1. Fetches all Jira projects accessible to the agent
2. For each project, searches for tickets in "To Do" status
3. Filters out already-processed tickets (checks `memory/processed-tickets.json`)
4. For each unprocessed ticket, runs the full pipeline:
   - Classify (bug/feature)
   - Generate RCA or TRD
   - Identify code changes
   - Assemble plan
   - Post to Jira
   - Transition to IN DEVELOPMENT
5. Reports summary of processed tickets

## Example
```
/intake

🧭 Pathfinder — Scanning all Jira boards...

Found 3 To-Do tickets across 2 projects:
  RP-400: "Fix conversation pagination timeout" (Bug)
  RP-401: "Add message reactions" (Story)
  DS-150: "Support batch inference" (Task)

Processing RP-400...
  → Classified: bug
  → RCA generated
  → Code changes: agent-gateway (2 files), communication-service (1 file)
  → Plan posted to Jira ✅
  → Transitioned to IN DEVELOPMENT ✅

Processing RP-401...
  → Classified: feature
  → TRD generated
  → Code changes: agent-gateway (3 files), communication-service (4 files), proto-definitions (1 file)
  → Plan posted to Jira ✅
  → Transitioned to IN DEVELOPMENT ✅

Processing DS-150...
  → Classified: feature
  → TRD generated
  → Code changes: ai-proxy (2 files)
  → Plan posted to Jira ✅
  → Transitioned to IN DEVELOPMENT ✅

🧭 Intake complete — 3/3 tickets processed
```
