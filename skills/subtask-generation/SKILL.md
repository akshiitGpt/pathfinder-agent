---
name: subtask-generation
description: Methodology for decomposing implementation plans into discrete, sequenced subtasks.
---

# Subtask Generation Skill

## Purpose

Break a Pathfinder implementation plan into actionable subtasks that map to individual units of work. Good subtasks make it easy for developers to:
- See what to do first
- Work on pieces in parallel when possible
- Track progress granularly
- Review changes in smaller, focused PRs

## Methodology

### Step 1: Assess Whether Subtasks Are Needed

| Complexity | Repos Affected | Generate Subtasks? |
|------------|---------------|-------------------|
| S          | 1             | No                |
| M          | 1             | Only if 3+ distinct logical changes |
| M          | 2+            | Yes               |
| L          | Any           | Yes               |
| XL         | Any           | Yes               |

### Step 2: Choose a Decomposition Strategy

Pick the strategy that best fits the work:

**Strategy A — By Execution Phase** (best for sequential, cross-repo work)
- Each phase in the execution order = one subtask
- Use when: changes must happen in a strict order (e.g., proto first, then service, then gateway)
- Advantage: clear dependency chain

**Strategy B — By Repository** (best for parallel, independent repo work)
- Each repo's changes = one subtask
- Use when: repos can be modified independently
- Advantage: enables parallel development

**Strategy C — By Logical Unit** (best for single-repo, multi-concern work)
- Group related changes by concern (data model, API, business logic, tests)
- Use when: all changes are in one repo but touch different layers
- Advantage: focused code reviews

**Strategy D — Hybrid** (for complex, XL issues)
- Combine strategies: phase-level subtasks that group repo-level work
- Use when: XL complexity with 3+ repos and strict ordering
- Advantage: balances granularity with manageability

### Step 3: Write Subtask Titles

Good titles are:
- **Action-oriented** — start with a verb (Add, Update, Implement, Fix, Create, Migrate)
- **Specific** — mention the component/repo/concept
- **Concise** — under 80 characters

Examples:
- "Add reaction model and MongoDB collection" (good)
- "Backend changes" (bad — too vague)
- "Implement the new reaction feature data model layer in communication-service" (bad — too long)

### Step 4: Write Subtask Descriptions

Each description should include:

```markdown
**Repo:** {repo-name}

**Files to change:**
- `path/to/file.py` — {what to change}
- `path/to/other.py` — {what to change}

**What to do:**
{1-3 sentences describing the change}

**Dependencies:**
- Depends on: {subtask title or "none"}
- Blocks: {subtask title or "none"}
```

### Step 5: Set Dependencies

Map the dependency graph:
- Which subtasks must complete before others can start?
- Which subtasks can run in parallel?
- Use Linear's `blocks`/`blockedBy` relations to encode this

### Verification Gates

1. **Completeness check:** Do all files from the code change map appear in at least one subtask?
2. **No orphan work:** Is every subtask connected to the parent issue via `parentId`?
3. **Reasonable count:** Are there 2-6 subtasks? If more than 6, consider grouping. If fewer than 2, subtasks aren't needed.
4. **Actionable:** Can a developer pick up any unblocked subtask and start working without needing to read the other subtasks first?
5. **No overlap:** Do any two subtasks modify the same file? If yes, they should probably be merged or one should block the other.
