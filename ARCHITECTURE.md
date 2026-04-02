# Architecture

Pathfinder is an autonomous pre-coding agent built as an OpenClaw workspace. It uses a layered architecture: skills define the analysis methodology, agents execute the work, and the knowledge graph provides codebase context.

## How It Works

```
Heartbeat (every 30 min)
         │
         ▼
    ┌─────────────────┐
    │  Scan Linear     │  Fetch all Todo issues across all teams
    │  Teams           │
    └────────┬────────┘
             │ for each issue
             ▼
    ┌─────────────────┐
    │  Issue           │  Read issue, classify: bug or feature
    │  Classifier      │  Extract: summary, context, affected areas
    └────────┬────────┘
             │
        ┌────┴────┐
        │         │
    bug │         │ feature
        ▼         ▼
  ┌──────────┐  ┌──────────┐
  │ RCA      │  │ TRD      │
  │ Agent    │  │ Agent    │
  └────┬─────┘  └────┬─────┘
       │              │
       └──────┬───────┘
              ▼
    ┌─────────────────┐
    │  Repo Scanner    │  Consult knowledge graph → identify repos
    │                  │  Go into each repo → find exact change locations
    └────────┬────────┘
             ▼
    ┌─────────────────┐
    │  Plan Writer     │  Assemble: RCA/TRD + code changes + plan
    │                  │  Post to Linear issue as comment
    └────────┬────────┘
             │ if M(multi-repo)/L/XL
             ▼
    ┌─────────────────┐
    │  Subtask         │  Break plan into Linear subtasks
    │  Generator       │  Set parentId, dependencies, sequencing
    └────────┬────────┘
             │
             ▼
    Transition issue → In Progress
```

## Directory Layout

```
pathfinder/
├── SOUL.md                 Agent personality, rules, principles
├── IDENTITY.md             Name, vibe, emoji
├── HEARTBEAT.md            30-min autonomous cycle definition
├── ARCHITECTURE.md         This file — how it all connects
├── README.md               Overview and quick start
├── TOOLS.md                Tooling reference (linear.sh skill, gh, etc.)
│
├── knowledge-graph/        Obsidian vault — repo knowledge base
│   ├── repos/              Per-repo knowledge files
│   │   ├── agent-platform-v2.md
│   │   ├── agent-gateway.md
│   │   ├── communication-service.md
│   │   └── ai-proxy.md
│   ├── connections/        Inter-repo dependency maps
│   │   └── service-map.md
│   └── README.md           How to use the knowledge graph
│
├── skills/                 Analysis methodologies (the knowledge base)
│   ├── ticket-analysis/    How to read and classify issues
│   ├── rca/                Root cause analysis methodology
│   ├── trd-generation/     TRD structure and best practices
│   ├── code-change-detection/  How to find affected code
│   ├── subtask-generation/ How to decompose plans into subtasks
│   └── knowledge-graph/    How to consult and maintain the graph
│
├── agents/                 Sub-agent definitions
│   ├── ticket-classifier.md   Read and classify issues
│   ├── rca-agent.md           Root cause analysis for bugs
│   ├── trd-agent.md           TRD generation for features
│   ├── repo-scanner.md        Find code change locations
│   ├── plan-writer.md         Assemble and post final plan
│   └── subtask-generator.md   Break plan into Linear subtasks
│
├── commands/               Slash commands
│   ├── intake.md           /intake — process all Todo issues
│   ├── analyze.md          /analyze <key> — analyze a specific issue
│   ├── rca.md              /rca <key> — generate RCA
│   ├── trd.md              /trd <key> — generate TRD
│   └── graph.md            /graph — rebuild knowledge graph
│
└── templates/              Output templates
    ├── trd-template.md     TRD document structure
    ├── rca-template.md     RCA document structure
    └── plan-template.md    Final plan structure
```

## Key Concepts

### Skills vs Agents
- **Skills** are knowledge documents — they describe HOW to analyze (RCA methodology, TRD structure, etc.)
- **Agents** are executors — they READ a skill and DO the work
- Each agent loads the relevant skill + knowledge graph context before producing output

### The Knowledge Graph
The Obsidian vault is Pathfinder's understanding of the codebase landscape. Each repo file contains:
- What the service does
- Tech stack and key dependencies
- Architecture and key files
- API surface (routes, gRPC methods, Kafka topics)
- Common change patterns

The connections file maps how services talk to each other — gRPC calls, Kafka topics, shared models.

### Issue Classification
Pathfinder classifies issues using signals from the Linear issue:
- **Bug signals:** "Bug" label, error descriptions, "fix", "broken", "regression", stack traces
- **Feature signals:** "Feature"/"Improvement" label, "add", "implement", "new", "support", "enable"
- **Ambiguous:** When unclear, Pathfinder reads linked issues and comments for additional context

### HITL Gate
Pathfinder does NOT auto-assign or start coding. Moving to "In Progress" is the signal that:
1. Analysis is complete
2. A plan with RCA/TRD + code changes is attached
3. A human should review the plan before coding begins
