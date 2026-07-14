# Invention Disclosure Sketch — Safety-Rated High-Load Autonomous Hot-Swap Coupling

**Purpose:** a concrete starting point to hand to a mechanical/safety engineer (feasibility) and a *Patentanwalt* (novelty/FTO search and drafting). This is a working sketch from a preliminary knock-out scan, **not** a clearance search or legal advice. Individual features below are known in isolation; the argued inventive step is the **specific combination applied to autonomous swapping of a load-bearing, human-adjacent locomotion/limb module**.

## 1. The gap this targets

Two adjacent bodies of art are mature and densely patented, but neither covers the intersection:

- **Self-reconfigurable modular-robot (SRMR) connectors** — e.g. US 7,850,388 (autonomous, genderless docking supporting "structural load bearing, communications and power sharing"); US 8,234,950 (USC SuperBot genderless connector with mechanical linkage, power sharing, communication, docking guidance). These are **autonomous and self-reconfiguring but small-load** (lab modules; representative connectors carry only tens–a few hundred N / tens of Nm) and are **not built around a functional-safety-rated interlock**.
- **Industrial tool changers** — e.g. US 4,551,903 (simultaneous mechanical + electrical + fluid connection, energized on coupling / interrupted on separation); ATI-style fail-safe cam+ball locks with air-pressure fail-safe and lock/unlock proximity sensors; DE 3,705,123; US 8,500,132; US 10,047,908. These are **high-load but for end-of-arm tools on a fixed arm**, are typically pneumatic/human-actuated, energize all lines **simultaneously**, and are **not** designed for autonomous self-reconfiguration of *locomotion* nor around safety-rated E-stop/STO continuity for a limb that bears body weight over a human.

**Target whitespace:** high structural/locomotion load **+** autonomous hot-swap **+** real-time data **+** a *certifiable* safety interlock, for a module a human-adjacent robot stands, walks, or rides on.

## 2. Candidate mechanism — distinguishing features

Seven features; the inventive core is the **combination** of A/B (load-safe locking) + C (safety-sequenced engagement) + D/E (safety-gated load enable).

**A. Load-energized, passive-lock / active-unlock structural lock.** The support-load path passes through a wedge/cam interface arranged so that **increasing external support load increases the locking normal force** (self-energizing). Default state is mechanically locked; power is required only to *unlock*. On power/pressure loss under load, the joint does not release — it locks harder.
 *Differs from:* ATI-type locks that are **held by air pressure** and can release below a pressure threshold; SRMR SMA/hook latches not designed for load-proportional locking.

**B. Redundant, independently-sensed dual structural latch.** Two independent latches share the load path such that **single-latch failure does not drop the module** (single-fault tolerance for a load-bearing limb).
 *Differs from:* single-locking-mechanism tool changers and SRMR connectors.

**C. Physically staged make/break sequence: structural → safety → power → data.** Contact/latch geometry (graded pin lengths, latch-travel gating) enforces, on docking: (1) structural lock engages **and is proven**, then (2) a hardwired safety loop (dual-channel E-stop/STO) closes and is proven, then (3) power mates, then (4) data enumerates. On undock, strict reverse order, with **actuation disabled before load release**.
 *Differs from:* US 4,551,903 / DE 3,705,123, which energize mechanical + electrical + fluid **simultaneously**; SRMR connectors (no safety sequencing).

**D. Dual-channel, diverse, cross-monitored lock proof.** Two independent sensors of **different physical principle** report lock state to the robot's safety controller, enabling a Category-3/PL d (or SIL-2-style) **proof of mechanical capture** as a precondition for actuation, with hardwired redundant safety continuity routed through the coupling.
 *Differs from:* ATI's single-type proximity lock/unlock sensors, which are not integrated as a safety-rated dual-channel enable gate.

**E. Safety-relevant admission control tied to module ID / manifest.** Before enabling load-bearing actuation, the safety controller reads the module's electronic ID / capability manifest (rated load, safety rating, required safety-loop topology) and **refuses to permit load or actuation** if the declared rating is insufficient or the lock/safety proofs fail. (Ties directly to your VMM concept, but as a *safety-gated* enable/refuse, not just descriptive metadata.)
 *Differs from:* AAS/tool-changer electronic IDs, which are descriptive and not tied to a safety-rated load-bearing enable gate.

**F. Two-stage interface: large-AoA compliant capture, then rigidize to high stiffness.** A large area-of-acceptance compliant capture stage tolerates autonomous-docking misalignment (SRMR strength), followed by a **preload/rigidize stage** that converts to a backlash-free, high-stiffness load path rated for locomotion moments (tool-changer strength).
 *Differs from:* SRMR connectors (compliant but low-stiffness/low-load) and tool changers (stiff but small area-of-acceptance / precise approach).

**G. Anti-backdrive lock for dynamic locomotion.** Self-locking/one-way geometry so cyclic locomotion shock and vibration cannot progressively unlock the joint; active power needed only to unlock.
 *Differs from:* quasi-static tool-holding assumptions.

## 3. How each feature maps to the closest prior art

| Feature | US 7,850,388 (SRMR autonomous dock) | US 8,234,950 (SuperBot) | ATI-type tool changer | US 4,551,903 |
|---|---|---|---|---|
| A. Load-energized passive-lock | not addressed | not addressed | air-pressure held (opposite) | not addressed |
| B. Redundant dual latch | single mechanism | single mechanism | single mechanism | single mechanism |
| C. Safety-sequenced make/break | no | no | simultaneous | **simultaneous (explicit)** |
| D. Dual-channel safety-rated lock proof | no | no | single-type sensors | no |
| E. Safety-gated manifest admission | no | no | no | no |
| F. Compliant-capture→rigidize (high load) | compliant/low load | compliant/low load | stiff/small AoA | stiff |
| G. Anti-backdrive for locomotion | not addressed | not addressed | one-way clutch (manual variant) | no |

## 4. Candidate claim seeds (for the attorney to shape)

**Independent — apparatus.** A coupling for releasably joining a load-bearing module to a robot, comprising: complementary first and second coupling units; a structural locking mechanism defining a support-load path and arranged such that an increasing external support load increases a locking force of the mechanism, the mechanism being biased to a locked state and requiring actuation to unlock; a hardwired safety-signal path, a power path, and a data path routed through the coupling; wherein the coupling units are configured to engage the paths in a predetermined sequence in which the structural locking mechanism engages before the safety-signal path, the safety-signal path before the power path, and the power path before the data path.

**Independent — system.** A robot comprising the coupling and a safety controller configured to permit load-bearing actuation across the coupling only after (i) dual-channel confirmation of the locked state, (ii) confirmation of integrity of the safety-signal path, and (iii) validation that a rating declared by the module satisfies a load and safety requirement.

**Independent — method.** A method of autonomously exchanging a load-bearing module of a robot, comprising: compliant capture of the module within an area of acceptance; rigidizing the interface to a high-stiffness load path; confirming a locked state via two independent sensing channels; closing and confirming a hardwired safety loop through the coupling; mating power; enumerating a data link and validating a module rating; and only then enabling load-bearing actuation; wherein disconnection reverses the sequence and disables actuation before releasing the locked state.

**Dependent seeds.** Load-energized wedge/cam geometry (A); redundant dual latch with independent sensing (B, D); diverse-principle lock sensors (D); large-AoA compliant capture followed by preload (F); anti-backdrive self-locking geometry (G); fault-under-load behavior that maintains the locked state and signals safe-torque-off rather than releasing; refusal to enable actuation on insufficient declared rating (E); genderless coupling units; the same coupling accepting non-locomotion modules (platform tie-in).

## 5. Claim strategy

- **Do not** claim "a genderless load-bearing connector transferring power and data for a self-reconfigurable robot" alone — that is squarely owned (US 7,850,388 / US 8,234,950).
- Put the **novel combination in the independent claim**: (load-energized *or* redundant-latch locking) **+** (safety-sequenced make/break) **+** (safety-rated proof gating load actuation). This trio is where the whitespace is.
- **EPO:** frame the technical effect as reliable single-fault-tolerant load retention and a provable safe state during autonomous load-bearing exchange — concrete, technical, COMVIK-friendly.
- **CNIPA:** file the four-part suite (method / apparatus / robot / storage-medium); present as a technical solution to the technical problem of safely exchanging a load-bearing module on a human-adjacent robot.
- Keep the safety-sequencing method claim independent of the specific lock geometry, so it survives if a competitor uses a different lock.

## 6. Next steps / caveats

1. Have a mechanical + functional-safety engineer pressure-test feasibility (esp. A, D, F) and put real numbers on load, stiffness, and cycle life — the claims need concrete values.
2. Commission a **professional novelty/FTO search** (Espacenet / DEPATISnet / CNIPA / commercial DB) focused on features A + C + D combined; check for prosthetics/exoskeleton and battery-swap docking art not covered here.
3. **Report as a service invention to NEURA (ArbnErfG)** before disclosing; **file before any public disclosure** (no EPO grace period).
4. Individual features are known in isolation; commercial and patent value depends on a genuinely novel *specific* mechanism and on execution — treat this as a hypothesis to disprove quickly, not a finished invention.
