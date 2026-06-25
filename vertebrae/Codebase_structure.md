# QNC — Complete Codebase Architecture Blueprint

> **Document scope:** Based on `QNC软件架构设计.txt` (Software Architecture Design) and  
> `QNC_产品策略与概念审查.md` (Product Strategy & Concept Review).  
> Target: production-grade, scalable, maintainable monorepo.

---

## Table of Contents

1. [High-Level Architecture Overview](#1-high-level-architecture-overview)
2. [Recommended Tech Stack](#2-recommended-tech-stack)
3. [Folder and Module Structure](#3-folder-and-module-structure)
4. [Separation of Concerns](#4-separation-of-concerns)
5. [Backend Architecture](#5-backend-architecture)
6. [Frontend Architecture](#6-frontend-architecture)
7. [Database Layer Structure](#7-database-layer-structure)
8. [API Design Organization](#8-api-design-organization)
9. [Shared Utilities and Libraries](#9-shared-utilities-and-libraries)
10. [Authentication and Authorization Structure](#10-authentication-and-authorization-structure)
11. [Configuration and Environment Management](#11-configuration-and-environment-management)
12. [Testing Structure](#12-testing-structure)
13. [DevOps / Deployment Structure](#13-devops--deployment-structure)
14. [Naming Conventions](#14-naming-conventions)
15. [Scalability and Maintainability Considerations](#15-scalability-and-maintainability-considerations)
16. [Design Patterns](#16-design-patterns)
17. [Example File Tree](#17-example-file-tree)
18. [Assumptions](#18-assumptions)
19. [Improvement Suggestions](#19-improvement-suggestions)

---

## 1. High-Level Architecture Overview

QNC is a **software-defined robotics gateway** with three clearly distinct runtime tiers:

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         MANAGEMENT PLANE                                │
│                  (Web Dashboard / CLI / REST API)                       │
│          Device registry · Descriptor CRUD · Health monitoring          │
└───────────────────────────┬─────────────────────────────────────────────┘
                            │  REST / WebSocket (JSON)
┌───────────────────────────▼─────────────────────────────────────────────┐
│                          CONTROL PLANE                                  │
│                    QNC Core Gateway Service                             │
│  ┌──────────────┐  ┌──────────────────┐  ┌───────────────────────────┐ │
│  │CapabilityMgr │  │DescriptorEngine  │  │  BridgeStateMachine       │ │
│  │              │  │  (JSON → cmds)   │  │  DISCONNECTED→OPERATIONAL │ │
│  └──────────────┘  └──────────────────┘  └───────────────────────────┘ │
│  ┌──────────────────────────────────────────────────────────────────┐   │
│  │                  Universal IDL Layer                             │   │
│  │   GripperCommand · GripperState · GripperCapabilities            │   │
│  └──────────────────────────────────────────────────────────────────┘   │
└───────────────────────────┬─────────────────────────────────────────────┘
                            │  Protocol-specific adapters
┌───────────────────────────▼─────────────────────────────────────────────┐
│                           DATA PLANE                                    │
│               Protocol Adaptation Layer                                 │
│  ┌────────────┐ ┌──────────┐ ┌──────────────┐ ┌────────────────────┐   │
│  │Modbus RTU  │ │ CAN Bus  │ │  TCP/IP SDK  │ │  Serial ASCII      │   │
│  │ Adapter    │ │ Adapter  │ │  Adapter     │ │  Adapter           │   │
│  └────────────┘ └──────────┘ └──────────────┘ └────────────────────┘   │
│                          [VendorX Adapter]                              │
└───────────────────────────┬─────────────────────────────────────────────┘
                            │  RS-485 / CAN / Ethernet / USB-Serial
              ┌─────────────▼───────────────────┐
              │         Physical Devices         │
              │  Grippers · Sensors · Actuators  │
              └──────────────────────────────────┘
```

### Data Flow (read path)

```
Device HW → Adapter (raw bytes)
         → IDL mapping (GripperState)
         → Capability-gated topic publish (DDS / FastDDS)
         → Robot Platform subscribes
         → Dashboard WebSocket stream
```

### Data Flow (write path)

```
Robot Platform sends GripperCommand (IDL)
→ BridgeStateMachine validates OPERATIONAL
→ DescriptorEngine resolves capability
→ Adapter translates to wire protocol
→ Device executes
→ GripperState update published
```

---

## 2. Recommended Tech Stack

### Decision rationale

The documents specify DDS/FastDDS as the real-time messaging substrate (NeuraSync), JSON descriptors for device semantics, and multi-protocol industrial fieldbus support. The management layer is implicitly web-based. No language is mandated.

| Layer | Choice | Rationale |
|---|---|---|
| **Gateway core** | **Python 3.11+** | Rich robotics/industrial ecosystem (pymodbus, python-can, FastDDS Python bindings); rapid adapter authoring; strong async support |
| **High-perf adapter hot path** | **C++ 17 (optional)** | For adapters where <1 ms latency is critical; wrap as Python extension via pybind11 |
| **DDS messaging** | **Eclipse Cyclone DDS / Fast-DDS** | Open-source; ROS 2 compatible; supports Python and C++ |
| **IDL definition** | **OMG IDL 4.x → DDS-XTYPES** | Standard; language-agnostic; tooling generates Python/C++ stubs |
| **REST / WebSocket API** | **FastAPI (Python)** | Async; OpenAPI auto-generation; Pydantic models align with IDL types |
| **Frontend dashboard** | **React 18 + TypeScript** | Component ecosystem; excellent WebSocket support; strong typing |
| **Frontend state** | **Zustand** | Lightweight; fits real-time streaming data; avoids Redux boilerplate |
| **UI library** | **shadcn/ui + Tailwind CSS** | Consistent industrial-grade UI |
| **Primary database** | **PostgreSQL 15** | Device registry, descriptor store, event log; ACID; JSONB for descriptors |
| **Time-series store** | **TimescaleDB** (PostgreSQL extension) | Gripper telemetry; native SQL; avoids separate TSDB ops |
| **Cache / pub-sub** | **Redis 7** | Session cache; real-time state cache; optional pub-sub bridge |
| **Message queue** | **RabbitMQ** (optional) | Async command dispatch; decouples dashboard from gateway |
| **Config management** | **Pydantic Settings + dotenv** | Type-safe; environment-driven |
| **Container runtime** | **Docker + Docker Compose** | Dev parity; also Kubernetes-ready |
| **Container orchestration** | **Kubernetes (k3s for edge)** | Gateway runs on edge compute; k3s is lightweight for ARM/x86 |
| **Observability** | **Prometheus + Grafana** | Metrics; latency histograms per adapter |
| **Tracing** | **OpenTelemetry** | Distributed tracing across gateway + API layers |
| **CI/CD** | **GitHub Actions** | Pipeline definition; artifact publishing |

### Alternative considered: Go for gateway core

Go offers better goroutine concurrency and lower memory footprint. However, Python's pymodbus, python-can, and FastDDS bindings are significantly more mature. **Recommendation: Python with optional C++ extensions for hot paths.**

---

## 3. Folder and Module Structure

QNC uses a **monorepo** with Nx-style workspace tooling. Each package is independently versioned and deployable.

```
qnc/
├── packages/
│   ├── gateway/          # Core gateway service (Python)
│   ├── api-server/       # REST + WebSocket management API (Python/FastAPI)
│   ├── dashboard/        # Web UI (React/TypeScript)
│   ├── idl/              # IDL definitions + generated stubs
│   ├── adapters/         # Protocol adapters (one sub-package per protocol)
│   │   ├── modbus/
│   │   ├── canbus/
│   │   ├── tcpip/
│   │   ├── serial/
│   │   └── vendorx/
│   ├── descriptor-engine/ # JSON descriptor parser + executor
│   ├── shared-py/        # Shared Python utilities
│   └── shared-ts/        # Shared TypeScript types and utilities
├── infra/
│   ├── docker/
│   ├── k8s/
│   └── terraform/
├── scripts/
├── docs/
└── tests/
    ├── integration/
    ├── e2e/
    └── hardware-in-loop/
```

---

## 4. Separation of Concerns

```
┌──────────────────────────────────────────────────────────────────┐
│  PRESENTATION       Dashboard (React)                            │
│                     CLI tool (Typer/Python)                      │
├──────────────────────────────────────────────────────────────────┤
│  APPLICATION        api-server/routers (FastAPI route handlers)  │
│                     gateway/services (business logic)            │
│                     gateway/state_machine                        │
│                     descriptor-engine/executor                   │
├──────────────────────────────────────────────────────────────────┤
│  DOMAIN             idl/ (GripperCommand, GripperState, etc.)    │
│                     gateway/domain/capabilities                  │
│                     gateway/domain/errors                        │
├──────────────────────────────────────────────────────────────────┤
│  INFRASTRUCTURE     adapters/* (Modbus, CAN, TCP, Serial)        │
│                     gateway/transport/dds_publisher              │
│                     api-server/db/repositories                   │
│                     api-server/cache/redis_client                │
└──────────────────────────────────────────────────────────────────┘
```

**Key invariants:**
- Domain layer has **zero** external dependencies (no DB, no network, no DDS)
- Adapters depend on domain IDL types, never on application services
- Application services depend on domain; inject infrastructure via interfaces
- The DDS publish path is **never blocked** by database I/O

---

## 5. Backend Architecture

### 5.1 Gateway Service (`packages/gateway/`)

The gateway is the real-time core. It is **not** a web server — it runs as a long-lived process managing device connections and DDS publishing.

```
gateway/
├── __init__.py
├── main.py                    # Entry point: initialise DDS, load adapters, start loop
├── config.py                  # Pydantic settings
├── domain/
│   ├── __init__.py
│   ├── commands.py            # GripperCommand dataclass (mirrors IDL)
│   ├── state.py               # GripperState, GripStatus enum
│   ├── capabilities.py        # GripperCapabilities dataclass
│   ├── errors.py              # ErrorSeverity enum: WARNING / CRITICAL / FATAL
│   └── events.py              # Internal domain events (DeviceConnected, etc.)
├── state_machine/
│   ├── __init__.py
│   ├── bridge_fsm.py          # BridgeState enum + FSM implementation
│   │                          # DISCONNECTED→PROTOCOL_DETECTED→INITIALIZING→OPERATIONAL
│   └── transitions.py         # Transition guards and actions
├── services/
│   ├── __init__.py
│   ├── capability_service.py  # Reads capabilities; caches; exposes to API
│   ├── command_service.py     # Validates and dispatches GripperCommand
│   ├── discovery_service.py   # Hardware ID → descriptor lookup (local + cloud)
│   └── telemetry_service.py   # Batches GripperState → TimescaleDB
├── transport/
│   ├── __init__.py
│   ├── dds_node.py            # CycloneDDS participant, publisher, subscriber
│   ├── dds_topics.py          # Topic name constants + QoS profiles
│   └── redis_bridge.py        # Optional: bridge DDS to Redis pub/sub
├── registry/
│   ├── __init__.py
│   ├── adapter_registry.py    # Maps protocol → adapter class (plugin pattern)
│   └── device_registry.py     # Runtime device connection state
└── ports/
    ├── __init__.py
    ├── adapter_port.py        # Abstract base: IDeviceAdapter interface
    └── descriptor_port.py     # Abstract base: IDescriptorStore interface
```

#### Bridge State Machine

```python
# gateway/state_machine/bridge_fsm.py

from enum import Enum, auto
from dataclasses import dataclass, field
from typing import Optional
import logging

class BridgeState(Enum):
    DISCONNECTED        = auto()
    PROTOCOL_DETECTED   = auto()
    INITIALIZING        = auto()
    OPERATIONAL         = auto()
    FAULTED             = auto()

VALID_TRANSITIONS: dict[BridgeState, set[BridgeState]] = {
    BridgeState.DISCONNECTED:      {BridgeState.PROTOCOL_DETECTED},
    BridgeState.PROTOCOL_DETECTED: {BridgeState.INITIALIZING, BridgeState.DISCONNECTED},
    BridgeState.INITIALIZING:      {BridgeState.OPERATIONAL, BridgeState.FAULTED, BridgeState.DISCONNECTED},
    BridgeState.OPERATIONAL:       {BridgeState.DISCONNECTED, BridgeState.FAULTED},
    BridgeState.FAULTED:           {BridgeState.DISCONNECTED},
}

@dataclass
class BridgeFSM:
    device_id: str
    state: BridgeState = BridgeState.DISCONNECTED
    _log: logging.Logger = field(init=False, repr=False)

    def __post_init__(self):
        self._log = logging.getLogger(f"bridge.{self.device_id}")

    def transition(self, target: BridgeState) -> None:
        if target not in VALID_TRANSITIONS[self.state]:
            raise InvalidTransitionError(self.state, target)
        self._log.info("FSM %s -> %s", self.state.name, target.name)
        self.state = target

    @property
    def is_operational(self) -> bool:
        return self.state == BridgeState.OPERATIONAL
```

#### Adapter Interface (Port)

```python
# gateway/ports/adapter_port.py

from abc import ABC, abstractmethod
from ..domain.commands import GripperCommand
from ..domain.state import GripperState
from ..domain.capabilities import GripperCapabilities

class IDeviceAdapter(ABC):
    """
    Every protocol adapter must implement this interface.
    The gateway core never imports concrete adapter classes directly —
    only this interface. Adapters are loaded via the registry.
    """

    @abstractmethod
    async def connect(self) -> None: ...

    @abstractmethod
    async def disconnect(self) -> None: ...

    @abstractmethod
    async def read_capabilities(self) -> GripperCapabilities: ...

    @abstractmethod
    async def send_command(self, cmd: GripperCommand) -> None: ...

    @abstractmethod
    async def read_state(self) -> GripperState: ...

    @abstractmethod
    async def health_check(self) -> bool: ...
```

### 5.2 API Server (`packages/api-server/`)

FastAPI application providing REST endpoints for the dashboard and external integrations. Runs separately from the gateway — communicates with it via Redis or direct IPC.

```
api-server/
├── __init__.py
├── main.py                    # FastAPI app factory
├── config.py
├── routers/
│   ├── __init__.py
│   ├── devices.py             # GET/POST/DELETE /devices
│   ├── descriptors.py         # CRUD /descriptors
│   ├── capabilities.py        # GET /devices/{id}/capabilities
│   ├── commands.py            # POST /devices/{id}/commands
│   ├── telemetry.py           # GET /devices/{id}/telemetry (REST + WebSocket)
│   ├── health.py              # GET /health, /readyz, /livez
│   └── auth.py                # POST /auth/token, /auth/refresh
├── schemas/                   # Pydantic request/response models
│   ├── __init__.py
│   ├── device.py
│   ├── descriptor.py
│   ├── command.py
│   ├── state.py
│   └── auth.py
├── services/                  # Application-layer use cases
│   ├── __init__.py
│   ├── device_service.py
│   ├── descriptor_service.py
│   └── command_dispatch_service.py
├── db/
│   ├── __init__.py
│   ├── base.py                # SQLAlchemy declarative base
│   ├── session.py             # Async engine + session factory
│   ├── models/                # ORM models
│   │   ├── device.py
│   │   ├── descriptor.py
│   │   ├── event_log.py
│   │   └── user.py
│   └── repositories/          # Repository pattern: one class per aggregate
│       ├── __init__.py
│       ├── device_repo.py
│       ├── descriptor_repo.py
│       └── event_log_repo.py
├── cache/
│   ├── __init__.py
│   └── redis_client.py        # Async Redis; used for session + state cache
├── middleware/
│   ├── __init__.py
│   ├── auth_middleware.py     # JWT validation
│   ├── rate_limit.py          # Sliding-window rate limiter
│   └── request_id.py          # X-Request-ID injection
└── dependencies/
    ├── __init__.py
    ├── auth.py                # FastAPI Depends() for current user
    └── db.py                  # FastAPI Depends() for DB session
```

---

## 6. Frontend Architecture

### 6.1 Dashboard (`packages/dashboard/`)

React 18 + TypeScript SPA. Communicates with `api-server` over REST and WebSocket.

```
dashboard/
├── public/
├── src/
│   ├── main.tsx
│   ├── App.tsx
│   ├── router/
│   │   └── index.tsx           # React Router v6 route definitions
│   ├── pages/
│   │   ├── Devices/
│   │   │   ├── DeviceList.tsx
│   │   │   ├── DeviceDetail.tsx
│   │   │   └── DeviceCreate.tsx
│   │   ├── Descriptors/
│   │   │   ├── DescriptorList.tsx
│   │   │   └── DescriptorEditor.tsx  # Monaco-based JSON editor
│   │   ├── Telemetry/
│   │   │   └── TelemetryDashboard.tsx
│   │   └── Auth/
│   │       ├── Login.tsx
│   │       └── Profile.tsx
│   ├── components/
│   │   ├── ui/                 # shadcn/ui re-exports
│   │   ├── GripperStateCard/
│   │   ├── CapabilityBadge/
│   │   ├── BridgeStatusIndicator/
│   │   ├── TelemetryChart/     # Recharts-based real-time chart
│   │   └── CommandPanel/       # Send GripperCommand UI
│   ├── hooks/
│   │   ├── useDeviceStream.ts  # WebSocket → Zustand state sync
│   │   ├── useCapabilities.ts
│   │   └── useAuth.ts
│   ├── stores/                 # Zustand stores
│   │   ├── deviceStore.ts
│   │   ├── telemetryStore.ts
│   │   └── authStore.ts
│   ├── api/                    # Generated OpenAPI client (openapi-typescript-codegen)
│   │   ├── client.ts
│   │   └── endpoints/
│   ├── types/                  # TypeScript types mirroring IDL domain models
│   │   ├── gripper.ts
│   │   ├── descriptor.ts
│   │   └── device.ts
│   └── utils/
│       ├── formatters.ts
│       └── validators.ts
├── tsconfig.json
├── vite.config.ts
└── package.json
```

### 6.2 State Management Strategy

```
WebSocket stream
      │
      ▼
useDeviceStream hook
      │ parses GripperState JSON
      ▼
telemetryStore (Zustand)   ──→  TelemetryChart (subscribes)
deviceStore (Zustand)       ──→  BridgeStatusIndicator
                                 GripperStateCard
```

Feature flags derived from `GripperCapabilities` are stored in `deviceStore` and gate UI elements (e.g., force control slider is rendered only if `has_force_control === true`).

---

## 7. Database Layer Structure

### 7.1 Schema design

```sql
-- Device registry
CREATE TABLE devices (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name            TEXT NOT NULL,
    protocol        TEXT NOT NULL,            -- 'modbus_rtu' | 'canbus' | 'tcpip' | 'serial'
    connection_uri  TEXT NOT NULL,            -- e.g. 'modbus+rtu:///dev/ttyUSB0?baudrate=115200'
    hardware_id_raw INTEGER,                  -- ADC value only; interpreted by descriptor engine
    descriptor_id   UUID REFERENCES descriptors(id),
    bridge_state    TEXT NOT NULL DEFAULT 'DISCONNECTED',
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- JSON descriptor store
CREATE TABLE descriptors (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name            TEXT NOT NULL,
    vendor          TEXT,
    model           TEXT,
    protocol        TEXT NOT NULL,
    version         SEMVER NOT NULL DEFAULT '1.0.0',
    schema          JSONB NOT NULL,           -- Full descriptor JSON
    checksum        TEXT NOT NULL,            -- SHA-256 of schema
    is_validated    BOOLEAN NOT NULL DEFAULT FALSE,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);
CREATE INDEX idx_descriptors_vendor_model ON descriptors(vendor, model);

-- Capability cache (materialised from descriptor + live device)
CREATE TABLE device_capabilities (
    device_id           UUID PRIMARY KEY REFERENCES devices(id) ON DELETE CASCADE,
    has_force_control   BOOLEAN NOT NULL DEFAULT FALSE,
    has_velocity_control BOOLEAN NOT NULL DEFAULT FALSE,
    has_multi_finger    BOOLEAN NOT NULL DEFAULT FALSE,
    has_tactile_sensor  BOOLEAN NOT NULL DEFAULT FALSE,
    num_dof             SMALLINT NOT NULL DEFAULT 1,
    max_force_n         REAL,
    raw_capabilities    JSONB,               -- Full deserialized GripperCapabilities
    refreshed_at        TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- Telemetry (TimescaleDB hypertable)
CREATE TABLE gripper_telemetry (
    time            TIMESTAMPTZ NOT NULL,
    device_id       UUID NOT NULL REFERENCES devices(id),
    position        REAL,
    effort          REAL,
    grip_status     TEXT,
    error_code      SMALLINT,
    is_activated    BOOLEAN
);
SELECT create_hypertable('gripper_telemetry', 'time');
CREATE INDEX ON gripper_telemetry (device_id, time DESC);

-- Immutable audit / event log
CREATE TABLE event_log (
    id          BIGSERIAL PRIMARY KEY,
    occurred_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    device_id   UUID REFERENCES devices(id),
    event_type  TEXT NOT NULL,
    severity    TEXT NOT NULL,               -- INFO | WARNING | CRITICAL | FATAL
    payload     JSONB
);
CREATE INDEX idx_event_log_device_time ON event_log(device_id, occurred_at DESC);

-- Users
CREATE TABLE users (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email           TEXT UNIQUE NOT NULL,
    hashed_password TEXT NOT NULL,
    role            TEXT NOT NULL DEFAULT 'operator',  -- admin | operator | viewer
    is_active       BOOLEAN NOT NULL DEFAULT TRUE,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

### 7.2 Repository Pattern

```python
# api-server/db/repositories/device_repo.py

from uuid import UUID
from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy import select
from ..models.device import DeviceModel
from ...schemas.device import DeviceCreate

class DeviceRepository:
    def __init__(self, session: AsyncSession) -> None:
        self._session = session

    async def get_by_id(self, device_id: UUID) -> DeviceModel | None:
        result = await self._session.execute(
            select(DeviceModel).where(DeviceModel.id == device_id)
        )
        return result.scalar_one_or_none()

    async def list_all(self) -> list[DeviceModel]:
        result = await self._session.execute(select(DeviceModel))
        return list(result.scalars().all())

    async def create(self, data: DeviceCreate) -> DeviceModel:
        device = DeviceModel(**data.model_dump())
        self._session.add(device)
        await self._session.flush()
        return device
```

### 7.3 Migrations

Use **Alembic** for schema migrations:

```
api-server/
└── alembic/
    ├── env.py
    ├── alembic.ini
    └── versions/
        ├── 0001_initial_schema.py
        ├── 0002_add_device_capabilities.py
        └── 0003_add_telemetry_hypertable.py
```

---

## 8. API Design Organization

### 8.1 REST API surface

All endpoints under `/api/v1/`. Version prefix in path (not header) for reverse-proxy compatibility.

```
Authentication
  POST   /api/v1/auth/token          # Issue JWT (username + password)
  POST   /api/v1/auth/refresh        # Refresh access token
  DELETE /api/v1/auth/token          # Logout (revoke refresh token)

Devices
  GET    /api/v1/devices             # List all devices
  POST   /api/v1/devices             # Register new device
  GET    /api/v1/devices/{id}        # Get device detail + current bridge state
  PUT    /api/v1/devices/{id}        # Update device config
  DELETE /api/v1/devices/{id}        # Deregister device

  GET    /api/v1/devices/{id}/capabilities    # Current GripperCapabilities
  GET    /api/v1/devices/{id}/state           # Current GripperState (polled)
  POST   /api/v1/devices/{id}/commands        # Dispatch GripperCommand
  GET    /api/v1/devices/{id}/events          # Paginated event log

Descriptors
  GET    /api/v1/descriptors         # List available descriptors
  POST   /api/v1/descriptors         # Upload / register new descriptor
  GET    /api/v1/descriptors/{id}    # Get full descriptor JSON
  PUT    /api/v1/descriptors/{id}    # Update descriptor (creates new version)
  DELETE /api/v1/descriptors/{id}

  POST   /api/v1/descriptors/{id}/validate   # Trigger validation against a device

Telemetry
  GET    /api/v1/devices/{id}/telemetry      # Query historical (time range, resolution)
  WS     /api/v1/devices/{id}/stream         # WebSocket real-time state stream

Health
  GET    /api/v1/health              # Liveness
  GET    /api/v1/readyz              # Readiness (DB + Redis connectivity)
  GET    /api/v1/metrics             # Prometheus text format
```

### 8.2 WebSocket message protocol

```typescript
// From server to client
interface DeviceStreamMessage {
  type: "state" | "event" | "capability_update" | "bridge_state_change";
  device_id: string;
  timestamp: string;          // ISO-8601
  payload: GripperState | EventPayload | GripperCapabilities | BridgeStatePayload;
}

// From client to server (command injection via WebSocket)
interface DeviceCommandMessage {
  type: "command";
  request_id: string;
  payload: GripperCommandPayload;
}
```

### 8.3 Error response schema

```json
{
  "error": {
    "code": "DEVICE_NOT_OPERATIONAL",
    "message": "Device abc123 is in state INITIALIZING, not OPERATIONAL",
    "severity": "WARNING",
    "request_id": "req_01HX...",
    "timestamp": "2024-11-01T10:23:45Z"
  }
}
```

### 8.4 Latency contract

As noted in the product review, the 8 ms "typical" figure is protocol-dependent. The API exposes a `latency_profile` per device in the capabilities response:

```json
{
  "latency_profile": {
    "typical_ms": 42,
    "formula": "((10 * byte_size) / baud_rate) * 1000 + processing_overhead_ms",
    "baud_rate": 9600,
    "byte_size": 10
  }
}
```

---

## 9. Shared Utilities and Libraries

### 9.1 `packages/shared-py/`

```
shared-py/
├── qnc_shared/
│   ├── __init__.py
│   ├── logging.py             # Structured JSON logger (structlog)
│   ├── tracing.py             # OpenTelemetry span helpers
│   ├── retry.py               # Exponential backoff decorator
│   ├── pagination.py          # Cursor-based pagination helpers
│   ├── serialization.py       # IDL ↔ JSON round-trip helpers
│   ├── validation.py          # JSON Schema validator for descriptors
│   └── errors/
│       ├── __init__.py
│       ├── base.py            # QNCBaseError with severity
│       ├── device_errors.py
│       └── protocol_errors.py
└── pyproject.toml
```

### 9.2 `packages/shared-ts/`

```
shared-ts/
├── src/
│   ├── types/
│   │   ├── gripper.ts         # GripperCommand, GripperState, GripperCapabilities
│   │   ├── device.ts
│   │   ├── descriptor.ts
│   │   └── events.ts
│   ├── utils/
│   │   ├── formatters.ts
│   │   └── validators.ts
│   └── constants/
│       ├── bridgeStates.ts
│       └── gripStatus.ts
├── tsconfig.json
└── package.json
```

### 9.3 `packages/idl/`

IDL source files and generated language bindings:

```
idl/
├── src/
│   ├── GripperCommand.idl
│   ├── GripperState.idl
│   ├── GripperCapabilities.idl
│   └── ErrorSeverity.idl
├── generated/
│   ├── python/                # Auto-generated by idlc or cyclone-dds tooling
│   │   ├── GripperCommand.py
│   │   ├── GripperState.py
│   │   └── GripperCapabilities.py
│   └── cpp/                   # For C++ performance-critical adapters
│       ├── GripperCommand.hpp
│       └── ...
└── Makefile                   # `make idl` regenerates all bindings
```

Example IDL:

```idl
// idl/src/GripperCommand.idl
module qnc {
  module idl {

    enum GripperMode {
      POSITION,
      FORCE,
      VELOCITY,
      PRESET
    };

    @topic
    struct GripperCommand {
      @key string<64> device_id;
      @optional float target_position;   // 0.0 (closed) – 1.0 (open)
      @optional float max_effort;        // Newtons; 0 = no limit
      GripperMode mode;
      boolean activate;
      boolean stop;
    };

  };
};
```

---

## 10. Authentication and Authorization Structure

### 10.1 Authentication flow

```
Client → POST /api/v1/auth/token { email, password }
       ← { access_token (JWT, 15min), refresh_token (opaque, 7d) }

Client → GET /api/v1/devices  [Authorization: Bearer <access_token>]
       ← 200 OK

Client → POST /api/v1/auth/refresh { refresh_token }
       ← { access_token (new) }
```

Refresh tokens are stored in Redis with device fingerprint binding. Revocation is immediate (delete from Redis).

### 10.2 JWT payload structure

```json
{
  "sub": "user-uuid",
  "email": "operator@example.com",
  "role": "operator",
  "iat": 1700000000,
  "exp": 1700000900,
  "jti": "unique-token-id"
}
```

### 10.3 Role-based access control (RBAC)

| Role | Permissions |
|---|---|
| `admin` | Full access: user management, descriptor upload, device delete |
| `operator` | Send commands, view telemetry, manage descriptors |
| `viewer` | Read-only: device list, telemetry, capabilities |

```python
# api-server/dependencies/auth.py

from fastapi import Depends, HTTPException, status
from .models import User, Role

class RequireRole:
    def __init__(self, *roles: Role):
        self.roles = set(roles)

    def __call__(self, current_user: User = Depends(get_current_user)) -> User:
        if current_user.role not in self.roles:
            raise HTTPException(status_code=status.HTTP_403_FORBIDDEN)
        return current_user

require_admin    = RequireRole(Role.ADMIN)
require_operator = RequireRole(Role.ADMIN, Role.OPERATOR)
require_viewer   = RequireRole(Role.ADMIN, Role.OPERATOR, Role.VIEWER)
```

### 10.4 Gateway-level authentication

The gateway service itself does not expose a public API. It communicates internally via:
- Redis pub/sub (over private network)
- DDS (domain-isolated; not internet-accessible)
- Unix socket for co-located api-server

---

## 11. Configuration and Environment Management

### 11.1 Layered configuration

```
Priority (high → low):
  1. Environment variables (12-factor)
  2. .env.local (developer override, git-ignored)
  3. .env.{environment} (staging / production, vault-managed)
  4. config/defaults.yaml (committed defaults)
```

### 11.2 Pydantic Settings classes

```python
# gateway/config.py

from pydantic_settings import BaseSettings, SettingsConfigDict
from pydantic import Field

class GatewayConfig(BaseSettings):
    model_config = SettingsConfigDict(env_prefix="QNC_GW_", env_file=".env")

    # DDS
    dds_domain_id: int = Field(default=0, ge=0, le=232)
    dds_participant_name: str = "qnc_gateway"

    # Telemetry
    telemetry_flush_interval_ms: int = 250
    telemetry_batch_size: int = 100

    # Adapter poll rates
    adapter_poll_interval_ms: int = 100

    # Database (write path for telemetry)
    db_url: str = Field(..., alias="DATABASE_URL")

    # Redis
    redis_url: str = Field(default="redis://localhost:6379/0")

    # Logging
    log_level: str = "INFO"
    log_format: str = "json"   # json | text

    # Discovery
    descriptor_db_url: str | None = None  # Cloud descriptor lookup URL

class ApiServerConfig(BaseSettings):
    model_config = SettingsConfigDict(env_prefix="QNC_API_", env_file=".env")

    host: str = "0.0.0.0"
    port: int = 8080
    cors_origins: list[str] = ["http://localhost:3000"]
    jwt_secret: str = Field(..., min_length=32)
    jwt_algorithm: str = "HS256"
    access_token_expire_minutes: int = 15
    refresh_token_expire_days: int = 7
    db_url: str = Field(..., alias="DATABASE_URL")
    redis_url: str = Field(default="redis://localhost:6379/0")
```

### 11.3 Environment files

```
.env.example          # Template committed to repo (no secrets)
.env.local            # Local dev (git-ignored)
.env.test             # CI test values (no real hardware)
.env.staging          # Managed by Vault / Kubernetes Secrets
.env.production       # Same, production values
```

---

## 12. Testing Structure

### 12.1 Test pyramid

```
              ┌──────────────────┐
              │   Hardware-in-   │  ← Real physical devices
              │   Loop (HIL)     │    Scheduled nightly
              ├──────────────────┤
              │  End-to-End (E2E)│  ← Playwright: dashboard + API + mock device
              ├──────────────────┤
              │   Integration    │  ← Docker Compose: API + DB + Redis + mock DDS
              ├──────────────────┤
              │  Unit Tests      │  ← No I/O; fast; covers domain + services
              └──────────────────┘
```

### 12.2 Test layout

```
tests/
├── unit/
│   ├── gateway/
│   │   ├── test_bridge_fsm.py
│   │   ├── test_capability_service.py
│   │   ├── test_command_validation.py
│   │   └── test_descriptor_engine.py
│   ├── api-server/
│   │   ├── test_device_service.py
│   │   └── test_auth.py
│   └── adapters/
│       ├── test_modbus_adapter.py      # Uses mock serial port
│       └── test_canbus_adapter.py
├── integration/
│   ├── conftest.py                     # Starts Docker Compose services
│   ├── test_device_lifecycle.py        # Register → connect → command → state
│   ├── test_descriptor_upload.py
│   ├── test_websocket_stream.py
│   └── test_hot_plug.py               # Exercises FSM DISCONNECTED→OPERATIONAL
├── e2e/
│   ├── playwright.config.ts
│   ├── specs/
│   │   ├── device-registration.spec.ts
│   │   ├── send-command.spec.ts
│   │   └── telemetry-dashboard.spec.ts
│   └── fixtures/
└── hardware-in-loop/
    ├── conftest.py                     # Requires QNC_HIL_DEVICE_URI env var
    ├── test_modbus_robotiq.py
    └── test_canbus_dh_ag95.py
```

### 12.3 Key test patterns

```python
# tests/unit/gateway/test_bridge_fsm.py

import pytest
from gateway.state_machine.bridge_fsm import BridgeFSM, BridgeState

def test_happy_path_to_operational():
    fsm = BridgeFSM(device_id="dev-001")
    fsm.transition(BridgeState.PROTOCOL_DETECTED)
    fsm.transition(BridgeState.INITIALIZING)
    fsm.transition(BridgeState.OPERATIONAL)
    assert fsm.is_operational

def test_cannot_skip_from_disconnected_to_operational():
    fsm = BridgeFSM(device_id="dev-001")
    with pytest.raises(InvalidTransitionError):
        fsm.transition(BridgeState.OPERATIONAL)

def test_operational_to_faulted():
    fsm = BridgeFSM(device_id="dev-001")
    # ... transition to OPERATIONAL ...
    fsm.transition(BridgeState.FAULTED)
    assert not fsm.is_operational
```

```python
# Adapter test with mock serial
# tests/unit/adapters/test_modbus_adapter.py

import pytest
from unittest.mock import AsyncMock, patch
from adapters.modbus.adapter import ModbusRtuAdapter
from gateway.domain.commands import GripperCommand, GripperMode

@pytest.fixture
def mock_modbus_client():
    with patch("adapters.modbus.adapter.ModbusSerialClient") as mock:
        mock.return_value.connect = AsyncMock(return_value=True)
        mock.return_value.read_holding_registers = AsyncMock(
            return_value=MockRegisters(registers=[512, 0, 1])
        )
        yield mock

async def test_read_state_maps_registers(mock_modbus_client):
    adapter = ModbusRtuAdapter(uri="modbus+rtu:///dev/ttyUSB0", descriptor={...})
    await adapter.connect()
    state = await adapter.read_state()
    assert state.current_position == pytest.approx(512 / 1000.0, abs=0.01)
    assert state.is_activated is True
```

---

## 13. DevOps / Deployment Structure

### 13.1 Docker composition

```
infra/docker/
├── Dockerfile.gateway         # Python gateway process
├── Dockerfile.api-server      # FastAPI service
├── Dockerfile.dashboard       # nginx-served React build
└── docker-compose.yml         # Full local dev stack
```

```yaml
# infra/docker/docker-compose.yml (development)
version: "3.9"
services:
  postgres:
    image: timescale/timescaledb:latest-pg15
    environment:
      POSTGRES_DB: qnc
      POSTGRES_USER: qnc
      POSTGRES_PASSWORD: dev_password
    ports: ["5432:5432"]
    volumes: [pgdata:/var/lib/postgresql/data]

  redis:
    image: redis:7-alpine
    ports: ["6379:6379"]

  gateway:
    build:
      context: ../../
      dockerfile: infra/docker/Dockerfile.gateway
    depends_on: [postgres, redis]
    environment:
      DATABASE_URL: postgresql+asyncpg://qnc:dev_password@postgres/qnc
      QNC_GW_REDIS_URL: redis://redis:6379/0
      QNC_GW_LOG_LEVEL: DEBUG
    devices:
      - "/dev/ttyUSB0:/dev/ttyUSB0"   # Pass-through for Modbus device
    privileged: false
    cap_add: [NET_RAW]                # For CAN socket access

  api-server:
    build:
      context: ../../
      dockerfile: infra/docker/Dockerfile.api-server
    depends_on: [postgres, redis]
    ports: ["8080:8080"]
    environment:
      DATABASE_URL: postgresql+asyncpg://qnc:dev_password@postgres/qnc
      QNC_API_JWT_SECRET: dev_secret_change_in_production_32chars

  dashboard:
    build:
      context: ../../
      dockerfile: infra/docker/Dockerfile.dashboard
    ports: ["3000:80"]
    environment:
      VITE_API_BASE_URL: http://api-server:8080

volumes:
  pgdata:
```

### 13.2 Kubernetes (edge deployment with k3s)

```
infra/k8s/
├── namespace.yaml
├── gateway/
│   ├── deployment.yaml        # DaemonSet on edge nodes with USB/CAN access
│   ├── configmap.yaml
│   └── secret.yaml            # DB URL, Redis URL (sealed-secrets or Vault)
├── api-server/
│   ├── deployment.yaml        # 2+ replicas behind LoadBalancer
│   ├── service.yaml
│   ├── hpa.yaml               # HorizontalPodAutoscaler
│   └── ingress.yaml           # TLS termination via cert-manager
├── dashboard/
│   ├── deployment.yaml
│   └── service.yaml
├── postgres/
│   └── statefulset.yaml       # Or use managed RDS
└── redis/
    └── deployment.yaml        # Or use managed ElastiCache
```

**Gateway deployment note:** The gateway needs access to physical serial/CAN ports. It is deployed as a `DaemonSet` with `hostPath` volume mounts for `/dev/ttyUSB*` and a `NET_RAW` capability for CAN sockets. This is the only privileged workload.

### 13.3 CI/CD pipeline

```yaml
# .github/workflows/ci.yml (abbreviated)
name: CI

on: [push, pull_request]

jobs:
  test-gateway:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with: { python-version: "3.11" }
      - run: pip install -e packages/gateway[test]
      - run: pytest tests/unit/gateway -v --cov=gateway

  test-adapters:
    runs-on: ubuntu-latest
    steps:
      - run: pytest tests/unit/adapters -v

  integration-tests:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: timescale/timescaledb:latest-pg15
        ...
      redis:
        image: redis:7-alpine
    steps:
      - run: pytest tests/integration -v

  build-images:
    needs: [test-gateway, test-adapters, integration-tests]
    runs-on: ubuntu-latest
    steps:
      - uses: docker/build-push-action@v5
        with:
          context: .
          file: infra/docker/Dockerfile.gateway
          tags: ghcr.io/org/qnc-gateway:${{ github.sha }}
          push: ${{ github.ref == 'refs/heads/main' }}
```

### 13.4 Observability

```
infra/monitoring/
├── prometheus/
│   ├── prometheus.yml
│   └── rules/
│       ├── gateway_alerts.yml     # adapter_error_rate > 0.05
│       └── latency_alerts.yml     # p99 latency > 100ms
└── grafana/
    ├── datasources/
    └── dashboards/
        ├── gateway_overview.json
        └── device_telemetry.json
```

Key metrics to instrument:

```python
# shared-py/qnc_shared/metrics.py

from prometheus_client import Counter, Histogram, Gauge

adapter_commands_total = Counter(
    "qnc_adapter_commands_total",
    "Total adapter commands dispatched",
    ["device_id", "protocol", "mode"]
)

adapter_latency_seconds = Histogram(
    "qnc_adapter_latency_seconds",
    "Adapter round-trip latency",
    ["device_id", "protocol"],
    buckets=[0.001, 0.005, 0.01, 0.025, 0.05, 0.1, 0.25, 0.5, 1.0]
)

device_bridge_state = Gauge(
    "qnc_device_bridge_state",
    "Bridge state (0=DISCONNECTED, 3=OPERATIONAL)",
    ["device_id"]
)
```

---

## 14. Naming Conventions

### Python

| Element | Convention | Example |
|---|---|---|
| Modules | `snake_case` | `bridge_fsm.py` |
| Classes | `PascalCase` | `ModbusRtuAdapter` |
| Functions / methods | `snake_case` | `read_capabilities()` |
| Constants | `UPPER_SNAKE_CASE` | `DEFAULT_TIMEOUT_MS` |
| Private attributes | `_snake_case` | `_session` |
| Type aliases | `PascalCase` | `DeviceId = UUID` |
| Protocols (abstract) | `I` prefix | `IDeviceAdapter` |
| Test functions | `test_<what>_<condition>` | `test_send_command_when_not_operational` |

### TypeScript / React

| Element | Convention | Example |
|---|---|---|
| React components | `PascalCase` | `GripperStateCard` |
| Hooks | `use` prefix | `useDeviceStream` |
| Files (components) | `PascalCase` | `DeviceList.tsx` |
| Files (hooks/utils) | `camelCase` | `useDeviceStream.ts` |
| Types / Interfaces | `PascalCase` | `GripperState` |
| Zustand stores | `camelCase + Store` | `deviceStore.ts` |
| Constants | `UPPER_SNAKE_CASE` | `BRIDGE_STATES` |
| API endpoints | `camelCase` vars, kebab-case paths | `/api/v1/device-capabilities` |

### Database

| Element | Convention | Example |
|---|---|---|
| Tables | `snake_case` plural | `devices`, `event_log` |
| Columns | `snake_case` | `hardware_id_raw` |
| Indexes | `idx_<table>_<columns>` | `idx_event_log_device_time` |
| Foreign keys | `fk_<table>_<ref>` | `fk_devices_descriptor` |
| Enums (PG type) | `snake_case` | `bridge_state_enum` |

### IDL / DDS

| Element | Convention | Example |
|---|---|---|
| Module namespaces | `lowercase` | `qnc.idl` |
| Types | `PascalCase` | `GripperCommand` |
| Enum values | `UPPER_SNAKE_CASE` | `OBJECT_SLIPPED` |
| Topic names | `snake_case` | `gripper_state` |

---

## 15. Scalability and Maintainability Considerations

### Horizontal scaling

| Component | Strategy |
|---|---|
| **API server** | Stateless → multiple replicas behind load balancer. Session in Redis. |
| **Gateway** | One process per edge node (device locality). Gateway does not scale horizontally on same node — use DaemonSet pattern. |
| **Database** | Read replicas for dashboard queries; write primary for telemetry ingest. TimescaleDB chunk pruning for retention. |
| **Telemetry ingest** | Gateway writes to Redis stream; a separate ingest worker batches to TimescaleDB. This isolates latency-critical DDS path from DB I/O. |

### Protocol adapter isolation

Each adapter runs in its own `asyncio.Task` with a dedicated error boundary. A fault in the Modbus adapter cannot crash the CAN adapter. Adapter processes can optionally be isolated as sub-processes with `multiprocessing` for true memory isolation.

### Descriptor versioning

Descriptors are **immutable once published**. Updates create new versions. Devices reference a descriptor version by ID. This ensures rollback capability and audit integrity.

### WiFi / reliability guardrail

As flagged in the strategy review: **WiFi is marked as monitoring-only at the API layer**. The `connection_uri` schema for devices rejects `transport=wifi` for `device_type=gripper_control`. Wired Ethernet or serial is enforced for active control paths.

### Plugin system for adapters

New adapters are registered via Python entry points, not hard-coded imports:

```toml
# adapters/modbus/pyproject.toml
[project.entry-points."qnc.adapters"]
modbus_rtu = "qnc_adapter_modbus:ModbusRtuAdapter"
```

```python
# gateway/registry/adapter_registry.py
import importlib.metadata

def load_adapters() -> dict[str, type[IDeviceAdapter]]:
    return {
        ep.name: ep.load()
        for ep in importlib.metadata.entry_points(group="qnc.adapters")
    }
```

This allows third-party vendors to ship adapters as Python packages without modifying the core gateway.

---

## 16. Design Patterns

| Pattern | Where applied | Rationale |
|---|---|---|
| **Ports & Adapters (Hexagonal)** | Gateway core | Domain IDL never touches hardware. All I/O injected via `IDeviceAdapter` port. |
| **Repository** | api-server DB layer | Decouples service logic from SQLAlchemy ORM details. Easy to mock in tests. |
| **State Machine** | Bridge FSM | Explicit, testable state transitions. Prevents false-Ready race condition identified in review. |
| **Strategy** | Protocol adapter selection | Adapters are interchangeable strategies behind `IDeviceAdapter` interface. |
| **Factory / Registry** | Adapter loading | Entry-point plugin registry; open/closed principle for new protocols. |
| **Observer / Pub-Sub** | DDS topics, WebSocket streams | Decoupled real-time data distribution. |
| **Command Object** | `GripperCommand` | Encapsulates all parameters; serializable; auditable. |
| **Decorator** | Retry, metrics, tracing | Applied to adapter methods via `@with_retry`, `@track_latency` decorators. |
| **Outbox / Transactional** | Event log writes | DB insert + event publish in one transaction; prevents lost events. |
| **Circuit Breaker** | Adapter connections | After N consecutive failures, trip to FAULTED without hammering the device. |

```python
# Decorator example
# shared-py/qnc_shared/retry.py

import functools
import asyncio
import logging

def with_retry(max_attempts: int = 3, backoff_base: float = 0.5):
    def decorator(fn):
        @functools.wraps(fn)
        async def wrapper(*args, **kwargs):
            for attempt in range(1, max_attempts + 1):
                try:
                    return await fn(*args, **kwargs)
                except Exception as exc:
                    if attempt == max_attempts:
                        raise
                    wait = backoff_base * (2 ** (attempt - 1))
                    logging.warning("Retry %d/%d after %.2fs: %s", attempt, max_attempts, wait, exc)
                    await asyncio.sleep(wait)
        return wrapper
    return decorator
```

---

## 17. Example File Tree

```
qnc/
├── .github/
│   └── workflows/
│       ├── ci.yml
│       ├── release.yml
│       └── hil-tests.yml
├── .env.example
├── .gitignore
├── pyproject.toml                      # Root workspace (uv / hatch)
├── package.json                        # Root JS workspace (pnpm)
├── README.md
│
├── packages/
│   │
│   ├── idl/                            # IDL definitions + generated stubs
│   │   ├── src/
│   │   │   ├── GripperCommand.idl
│   │   │   ├── GripperState.idl
│   │   │   ├── GripperCapabilities.idl
│   │   │   └── ErrorSeverity.idl
│   │   ├── generated/
│   │   │   ├── python/
│   │   │   │   ├── GripperCommand.py
│   │   │   │   ├── GripperState.py
│   │   │   │   └── GripperCapabilities.py
│   │   │   └── cpp/
│   │   │       └── ...
│   │   └── Makefile
│   │
│   ├── gateway/                        # Real-time gateway core
│   │   ├── gateway/
│   │   │   ├── __init__.py
│   │   │   ├── main.py
│   │   │   ├── config.py
│   │   │   ├── domain/
│   │   │   │   ├── __init__.py
│   │   │   │   ├── commands.py
│   │   │   │   ├── state.py
│   │   │   │   ├── capabilities.py
│   │   │   │   ├── errors.py
│   │   │   │   └── events.py
│   │   │   ├── state_machine/
│   │   │   │   ├── __init__.py
│   │   │   │   ├── bridge_fsm.py
│   │   │   │   └── transitions.py
│   │   │   ├── services/
│   │   │   │   ├── __init__.py
│   │   │   │   ├── capability_service.py
│   │   │   │   ├── command_service.py
│   │   │   │   ├── discovery_service.py
│   │   │   │   └── telemetry_service.py
│   │   │   ├── transport/
│   │   │   │   ├── __init__.py
│   │   │   │   ├── dds_node.py
│   │   │   │   ├── dds_topics.py
│   │   │   │   └── redis_bridge.py
│   │   │   ├── registry/
│   │   │   │   ├── __init__.py
│   │   │   │   ├── adapter_registry.py
│   │   │   │   └── device_registry.py
│   │   │   └── ports/
│   │   │       ├── __init__.py
│   │   │       ├── adapter_port.py
│   │   │       └── descriptor_port.py
│   │   ├── tests/
│   │   │   ├── unit/
│   │   │   │   ├── test_bridge_fsm.py
│   │   │   │   ├── test_capability_service.py
│   │   │   │   └── test_command_service.py
│   │   │   └── conftest.py
│   │   └── pyproject.toml
│   │
│   ├── api-server/                     # Management REST + WebSocket API
│   │   ├── api_server/
│   │   │   ├── __init__.py
│   │   │   ├── main.py
│   │   │   ├── config.py
│   │   │   ├── routers/
│   │   │   │   ├── __init__.py
│   │   │   │   ├── devices.py
│   │   │   │   ├── descriptors.py
│   │   │   │   ├── capabilities.py
│   │   │   │   ├── commands.py
│   │   │   │   ├── telemetry.py
│   │   │   │   ├── health.py
│   │   │   │   └── auth.py
│   │   │   ├── schemas/
│   │   │   │   ├── __init__.py
│   │   │   │   ├── device.py
│   │   │   │   ├── descriptor.py
│   │   │   │   ├── command.py
│   │   │   │   ├── state.py
│   │   │   │   └── auth.py
│   │   │   ├── services/
│   │   │   │   ├── __init__.py
│   │   │   │   ├── device_service.py
│   │   │   │   ├── descriptor_service.py
│   │   │   │   └── command_dispatch_service.py
│   │   │   ├── db/
│   │   │   │   ├── __init__.py
│   │   │   │   ├── base.py
│   │   │   │   ├── session.py
│   │   │   │   ├── models/
│   │   │   │   │   ├── device.py
│   │   │   │   │   ├── descriptor.py
│   │   │   │   │   ├── event_log.py
│   │   │   │   │   └── user.py
│   │   │   │   └── repositories/
│   │   │   │       ├── __init__.py
│   │   │   │       ├── device_repo.py
│   │   │   │       ├── descriptor_repo.py
│   │   │   │       └── event_log_repo.py
│   │   │   ├── cache/
│   │   │   │   ├── __init__.py
│   │   │   │   └── redis_client.py
│   │   │   ├── middleware/
│   │   │   │   ├── __init__.py
│   │   │   │   ├── auth_middleware.py
│   │   │   │   ├── rate_limit.py
│   │   │   │   └── request_id.py
│   │   │   └── dependencies/
│   │   │       ├── __init__.py
│   │   │       ├── auth.py
│   │   │       └── db.py
│   │   ├── alembic/
│   │   │   ├── env.py
│   │   │   ├── alembic.ini
│   │   │   └── versions/
│   │   │       ├── 0001_initial_schema.py
│   │   │       └── 0002_add_timescale_hypertable.py
│   │   ├── tests/
│   │   │   ├── unit/
│   │   │   │   ├── test_device_service.py
│   │   │   │   └── test_auth.py
│   │   │   └── conftest.py
│   │   └── pyproject.toml
│   │
│   ├── descriptor-engine/              # JSON descriptor parser + executor
│   │   ├── descriptor_engine/
│   │   │   ├── __init__.py
│   │   │   ├── parser.py              # JSON Schema validation
│   │   │   ├── executor.py            # SET(capability, value) → register mapping
│   │   │   ├── resolver.py            # hardware_id ADC → descriptor lookup
│   │   │   └── models.py              # DescriptorSchema Pydantic model
│   │   ├── schemas/
│   │   │   └── descriptor_schema.json # JSON Schema for descriptor validation
│   │   ├── tests/
│   │   │   ├── test_executor.py
│   │   │   ├── test_resolver.py
│   │   │   └── fixtures/
│   │   │       └── sample_descriptor.json
│   │   └── pyproject.toml
│   │
│   ├── adapters/
│   │   ├── modbus/                     # Modbus RTU / TCP adapter
│   │   │   ├── qnc_adapter_modbus/
│   │   │   │   ├── __init__.py
│   │   │   │   ├── adapter.py
│   │   │   │   ├── register_map.py    # Descriptor-driven register resolution
│   │   │   │   └── constants.py
│   │   │   ├── tests/
│   │   │   │   └── test_adapter.py
│   │   │   └── pyproject.toml
│   │   ├── canbus/                     # CAN Bus adapter (python-can)
│   │   │   ├── qnc_adapter_canbus/
│   │   │   │   ├── __init__.py
│   │   │   │   ├── adapter.py
│   │   │   │   └── canopen_profile.py
│   │   │   ├── tests/
│   │   │   └── pyproject.toml
│   │   ├── tcpip/                      # TCP/IP SDK adapter
│   │   │   ├── qnc_adapter_tcpip/
│   │   │   │   ├── __init__.py
│   │   │   │   ├── adapter.py
│   │   │   │   └── sdk_wrapper.py
│   │   │   ├── tests/
│   │   │   └── pyproject.toml
│   │   ├── serial/                     # Serial ASCII adapter
│   │   │   ├── qnc_adapter_serial/
│   │   │   │   ├── __init__.py
│   │   │   │   ├── adapter.py
│   │   │   │   └── frame_parser.py
│   │   │   ├── tests/
│   │   │   └── pyproject.toml
│   │   └── vendorx/                    # Example vendor extension
│   │       ├── qnc_adapter_vendorx/
│   │       │   ├── __init__.py
│   │       │   ├── adapter.py
│   │       │   └── vendorx_idl.py     # VendorX-specific IDL types
│   │       ├── tests/
│   │       └── pyproject.toml
│   │
│   ├── shared-py/                      # Shared Python utilities
│   │   ├── qnc_shared/
│   │   │   ├── __init__.py
│   │   │   ├── logging.py
│   │   │   ├── tracing.py
│   │   │   ├── retry.py
│   │   │   ├── pagination.py
│   │   │   ├── serialization.py
│   │   │   ├── validation.py
│   │   │   ├── metrics.py
│   │   │   └── errors/
│   │   │       ├── __init__.py
│   │   │       ├── base.py
│   │   │       ├── device_errors.py
│   │   │       └── protocol_errors.py
│   │   ├── tests/
│   │   └── pyproject.toml
│   │
│   ├── dashboard/                      # React management UI
│   │   ├── public/
│   │   │   └── favicon.ico
│   │   ├── src/
│   │   │   ├── main.tsx
│   │   │   ├── App.tsx
│   │   │   ├── router/
│   │   │   │   └── index.tsx
│   │   │   ├── pages/
│   │   │   │   ├── Devices/
│   │   │   │   │   ├── DeviceList.tsx
│   │   │   │   │   ├── DeviceDetail.tsx
│   │   │   │   │   └── DeviceCreate.tsx
│   │   │   │   ├── Descriptors/
│   │   │   │   │   ├── DescriptorList.tsx
│   │   │   │   │   └── DescriptorEditor.tsx
│   │   │   │   ├── Telemetry/
│   │   │   │   │   └── TelemetryDashboard.tsx
│   │   │   │   └── Auth/
│   │   │   │       ├── Login.tsx
│   │   │   │       └── Profile.tsx
│   │   │   ├── components/
│   │   │   │   ├── ui/
│   │   │   │   ├── GripperStateCard/
│   │   │   │   │   ├── GripperStateCard.tsx
│   │   │   │   │   └── GripperStateCard.test.tsx
│   │   │   │   ├── CapabilityBadge/
│   │   │   │   ├── BridgeStatusIndicator/
│   │   │   │   ├── TelemetryChart/
│   │   │   │   └── CommandPanel/
│   │   │   ├── hooks/
│   │   │   │   ├── useDeviceStream.ts
│   │   │   │   ├── useCapabilities.ts
│   │   │   │   └── useAuth.ts
│   │   │   ├── stores/
│   │   │   │   ├── deviceStore.ts
│   │   │   │   ├── telemetryStore.ts
│   │   │   │   └── authStore.ts
│   │   │   ├── api/
│   │   │   │   ├── client.ts
│   │   │   │   └── endpoints/
│   │   │   │       ├── devices.ts
│   │   │   │       ├── descriptors.ts
│   │   │   │       └── auth.ts
│   │   │   ├── types/
│   │   │   │   ├── gripper.ts
│   │   │   │   ├── descriptor.ts
│   │   │   │   └── device.ts
│   │   │   └── utils/
│   │   │       ├── formatters.ts
│   │   │       └── validators.ts
│   │   ├── tsconfig.json
│   │   ├── vite.config.ts
│   │   └── package.json
│   │
│   └── shared-ts/                      # Shared TypeScript types
│       ├── src/
│       │   ├── types/
│       │   │   ├── gripper.ts
│       │   │   ├── device.ts
│       │   │   └── events.ts
│       │   ├── utils/
│       │   │   └── formatters.ts
│       │   └── constants/
│       │       └── bridgeStates.ts
│       ├── tsconfig.json
│       └── package.json
│
├── infra/
│   ├── docker/
│   │   ├── Dockerfile.gateway
│   │   ├── Dockerfile.api-server
│   │   ├── Dockerfile.dashboard
│   │   └── docker-compose.yml
│   ├── k8s/
│   │   ├── namespace.yaml
│   │   ├── gateway/
│   │   │   ├── daemonset.yaml
│   │   │   ├── configmap.yaml
│   │   │   └── secret.yaml
│   │   ├── api-server/
│   │   │   ├── deployment.yaml
│   │   │   ├── service.yaml
│   │   │   ├── hpa.yaml
│   │   │   └── ingress.yaml
│   │   ├── dashboard/
│   │   ├── postgres/
│   │   └── redis/
│   ├── terraform/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── modules/
│   │       ├── eks/
│   │       ├── rds/
│   │       └── elasticache/
│   └── monitoring/
│       ├── prometheus/
│       │   ├── prometheus.yml
│       │   └── rules/
│       └── grafana/
│           └── dashboards/
│
├── scripts/
│   ├── dev-setup.sh                    # Bootstrap local dev environment
│   ├── generate-idl.sh                 # Regenerate IDL bindings
│   ├── seed-descriptors.py             # Load sample descriptors into DB
│   └── check-latency.py               # Latency formula calculator
│
├── docs/
│   ├── architecture/
│   │   ├── decisions/                  # Architecture Decision Records (ADRs)
│   │   │   ├── 001-monorepo.md
│   │   │   ├── 002-python-gateway.md
│   │   │   └── 003-hexagonal-adapters.md
│   │   └── diagrams/
│   ├── api/
│   │   └── openapi.yaml               # Auto-generated; committed for review
│   └── descriptors/
│       ├── schema-reference.md
│       └── writing-a-descriptor.md
│
└── tests/
    ├── integration/
    │   ├── conftest.py
    │   ├── test_device_lifecycle.py
    │   ├── test_descriptor_upload.py
    │   ├── test_websocket_stream.py
    │   └── test_hot_plug.py
    ├── e2e/
    │   ├── playwright.config.ts
    │   └── specs/
    │       ├── device-registration.spec.ts
    │       ├── send-command.spec.ts
    │       └── telemetry-dashboard.spec.ts
    └── hardware-in-loop/
        ├── conftest.py
        ├── test_modbus_robotiq.py
        └── test_canbus_dh_ag95.py
```

---

## 18. Assumptions

The following were inferred from the documents and marked here for explicit review:

1. **Management plane is web-based.** The documents do not specify a UI, but a management dashboard is assumed given the need for device registration, descriptor CRUD, and telemetry monitoring.
2. **Single edge node per QNC instance.** The gateway is assumed to run on a single Linux host with direct hardware access. Multi-node clustering is a future concern.
3. **Python is acceptable for the gateway hot path.** If sub-millisecond latency is required for all adapters, some adapters may need C++ reimplementation.
4. **Authentication is needed.** The product review implies multi-user deployment; JWT + RBAC is assumed.
5. **NeuraSync = FastDDS.** The document uses both terms; they are treated as the same DDS transport.
6. **Descriptor cloud repository is a future feature.** The `discovery_service` structure supports it but the initial implementation uses only local DB lookup.
7. **E-Stop is handled by hardware bypass.** As per the product review recommendation, the QNC software layer is explicitly not in the safety-critical E-Stop path. This constraint is enforced architecturally (no E-Stop command in IDL).

---

## 19. Improvement Suggestions

Based on weaknesses identified in the product strategy review:

### Critical

| Issue | Fix in architecture |
|---|---|
| **Hardware ID leaks device semantics** | `hardware_id_raw` stored as raw ADC integer in DB. Interpretation happens only in `descriptor_engine/resolver.py`. Gateway transport layer never maps HW ID to device name. |
| **WiFi for real-time control** | `connection_uri` schema validation rejects `transport=wifi` for `mode=control`. WiFi-connected devices are `monitoring_only=True` in the DB schema. |
| **State sync race in hot-plug** | `BridgeFSM` requires explicit `OPERATIONAL` state before any `GripperCommand` is accepted. API returns `409 CONFLICT` with `DEVICE_NOT_OPERATIONAL` error if command sent during `INITIALIZING`. |
| **Descriptor ownership ambiguity** | `DescriptorEngine` is the only component that parses descriptors. Gateway is a pure protocol translator. The `executor.py` in `descriptor-engine` maps `SET(capability, value)` → register address. This resolves the bridge-vs-device-manager ambiguity in favour of a "smart translator" model. |

### Important

| Issue | Fix |
|---|---|
| **Latency single-value claim** | `latency_profile` object in capabilities response exposes formula, baud rate, and message size. Tested in `tests/unit/test_latency_calculator.py`. |
| **No descriptor library at launch** | `scripts/seed-descriptors.py` pre-populates DB with validated descriptors for Robotiq 2F-85, DH-Robotics AG95, Festo DHZE, and 10 others. This is a day-one deliverable. |
| **Standardisation paradox** | `docs/descriptors/writing-a-descriptor.md` with schema auto-completion in the `DescriptorEditor` (Monaco + JSON Schema) dramatically reduces authoring time. |
| **Safety gap** | Architecture explicitly excludes E-Stop from software IDL. Hardware bypass wiring guide added to `docs/safety/estop-bypass.md`. All deployment manifests include a required checklist annotation. |

### Nice-to-have

- **gRPC streaming** alternative to WebSocket for higher-throughput telemetry scenarios
- **WASM-compiled descriptor executor** for client-side validation in the dashboard without a round-trip
- **OpenTelemetry-native IDL annotations** for automatic span propagation from GripperCommand dispatch to physical device acknowledgement
