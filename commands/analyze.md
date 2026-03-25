---
description: Analyze a specific Jira ticket — classify, generate RCA/TRD, identify code changes, post plan.
---

# /analyze

Usage: `/analyze <ticket-key>`

## What it does
1. Fetches the specified Jira ticket
2. Runs the full Pathfinder pipeline on it:
   - Classify (bug/feature)
   - Generate RCA or TRD
   - Identify code changes
   - Assemble plan
   - Post to Jira
   - Transition to IN DEVELOPMENT
3. Reports the result

## Example
```
/analyze RP-365

🧭 Pathfinder — Analyzing RP-365...

Ticket: "Ticket Intake & Analysis — RCA, TRDs, subtask generation, knowledge graph"
Classification: feature
Priority: Medium

Generating TRD...
  → Scope: Pre-coding automation agent
  → Affected repos: None (new project)
  → Complexity: XL

TRD posted to RP-365 ✅
Transitioned to IN DEVELOPMENT ✅
```
