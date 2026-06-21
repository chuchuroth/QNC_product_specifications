# NeuraSync Ecosystem — Consolidated Technical Source of Truth

Updated: March 2026

## 1. Purpose and Scope

This document consolidates prior NeuraSync documentation into one canonical reference for:

- NeuraSync architecture and repository roles
- FastDDS → NeuraSync migration work in `neurasync-robot-client`
- IDL toolchain and interface package ecosystem
- DDS feature coverage, QoS profiles, and known limitations
- Code quality findings and prioritized recommendations

It supersedes and replaces previous split documents.

---

## 2. Workspace and Ecosystem Overview

NeuraSync is an opinionated communication framework built on top of eProsima FastDDS, with standardized factories, QoS presets, node lifecycle APIs, IDL generation tooling, and visualization utilities.

### 2.1 Primary repositories and responsibilities

| Repository | Role |
|---|---|
| `neura-sync` | Core DDS abstraction (`NeuraSyncNode`, factories, pub/sub/service wrappers) |
| `neurasync-dashboard` | Real-time DDS visualization and dynamic topic inspection |
| `neura_sync_idl_tools` | CMake macro package for IDL → C++ generation |
| `neuraverse-neurasync-idls` | Fleet-wide IDL catalog and distributable interface packages |
| `Fast-CDR` | CDR serialization dependency |
| `Fast-DDS-Gen` | IDL generator toolchain dependency |

### 2.2 Baseline technology stack

- C++17
- eProsima FastDDS 3.4.1
- eProsima Fast-CDR 2.3.4 (core baseline)
- GoogleTest
- CMake 3.16+
- Debian packaging via CPack/JFrog workflows
- Dashboard UI stack: Dear ImGui + ImPlot + GLFW + OpenGL3

---

## 3. Architecture

### 3.1 Layered architecture

1. **Application layer**
   - Robot control applications
   - QNC ModbusRTU↔DDS bridge
   - NeuraSync dashboard

2. **NeuraSync abstraction layer**
   - `NeuraSyncNode`
   - `ParticipantFactory`, `PublisherFactory`, `SubscriberFactory`
   - `ServiceClient` / `ServiceServer`

3. **Middleware layer**
   - FastDDS runtime entities and RTPS transport
   - Fast-CDR serialization
   - foonathan memory allocator

4. **Transport layer**
   - Shared memory (local real-time)
   - UDP/TCP networking
   - Data-sharing and zero-copy transport modes where configured

### 3.2 Build dependency hierarchy

- `neura_sync_meta`
  - `neura_sync_core_msgs` (IDL-generated core messages)
  - `neura_sync_utils`
  - `neura_sync_config`
  - `neura_sync` (core runtime library)
- `neura_sync_idl_tools` provides `generate_msg_from_idl()` used across message packages

---

## 4. Module Analysis

### 4.1 `neura-sync` (core library)

Core API surface includes:

- `NeuraSyncNode`
- `Publisher<T, PubSubType>` / `Subscriber<T, PubSubType>`
- `ParticipantFactory` (participant creation presets)
- `PublisherFactory` / `SubscriberFactory` (QoS profile-based entity creation)
- `ServiceClient` / `ServiceServer` (request/reply over topic pairs)

Key design patterns:

- Template-based type safety over IDL-generated topic types
- Factory-driven QoS defaults
- Pimpl encapsulation in `NeuraSyncNode`
- Service correlation via `SampleIdentity`

### 4.2 `neura_sync_config` and transport presets

`ParticipantQosFactory` and `PubsubQosFactory` expose configurable presets:

- Shared memory only
- Default UDP multicast
- UDP with IP constraint
- SHM+UDP hybrid
- SHM peer-to-peer unicast
- Discovery server/client mode
- XML-driven participant configuration

Notable default constant:

- `SHM_DEFAULT_SEGMENT_SIZE_BYTES = 1,048,576` (1 MiB)

### 4.3 `neura_sync_utils`

- Thread-safe logger singleton with multiple levels
- Timing/IP utility helpers
- Optional trace macros and real-time thread utilities

### 4.4 `neura_sync_core_msgs` (internal message layer)

Contains core IDL definitions for robot commands, states, multi-robot state, Modbus abstraction, external PDO, tool/safety IO, time, and test service request/response types.

### 4.5 `neurasync-dashboard`

Major capabilities:

- Dynamic topic discovery and runtime type introspection
- Multi-connection manager for robot sessions
- Real-time plotting (ImPlot)
- Online statistics engines (P² / T-Digest / traditional)
- Formula processor for derived signals
- CSV recording and configurable buffering

### 4.6 `neuraverse-neurasync-idls`

Robot/system interface packages include:

- `neura_core_interfaces`
- `mipa_mav_interfaces`
- `lara_maira_interfaces`
- `4ne1_core_interfaces`
- `shd_interfaces`

Design conventions:

- ROS-style `Header` and time representations in core interfaces
- Bounded sequences for deterministic memory behavior
- `@Key` usage for DDS instance discrimination where needed
- Cross-package IDL includes (for example, `mipa_mav_interfaces` importing `neura_core_interfaces` definitions)

Distribution model:

- Built and packaged per sub-package as Debian artifacts
- Published through JFrog Artifactory pipelines
- Optional ROS2/ament exports when `ament_cmake` is available

### 4.7 `neura_sync_idl_tools`

Provides the reusable `generate_msg_from_idl()` macro.

Signature:

```cmake
generate_msg_from_idl(
  <lib_target_name>
  <IDL_FILES>
  <IDL_INCLUDE_DIRS>
  <GENERATED_DIR>
  [SafeDDS]
)
```

Backend behavior:

- FastDDS path (default): `fastddsgen` (v4.2.0 baseline) generating headers + sources
- SafeDDS path (optional): `safeddsgen` header-only generation

SafeDDS build caveat:

- Header-only targets still require an explicit custom target dependency to force generation during build.

---

## 5. DDS Feature Coverage and QoS Model

### 5.1 Feature coverage summary

Implemented/wrapped:

- Publish/subscribe abstractions
- Request/reply service abstraction over DDS topics
- Discovery modes (simple and server/client)
- Shared-memory, UDP, and server-driven transport patterns
- Data-sharing and zero-copy factory profiles
- Keyed topics (`@Key` in IDL)
- Deadline/liveliness parameters and callbacks
- XML profile support

Not currently implemented/exposed:

- DDS Security
- Content-filtered topics
- Persistence service integration

### 5.2 QoS profile matrix

| Profile | Reliability | Durability | History | Publish Mode | Typical Use |
|---|---|---|---|---|---|
| `Default` | BEST_EFFORT | VOLATILE | KEEP_LAST(10) | ASYNC | High-rate telemetry |
| `ControlData` | BEST_EFFORT | VOLATILE | KEEP_LAST(1) | SYNC | Control loops |
| `BigData` | BEST_EFFORT | VOLATILE | KEEP_LAST(3) | ASYNC | High-volume streams |
| `BigFile` | RELIABLE | VOLATILE | KEEP_LAST(3) | ASYNC | Large payload transfer |
| `CriticalData` | RELIABLE | VOLATILE | KEEP_LAST(20) | SYNC | Safety/error/command-critical paths |
| `DataSharing` | RELIABLE | VOLATILE | KEEP_LAST(20) | SYNC | Intra-host IPC |
| `ZeroCopy` | RELIABLE | VOLATILE | KEEP_LAST(20) | SYNC | Low-latency local data paths |

---

## 6. FastDDS → NeuraSync Migration in `neurasync-robot-client`

### 6.1 Scope and impacted files

Migration replaced direct FastDDS entity management with NeuraSync factory APIs in:

- `CMakeLists.txt`
- `src/gripper_control_dds.cpp`
- `deploy_on_qnc/qnc_bridge.cpp`
- `src/gripper_control.cpp` (dead conditional DDS block removed)

### 6.2 Build and generation changes

- Removed legacy CMake toggles (`BUILD_WITH_DDS`, `BUILD_WITH_NEURASYNC`)
- Added `find_package(neura_sync REQUIRED)` and linked `neura_sync::neura_sync`
- Switched to pre-generated types from `neurasync/generated/`
- Avoided incompatible system `fastddsgen` 2.3.0 output with FastDDS 3.4.1 headers

IDL regeneration guidance (when IDL changes):

- Use FastDDS 3.x-compatible `fastddsgen` (v4.2.0 baseline; any compatible 4.x acceptable)
- Regenerate into `neurasync/generated/` and rebuild consumers

### 6.3 QoS migration notes for Modbus bridge topics

Command and response/statistics reliability were strengthened by aligning key paths to `CriticalData` where required.

Important compatibility note:

- Legacy command flows used `TRANSIENT_LOCAL` durability in some paths.
- NeuraSync presets are `VOLATILE`.
- Delivery assurance is maintained through explicit endpoint matching waits before first publish (`wait_for_bridge_match()` / `wait_for_publisher_match()`).

### 6.4 Runtime architecture shift in QNC bridge

Old pattern:

- Polling loop calling `take_next_sample()` directly

New pattern:

- Subscriber callbacks enqueue incoming requests
- Main loop drains queues and performs serial Modbus transactions single-threadedly

This callback→queue bridge preserves thread-safe serial bus access while adopting asynchronous listener-based DDS reception.

### 6.5 Lifecycle and matching differences

Raw FastDDS approach:

- Manual create/delete of participant, publisher/subscriber, topics, and data writers/readers

NeuraSync approach:

- Factory-created entities managed through node lifecycle
- Reduced teardown complexity via node reset and participant cleanup wrappers

---

## 7. IDL Toolchain and Interface Package Topology

### 7.1 Logical split

- `neura_sync_idl_tools` = generation toolchain package
- `neuraverse-neurasync-idls` = interface-definition/data package collection

### 7.2 Dependency and consumption model

- Internal middleware (`neura_sync_core_msgs`) uses the same IDL generation macro
- Robot/application-specific projects (including QNC-related targets) can define local IDL and generate code directly
- `neuraverse-neurasync-idls` packages are primarily distributed for external consumers/integrators

---

## 8. Data-Flow Patterns

### 8.1 Publish path

1. Application obtains publisher through a factory preset
2. Type registration/topic creation and writer setup occur under factory logic
3. Application sets a publish buffer and invokes publish
4. FastDDS transport layer handles delivery over configured transport

### 8.2 Subscribe path

1. DataReader listener receives transport data
2. Sample deserialization occurs via generated Fast-CDR code
3. User callback handles typed message payload

### 8.3 Service path

1. Client publishes request to `<service>/request`
2. Server callback computes response and correlates with request identity
3. Server publishes to `<service>/response`
4. Client resolves the pending promise/future using related sample identity

---

## 9. Quality Assessment and Known Technical Debt

### 9.1 Strengths

- Clear modular decomposition across config/utils/core/messages
- Strong API simplification over raw DDS boilerplate
- Practical QoS profile taxonomy for robotics traffic classes
- Mature packaging/distribution patterns
- High-value dynamic dashboard capabilities for observability/debugging

### 9.2 Current risks/limitations

| Area | Observation | Impact |
|---|---|---|
| Factory map initialization | Potential thread-safety risk in singleton maps during concurrent first access | Medium |
| Participant ownership model | Raw participant pointer lifetime discipline still required in some paths | Medium |
| Service pending map cleanup | Client timeout cleanup can be manual, enabling accumulation if misused | Medium |
| Hardcoded defaults | Some network/runtime defaults remain embedded | Low |
| Security/filtering/persistence | Not surfaced/implemented in current abstraction | Medium for broader/untrusted deployments |
| Version coupling | Fast-CDR version assumptions may diverge across components | Low-to-medium operational fragility |

---

## 10. Consolidated Recommendations

1. Add synchronization around per-participant factory singleton map creation.
2. Strengthen participant lifetime safety (RAII wrapper or smart-pointer deleter model where feasible).
3. Add optional service-request timeout policies with automatic pending-map cleanup.
4. Externalize network defaults and transport constants into configuration.
5. Evaluate DDS Security for deployments outside trusted LAN boundaries.
6. Keep FastDDS/Fast-CDR/generator versions aligned across core and dashboard runtime environments.

---

## 11. Canonical Notes

- NeuraSync is a wrapper/ecosystem over FastDDS, not a clean-room DDS reimplementation.
- The migration in this workspace intentionally prioritizes stable delivery via endpoint matching over TRANSIENT_LOCAL durability behavior.
- The IDL ecosystem intentionally separates toolchain concerns from interface package ownership.
