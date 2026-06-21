




 presentation_talking_script


 the current mipa structure:
 
this is the current mipa structure, i got here the whole technical specifications, but you are probably more familiar than i am, so i will skip this part and get to the point, like what is my plan, and idea at this moment.


 system has two layers: upperbody highcompute intelligence and a chassislevel realtime mobility and safety layer, interconnected by a 1gigabit Ethernet backbone, the Ethernet connect: Upperbody AI compute, and Arm control modules, and The chassis IPC

 The upper body vision system runs on dual NVIDIA Jetson Orin modules, handling highbandwidth camera streams and perception workloads. The chassis sensors—LiDAR, IMU, ultrasonics—are handled by an industrial PC running ROS 2. This separation isolates realtime navigation and safety from heavy vision compute,










**MiPA-X uses **dual NVIDIA Jetson Orin modules** as its primary AI and vision processors, with a major vision pipeline**, handeln:
* Multi-camera RGB-D vision processing
* AI inference and decision making
* 720p video streaming at 30 frames per second
* Audio processing, including microphone array input and echo cancellation
* 


+ AI workloads is High-latency-tolerant, like vision, perception, inference, and planning, which are isolated from hard real-time control loops that must operate at kilohertz rates without interruption.

so tasks can be isolated —vision pipelines can scale independently from higher-level reasoning and interaction logic.

Supporting the Jetsons is a **Verdin iMX8M Plus System-on-Module**, which serves as a **real-time bridge** between high-level planning and physical actuation. This module interfaces directly with the dual robotic arms, managing:

* Real-time trajectory execution
* Dual CAN bus communication
* ROS 2 driver nodes for arm control


At the lowest level of the system sits an **STM32H7 microcontroller**, connected over CAN at 1 megabit per second.

This MCU runs a **1-millisecond real-time control loop**, handling:

* Low-level motor commands
* Power distribution and monitoring
* Emergency stop handling
* Hardware watchdogs and safety interlocks

This design ensures that even in the event of higher-level compute overload or network congestion, the robot remains electrically and mechanically safe.

The chassis contains an **industrial PC running ROS 2**, acting as the central coordinator for navigation and base mobility.

Its responsibilities include:

* SLAM and mapping
* IMU and ultrasonic sensor fusion
* Wheel odometry and motion control
* Coordination with the upper-body compute stack

Sensor fusion is performed via EKF at 50 hertz, combining IMU data with wheel odometry for stable localization.



# Sensor Architecture Overview

# MiPA-X is heavily sensor-driven. The sensor coverage is thorough (the Field of View for perception is seriously designed). The arms are 7-Degree of freedom, with 600mm linear lift give enough manipulation capability.
+ i think they are expert on sensor fusion, implement quite well, so i don't think i would change anything radically on that front, just to make the most of it, and maybe reorgnise the structure a little bit to make the resourse magament a bit more reasonable and efficient, because i think the sensors setting could be a little overkill
+ the only changement i would made at the moment is probably to add a lightweight language module in one of the orins, so the two jetson orins to have different focus.

+ Orin #1 is dedicated entirely to perception Vision, audio, sensor fusion, and scene understanding all live here.
+ Orin #2 handles cognition Task planning, dialog, speech output, and a **lightweight language model**, supported by a deterministic behavior state machine.

so The state machine is a classic rule based algorithm, guarantees predictable behavior, while the language model adds flexibility for natural interaction.

If the language model fails, the robot does not fail—it falls back to rule-based behavior.


#### in the vision stack(on the upper body)

+ multilayered sensing
+ separate global perception, local precision sensing, and redundant safety sensing, and then fuse them through ROS 2 into a single coherent world model.
+ The backbone is a wideangle RGBD camera with a 190degree horizontal and 114degree vertical field of view. At 1280 by 720 resolution and 30 Hz, it provides dense spatial awareness out to 100 meters and acts as the main input for scene understanding, obstacle detection, and semantic perception.
+ Complementing this is a secondary RGBD camera with a narrower 86 by 58 degree field of view, taskfocused. It’s positioned to cover the manipulation zone, and obstacles approaching from outside the primary RGBD frustum. Distortion is handled explicitly through a calibrated fisheye model, which keeps them usable for downstream perception pipelines.
+ dual fisheye cameras, mounted front and rear. Each provides a 120degree horizontal field of view at full HD resolution. These cameras are less about metric depth and more about situational awareness—detecting motion, humans
+ the 335LG precision depth sensor. This is a narrowfield, highaccuracy depth unit—18 degrees vertical, 30 degrees horizontal—mounted lower on the platform. Its role is very specific: precise ranging for heightcritical or geometrysensitive tasks, where wideangle depth becomes noisy or ambiguous.
+ vision sensors are managed through ROS 2 topics with clear frame hierarchies, deterministic fusion 



#### navigation and localization layer(on the chassis)

+ center is a 360degree 3D LiDAR, providing full horizontal coverage with a ±15degree vertical field. Running at 10 Hz with up to 32 vertical layers, this sensor is the backbone of SLAM and global mapping. It gives us reliable geometry regardless of lighting conditions and acts as the primary reference for largescale spatial consistency.
+ Alongside it is a 2D LiDAR, which serves two purposes. First, it provides a lightweight, fast obstacle detection layer for classic navigation stacks. Second, it acts as a fallback and simulationfriendly sensor, ensuring continuity across hardware configurations and development environments.
+ Orientation and motion are handled by a 6DOF IMU, running up to 100 Hz. This IMU is tightly integrated through an EKF, fusing angular velocity with wheel odometry to produce a stable `/odom` output. 
+ This fusion layer is critical—it smooths motion, stabilizes perception pipelines, and improves SLAM robustness during aggressive maneuvers.
+ MiPAX uses eight ultrasonic sensors, evenly distributed around the base to provide full 360degree closerange coverage. Each sensor operates at 30 Hz with a 75degree sensing cone and a minimum range down to 2 centimeters. These sensors are intentionally simple and independent. They are not used for mapping—they exist to stop the robot from hitting things, especially in blind spots or failure cases where higherlevel perception degrades.

All of this sensing hardware feeds into a distributed compute architecture.



+ In total, the sensor suite can generate over 200 megabytes per second of raw data. On a 1gigabit Ethernet backbone, that’s more than 170 percent utilization. This forces the system to rely on compression, selective streaming, and intelligent topic throttling, rather than assuming unlimited bandwidth.

+ Vision for understanding
+ LiDAR for geometry
+ IMU for motion
+ Ultrasonics for safety



MiPA-X features a **dual 7-degree-of-freedom arm configuration**, totaling 14 actuated joints.

Each arm is powered by **T-Motor AK80-6 actuators**,  real-time motor and power communication are communicating over dual isolated CAN buses, preventing interference from high-volume sensor traffic.

* One bus dedicated to the left arm
* One bus dedicated to the right arm

Control runs at **300 hertz per arm**, with approximately 67% bus utilization, leaving sufficient safety margin.


# **System Status and Ongoing Improvements**

As of version 3.1, core functionality—including dual Jetsons, LiDAR, ultrasonic safety, and multi-camera vision—is complete.

Current work in version 3.7 focuses on:

* Chassis size optimization
* Battery accessibility and user-swapping
* Improved cable management and serviceability
* Structural changes for easier maintenance

These refinements are aimed at transitioning from a functional prototype to a production-ready system.




---
---
---

# *MiPA-X Minimum Viable Software Architecture*

from product perspective, what is the minimum requirement for a service robot. what must a robot can. this is the point where i rethink the whole structure.

basically it can see, walk, touch, hear, talk, think, and understand its environment— while being **safely**, so that is priority.


---

### 1. Design Philosophy

Our architecture is guided by four core ideas.

First, **safety first**.
Hardware emergency stops, real-time collision detection, watchdogs, and strict motion limits always override AI behavior. higher layers can request but never override lower layers. the safety case doesn't depend on AI working correctly, because AI is sandboxed above a deterministic foundation.

Second, **graceful degradation**.
If part of the system fails—an AI model crashes, a compute module goes offline—the robot does not behave unpredictably. It steps down into simpler, safer modes, all the way to a complete safe stop if needed.

Third, **lightweight AI**.
We are not chasing perfection in version one. The goal is functionality that works reliably today, with a clean upgrade path for tomorrow.

And finally, **modularity**.
Every major capability—perception, cognition, navigation, control—can evolve independently without breaking the system.

---

### 2. High-Level System Overview 


At a system level, MiPA-X is organized around **clear separation of responsibility**. here is a little changement i made, maybe make the two orins to have different focus.
At the center are **two NVIDIA Jetson Orin modules**, which together act as the robot’s brain.

* **Orin #1** is dedicated entirely to **perception**
  Vision, audio, sensor fusion, and scene understanding all live here.

* **Orin #2** handles **cognition**
  Task planning, dialog, speech output, and a lightweight language model.

These Orins connect over Ethernet to a **chassis IPC**, which runs ROS 2 and is responsible for **navigation and safety-critical motion decisions**.

Finally, at the lowest level, **real-time motor and arm control** is handled by dedicated controllers over CAN bus.
This ensures that even if higher-level software misbehaves, the robot remains physically safe.


---

### 3. Layered Software Architecture

The software stack is organized into **six clean layers**.

At the bottom is **hardware**—sensors, actuators, and power.

Above that is **control**, which includes motor control, firmware limits, and emergency stop handling.

Next is **navigation**, where SLAM, localization, and path planning live.

On top of navigation is **perception**—vision, audio, and sensor fusion.

Then comes **cognition**, which includes decision-making, planning, and language understanding.

And finally, at the top, **applications and user interaction**—touchscreen UI, voice interaction, and external APIs.

Each layer only depends on the layers below it.
This keeps the system predictable, testable, and safe.

---

### 4. Safety Architecture

Safety deserves special attention, because it is not just a feature—it is the system’s backbone.

We define **five safety priority levels**.

* **Priority zero** is the **hardware emergency stop**.
  It bypasses all software and stops motion immediately.

* **Priority one** is **collision detection**, using ultrasonic sensors and LiDAR with sub-100 millisecond response.

* **Priority two** enforces **soft limits**—joint angles, velocities, and forces.

* **Priority three** adds **behavior limits**, like speed restrictions near people or no-go zones.

* **Priority four** consists of **watchdog timers**, ensuring that any loss of communication leads to a safe stop.

At the center of this is a mandatory **Safety Monitor node**, continuously validating system health and enforcing these rules.

No AI decision—no matter how intelligent—can override safety.

---

### 5. Perception and Cognition

All perception is centralized on **Orin #1**.

The robot processes camera feeds at 30 frames per second, detects people and objects, builds a 3D scene graph, and fuses this with LiDAR and IMU data.
The result is a **single, unified world model** that the rest of the system can trust.

Audio follows a similar pattern:
wake word detection, speech recognition, and intent classification run locally, with no dependency on the cloud.

On **Orin #2**, cognition is handled using a **lightweight language model**, supported by a deterministic behavior state machine.

This combination is intentional.
The state machine guarantees predictable behavior, while the language model adds flexibility for natural interaction.

If the language model fails, the robot does not fail—it falls back to rule-based behavior.

---

### 6. Behavior and Interaction

MiPA-X operates through a clearly defined behavior loop.

It starts idle.
When it hears its wake word, it listens.
It thinks—briefly—and then acts.

That action might be navigating, speaking, grasping an object, or asking a clarifying question.

At any point, a safety trigger can force an immediate transition into an emergency safe state.

For the user, interaction is simple and intuitive:
voice commands, a touchscreen interface, and clear visual feedback through LEDs.

---

### 7. Development Roadmap

This architecture supports a **phased development approach**.

In the first eight weeks, we focus on safety, navigation, basic perception, and simple voice interaction.

The next phase adds language understanding and arm manipulation.

Only after the full system is stable do we move into optimization, testing, and refinement.

This staged approach ensures that we always have a **working, safe robot**, even early in development.

---

### 8. Closing

To summarize:

This Minimum Viable Software Architecture gives us a **safe, modular, and future-proof foundation**.

It prioritizes human safety above all else.
It centralizes perception for clarity and performance.
It separates AI from motion control for reliability.
And it embraces lightweight, upgradeable intelligence rather than brittle complexity.

With this foundation, MiPA-X can already navigate indoor spaces, understand basic commands, interact naturally with people, and perform simple service tasks—while remaining predictable and safe.

Most importantly, this architecture is not an endpoint.
It is a **platform**—designed to grow as our AI, hardware, and ambitions evolve.



---
# refactoring
Although I don’t yet have access to the previous software codebase, my plan is to adopt it and refactor the legacy system into a new architecture.

overall goal of refactoring of the existing mipax codebase into MiPA X,

* Hard real-time control running at the millisecond level
* High-bandwidth perception and navigation on a GPU-accelerated compute platform
* And distributed autonomy implemented through skills and behavior orchestration

To achieve this, the codebase has been restructured into well-defined ROS 2 packages, each aligned with a specific responsibility in the system.

---

### **Package Renaming and Core Structure**

At the top level, we retain **`mipax`** as a meta-package. Its role is orchestration and dependency aggregation, not logic.

The foundational layer is **`mipax_base`**, which contains:

* Base configuration files
* The EKF state estimation setup
* And the ros2_control hardware abstraction layer

This package defines the system’s physical truth: frames, sensors, filters, and baseline control parameters.

From there, we introduce **`mipax_control`**, which is a key architectural upgrade.

---

### **Real-Time Control Architecture**

`mipax_control` formalizes the **1 millisecond real-time chassis control loop**.

This loop is executed on an **STM32H7 microcontroller**, communicating with the main compute platform over **CAN bus**. The responsibilities here are tightly scoped:

* Motor command execution
* Encoder feedback handling
* Deterministic control timing
* And hardware safety boundaries

On the ROS 2 side, this is exposed through a **ros2_control hardware interface**, ensuring compatibility with standard controllers while preserving real-time guarantees.

The key principle here is that **nothing non-deterministic crosses into this loop**. High-level logic, perception, and planning stay strictly upstream.

---

### **Compute and Perception Layer**

Moving upward, the primary compute platform is the **Jetson AGX Orin**.

To reflect this, we introduce **`mipax_perception`** as a first-class package. This includes:

* Camera and LiDAR integration
* Sensor fusion pipelines
* GPU-accelerated perception nodes

By isolating perception into its own package, we gain flexibility in upgrading sensors, swapping algorithms, or distributing workloads across multiple compute nodes without impacting control or navigation.

---

### **Navigation and Simulation**

Navigation is handled by **`mipax_navigation`**, which encapsulates Nav2 integration, maps, and planning configurations.

Simulation support is provided by **`mipax_gazebo`**, which mirrors the real robot’s interfaces as closely as possible. This ensures that control, navigation, and skills can be validated in simulation with minimal divergence from hardware behavior.

Robot models, URDFs, meshes, and kinematic descriptions live in **`mipax_description`**, keeping physical representation cleanly separated from logic.

---

### **Skills and Distributed Autonomy**

At the highest level, we evolve **`mipax_neuraverse`** into **`mipax_skills`**.

This package integrates **Neuraverse Skills**, enabling:

* High-level behavior trees
* Task abstraction
* And multi-robot coordination primitives

Rather than embedding autonomy logic directly into navigation or control nodes, MiPA X treats autonomy as a **composable skill layer**. This makes behaviors portable, testable, and scalable across fleets.

Integration with **NeuraSync** further allows monitoring, orchestration, and distributed execution across multiple robots.

---

### **Why This Matters**

This refactoring delivers several concrete benefits:

* **Deterministic real-time control**, isolated from high-level computation
* **Modular scalability**, from single robots to coordinated fleets
* **Hardware abstraction**, enabling future platform changes with minimal disruption
* And a **clean separation between control, perception, navigation, and autonomy**

Most importantly, MiPA X positions the platform not as a single robot stack, but as a **foundation for extensible robotic systems**.

---

---
---


# i got an idea

the idea is: **building a standardized robotic platform that can be equipped for any task, and establishing the interface as a standard for all physical hardware, the locomotives, wheels and legs, end-effectors, the hands.**


+ hardware first, maybe the robotic battle will be won at the interface layer, so we need to define hardware standards, the physical interface
+ If our vertebrae interface becomes the hardware standard, we become the necessary partner for any software ecosystem. If we are the first to deliver such a platform, there could be hundreds of end-effectors compatible with our robot. In that case, we would be launching an ecosystem rather than merely shipping a single product. 
+ The "capability bus" concept, software that describes what the robot can do. If a user swaps wheeled mobility for tracked, the system doesn't need reconfiguration; it just updates the capability manifest.

+ the problem with humanoid hand is it is too difficult to perfection. this approach would make a product that is more reliable than a half-functional humanoid hand and it is eaiser to perfection (narrow task perfection), because it is based on the embedded-systems industry, which has been developing for decades, and we would have an entire manufacturing ecosystem supporting it.  

+ standardized interface, it should be visible in the design—a prominent, elegant mounting point that says "this is where capability attaches." The current end-effectors (the grippers shown) look like standard robot grippers, not like the first instance of an ecosystem. What's the physical interface spec? How does tool-changing work? Is it manual or automatic? These questions should be driving the arm design.
if that is the direction, we should reconsider the whole structure and put "modularity " in the first place.
The architecture should be classical, minimalist, with simplified functionality, zero redundancy, and a strong emphasis on safety. It should be compatible with third-party ecosystems, effectively creating a ‘railroad’ for the industry—built on our foundation and our groundwork. 





## baby protector - inflatable safety shell
this is a baby protector idea i had, sort like airbag idea, if something urgent happens, a soft end-effector should pop out and creates a safe boundary without grabbing. 
i really like this idea, i think we should patent it. before someone else took it. 
The inflatable safety shell concept is genuinely novel and worth protecting.

---
---

## end-effector

**Let me show you a list of end-effectors just to give you a sense of how large this market could be and how much potential. For each end-effector, a new skill would be delivered via the Neuraverse, and with each new skill plus its corresponding end-effector, the robot effectively becomes a new service robot.**

This is when the platform economics really starts to boom. Here's an interactive catalog with **197 end-effectors** across **15 categories**. You can filter, search, and sort by priority or market size.

**Summary by Category:**

| Category | Count | Key Examples |
|----------|-------|--------------|
| Kitchen & Food | 18 | Knife tool, dishwasher loader, coffee maker interface |
| Cleaning | 14 | Vacuum nozzle, window squeegee, toilet brush |
| Logistics & Handling | 12 | Package lifter, bin picker, shelf stocker |
| Maintenance & Repair | 16 | Screwdriver set, drill, paint roller |
| Healthcare & Medical | 12 | Vital signs monitor, medication cart, patient lift |
| Household | 9 | Universal gripper, door handle operator, plug interface |
| Health & Personal Care | 14 | Medication dispenser, compression sock helper, emergency alert |
| Childcare & Pet Care | 12 | Bottle warmer, litter scoop, child safety blocker |
| Garden & Outdoor | 14 | Pruning shears, watering wand, snow shovel |
| Laundry & Fabric | 11 | Clothes folder, washer/dryer loader, bed maker |
| Safety & Security | 10 | Fire extinguisher operator, inflatable barrier, patrol camera |
| Commercial & Retail | 10 | Cash register, price tag applicator, shelf stocker |
| Office & Productivity | 10 | Document scanner, keyboard controller |
| Industrial & Workshop | 12 | Welding torch, assembly gripper, CNC tender |
| Entertainment | 8 | Camera controller, DJ equipment |

**The key insight your instinct captured:**

This list of 197 is just what I could enumerate in one pass. The actual market is **unbounded** because:

1. **Every human tool is a candidate** — anything designed for human hands can be redesigned for a standardized robot mount
2. **New tools emerge constantly** — as industries evolve, new end-effectors become relevant
3. **Task-specific optimization beats general-purpose** — a dedicated vegetable chopper will outperform a humanoid hand with a knife
4. **Third-party ecosystem** — once your interface is standard, other companies will build end-effectors you haven't imagined

**This is why the platform play is correct.** You're not selling a robot—you're selling access to an infinite catalog of physical capabilities. Every end-effector is:
- A potential SKU you sell
- A potential third-party product that increases your platform value
- A reason for customers to buy into your ecosystem

The Beetle analogy extends: the Beetle wasn't valuable because it was one car. It was valuable because it created a market for accessories, modifications, and services. Your robot platform could do the same for physical automation.


---
---



## a little thoughts on industrial design
+ persnally i like this design very much, it has this cute approachable image, like a chess piece or a penguin. That could become iconic. not threatening like a humanoid. 
+ Large footprint (879mm depth). This takes up significant space. Where does it live when not in use?
+ The current design would fit well in a hospital, hotel, or commercial setting. It looks professional. But the Beetle wasn't professional—it was charming, humble, a little goofy.
+ Consider: why are pets appealing? They're smaller than us. We look down at them. They fit in our space without dominating it. A 1.46m robot with two industrial arms is not something you look down at—it's something that shares your height, your eye level, your space as an equal. That's psychologically very different.

+ so because i benchmark it with beetle, so I was thinking about price, it should be affordable, but given the sensors and actuators, it's not a mass-market consumer device—it's a prosumer or light-commercial device. it all comes down to who is customer, But then the design language should match. "cute" is not so important if the price point means your first customers are tech enthusiasts, small businesses, or researchers. Let those customers prove out the platform, then design the mass-market version.






~~The first proposal is to make minor modifications to the existing robot prototype. The hardware would remain almost unchanged; the focus would be on software adjustments—maximizing the use of sensors, reducing costs, and trimming some redundant functions where appropriate, while largely preserving the original system framework.~~

~~The second idea is bit radical, next level thing, maybe we should redesign it anyhow, we might as well take a big step and fundamentally redesign the entire system architecture. I have defined a completely new direction.~~
















## reach is more efficient than locomotion

that a stationary base with long-reach arms beats a walking robot that must reposition—is physically correct and economically sound. Moving mass is expensive. Extending reach is cheap.


I'll create some conceptual design visualizations for the modular platform architecture.Here's an interactive visualization of the conceptual designs. You can explore each section:

**Vertebrae Interface** — The symmetric connector design showing power contacts, data bus routing, E-stop signals, and mounting geometry. The 8-position radial symmetry enables 45° rotation options.

**Mobility Modules** — Four variants (wheeled, tracked, legged, snake-like) all connecting through the same vertebrae interface. Each shows key specs and development priority.

**Reach System** — The telescopic arm architecture demonstrating the core principle: base stays still, arm does the reaching. Toggle between retracted/mid/extended states to see the 3:1 reach-to-body ratio.

**System Configurations** — Four example configurations built from the same modular components: Home Assistant, Industrial Inspector, Child Safety Monitor, and Research Platform.

**Consumer vs Developer** — Side-by-side comparison showing how the same core architecture can be presented as a friendly consumer shell or an exposed developer platform.

These are conceptual sketches to guide discussion and engineering direction—not final designs. The key insight they illustrate: modularity enables many products from one platform, but the *presentation* of that modularity differs by market.




---
---










 
 
 





---
## the specification framework

The vertebrae interface specification is important. Everything else—the mobility modules, the reach modules, the consumer shell—depends on getting this right. And once you publish it (even internally), changing it becomes expensive because every module designed against it must be revised. so this is the initial specification structure. this is a working document that outlines: Mechanical interface , Electrical interface , Data interface , Safety interface , Thermal and environmental , Module classification (defining what makes something a "mobility module" vs "reach module" vs "intervention module" and the minimum requirements for each class). this is a skeleton to fill in with actual engineering decisions, and a checklist of decisions you must make before detailed design begins. 

**The vertebrae interface specification is your most important artifact.** Everything else—the mobility modules, the reach modules, the consumer shell—depends on getting this right. And once you publish it (even internally), changing it becomes expensive because every module designed against it must be revised.

I can help you draft the initial specification structure. Would you like me to create a working document that outlines:

1. **Mechanical interface** — mounting geometry, load ratings, quick-connect mechanism options, orientation constraints

2. **Electrical interface** — power rails (48V/24V as you noted), connector standards, current limits per module class

3. **Data interface** — protocol options (CAN-FD vs EtherCAT vs hybrid), message formats, capability declaration schema

4. **Safety interface** — E-stop signal propagation, watchdog requirements, fault isolation rules

5. **Thermal and environmental** — heat dissipation paths, IP rating targets, operating temperature range

6. **Module classification** — defining what makes something a "mobility module" vs "reach module" vs "intervention module" and the minimum requirements for each class

This would give you a skeleton to fill in with actual engineering decisions, and a checklist of decisions you *must* make before detailed design begins.



---

Here's your Vertebrae Interface Specification Framework.

**What's in the document:**

The specification is structured as a working document with six main sections, each containing both firm requirements and open decisions marked for engineering input:

**Section 1: Mechanical Interface** — Mounting geometry (symmetric vertebra design), load ratings with required analyses, and three quick-connect mechanism options (bayonet, threaded ring, lever-cam) for your team to prototype and test.

**Section 2: Electrical Interface** — Dual power rails (48V/24V), connector pinout allocation, and open decisions around hot-swap capability and power budgeting.

**Section 3: Data Interface** — CAN-FD for real-time control (L0/L1) with message ID allocation by priority class, Ethernet for high-bandwidth data (L2+), and the capability declaration schema in JSON format.

**Section 4: Safety Interface** — E-Stop specification (hardwired, normally closed, independent of software), hardware-enforced limits that cannot be overridden, fault isolation requirements, and relevant certification standards (ISO 10218, ISO 13482, IEC 61508).

**Section 5: Thermal and Environmental** — Operating conditions for both consumer and developer variants, heat dissipation budgets per module class, and EMC requirements.

**Section 6: Module Classification** — Detailed requirements for each module class (Mobility, Reach, Sensor, Intervention, Compute, Storage) with development priorities noted.

**Appendices** include a decision log template, open questions list, reference documents, and revision history.

Every [TBD] and "Open Decision" marker represents a choice your engineering team must make. The document is designed to be iterated on—fill in the blanks as you complete analyses and make decisions.



---
