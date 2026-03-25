# ai-proxy

> OpenRouter proxy gateway — routes LLM requests to AI models via OpenRouter.

**Last updated:** 2026-03-25

## Overview

The AI proxy is a FastAPI-based gateway that proxies requests to OpenRouter API. It provides an OpenAI-compatible interface (`/v1/chat/completions`) that works with LangChain, LangGraph, OpenAI SDK, and other clients. It's a complete pass-through proxy — no request/response modification — with streaming support, tool calls, and observability via OpenTelemetry and Langfuse.

## Tech Stack

| Component | Technology |
|-----------|-----------|
| Language | Python 3.11+ |
| Package Manager | uv |
| Framework | FastAPI + Uvicorn |
| HTTP Client | httpx |
| Database | MongoDB (Motor) — for logging/analytics |
| Messaging | Kafka (aiokafka) |
| RPC | gRPC (for inter-service communication) |
| Observability | OpenTelemetry, structlog, Langfuse |
| Validation | Pydantic v2, pydantic-settings |
| Testing | pytest, pytest-asyncio, pytest-cov, pytest-mock, respx |

## Architecture

```
Client Request (OpenAI-compatible)
    POST /v1/chat/completions
    → FastAPI Router
        → Proxy Service
            → httpx → OpenRouter API
        → Response (JSON or SSE stream)
```

## Key Directories

| Directory | Purpose |
|-----------|---------|
| `app/api/` | API route definitions |
| `app/core/` | Configuration, settings |
| `app/services/` | Proxy logic, streaming handler |
| `app/models/` | Request/response models |
| `app/middleware/` | Request middleware |
| `app/grpc_/` | gRPC stubs (inter-service) |
| `app/helper/` | Utility helpers |
| `app/utils/` | Shared utilities |
| `tests/` | Test suite |

## API Surface

### REST Endpoints
| Method | Path | Purpose |
|--------|------|---------|
| GET | `/health` | Health check |
| POST | `/v1/chat/completions` | OpenAI-compatible chat completions (streaming + non-streaming) |

### Environment Variables
| Variable | Default | Purpose |
|----------|---------|---------|
| `OPENROUTER_API_KEY` | — | OpenRouter API key |
| `OPENROUTER_BASE_URL` | `https://openrouter.ai/api/v1` | OpenRouter base URL |
| `HOST` | `0.0.0.0` | Server host |
| `PORT` | `5511` | Server port |
| `LOG_LEVEL` | `info` | Logging level |

## Domain Concepts

This repo owns:
- **LLM routing** — proxying requests to OpenRouter
- **Streaming** — SSE streaming of LLM responses
- **Tool calls** — function calling pass-through
- **Model selection** — routing to specific models via OpenRouter
- **Usage tracking** — via Langfuse and OTel

## Common Change Patterns

| When you need to... | Look at... |
|---------------------|-----------|
| Add request transformation | `app/services/` (proxy service) |
| Add new endpoint | `app/api/` + `app/models/` |
| Modify streaming behavior | `app/services/` (streaming handler) |
| Add middleware | `app/middleware/` |
| Change model routing | `app/core/` (config/settings) |
| Add Kafka integration | `app/services/` + config |

## Code Style

- PEP 8, type hints
- async/await throughout
- Pydantic v2 for models
- structlog for logging
- pytest with asyncio_mode=auto

## Connections

- ← [[agent-gateway]] (HTTP: LLM requests)
- ← [[agent-platform-v2]] (HTTP: LLM requests from agent loop)
- → OpenRouter API (HTTPS: model inference)
- → MongoDB (logging/analytics)
- → Kafka (event streaming)
- → Langfuse (usage tracking)
