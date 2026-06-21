**QNC Definition (from hardware perspective):**
A physical hardware module installed between a robot control system and industrial end effectors (grippers, sensors, motors, etc.). It handles raw fieldbus communications (Modbus RTU, IO-Link, EtherCAT, etc.), allowing robot developers to avoid writing device drivers. Device semantics are stored on the robot platform as JSON descriptors; QNC is responsible only for protocol-layer data transport.

**Strategy:**
A phased product strategy—launching the bridge first and then building an intelligent module platform—is more feasible than attempting both simultaneously.

### Replace Marketing Claims with Measurable Interface Commitments

The current Design Philosophy section (README) reads more like sales copy embedded within a technical reference document. For infrastructure documentation, value propositions should be expressed as verifiable interface contracts:

* “Adding a new device only requires a JSON descriptor; no firmware recompilation is needed.” *(verifiable)*
* “A descriptor for any Modbus RTU device can be generated solely from the manufacturer's datasheet.” *(verifiable and testable)*
* “Protocol debugging is isolated within the QNC subsystem.” *(verifiable)*

---

## Issue 1: The Principle “QNC Handles Protocols, Not Devices” Is Architecturally Unstable

To convert a `GenericCommand{address=259, value=500}` into a valid Modbus RTU frame, QNC must know:

* Which Modbus function code to use (FC03, FC04, FC06, FC16 — not interchangeable)
* Whether address 259 is a holding register or an input register
* The correct byte order and data encoding format

These decisions are themselves device semantics.

A “pure protocol bridge” that accepts:

```text
WriteCommand{slave=1, reg=259, val=500}
```

is not truly generic—it is merely a Modbus API wrapper that assumes the caller already knows the device register map.

The real abstraction is not **“no device semantics”**, but rather:

> **“Device semantics are delegated to the caller.”**

This is a weaker but more accurate statement, and it still provides genuine value.

### Recommendation

Reframe the core principle honestly:

> “QNC moves device-semantic complexity out of firmware and into declarative configurations managed by the robot platform.”

This remains a strong value proposition and can withstand engineering scrutiny, whereas a “zero device semantics” claim cannot.

---

## Issue 2: The JSON Descriptor Model Is Underdefined as a Product Feature

### Problem

The JSON descriptor concept—the carrier of device intelligence—is both the most important feature in the QNC value proposition and the least defined.

in the repository references a `ModbusRTU_DeviceDescriptor_Schema.json`.

However, the following questions remain unanswered:

* Who creates and maintains descriptors? (Device manufacturers? Integrators? A community library?)
* How are descriptors versioned and distributed?
* How is descriptor correctness validated?
* Can an invalid descriptor crash a robot?
* Is there a certified registry of descriptors for well-known devices?

The entire claim of **“supporting a new device in 30 minutes”** depends on descriptor creation being extremely simple.

If writing a correct descriptor requires:

* Reading a 200-page device manual
* Calculating scaling factors
* Determining proper function codes
* Testing edge cases

then the actual time savings will be significantly lower than advertised.

### Recommendation

The descriptor ecosystem—authoring tools, validation toolchains, certified libraries, and distribution mechanisms—must be defined as a first-class product feature rather than a footnote.

Without it, the value proposition still depends on work the customer must perform themselves.

---

## Issue 3: “Protocol Bridge” and “Universal Connector” Are Different Product Archetypes

### Problem

The name **Quick Network Connector** implies a physical connectivity product—a device used to connect arbitrary equipment.

The description of a **protocol bridge**, however, suggests a software-defined translation layer.

These are two different product categories:

### Connector Product

Competes with:

* Anybus multi-protocol I/O modules
* Hardware protocol gateways
* Robot tool changers

Differentiation:

* Mechanical compatibility
* Plug-and-play deployment

### Bridge Product

Competes with:

* OPC-UA servers
* Modbus-to-DDS gateways
* Open-source communication libraries
* Robot SDK integration layers

Differentiation:

* Software abstraction

The PRD implicitly acknowledges this tension through concepts such as the “soft connector” and “rubber wristband” mechanical vision, but never resolves what the product fundamentally is.

The answer to **“What is QNC?”** should be a single sentence that both engineers and customers can use consistently.

### Recommendation (Draft)

> “QNC is a hardware module installed between a robot flange and any industrial end effector, providing software-configurable communication bridging so that robots do not require device-specific firmware.”

Every feature decision should then be evaluated against this definition.

---

## Issue 4: Product Boundaries Are Undefined at the Most Critical Point

### Problem

QNC positions itself between robots and end effectors.

However, the most important architectural question remains unanswered:

> Where does device intelligence actually reside—in the robot platform or in QNC?

This is not a minor implementation detail.

It determines:

* Who is responsible for device onboarding
* How updates are deployed
* What happens during hot-plug events
* Whether QNC can operate independently

Before implementation begins, the product’s system boundaries must be unambiguous.

### Recommendation

Create a single, explicit system boundary diagram that clearly answers:

* Where data is stored
* Who writes it
* Who reads it
* What happens to the data when QNC is disconnected

---

## Issue 5: Hot-Plugging and Disconnect-While-Moving Cannot Be Reconciled Without a Robot-Side Safety Contract

### Problem

QNC simultaneously claims to support:

1. Hot-plugging (device replacement without rebooting)
2. Emergency-stop behavior when a device disconnects during motion

These requirements point in opposite directions.

The system cannot determine whether a disconnection is:

* Intentional (hot-plug operation)
* Accidental (failure)

because it lacks the necessary robot-side context.

### Intentional Hot-Plug

* Robot is stopped
* Operator removes the device
* Safe
* Continue detection and reconfiguration

### Accidental Disconnect During Motion

* Robot is moving
* Device falls off or loses communication
* Critical event
* Trigger emergency stop

QNC alone cannot distinguish between these scenarios.

The PRD states that the robot **“should”** (not **“must”**) trigger an emergency stop, placing responsibility on the robot platform.

However, QNC provides no reliable mechanism for the robot to communicate its motion state, which is precisely the information required to classify disconnections correctly.

### Recommendation

Define a `RobotMotionState` input interface through which the robot continuously publishes its current state:

* Moving
* Idle

QNC can then use this information to determine whether a disconnect is:

* A valid hot-plug event
* A safety-critical fault requiring a protective response

This is a simple, concrete addition that closes the architectural gap.

---

## Issue 6: The Competitive Differentiation Against Anybus and Hilscher Is Unresolved

### Problem

Commercial solutions such as Anybus CompactCom and Hilscher netX already provide:

* Industrial protocol conversion (Modbus, EtherCAT, EtherNet/IP, PROFINET, etc.)
* Industrial certifications
* Hardware-validated multi-protocol gateways
* Products specifically marketed for robot tool integration

The PRD contains no competitive analysis.

The claimed differentiation (NeuraSync/DDS integration) only matters within the NeuraSync ecosystem.

From the perspective of a customer integrating a UR or FANUC robot, Anybus is a more established and widely certified solution.

### Recommendation

Add a dedicated competitive positioning section to the PRD.

Potential differentiators may include:

**(a) Native DDS integration**

* Optimized for low-latency robotic software stacks

**(b) Descriptor-driven onboarding**

* Eliminates firmware modifications for supported devices

**(c) Flange-mounted form factor**

* Designed specifically for robot end-effectors

These differentiators are worth articulating precisely and defending explicitly.

---

## Issue 7: The Descriptor Model Creates Long-Term Ecosystem Dependencies

### Problem

A descriptor-based architecture only delivers its promised value if a substantial library of prevalidated descriptors exists.

The documentation currently makes no mention of such a library.

This creates a bootstrap problem:

The first customer using an unsupported device experiences essentially the same workflow as writing a driver from scratch—they must create and validate the descriptor themselves.

This is a well-known pattern in developer tooling:

* ROS 2 succeeded in part because of its large package ecosystem.
* IDE plugin architectures depend on plugin marketplaces.

Without a planned descriptor registry, the promise of **“30-minute device support”** applies only to devices that already have descriptors.

At launch, that number is effectively zero.

### Recommendation

Define a **QNC Device Descriptor Registry** as a first-class deliverable that ships alongside v1.0, even if it initially contains only 5–10 of the most commonly used grippers.

The registry itself becomes:

* A competitive moat
* A core product asset
* The primary driver behind reduced integration time

Without it, the descriptor architecture remains a technical mechanism rather than a customer-visible advantage.
