# Pathfinder 🧭

> Autonomous pre-coding agent that analyzes Jira tickets, generates TRDs and RCAs, identifies code changes, and prepares everything before development begins.

Pathfinder is an AI-powered pre-coding agent built on [OpenClaw](https://openclaw.com). It runs every 30 minutes, scans Jira boards for To-Do tickets, classifies them as bugs or features, performs deep analysis, and produces actionable development plans — all before a developer touches the code.

## What Pathfinder Does

Every 30 minutes:

1. **Scans Jira boards** for tickets in To-Do status
2. **Classifies** each ticket as a bug or a feature
3. **Bug** → generates a Root Cause Analysis (RCA)
4. **Feature** → generates a Technical Requirement Document (TRD)
5. **Consults the knowledge graph** (Obsidian vault) to understand repo landscape and connections
6. **Identifies affected repos** and pinpoints exact code change locations
7. **Assembles a plan** with TRD/RCA + code change mapping
8. **Posts the plan** back to the Jira ticket as a comment
9. **Transitions the ticket** to "IN DEVELOPMENT"

## The Knowledge Graph

Pathfinder maintains an Obsidian vault (`knowledge-graph/`) that maps:

- **Each repository** — what it does, its tech stack, key files, architecture
- **Inter-repo connections** — which services call which, shared protocols, Kafka topics, gRPC contracts
- **Domain concepts** — agents, conversations, messages, users, platforms
- **Change patterns** — when X changes, Y is usually affected

This graph is Pathfinder's memory of the codebase landscape. It consults the graph before identifying code changes, so it knows *where* things live and *how* they connect.

## Connected Repositories

| Repo | Stack | Role |
|------|-------|------|
| `agent-platform-v2` | Python/Poetry/FastAPI | Core agent execution platform |
| `agent-gateway` | Python/Poetry/FastAPI | Developer API gateway (REST/SSE/gRPC) |
| `communication-service` | Python/Poetry/gRPC/Beanie | Conversation & message storage (MongoDB) |
| `ai-proxy` | Python/uv/FastAPI | OpenRouter proxy for LLM calls |

## Agent Architecture

Pathfinder spawns specialized sub-agents for each phase:

| Agent | Role |
|-------|------|
| **Ticket Classifier** | Read ticket, classify as bug/feature, extract context |
| **RCA Agent** | Root cause analysis for bugs — trace symptoms to source |
| **TRD Agent** | Technical requirement doc for features — scope, design, acceptance criteria |
| **Repo Scanner** | Scan repos, find exact files/functions that need changes |
| **Plan Writer** | Assemble final plan, post to Jira, transition ticket |

See [ARCHITECTURE.md](ARCHITECTURE.md) for the full flow.

## Commands

```bash
# Process all To-Do tickets across all boards
/intake

# Analyze a specific ticket
/analyze RP-365

# Generate RCA for a bug ticket
/rca RP-400

# Generate TRD for a feature ticket
/trd RP-401

# Rebuild the knowledge graph
/graph
```

## Installation

Pathfinder runs as an [OpenClaw](https://openclaw.com) workspace agent.

1. Install OpenClaw
2. Clone this repo into your OpenClaw workspace directory
3. Copy `USER.md.template` to `USER.md` and fill in your details
4. Configure the agent in your OpenClaw config with a 30-minute heartbeat
5. Populate the knowledge graph (or run `/graph` to auto-generate)

## Key Design Decisions

- **30-minute heartbeat** — frequent enough to catch new tickets, not so frequent it spams
- **Classify first, then analyze** — don't waste time generating a TRD for a bug
- **Knowledge graph is the brain** — without repo context, code change identification is guessing
- **Post to Jira, don't create PRs** — Pathfinder prepares the plan, developers execute it
- **HITL gate** — the ticket moves to IN DEVELOPMENT as a signal for human review before coding starts
- **Never modify code** — Pathfinder reads, analyzes, and plans. It never writes production code.

## License

MIT — see [LICENSE](LICENSE)

---

Built by [Akshit Gupta](https://github.com/akshitgupta) — [Ruh AI](https://ruh.ai).

*"Scout the terrain before you march."*
