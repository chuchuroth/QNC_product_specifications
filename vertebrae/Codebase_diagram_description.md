# QNC — C++ Architecture Diagram Description (qnc-cpp/)

This document accompanies `qnc_structural_diagram` and reflects the C++
implementation under `qnc-cpp/` . The diagram is structured around the three views:
the **build artefacts** (`apps/`), the **service composition** that lives
inside each app (`services/`), and the **static library layer** (`libs/`)
that supplies their behaviour.

================================================================================
1. Component descriptions
================================================================================


### External consumers (outside the repo)

The dashboard and operator CLI are external to `qnc-cpp/`. They consume the
Drogon-based REST and WebSocket interface exposed by `qnc-api-server`. The
diagram includes them only to show the entry point; their source lives in
the Python sibling repo and downstream tooling repos.

### apps/ — build artefacts

Three executables are produced by the CMake build:

- **qnc-gateway** (`apps/gateway/`) — the real-time daemon. Wires
  configuration → logging → `services::gateway::gateway_runtime` and runs
  under `core::application` lifecycle management with signal handlers.
- **qnc-api-server** (`apps/api-server/`) — the Drogon HTTP + WebSocket
  process. Owns the long-lived `services::api::api_runtime` and the
  `auth_filter`/`rate_limit_filter`/`request_id_filter` chain that every
  protected route walks through.
- **qnc-descriptor-validate** (`apps/tools/`) — CLI11 utility that pipes a
  JSON descriptor through `qnc-descriptor`'s parser for offline validation.
  Used in CI and by device integrators authoring new descriptors.

### services/gateway — real-time core

Composed by `gateway_runtime`:

- `device_registry` — thread-safe map from `device_id` → `device_connection`
  (adapter + FSM + cached capabilities).
- `command_service` — FSM-gated dispatch. Asserts `OPERATIONAL`, validates
  capability gating (force/velocity/preset/monitoring_only), records
  Prometheus latency, and propagates structured errors.
- `capability_service` — refreshes capabilities from the adapter and
  publishes them to DDS and Redis.
- `telemetry_service` — startable component that owns a bounded SPSC queue
  and a dedicated flusher thread. The PD publish path never blocks the
  command path.

### services/api-server — Drogon controllers

Filter chain (registered in this order): `request_id_filter` →
`auth_filter` (verifies JWT via `qnc-auth::jwt_issuer` and attaches decoded
claims to the request) → `rate_limit_filter` (per-user / per-IP sliding
window with garbage-collected buckets) → controller.

Controllers:

- `health_controller` — `/health`, `/readyz`, `/livez`, `/metrics`
  (Prometheus text format). Anonymous.
- `auth_controller` — `/auth/token` (login), `/auth/refresh` (rotating
  refresh), `/auth/token DELETE` (revoke).
- `devices_controller` — CRUD over the device registry, rejects WiFi-only
  control URIs at the schema level.
- `descriptors_controller` — upload + retrieve descriptors; computes a
    stable SHA-256 checksum, rejects duplicates with 409.
- `commands_controller` — accepts a `GripperCommand` JSON, enqueues onto
  Redis stream `qnc:commands` for the gateway to consume, and writes an
  audit row to `event_log`.
- `telemetry_ws_controller` — `WS /devices/{id}/stream`. Authenticates via
  `?token=`, subscribes to `qnc:state:{id}` on Redis, forwards JSON frames.
- `iolink_controller` — IO-Link parameter R/W, batch read, diagnostics,
  IODD metadata, process-data snapshot, and master discovery (see §3).

### core/ — composition

- `container` — header-only type-safe DI container with singleton +
  factory scopes.
- `config` — layered config (defaults → TOML file → env). Strong types per
  service (`gateway_config`, `api_server_config`).
- `lifecycle` — `application` owns a registration-ordered list of
  `startable` components, installs SIGINT/SIGTERM handlers, and unwinds
  on signal in reverse order.

### libs/qnc-common — cross-cutting utilities

Linked by every other library:

- `logging` — async spdlog with JSON sink in production and per-thread
  context binding (`request_id`, `device_id`).
- `error` — typed exception hierarchy with stable `error_code` + `severity`
  + default HTTP status. Every protected route maps these to the standard
  error envelope.
- `retry` — exponential backoff with jitter, predicate-driven.
- `metrics` — process-wide `prometheus-cpp` registry with shared
  collectors for adapter, DDS, HTTP, and telemetry paths.
- `uuid` — minimal RFC 4122 v4 implementation (avoids dragging in Boost
  UUID).
- `string_utils` — splitting, query parsing, case-insensitive comparison.

### libs/qnc-domain — value types + FSM

Pure data + the `bridge_fsm`:

- `gripper_command` / `gripper_state` / `gripper_capabilities` /
  `latency_profile`.
- `bridge_fsm` — five states (`DISCONNECTED` → `PROTOCOL_DETECTED` →
  `INITIALIZING` → `OPERATIONAL` → `FAULTED`). Bitmask transition table for
  O(1) validity check, single mutex for thread safety, listener fan-out
  for observers. `assert_operational()` is the hot-path gate on every
  command dispatch.
- `domain_events` — `std::variant` of internal events (connect, fault,
  state-transitioned).

### libs/qnc-idl — Protobuf-based wire contract

Protobuf is the C++ build's chosen IDL (the Python sibling used OMG IDL).
`.proto` definitions live under `proto/qnc/idl/v1/`:

- `GripperCommand`, `GripperState`, `GripperCapabilities`, `ErrorSeverity`.

Two converter layers:

- `wire_converters` — domain ↔ Protobuf for binary wire / DDS.
- `wire_json` — domain ↔ JSON for the WebSocket and Redis bridge.

### libs/qnc-descriptor — JSON descriptor pipeline

- `descriptor_model` — typed mirror of the schema. Now carries an optional
  `iolink_profile` section + new wire functions (`isdu_read/write`,
  `pd_read/write`) + new encodings (`uint8/int8/uint32/int64/iolink_*`).
- `parser` — JSON Schema validation via valijson; produces the typed
  model.
- `executor` — pure mapping from `gripper_command` to a `execution_plan`
  of wire writes. Adapters consume the plan to dispatch actual I/O.
- `resolver` — `hardware_id → descriptor_model` lookup, used by the
  discovery path on hot-plug events.

### libs/qnc-adapters — protocol adapter plugins

Adapter contract + the five concrete adapters:

- `adapter_port` (abstract): `connect`, `disconnect`, `read_capabilities`,
  `send_command`, `read_state`, `health_check`.
- `parameter_capable` (optional mixin, **new**): `read_parameter`,
  `write_parameter`, `read_parameters`, `decode_parameter`,
  `known_parameters`. Lets a protocol expose its full parameter dictionary
  without bloating `adapter_port`.
- `adapter_registry` — protocol-id → factory map, populated at static-init
  time via `QNC_REGISTER_ADAPTER`.

Concrete adapters:

- `modbus` — libmodbus, RTU and TCP, with `register_codec` for
  big-endian encode/decode.
- `canbus` — SocketCAN raw + CANopen DS-402 SDO write (Linux only).
- `tcpip` — Boost.Asio TCP with newline / length-prefix framing.
- `serial` — Boost.Asio serial + ASCII `frame_parser`.
- `vendorx` — synthetic adapter for CI + UI demos.
- **`iolink`** (new) — bridges to IO-Link via `qnc-iolink` (see §2).

### libs/qnc-iolink — IO-Link protocol stack

The newest library, isolated behind `QNC_ENABLE_IOLINK`. Layers:

- **Domain types**: `types` (strong enums for `vendor_id`, `device_id`,
  `port_number`, `parameter_address`, `master_uri`, `port_address`),
  `parameter` (variant aligned with IODD data types), `process_data`,
  `event`, `device_info` / `port_status`.
- **IODD subsystem**: `iodd_document` (typed model), `iodd_parser`
  (pugixml; tolerates `data_type` suffixes like `UIntegerT.16`),
  `iodd_repository` (thread-safe cache + filesystem auto-discovery).
- **Codec**: `parameter_codec` — big-endian encode/decode for every
  IODD `data_type` (Boolean, UInteger/Integer of arbitrary bit length,
  Float32, String, OctetString, Record, Array). Bit-aligned field
  extraction for packed records.
- **Master backends**: abstract `master_backend` + concrete `http_master_backend`
  (Boost.Beast, IFM/Murr-style REST/JSON conventions, base64-encoded ISDU
  payloads) and `simulator_backend` (in-process, supports fault injection,
  used by every IO-Link CI test).
- **Registry**: `master_registry` deduplicates backend instances across
  ports that share a master endpoint; one HTTP keepalive multiplexes all
  ports on one master.
- **Per-port supervisor**: `iolink_port` owns the cached backend, the
  TTL-bounded `parameter_cache`, the cyclic PD polling thread, and the
  reference to the loaded IODD.

### libs/qnc-transport — outbound publishing

- `dds_publisher` — abstract façade. Cyclone DDS implementation when
  `QNC_ENABLE_DDS=ON`, otherwise a metrics-recording null backend so the
  rest of the platform compiles + tests without DDS.
- `dds_topics` — string constants + QoS profiles (`best_effort` for state,
  `reliable+transient_local` for capabilities and events).
- `redis_bridge` — hiredis client for PUBLISH (state fan-out to the
  WebSocket) and XADD (telemetry stream consumed by the ingest worker).

### libs/qnc-persistence — PostgreSQL via libpqxx

- `connection_pool` — fixed-size pool, condition-variable acquire,
  bounded acquire timeout. libpqxx is synchronous; the pool gives the
  Drogon workers a non-blocking façade.
- Repositories: `device_repository`, `descriptor_repository`,
  `event_log_repository`, `user_repository`. Each consumes connection
  leases for the duration of one query / transaction.
- `migration_runner` — applies `*.sql` files in lexicographic order, tracks
  applied versions in `schema_migrations`. Two migrations ship: initial
  schema + TimescaleDB hypertable creation.

### libs/qnc-auth — authentication + authorisation

- `password_hasher` — **PBKDF2-HMAC-SHA256** (not bcrypt — keeps the
  dependency footprint to OpenSSL, which is already linked for jwt-cpp).
  Constant-time compare on verify, configurable iteration count.
- `jwt_issuer` — jwt-cpp, HS256, short-lived access + opaque refresh, JTI
  embedded for revocation.
- `refresh_store` — hiredis-backed, one-time-use semantics (atomic
  GET+DEL pipeline).
- `rbac` — three roles (`viewer`, `operator`, `admin`) + per-capability
  predicates checked inside each controller.

### External systems

- **Physical devices** — grippers / sensors / actuators reached through
  one of the protocol adapters. IO-Link devices sit behind an IO-Link
  master rather than connecting directly to the gateway host.
- **IO-Link masters** — IFM AL1352 / Murr MVK / Balluff USB. Speak
  REST/JSON to `http_master_backend` and SDCI on the device-facing side.
- **Robot platform** — ROS 2 / NeuraSync. Subscribes to `qnc/gripper_state`,
  publishes `qnc/gripper_command`. The gateway is the broker between the
  robot platform and physical devices.

### Data stores

- **PostgreSQL 15** — devices, descriptors, users, event_log.
- **TimescaleDB hypertable** — `gripper_telemetry` (provisioned by
  migration 0002). Retention policy drops samples older than 30 days.
- **Redis 7** — refresh tokens, state pub/sub (`qnc:state:{id}`),
  telemetry stream (`qnc:telemetry`), command stream (`qnc:commands`).

### Infra & observability (`infra/`)

- Docker — multi-stage Dockerfiles per app, plus a `docker-compose.yml`
  for the local stack (postgres + redis + gateway + api-server +
  prometheus + grafana).
- Kubernetes — DaemonSet for the gateway (host `/dev` mount + `CAP_NET_RAW`
  for SocketCAN), Deployment + HPA for the api-server, Ingress optional.
- Prometheus — scrapes `/api/v1/metrics` (api-server) and `:9100` (gateway).
- Grafana — Prometheus data source provisioned via `datasources/*.yml`.
- GitHub Actions — matrix CI across `{dev, release, asan, tsan}` ×
  `{gcc, clang}` × `iolink={on, off}` plus a smoke build with IO-Link
  disabled to keep the off-path honest.


================================================================================

2. IO-Link integration (new subsystem)
================================================================================

IO-Link is the first protocol QNC supports where the device sits behind a
**master** rather than being directly addressable. The integration follows
three rules:

1. **One adapter per port.** A master with N ports yields N
   `iolink_adapter` instances that share one cached `master_backend`. The
   master itself is never an adapter; the port is.
2. **Descriptor remains authoritative.** The QNC JSON descriptor's
   `register_map` continues to declare which ISDU indices satisfy which
   capability. The IODD is consulted at runtime only for typed decoding
   and event-code resolution.
3. **`parameter_capable` is opt-in.** The mixin lets `iolink_adapter`
   expose the full parameter dictionary via the api-server without
   bloating the existing `adapter_port` contract that every other
   adapter implements unchanged.

================================================================================

3. Key data flows
================================================================================

### Command dispatch (write path)

1. Robot platform publishes `GripperCommand` over DDS, **or** the
   dashboard issues `POST /devices/{id}/commands`.
2. Dashboard path: `commands_controller` validates the request, mints a
   `request_id`, appends an audit row to `event_log`, and XADDs the
   command onto Redis stream `qnc:commands`.
3. The gateway's command consumer dequeues the command and hands it to
   `command_service::dispatch`.
4. The FSM is asserted `OPERATIONAL`; capability gating rejects
   force/velocity/preset requests the device doesn't advertise.
5. `command_service` invokes `adapter_port::send_command` on the
   appropriate adapter.
6. The adapter consults `descriptor::executor`, produces an
   `execution_plan` of wire writes, and dispatches them. For IO-Link
   adapters: ISDU writes via `iolink_port::write_parameter`, PD writes
   via `master_backend::write_process_data`. Latency is recorded into
   `qnc_adapter_latency_seconds`.

### State read path

1. The adapter polls `read_state` — either by Modbus register reads,
   SocketCAN frames, or `iolink_port::read_process_data` consulting the
   IODD-declared `ProcessData.in` layout.
2. The result is a `gripper_state` value that `telemetry_service`
   enqueues into its SPSC queue.
3. The flusher thread publishes the state on DDS (`qnc/gripper_state`,
   best-effort QoS) **and** to Redis (`qnc:state:{id}`) **and** appends
   to the telemetry stream.
4. Robot platform receives the DDS message; the api-server's
   `telemetry_ws_controller` fans the Redis pub/sub stream out to every
   subscribed WebSocket client.

### IO-Link parameter read (cache hit)

1. Dashboard issues `GET /devices/{id}/iolink/parameters/64/0`.
2. `iolink_controller` resolves the adapter, casts to
   `parameter_capable`, and calls `read_parameter`.
3. `iolink_adapter` delegates to `iolink_port`, which checks the
   `parameter_cache` (TTL = `iolink_profile.parameter_cache_ttl_ms`,
   default 5 s). A cache hit returns immediately with the cached
   `parameter_value`.
4. The controller renders raw bytes + optional `decoded` (via
   `parameter_codec` against the IODD) as JSON.

### IO-Link parameter read (cache miss)

Same as above, but step 3 issues `master_backend::read_parameter`. The
HTTP backend serialises against its mutex (one keepalive per master),
issues `GET /iolink/v1/ports/{n}/parameters/{i}/{s}`, base64-decodes the
response, and populates the cache.

### IO-Link cyclic process data

1. When the adapter enters `OPERATIONAL`, `iolink_port::start_cyclic`
   spawns a dedicated thread that calls `read_process_data` every
   `cyclic_interval_ms` (default 10).
2. Each successful read flows through the adapter's PD decoder into a
   `gripper_state` and onto `telemetry_service`. Failures are counted;
   `max_consecutive_failures` trips the circuit breaker, marks the FSM
   `FAULTED`, and stops the thread.

### Device registration

1. `POST /api/v1/devices` with a descriptor reference.
2. `devices_controller` validates the connection URI (rejects
   `transport=wifi` for control mode), persists to PostgreSQL.
3. The gateway's discovery path picks up the new row, asks the
   `descriptor_repository` for the linked descriptor, and constructs
   the adapter via `adapter_registry::create(protocol, cfg)`.
4. For IO-Link: the constructor builds an `iolink_port`, which acquires
   the `master_backend` from `master_registry` (creating it if no other
   port on the same master already did) and loads the IODD from the
   `iodd_repository`.
5. The FSM walks `DISCONNECTED → PROTOCOL_DETECTED → INITIALIZING →
   OPERATIONAL`, capabilities are refreshed, cyclic polling starts.

### Telemetry ingest

The gateway never writes directly to TimescaleDB — that work happens on
an ingest worker (out of `qnc-cpp/`) that consumes the Redis stream
`qnc:telemetry` and bulk-inserts into the `gripper_telemetry` hypertable.
This keeps the gateway's hot path free of DB I/O.

================================================================================

4. Assumptions , Gap
===============================================================================

| # | Assumption / Gap | Notes |
|---|---|---|
| 1 | **Dashboard is external to this repo.** `qnc-cpp/` exposes Drogon REST + WS; the dashboard implementation lives in the Python sibling. | Diagram shows it only as a consumer. |
| 2 | **NeuraSync = Cyclone DDS.** Same convention as the Python build. When `QNC_ENABLE_DDS=OFF` (or Cyclone is unavailable at runtime), `make_dds_publisher` returns a null backend so the gateway still runs. | `dds_publisher.cpp` |
| 3 | **E-Stop is hardware-only.** No `EStop` field exists in the IDL by design. Documented in `docs/safety/estop-bypass.md`. | ADR-003 |
| 4 | **Ingest worker is out of repo.** The gateway publishes to Redis streams; the worker that drains into TimescaleDB is intentionally a separate process. | Same boundary as the Python sibling. |
| 5 | **IO-Link master vs port.** One `iolink_adapter` per port; the master is shared via `master_registry`. The api-server's adapter resolution lookup is currently a stub (`resolve_adapter` returns `nullptr` in `iolink_controller.cpp`) — the production wiring goes through a Redis pub/sub bridge between api-server and gateway, planned but not yet committed. | ADR-005 |
| 6 | **IODD repository auto-scan.** Scans `/usr/local/share/qnc/iodds/` at gateway startup. Per-device descriptors may also reference an explicit `iodd_path`. | `iodd_repository::scan_directory` |
| 7 | **HTTP master backend follows IFM-style conventions.** The exact REST surface differs by master vendor; `http_master_backend.cpp` is the integration point. Custom backends register via `QNC_REGISTER_IOLINK_BACKEND`. | `master_registry` |
| 8 | **Password hashing.** PBKDF2-HMAC-SHA256 with configurable iterations (default 600k); migration from bcrypt is documented but not automated. | `password_hasher.cpp` |
| 9 | **Drogon WebSocket session fan-out.** `telemetry_ws_controller` currently subscribes per-connection. A shared subscriber multiplexing multiple WS clients on one Redis subscription is on the backlog. | `telemetry_ws_controller.cpp` |
| 10 | **Tests cover the unit + adapter level.** Integration tests against a live IO-Link master are environment-specific and gated by `QNC_IOLINK_HIL_MASTER` (not in CI). | `tests/unit/iolink/*` |
