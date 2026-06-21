
## 🔧 1. Hardware & Networking Challenges

* Need for a **second Ethernet interface** (preferably not via USB).
* Explored options:

  * PCIe via **M.2 slot** (Intel i210 / i225v suggested).
  * USB 3.0 Ethernet as fallback (not preferred).
* Issues:

  * Lack of clear M.2 access on RB3 board.
  * PHY bring-up problems (eventually resolved).
  * GPIO, xHCI, and adapter driver modifications required.

---

## 🧠 2. Real-Time System Strategy (Xenomai vs PREEMPT_RT)

* Goal: **Hard real-time performance** on Qualcomm platform.
* Approach:

  * Port **Xenomai (Dovetail kernel)** into Qualcomm kernel.
  * Perform kernel merge + gap analysis.
* Consideration:

  * PREEMPT_RT as alternative (expected 5–10× higher latency vs Xenomai).
* Outcome:

  * Xenomai successfully booted on QCS6490.
  * Kernel tuning and validation ongoing.

---

## 🧪 3. Testing & Performance Results

* Extensive real-time validation completed:

  * All latency benchmarks passed (IRQ, kernel, user).
  * 1-hour stress test passed under heavy load.
* Observations:

  * System stable under extreme conditions.
  * Some **latency spikes remain** → suspected power management issues.

---

## 🧱 4. Software Stack & Build System

* Plan to maintain a **dedicated Yocto-based repo** for Qualcomm image.
* Integration tasks:

  * Merge Xenomai into Yocto build.
  * Align multiple Linux variants:

    * Qualcomm Linux
    * Yocto
    * Ubuntu
* OSTree compiled successfully.
* Software stack is functional and undergoing validation.

---

## ⚙️ 5. Board Bring-Up & Driver Work

* Progress:

  * Kernel boots with Xenomai.
  * GPIO and PHY issues eventually fixed.
  * Ethernet adapters (ix) working.
* Remaining:

  * Stabilization and bug fixing.
  * Migration to **Seco SOM board**.

---

## 🧩 6. Gunyah Hypervisor Issues

* Major blocker:

  * **Gunyah not appearing in device tree**.
  * UEFI error: `HypDtFixup... overlay skipped`.
* Impact:

  * Hypervisor not properly initialized.
* Needs:

  * Qualcomm documentation or UEFI source access.
  * Understanding DT overlay requirements.

---

## 🤝 7. Collaboration & Coordination

* Active collaboration between multiple engineers (Philippe, Jonathan, etc.).
* Plan:

  * Align work into **shared repositories**.
  * Sync efforts across teams (kernel, audio, networking).
* External dependency:

  * **Qualcomm support is critical**, especially for:

    * Gunyah hypervisor
    * Power management
    * NPU API

---

## 🚀 8. Long-Term Architecture Vision

* Target system:

  * **Dual Linux setup using Gunyah hypervisor**:

    * One **real-time OS (RT Linux)**.
    * One **general-purpose OS** (AI, applications).
* Goal:

  * Unified platform (“NeuraLinux”) across multiple Qualcomm chips.

---

## ⚠️ 9. Key Risks / Open Issues

* Gunyah hypervisor integration unclear.
* Power management causing latency spikes.
* Hardware limitations (M.2 access, PHY issues).
* Risk of **fragmented Linux stacks** if not aligned early.

---

## ✅ 10. Current Status (Bottom Line)

* Xenomai kernel: **working and validated**.
* Hardware bring-up: **mostly functional**.
* Software stack: **running, under testing**.
* Biggest blockers:

  * Gunyah integration
  * Qualcomm documentation/support
  * Final platform unification (Seco board)

---

