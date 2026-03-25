# agent-gateway

> Developer API gateway — REST/SSE entry point for all agent interactions.

**Last updated:** 2026-03-25

## Overview

The agent gateway is the front door to the Ruh AI platform. It exposes REST APIs for developers, handles authentication (JWT + API keys), manages sessions, streams responses via SSE, and routes requests to backend services via gRPC and Kafka. Every external request enters through this service.

## Tech Stack

| Component | Technology |
|-----------|-----------|
| Language | Python 3.13 |
| Package Manager | Poetry |
| Framework | FastAPI + Uvicorn |
| Database | Redis (caching/streams), MongoDB (via gRPC to communication-service) |
| Messaging | Kafka (aiokafka) |
| RPC | gRPC (to communication-service, user-service, agent-service) |
| Auth | JWT, API Keys, SDR-Auth-Key |
| Observability | OpenTelemetry + SigNoz, structlog |
| Validation | Pydantic v2 |
| Storage | Google Cloud Storage |
| External | Twilio, Telegram Bot API, OpenRouter |

## Architecture

```
Client Request (REST/SSE)
    → FastAPI Router (auth middleware: JWT/API-Key/SDR-Auth-Key)
        → Rate Limiting
        → OTel tracing
    → Service Layer
        → gRPC clients (communication-service, user-service, agent-service)
        → Kafka producer (agent_chat_requests)
        → Redis (caching, stream buffering)
    → Response (JSON or SSE stream)
```

## Key Files

| File/Directory | Purpose |
|---------------|---------|
| `app/routers/` | REST endpoint definitions |
| `app/core/` | Auth, config, middleware, dependencies |
| `app/services/` | Business logic layer |
| `app/schemas/` | Pydantic request/response models |
| `app/utils/redis/` | Redis client and stream utilities |
| `app/shared/` | Shared utilities |
| `app/platforms/` | Platform-specific logic (Telegram, SMS, etc.) |
| `app/grpc_/` | gRPC client stubs and wrappers |
| `app/helper/` | Helper utilities |
| `tests/` | Test suite (unit, integration, security, contract, smoke, e2e) |

## API Surface

### REST Endpoints (key ones)
- `POST /agents/chat/stream` — SSE streaming chat
- Agent management endpoints
- Session management endpoints
- API key management endpoints
- Webhook endpoints (Twilio, Telegram)

### Kafka Topics
| Topic | Direction | Purpose |
|-------|-----------|---------|
| `agent_chat_requests` | Producer | Send chat requests to agent-platform |
| `agent_chat_responses` | Consumer | Receive agent responses |
| `agent_session_deletion_requests` | Producer | Request session cleanup |
| `agent_task_requests` | Producer | Send task requests |

### Redis Key Patterns
| Pattern | Purpose |
|---------|---------|
| `{env}:agent:chat:requests` | Chat request queue |
| `{env}:agent:chat:responses:{conversation_id}-{request_id}` | Response stream |
| `{env}:summarisation:save` | Summarization cache |

### Authentication Schemes
| Scheme | Header | Purpose |
|--------|--------|---------|
| Bearer JWT | `Authorization: Bearer <token>` | User authentication |
| API Key | `X-API-Key: <key>` | Developer API access |
| SDR Auth | `X-SDR-Auth-Key: <key>` | Internal SDR auth |

## Domain Concepts

This repo owns:
- **API routing** — all external-facing endpoints
- **Authentication** — JWT, API key, SDR auth
- **Rate limiting** — per-key, per-user limits
- **SSE streaming** — real-time response streaming
- **Session management** — conversation sessions
- **Platform integrations** — Twilio, Telegram, webhooks
- **API key management** — CRUD for developer API keys

## Common Change Patterns

| When you need to... | Look at... |
|---------------------|-----------|
| Add a new REST endpoint | `app/routers/` + `app/schemas/` + `app/services/` |
| Add new auth scheme | `app/core/auth.py` or `app/core/dependencies.py` |
| Add gRPC client call | `app/grpc_/` + `app/services/` |
| Add Kafka topic | `app/services/` (producer) + config |
| Modify streaming | `app/utils/redis/` + `app/services/` |
| Add platform webhook | `app/platforms/` + `app/routers/` |
| Add rate limit rule | `app/core/` (rate limiting) |

## External Service Dependencies

| Service | Protocol | Port | Purpose |
|---------|----------|------|---------|
| [[communication-service]] | gRPC | 50055 | Conversation/message CRUD |
| user-service | gRPC | 50052 | User data |
| agent-service | gRPC | 50057 | Agent config |
| Redis | TCP | 6379 | Caching + streams |
| Kafka | TCP | 9092 | Async messaging |
| OpenRouter | HTTPS | — | LLM fallback |
| Twilio | HTTPS | — | SMS/voice |
| Telegram Bot API | HTTPS | — | Telegram integration |
| GCS | HTTPS | — | File storage |

## Code Style

- PEP 8, type hints throughout
- async/await for all I/O
- Pydantic v2 for all schemas
- structlog for logging
- OTel spans for tracing
- FastAPI Depends() for DI
- **NEVER run git commands without explicit user permission**

## Connections

- ← External clients (REST/SSE)
- → [[communication-service]] (gRPC: conversations, messages)
- → [[agent-platform-v2]] (Kafka: chat requests)
- → [[ai-proxy]] (HTTP: LLM calls, fallback)
- → user-service (gRPC: user data)
- → agent-service (gRPC: agent config)
