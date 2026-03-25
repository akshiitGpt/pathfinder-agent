# agent-platform-v2

> Core agent execution platform — where AI agents run, think, and use tools.

**Last updated:** 2026-03-25

## Overview

The agent platform is the brain of the Ruh AI system. It handles agent execution loops, tool orchestration, prompt management, and agent session state. When a user sends a message to an agent, this service processes it through the agent loop — selecting tools, calling LLMs, and producing responses.

## Tech Stack

| Component | Technology |
|-----------|-----------|
| Language | Python 3.x |
| Package Manager | Poetry |
| Framework | FastAPI (likely) |
| Async | asyncio throughout |
| Testing | pytest, pytest-asyncio, pytest-cov, pytest-mock |

## Architecture

```
Request (from agent-gateway via gRPC/Kafka)
    → Agent Loop (orchestration)
        → Tool Selection & Execution
        → LLM Call (via ai-proxy or direct)
        → Response Generation
    → Response (back to agent-gateway)
```

## Key Directories

| Directory | Purpose |
|-----------|---------|
| `app/agent/` | Core agent logic — loop, tools, execution |
| `app/shared/` | Shared utilities, config, logging |
| `app/` | Main application entry |
| `scripts/` | Utility scripts (e.g., `test_global_agent.py`) |
| `tests/` | Test suite |

## Domain Concepts

This repo owns:
- **Agent execution** — the main loop that processes user messages
- **Tool orchestration** — selecting, calling, and handling tool results
- **Prompt management** — system prompts, context building
- **Agent configuration** — agent settings, model selection

## API Surface

- Receives work from [[agent-gateway]] via gRPC or Kafka
- Calls [[ai-proxy]] for LLM inference
- Reads conversation history from [[communication-service]] via gRPC

## Common Change Patterns

| When you need to... | Look at... |
|---------------------|-----------|
| Add a new tool | `app/agent/tools/` |
| Modify the agent loop | `app/agent/` (main loop file) |
| Change prompt structure | `app/agent/` (prompt/context files) |
| Add a new agent config | `app/shared/config/` |

## Code Style

- Absolute imports from `app.` prefix
- PEP 8, type hints, max line 88 chars
- snake_case for vars/functions, PascalCase for classes
- All I/O operations async
- Structured logging via `app.shared.config.logging_config`
- **NEVER run git commands without explicit user permission**

## Connections

- → [[agent-gateway]] (receives requests from)
- → [[ai-proxy]] (sends LLM calls to)
- → [[communication-service]] (reads/writes conversation data via gRPC)
