# TOOLS.md — Pathfinder Tooling

## Core Tools

### Linear CLI (`linearis`)
Primary tool for all Linear operations. Install: `npm install -g linearis`
Auth: Set `LINEAR_API_TOKEN` env var or store in `~/.linear_api_token`

Key commands:
```bash
# List Todo issues across all teams
linearis issues search "" --status "Todo"

# Read a specific issue
linearis issues read ABC-123

# Search issues by keyword
linearis issues search "keyword" --team Backend

# Add a comment (post analysis plan)
linearis comments create ABC-123 --body "plan content here"

# Update issue status to In Progress
linearis issues update ABC-123 --status "In Progress"

# List teams
linearis teams list

# List projects
linearis projects list

# List active cycle/sprint
linearis cycles list --active
```

All output is JSON — pipe through `jq` for filtering.

### GitHub CLI (`gh`)
For repo operations: cloning, reading code, checking PRs.
Already authenticated on machine.

### File System Tools
- `Read` — read files in repos
- `Grep` — search for code patterns
- `Glob` — find files by pattern
- `Bash` — run shell commands

## APIs
- Linear: via `linearis` CLI
- GitHub: via `gh` CLI
