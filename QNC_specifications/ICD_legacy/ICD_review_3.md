# QNC Interface Control Document (ICD) Review Report

## 1. Factual Accuracy

| Issue | Reference | Explanation | Suggestion |
|-------|-----------|-------------|------------|
| **Modbus Slave ID Range** | Section 7.1.1 (Line 524) | States slave ID range is 1-247. While the Modbus spec allows 1-247, many industrial devices use 1-254 or specific reserved ranges. | Clarify if QNC strictly enforces 1-247 or supports the full 8-bit range (1-255). |
| **RS485 Termination** | Section 8.1 (Line 609) | States "120Ω at both ends". In a multi-drop bus, termination is only at the two *physical* ends. If QNC is in the middle, it shouldn't be terminated. | Specify if the 120Ω termination is switchable or fixed on the QNC hardware. |
| **IO-Link Power Supply** | Section 8.2 (Line 663) | Pin 1 (L+) is marked as "I" (Input). In an IO-Link Master (which QNC acts as), L+ is typically an *Output* to power the device. | Correct direction to "O" (Output) if QNC is providing power. |
| **ADC Resolution vs. Value** | Section 8.3 (Line 675-686) | 12-bit ADC (0-4095) with 3.3V ref and 10k pull-up. For a 2.2k resistor (AG95), the theoretical ADC value is $(2.2 / (10 + 2.2)) * 4095 \approx 738$. The document lists **1862**. | Recalculate and verify ADC mapping table values. |

## 2. Common Sense & Logical Consistency

| Issue | Reference | Explanation | Suggestion |
|-------|-----------|-------------|------------|
| **Latency Paradox** | Section 10.1 (Line 772) | Claims 8ms typical latency. However, Modbus RTU at 9600 baud takes ~1ms per character. A 10-byte request + 10-byte response + processing time will exceed 20ms. | Qualify the 8ms claim (e.g., "at 115200 baud") or update to more realistic industrial values. |
| **Timeout Contradiction** | Section 12 (Line 1016-1017) | Health check is 1000ms, but command timeout is 500ms. If a command times out, the health check might still pass, leading to inconsistent state. | Align timeouts or explain the hierarchy of "liveness" vs "responsiveness". |
| **Version Numbering** | Appendix E (Line 2691-2692) | v1.0 released Feb 18. v1.1 released Feb 19. A major functional jump (Handshaking, OTA) in 24 hours suggests versioning or release process issues. | Review release cycle or versioning semantics (Minor vs. Patch). |
| **Hot-Swap Sequence** | Section 10.3 (Line 812) | Sequence shows "Disconnect" then "Health Check Timeout". In high-speed robotics, waiting for 3 timeouts (3s) before notifying the robot is too slow. | Recommend a "Link Down" hardware interrupt or faster heartbeat for safety-critical apps. |

## 3. Definitions & Conceptual Clarity

| Issue | Reference | Explanation | Suggestion |
|-------|-----------|-------------|------------|
| **"Protocol" vs "Device"** | Section 1.4 (Line 83) | "QNC handles protocols, NOT devices." Yet Section 8.3 lists specific models (AG95, 2F-85) for Hardware ID mapping. | This is a "leaky abstraction". Define how Hardware ID mapping coexists with the "no device semantics" rule. |
| **NeuraSync Definition** | Appendix D (Line 2676) | Defined as "FastDDS-based". However, Section 4.1 shows it running over "Ethernet/WiFi". DDS behavior over WiFi is notoriously unstable without specific tuning. | Add a note on WiFi-specific QoS or recommended network topology. |
| **"Generic" Command Ambiguity** | Section 6 | The document pushes for the "Generic" interface but doesn't clearly define how `GenericCommand` maps to complex IO-Link ISDU or Modbus Function Codes beyond simple Read/Write. | Provide a mapping table for Generic -> Protocol-specific operations. |

## 4. Product Development Perspective

| Issue | Reference | Explanation | Suggestion |
|-------|-----------|-------------|------------|
| **Redundant Discovery** | Section 14 | QNC performs "Protocol Discovery" then asks the Robot to perform "Device Identification". This two-step process adds latency and complexity. | Consider allowing QNC to store a "Discovery Script" that performs both and returns a single "Device Ready" event. |
| **Configuration Overhead** | Section 16.1 | JSON descriptors are stored on the QNC (`/etc/qnc/devices/`). This contradicts the "Robot handles semantics" rule if the QNC needs to parse these to function. | Clarify if QNC parses these JSONs or if they are just "hosted" there for the robot to read. |
| **Safety Gap** | Section 2.2 (Line 135) | E-Stop protocols are "Out of Scope". For a bridge between a robot and a gripper, the lack of a hardware E-stop pass-through is a major blocker for industrial certs. | Consider adding a dedicated "Safe Torque Off" (STO) or hardware E-stop bypass interface. |

## 5. General Risks & Concerns

| Issue | Reference | Explanation | Suggestion |
|-------|-----------|-------------|------------|
| **Flash Wear** | Section 15 (Line 2141) | Frequent OTA updates and logging to `/var/log/qnc/` on an embedded system (likely SD card or eMMC) will cause premature flash failure. | Recommend using a RAM-disk for logs or limiting update frequency. |
| **Security of USB Update** | Section 15.6 (Line 2102) | "QNC auto-detects update file" from USB. This is a massive security hole (BadUSB) if not gated by a physical button or authenticated command. | Require a physical "Action" (e.g., hold button for 5s) to trigger USB firmware ingestion. |
| **Power Rail Monitoring** | Section 13.4 (Line 1105) | Fails handshake if 24V is 21.9V. Industrial 24V rails often dip during motor startup. This might cause "nuisance" failures. | Add a "Transient Dip" allowance or a time-averaged voltage check. |

