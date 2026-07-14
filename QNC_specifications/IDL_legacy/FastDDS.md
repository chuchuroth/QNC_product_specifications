
## The Role of FastDDS in Industrial Intelligent Gateways

Combining the product plan you mentioned earlier — an industrial intelligent gateway that supports multiple traditional industrial protocols (EtherCAT, PROFINET, Modbus, etc.) and provides OPC UA/MQTT upwards — **the addition of FastDDS will fundamentally change the positioning and value proposition of this product**.

In simple terms: **FastDDS does not replace any of your existing protocols; instead, it upgrades your gateway from a “protocol converter” to a “native robotic communication hub.”**

### I. What is FastDDS?

FastDDS (Fast Data Distribution Service) is a high-performance real-time communication framework developed by eProsima based on the OMG DDS standard. It has the following key features:

| Feature                  | Description |
|--------------------------|-------------|
| **Publish-Subscribe Model** | Decentralized, peer-to-peer communication between nodes with no central server |
| **Real-time Performance**   | Based on RTPS/UDP protocol, with latency as low as 0.2–1 ms |
| **Fine-grained QoS Control** | Precise configuration of reliability, latency, bandwidth, data lifespan, etc. |
| **Automatic Discovery**     | Nodes automatically discover each other upon coming online, no manual configuration required |
| **Default Middleware for ROS 2** | De facto standard in the robotics ecosystem |

**Key Understanding**: DDS is the “interface specification,” RTPS is the “underlying protocol,” and FastDDS is a “concrete implementation.”

### II. The Core Position of FastDDS in the Robotics Field

When you say “my robot communication uses FastDDS,” it means that all communication between nodes inside your robot system (most likely based on ROS 2) is using FastDDS as the data bus.

**Typical Scenario**:
- Perception nodes on the robot body (LiDAR, cameras) publish sensor data
- Planning nodes subscribe to map data and publish trajectory commands
- Control nodes subscribe to trajectories and publish joint commands to servo drives

**Architecture Diagram**:
```
[Perception Node] ──FastDDS──→ [Planning Node] ──FastDDS──→ [Control Node] ──→ Servo Drive
     ↑                      ↓                      ↓
     └──────FastDDS─────────┘                      └──(Conversion Required)──→ Industrial Ethernet
```

The problem is: **FastDDS is the “internal” communication bus of the robot, but communication between the robot and external devices (PLC, sensors, cloud) still requires traditional industrial protocols.**

### III. The Role of FastDDS in Your Gateway

#### Role 1: The Robot’s “Northbound Interface” (Most Important)

Your gateway can act as a **DomainParticipant in the FastDDS domain**, directly subscribing to all data topics published by the robot’s internal nodes, while publishing commands from external systems back to the robot.

```
┌─────────────────────────────────────────────────────────────┐
│                  Robot Internal (FastDDS Domain)             │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐                  │
│  │ Perception│  │ Planning │  │ Control  │                  │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘                  │
│       │             │             │                         │
│       └─────────────┼─────────────┘                         │
│                     │ FastDDS                                │
└─────────────────────┼───────────────────────────────────────┘
                      │
                      ▼
              ┌───────────────┐
              │   Your Gateway│ ◄── Acts as FastDDS Participant
              │(Industrial Intelligent Gateway)│
              └───────┬───────┘
                      │
        ┌─────────────┼─────────────┬─────────────┐
        ▼             ▼             ▼             ▼
   ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐
   │PROFINET │  │EtherCAT │  │Modbus   │  │OPC UA   │
   │ Master/Slave│ Master/Slave│ RTU/TCP │ Server  │
   └─────────┘  └─────────┘  └─────────┘  └─────────┘
        │             │             │             │
        ▼             ▼             ▼             ▼
   [PLC Network]   [Servo Drives]  [Sensors/Instruments]  [MES/Cloud]
```

**Specific Value**:
- The gateway can access all sensor data and status information without modifying the robot’s internal code
- The gateway can convert external commands (e.g., start/stop commands from a PLC) into FastDDS messages and publish them, allowing the robot to respond automatically
- This is “non-intrusive integration” — completely transparent to the robot system

#### Role 2: “Data Exchange Center” for Multi-Robot Systems

When there are multiple robots, the gateway can serve as a **central hub for cross-robot data exchange**:

```
Robot A (FastDDS Domain) ──→ Gateway (FastDDS Participant) ──→ Robot B (FastDDS Domain)
                              │
                              ▼
                       [Unified Data Logging / Analysis]
```

**Application Scenarios**:
- Multi-robot collaboration: Robot A detects an obstacle and notifies Robot B through the gateway
- Cluster scheduling: The gateway aggregates position information from all robots for unified scheduling

#### Role 3: “Bridge” Between Robot Data and IT Systems

FastDDS excels at real-time data distribution but does not directly provide standard interfaces to MES/ERP or cloud platforms. Your gateway can:

| Direction          | Conversion          | Target Protocol      |
|--------------------|---------------------|----------------------|
| Robot → Upper Layer| FastDDS → OPC UA    | MES, SCADA systems   |
| Robot → Cloud      | FastDDS → MQTT      | IoT platforms, remote monitoring |
| Robot → Database   | FastDDS → SQL/InfluxDB | Historical data storage and analysis |

### IV. Positioning Compared with Traditional Industrial Protocols

Among the protocols you are already focusing on, FastDDS is positioned as a **brand-new protocol that stands alongside PROFINET/EtherCAT but serves different scenarios**:

| Protocol                  | Typical Role                                      | Positioning in the Gateway |
|---------------------------|---------------------------------------------------|----------------------------|
| **EtherCAT / PROFINET**   | Motion control bus between robot body and servo drives | Gateway acts as slave to collect/send data |
| **Modbus RTU/TCP**        | Communication with simple devices such as sensors and instruments | Gateway acts as master to collect data |
| **OPC UA**                | Standardized interface with MES/ERP/SCADA         | Gateway acts as server to expose data |
| **MQTT**                  | Lightweight communication with cloud platforms    | Gateway acts as client to push data |
| **FastDDS**               | **Real-time data bus between internal robot nodes** | **Gateway acts as participant to directly subscribe/publish robot data** |

**Core Difference**:
- Other protocols are for “cross-system” communication (between robot and external world)
- FastDDS is for “intra-system” communication (between nodes inside the robot). The gateway’s role is to **bridge the internal and external worlds**.

### V. Impact on Your Product’s Commercial Value

#### 1. Opening New Target Customers

| Original Target Customers          | Newly Added Target Customers |
|------------------------------------|------------------------------|
| Robot system integrators           | **ROS 2 robot developers/manufacturers** |
| Factory automation departments     | **Multi-robot cluster scheduling system providers** |
| Equipment OEMs                     | **Robot-as-a-Service (RaaS) operators** |

#### 2. Differentiated Competitive Advantage

Most industrial gateways on the market **do not support FastDDS**. This means:
- Existing products can only communicate with ROS 2 robots indirectly via Modbus/TCP or OPC UA, resulting in low efficiency and poor real-time performance
- Your product **natively supports FastDDS**, allowing direct integration into the ROS 2 ecosystem — a unique selling point

#### 3. Technical Barrier

Implementing FastDDS integration requires:
- Understanding DDS QoS configuration (reliability, history, durability, etc.)
- Handling IDL (Interface Definition Language) code generation
- Possibly implementing **Discovery Server** mode in the gateway for large-scale deployments

These technical thresholds create a strong competitive moat.

### VI. Product Solution Adjustment Recommendations

#### Hardware Aspect

FastDDS is based on UDP/IP and has no special hardware requirements. Your existing netX 90 + main CPU solution is **fully compatible** — you only need to run the FastDDS application in the main CPU’s Linux system.

#### Software Architecture Aspect

It is recommended to add a **FastDDS Agent/Participant module** to the gateway:

```
┌─────────────────────────────────────────────────────┐
│                Your Gateway Software Architecture    │
├─────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  │
│  │ FastDDS     │  │ Modbus      │  │ PROFINET/   │  │
│  │ Participant │  │ Master      │  │ EtherCAT    │  │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘  │
│         │                │                │         │
│         └────────────────┼────────────────┘         │
│                          ▼                          │
│              ┌─────────────────────┐                │
│              │   Unified Data Bus   │                │
│              │ (Shared Memory/Data Lake) │           │
│              └──────────┬──────────┘                │
│                         │                           │
│         ┌───────────────┼───────────────┐           │
│         ▼               ▼               ▼           │
│   ┌──────────┐   ┌──────────┐   ┌──────────┐       │
│   │ OPC UA   │   │ MQTT     │   │ Local Storage│   │
│   │ Server   │   │ Client   │   │              │   │
│   └──────────┘   └──────────┘   └──────────┘       │
└─────────────────────────────────────────────────────┘
```

#### Deployment Modes

Depending on the scale of the robot system, different FastDDS deployment modes can be selected:

| Deployment Mode       | Applicable Scenario               | Gateway Role |
|-----------------------|-----------------------------------|--------------|
| **SIMPLE (Default)**  | Single robot, small scale         | One of the participants, automatic discovery |
| **Discovery Server**  | Multi-robot, cross-network        | Can act as a lightweight discovery server |
| **STATIC Discovery**  | Fixed topology, high performance requirements | Pre-configured participants |

### VII. Summary

| Dimension                    | Conclusion |
|------------------------------|----------|
| **Role of FastDDS**          | The gateway’s “northbound interface” connecting to the robot’s internal data bus — the native entry point for robot communication |
| **Does it replace existing protocols?** | No. It is an **additional protocol support oriented toward the ROS 2 robotics ecosystem** |
| **Impact on Product Positioning** | Upgrades the product from an “industrial protocol conversion gateway” to a “native robotic data hub” |
| **Commercial Value**         | Opens up the ROS 2 robot market and creates a differentiated competitive advantage |
| **Technical Feasibility**    | Fully feasible; existing hardware requires no changes, only a new software module needs to be added |

**One-sentence summary**: After adding FastDDS support, your product is no longer just a “multi-protocol converter,” but a **bidirectional bridge between the robot’s internal communication (FastDDS) afassdasnd the external world (PROFINET/EtherCAT/OPC UA/MQTT)** — precisely the critical missing link in modern smart factory robot system integration.
