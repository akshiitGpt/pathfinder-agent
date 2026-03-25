---
description: Generate a Root Cause Analysis for a specific bug ticket.
---

# /rca

Usage: `/rca <ticket-key>`

## What it does
1. Fetches the specified Jira ticket
2. Skips classification — treats it as a bug
3. Spawns the RCA Agent
4. Identifies code changes via Repo Scanner
5. Posts RCA + code changes to the ticket
6. Transitions to IN DEVELOPMENT

## Example
```
/rca RP-400

🧭 Pathfinder — RCA for RP-400

Symptom: POST /conversations/{id}/messages returns 500 for conversations with >50 messages
Root Cause: Missing compound index on (conversation_id, created_at) in communication-service
Fix: Add index to MessageModel, verify query uses it

Affected Files:
  communication-service:
    app/models/message_model.py — Add compound index
    tests/test_message_service.py — Add pagination test

RCA posted to RP-400 ✅
Transitioned to IN DEVELOPMENT ✅
```
