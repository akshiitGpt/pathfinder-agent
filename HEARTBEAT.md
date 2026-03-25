# HEARTBEAT.md — Pathfinder Autonomous Cycle

## Purpose

Every 30 minutes: scan Jira boards, find To-Do tickets, classify, analyze, plan, post, transition. No fluff.

---

## Checklist (run every heartbeat)

### 1. Scan All Jira Boards

Fetch all tickets in To-Do status across all Ruh AI Jira projects.

```
For each project:
  → Search: status = "To Do" AND updated >= -30m (or unprocessed)
  → Skip tickets already processed (check memory/processed-tickets.json)
  → Queue unprocessed tickets for analysis
```

If no new To-Do tickets → `HEARTBEAT_OK`

### 2. For Each Unprocessed Ticket

#### 2a. Read the Ticket
- Read title, description, comments, linked issues, parent epic
- Read issue type, priority, labels, components
- Check for attachments (error logs, screenshots, specs)

#### 2b. Classify: Bug or Feature
- Bug → proceed to RCA flow
- Feature → proceed to TRD flow
- Ambiguous → default to feature

#### 2c. Generate Analysis

**Bug path:**
1. Load `skills/rca/SKILL.md`
2. Identify symptom from ticket description
3. Consult `knowledge-graph/` to trace probable cause
4. Go into affected repo(s) — read actual code to confirm root cause
5. Generate RCA using `templates/rca-template.md`

**Feature path:**
1. Load `skills/trd-generation/SKILL.md`
2. Extract requirements from ticket description
3. Consult `knowledge-graph/` to identify affected services
4. Go into affected repo(s) — identify specific files and functions
5. Generate TRD using `templates/trd-template.md`

#### 2d. Identify Code Changes
1. Load `skills/code-change-detection/SKILL.md`
2. Read `knowledge-graph/connections/service-map.md` for cross-repo impacts
3. Read relevant `knowledge-graph/repos/*.md` files
4. Go into each affected repo — find exact files, functions, classes
5. Document: file path, function/class, what changes, why

#### 2e. Assemble Plan
1. Combine RCA/TRD + code changes into structured plan
2. Use `templates/plan-template.md` format
3. Include complexity estimate (S/M/L/XL)

#### 2f. Post to Jira
1. Add plan as a comment on the ticket
2. Transition ticket status → IN DEVELOPMENT
3. Record in `memory/processed-tickets.json`:
   ```json
   {
     "PROJ-123": {
       "processed": "2026-03-25T10:30:00Z",
       "classification": "feature",
       "repos_affected": ["agent-gateway", "communication-service"],
       "complexity": "M"
     }
   }
   ```

### 3. Post-Cycle Check

- Any tickets that failed analysis? → Log error, retry next cycle
- Knowledge graph gaps discovered? → Note them for `/graph` update

---

## Response Format

Nothing to do:
```
HEARTBEAT_OK — No new To-Do tickets found
```

Tickets processed:
```
🧭 Pathfinder — Processed 3 tickets

✅ RP-400 (bug) → RCA posted, moved to IN DEVELOPMENT
   Repos: agent-gateway, communication-service | Complexity: M

✅ RP-401 (feature) → TRD posted, moved to IN DEVELOPMENT
   Repos: agent-platform-v2 | Complexity: S

⚠️ RP-402 (feature) → Analysis incomplete — missing acceptance criteria
   Action: Added comment requesting clarification, left in To-Do
```

Error:
```
❌ RP-403 — Failed to analyze: could not access repo agent-platform-v2
   Action: Will retry next heartbeat
```
