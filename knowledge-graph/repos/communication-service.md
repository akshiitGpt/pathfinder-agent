# communication-service

> gRPC microservice for conversations and messages — the storage layer for all chat data.

**Last updated:** 2026-03-25

## Overview

The communication service is the data backbone for conversations and messages. It's a gRPC service backed by MongoDB (via Beanie ODM). It handles CRUD operations for conversations and messages, supports pagination, filtering, file versioning, and multi-platform data (Web, Telegram, SMS, Slack, Email). Every service that needs to read or write conversation data talks to this service.

## Tech Stack

| Component | Technology |
|-----------|-----------|
| Language | Python 3.11 |
| Package Manager | Poetry |
| Framework | gRPC (grpcio 1.71.0) |
| Database | MongoDB (async Motor + Beanie ODM) |
| Observability | OpenTelemetry + SigNoz, structlog, colorlog |
| Validation | Pydantic v2 |
| Testing | pytest, pytest-asyncio, factory-boy, faker, testcontainers |
| Linting | ruff, black (100 char), isort, mypy (strict) |

## Architecture

```
gRPC Request (from agent-gateway or other services)
    → CommunicationService (gRPC handler)
        → ConversationService / MessageService (business logic)
            → Beanie ODM
                → MongoDB
    → gRPC Response
```

## Key Files

| File | Purpose |
|------|---------|
| `app/services/communication_service.py` | Main gRPC service implementation |
| `app/services/conversation_service.py` | Conversation business logic |
| `app/services/message_service.py` | Message business logic |
| `app/models/conversation_model.py` | Conversation Beanie Document |
| `app/models/message_model.py` | Message Beanie Document |
| `app/db/mongo.py` | MongoDB connection and initialization |
| `app/utils/` | Utility functions |
| `scripts/migrations/` | Database migration scripts |
| `tests/` | Test suite (unit, integration) |

## Data Models

### Conversation
- Belongs to a user and organization
- Linked to an agent
- Has platform-specific behavior
- Statuses: active, archived, deleted

### Message
- Belongs to a conversation (via `conversation_id`)
- Sender types: USER, ASSISTANT
- Message types: USER_MESSAGE, WORKFLOW, CHAT, MCP
- Statuses: RUNNING, COMPLETED
- Supports file attachments with version history (APPEND/RESTORE)

### Key Enums
| Enum | Values |
|------|--------|
| ChatType | (various chat types) |
| Platform | WEB, TELEGRAM, SMS, SLACK, EMAIL |
| ConversationStatus | (active, archived, deleted) |
| SenderType | USER, ASSISTANT |
| MessageType | USER_MESSAGE, WORKFLOW, CHAT, MCP |
| MessageStatus | RUNNING, COMPLETED |

## API Surface

### gRPC Methods (CommunicationService)
- Conversation CRUD (create, get, list, update, delete)
- Message CRUD (create, get, list, update)
- Pagination and filtering
- File versioning operations

### Environment Variables
| Variable | Default | Purpose |
|----------|---------|---------|
| `GRPC_PORT` | 50055 | gRPC server port |
| `MONGO_HOST` | — | MongoDB host |
| `MONGO_PORT` | 27017 | MongoDB port |
| `MONGO_DB_NAME` | communication-service | Database name |

## Domain Concepts

This repo owns:
- **Conversations** — CRUD, status management, platform behavior
- **Messages** — CRUD, sender tracking, type classification
- **File versioning** — attachment versioning (APPEND/RESTORE)
- **Chat history** — pagination, filtering, ordering

## Common Change Patterns

| When you need to... | Look at... |
|---------------------|-----------|
| Add a field to conversations | `app/models/conversation_model.py` + `communication_service.py` |
| Add a field to messages | `app/models/message_model.py` + `message_service.py` |
| Add a new gRPC method | `app/services/communication_service.py` + proto file |
| Add a migration | `scripts/migrations/` |
| Change MongoDB queries | `app/services/conversation_service.py` or `message_service.py` |
| Add new enum values | `app/models/` (relevant model file) |

## Code Style

- PEP 8 with 100-char line limit
- Type hints everywhere, mypy strict
- async/await for all I/O
- Beanie Documents with `Settings` inner class
- `to_dict()` and `to_proto()` methods on models
- gRPC StatusCode for errors
- isort with black profile
- Singleton pattern for service instances
- **NEVER run git commands without explicit user permission**

## Connections

- ← [[agent-gateway]] (gRPC: conversation/message CRUD)
- ← [[agent-platform-v2]] (gRPC: read conversation history)
- → MongoDB (data storage)
