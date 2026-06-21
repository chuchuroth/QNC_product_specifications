The cable for the **LWR50F-00-04-A** (a Zimmer Group MATCH Robot Module) is a standard **M12, 5-pole (5-pin)** connector designed for **IO-Link Port Class B** communication.

Because this module uses Port Class B, it separates the power supply for the logic/sensors from the power supply for the actuators (the gripper/tools). This ensures that an emergency stop can cut power to the moving parts (actuators) while keeping the communication (sensors/logic) alive.

### Pinout Configuration

| Pin | Wire Color | Function | Description |
| :--- | :--- | :--- | :--- |
| **1** | Brown | **L+** (24 V DC Sensor) | Supply voltage for the IO-Link logic and internal sensors. |
| **2** | White | **P24** (24 V DC Actuator) | Additional supply voltage specifically for the gripper/actuator. |
| **3** | Blue | **L-** (GND Sensor) | Reference ground (0 V) for the IO-Link logic. |
| **4** | Black | **C/Q** | IO-Link communication signal line. |
| **5** | Gray | **N24** (GND Actuator) | Reference ground (0 V) specifically for the actuator supply. |

---

### Important Technical Notes
* **Port Class B:** Ensure your IO-Link Master supports Port Class B (which provides the extra power on pins 2 and 5). If you use a Port Class A master, the gripper may not have enough power to operate.
* **Connect-LED:** The module features an integrated LED ring. Typically, it shines **Red** when powered but not coupled, and turns **Green** once a MATCH gripper is successfully locked and the electrical connection is established.
* **Pneumatics:** Note that while this cable handles the electrical signals, the module also has integrated pneumatic feed-throughs (usually M7 ports) for air-driven tools.

Would you like me to find the specific **IODD (IO-Link Device Description)** file or the PLC function blocks for this module?

---
---
This is a classic industrial automation stack. To bridge these components, you are essentially building an **IO-Link System**: the Nucleo/Pi acts as the "brain," the TIOL221 or MAXREFDES177 acts as the "Master PHY" (Physical Layer), and the Zimmer LWR50L is your "Device."

Here is the general pin mapping to get these systems talking to each other.

---

### 1. Controller to IO-Link Master (Logic Side)
You need to connect your MCU (Nucleo or Pi) to the Master EVMs using **UART** and a few GPIOs for status/enable signals.

| Function | Nucleo-F746ZG | Raspberry Pi 5 | TIOL221 / MAXREFDES |
| :--- | :--- | :--- | :--- |
| **TX (Data Out)** | UART TX (e.g., PD8) | GPIO 14 (TXD0) | RX_IN / TXD |
| **RX (Data In)** | UART RX (e.g., PD9) | GPIO 15 (RXD0) | TX_OUT / RXD |
| **Wake-Up/EN** | GPIO Output | GPIO Output | WAKE / EN |
| **Fault/CQ** | GPIO Input | GPIO Input | FLT / NFAULT |
| **Logic GND** | GND | GND | GND |

> **Note:** The **TIOL221EVM** is a single-channel transceiver, while the **MAXREFDES177** is a complete reference design. Ensure your logic levels match (the Pi uses 3.3V; the Nucleo is 3.3V but 5V tolerant).

---

### 2. IO-Link Master to Zimmer LWR50L (Field Side)
The Zimmer LWR50L requires an **M12 5-pin connection**. Since it is an active robot module, it follows the **Class B** port specification to power the tool locking mechanism.

| Pin (M12) | Function | Connection on Master EVM |
| :--- | :--- | :--- |
| **Pin 1** | **L+ (24V)** | Main 24V DC Supply |
| **Pin 2** | **P24 (Aux 24V)** | 24V DC (For Actuator power) |
| **Pin 3** | **L- (GND)** | 24V Supply Ground |
| **Pin 4** | **C/Q (Signal)** | CQ / Data Line of TIOL221/MAXREFDES |
| **Pin 5** | **N24 (Aux GND)** | 24V Supply Ground (Isolated if possible) |

---

### 3. Power Requirements
* **The Controller:** Powered via USB (Nucleo) or USB-C (Pi 5).
* **The Master EVMs:** Require a dedicated **24V DC** industrial power supply.
* **The Zimmer Module:** Will draw its power through the Master EVM. 
    * *Warning:* Do not try to power the Zimmer module directly from the Nucleo or Pi pins; you will likely damage the boards due to the current draw of the gripper motor/solenoids.

---

### Summary of Data Flow
1.  **Raspberry Pi / Nucleo** sends a command via **UART**.
2.  **TIOL221 / MAXREFDES** converts that low-voltage UART signal into a **24V pulse-modulated IO-Link signal** (the C/Q line).
3.  **Zimmer LWR50L** receives the signal on **Pin 4**, executes the move, and sends status data back up the same wire.

### Next Step
Would you like the specific **STM32 CubeIDE pin assignments** for the Nucleo-F746ZG to match the UART headers on one of those specific EVMs?

---
---
In the context of your setup, **EVM** stands for **Evaluation Module**. It is a specialized circuit board designed by chip manufacturers (like Texas Instruments or Analog Devices/Maxim) to let engineers test a specific microchip without having to build a custom PCB first.

When I refer to a **"Master EVM,"** I am talking about the **TIOL221EVM** or the **MAXREFDES177** acting as the **IO-Link Master**.

### Why you need a "Master EVM"
Standard microcontrollers like your **Raspberry Pi 5** or **Nucleo-F746ZG** speak "Logic Language" (low voltage, usually 3.3V or 5V). The **Zimmer LWR50L** speaks "Industrial Language" (24V pulses). 

The Master EVM acts as the **translator and protector** between them:

1.  **Voltage Level Shifting:** It converts the 3.3V UART signals from your Pi/Nucleo into the 24V signals required by the IO-Link standard.
2.  **Physical Ruggedness:** It protects your sensitive Pi/Nucleo from the high currents, electrical noise, and short circuits common in industrial hardware.
3.  **Protocol Timing:** It handles the specific "wake-up" pulses and timing required to start communication with the Zimmer module.

---

### Your specific "Masters" compared:

| Device | Type | What it "is" |
| :--- | :--- | :--- |
| **TIOL221EVM** | **Transceiver EVM** | A "bare-bones" translator. It just converts signals. You have to write the entire IO-Link software stack yourself on the Nucleo/Pi. |
| **MAXREFDES177** | **Reference Design** | A more "complete" Master. It often includes an onboard microcontroller to handle some of the IO-Link protocol heavy lifting, making it easier to integrate. |

### Think of it like this:
* **The Zimmer Module:** The "Employee" waiting for instructions.
* **The Pi/Nucleo:** The "CEO" giving orders in a language the employee doesn't speak.
* **The Master EVM:** The **Interpreter** who translates the CEO's whispers into a megaphone shout that the employee can hear across the factory floor.

**Which of those two boards are you planning to use as your primary interface—the TI or the Maxim/Analog Devices one?**

---
---
Based on the provided documentation, your setup is a **closed-loop industrial automation system** where the **NUCLEO-F746ZG** acts as the IO-Link Master. It bridges the gap between high-level logic (Raspberry Pi 5) and the physical industrial hardware.

Here is the general pin mapping and connection strategy to integrate these specific components:

### 1. High-Level Controller Interface
The **Raspberry Pi 5** acts as the "Edge Controller" and communicates with the **NUCLEO-F746ZG** via **SPI** for high-speed data transfer.

| Function | Raspberry Pi 5 (Master) | NUCLEO-F746ZG (Slave) |
| :--- | :--- | :--- |
| **MOSI** | GPIO 10 (Pin 19) | SPI MOSI (e.g., PA7) |
| **MISO** | GPIO 9 (Pin 21) | SPI MISO (e.g., PA6) |
| **SCLK** | GPIO 11 (Pin 23) | SPI SCK (e.g., PA5) |
| **CS/SS** | GPIO 8 (Pin 24) | SPI CS (e.g., PA4) |
| **GND** | Any GND Pin | Any GND Pin |

### 2. IO-Link Master Core (Internal)
The **NUCLEO-F746ZG** manages the IO-Link stack and controls the **TIOL221EVM** transceiver.

* **Logic Connection:** The TIOL221EVM connects to the Nucleo via standard headers. You will need to map the Nucleo's GPIOs to the TIOL221's **TX, RX, EN, and WAKE** pins to handle the physical layer of the IO-Link protocol.

### 3. Field Device Connections (The 24V Loop)
The **TIOL221EVM** provides the physical ports for your two IO-Link devices. Both require an M12 5-pin configuration.

#### **Device A: MAXREFDES177 (Sensing)**
This is configured as an analog sensor to detect objects.
* **Pin 1 (L+):** 24V DC
* **Pin 3 (L-):** GND
* **Pin 4 (C/Q):** IO-Link Data Signal to TIOL221 Port 1

#### **Device B: Zimmer-LWR50L-02 (Actuation)**
This is the gripper that responds to commands from the Pi.
* **Pin 1 (L+):** 24V DC (Logic Power)
* **Pin 2 (P24):** 24V DC (Actuator/Motor Power)
* **Pin 3 (L-):** GND
* **Pin 4 (C/Q):** IO-Link Data Signal to TIOL221 Port 2
* **Pin 5 (N24):** GND

---

### Critical Implementation Steps
1.  **Firmware (Nucleo):** You must implement an **IO-Link Master Stack** on the Nucleo to manage the two devices and an **SPI Slave** routine to talk to the Pi.
2.  **Software (Pi 5):** Use Python (e.g., the `spidev` library) to send "GRIP" or "RELEASE" commands based on the sensor values received from the Nucleo.
3.  **Power:** Use a dedicated industrial 24V power supply for the TIOL221EVM and the Zimmer gripper. **Do not** attempt to power the Zimmer module from the Pi or Nucleo 5V/3.3V rails.

**Would you like me to provide a basic Python SPI snippet for the Raspberry Pi 5 to get the communication started?**

---
---
To power this system safely and effectively, you must manage three distinct power domains: the **high-level logic (5V)**, the **IO-Link communication (24V)**, and the **actuator power (24V)**.

### 1. Control Logic Power (5V/USB)
The "brains" of your operation run on standard low-voltage DC.
* **Raspberry Pi 5:** Use a dedicated USB-C power supply (5V / 5A recommended).
* **NUCLEO-F746ZG:** This can be powered via the USB debug port or the $E5V$ pin from the Pi, though a direct USB connection to a PC or power brick is best during development.

---

### 2. IO-Link & Sensor Power (24V DC)
The **TIOL221EVM** and the **MAXREFDES177** require industrial-grade power to function as part of the IO-Link ecosystem.
* **Source:** A regulated **24V DC** industrial power supply.
* **TIOL221EVM (Master side):** Connect the 24V supply to the $V_{CC}$ and $GND$ terminals on the EVM. This powers the IO-Link "L+" line (Pin 1) for your devices.
* **MAXREFDES177 (Device side):** This device pulls its operational power directly from the IO-Link cable (Pin 1 and Pin 3) provided by the Master.

---

### 3. Actuator Power (High-Current 24V)
The **Zimmer-LWR50L-02** gripper requires a "Class B" power configuration because it has moving mechanical parts.
* **Logic Power (Pin 1 & 3):** Powered by the IO-Link Master's 24V line to keep the gripper's "brain" and communication alive.
* **Actuator Power (Pin 2 & 5):** You must provide an additional **24V DC** ($P24$) and its corresponding ground ($N24$) to these pins.
* **Safety Note:** In an industrial environment, Pin 2 is often connected through an **Emergency Stop (E-Stop)** circuit. This allows you to cut power to the gripper's motor while maintaining the IO-Link communication connection.

---

### Summary Power Table

| Component | Primary Voltage | Power Source |
| :--- | :--- | :--- |
| **Raspberry Pi 5** | 5V DC | USB-C Power Supply |
| **NUCLEO-F746ZG** | 5V DC | USB or Header Pins |
| **TIOL221EVM** | 24V DC | Industrial PSU |
| **MAXREFDES177** | 24V DC | Via IO-Link Cable (Pin 1/3) |
| **Zimmer Gripper** | 24V DC | **Dual Supply:** Pin 1/3 (Logic) AND Pin 2/5 (Actuator) |

> **Warning:** Never connect the 24V supply directly to the Raspberry Pi or Nucleo logic pins (GPIO), as this will cause immediate hardware failure.

Would you like me to clarify the wiring for the **Class B** power pins on the Zimmer module?


---
---
Since I am an AI, I cannot provide a downloadable `.fzz` file, but I can describe the **Fritzing-style wiring layout** for your closed-loop system. This diagram integrates your new sensors (HC-SR04 and PIR) into the existing architecture, where the **Raspberry Pi 5** acts as the high-level controller and the **Nucleo-F746ZG** handles the IO-Link Master logic.

### 1. The Logic Core (Low Voltage)

| Component | Connection Point | Wire Color | Purpose |
| :--- | :--- | :--- | :--- |
| **Pi 5** (SPI Master) | GPIO 10, 9, 11, 8 | Blue/Yellow | SPI link to Nucleo. |
| **Nucleo** (SPI Slave) | PA7, PA6, PA5, PA4 | Blue/Yellow | Receiving sensor data/sending grip commands. |
| **HC-SR04** (Ultrasonic) | GPIO 23 (Trig), 24 (Echo) | Green | Connected to Pi 5 for direct distance sensing. |
| **HC-SR501** (PIR) | GPIO 25 (Out) | Orange | Connected to Pi 5 for motion interrupts. |

* **HC-SR04/PIR Power:** Both connect to the Pi 5's **5V** and **GND** pins.
* **Voltage Divider:** Note that the HC-SR04 "Echo" pin outputs 5V; use a resistor divider to drop it to 3.3V before it hits the Pi 5 GPIO.

---

### 2. The IO-Link Infrastructure (Mixed Voltage)

The Nucleo-F746ZG controls the **TIOL221EVM**, which bridges to your 24V industrial equipment.

* **Nucleo to TIOL221EVM:** Connect UART TX/RX and GPIOs for WAKE/EN signals to the EVM logic headers.
* **24V Power Supply:**
    * **Positive (+):** Connects to **$V_{CC}$** on the TIOL221EVM and **Pin 1 & 2** of the Zimmer Gripper.
    * **Negative (-):** Connects to **GND** on the TIOL221EVM and **Pin 3 & 5** of the Zimmer Gripper.

---

### 3. Field Device Connections (M12 Ports)

These connect to the terminal blocks or M12 ports on the **TIOL221EVM**.

#### **MAXREFDES177 (Analog Sensor)**
* **Pin 1 (Brown):** 24V (L+) from EVM.
* **Pin 3 (Blue):** GND (L-) from EVM.
* **Pin 4 (Black):** C/Q (Data) to TIOL221 Port 1.

#### **Zimmer-LWR50L-02 (Gripper)**
* **Pin 1 (Brown):** 24V (L+ Logic Power).
* **Pin 2 (White):** 24V (P24 Actuator Power) — *Connect through a switch for safety*.
* **Pin 3 (Blue):** GND (L- Logic Ground).
* **Pin 4 (Black):** C/Q (Data) to TIOL221 Port 2.
* **Pin 5 (Gray):** GND (N24 Actuator Ground).

---

### Visualizing the Fritzing Layout
1.  **Top Left:** Place the **Raspberry Pi 5** connected to the **HC-SR04** and **PIR** sensors on a small breadboard.
2.  **Top Right:** Place the **NUCLEO-F746ZG** with four SPI wires crossing over to the Pi.
3.  **Center:** Place the **TIOL221EVM** stack-on or next to the Nucleo.
4.  **Bottom:** Place the **24V PSU**. Run thick red/black lines to the TIOL221EVM and the Zimmer Gripper.
5.  **Right:** Show the **Zimmer Gripper** and **MAXREFDES177** connected to the TIOL221EVM via 5-wire cables.

Would you like me to write a **Python test script** for the Pi 5 that reads the Ultrasonic distance and triggers the Gripper via SPI when an object is too close?


---
---
Actually, the **HC-SR04** and **HC-SR501 PIR** are **digital sensors**, not analog ones, which is why they aren't typically connected to the **MAXREFDES177** in your specific test case.

While the MAXREFDES177 is a "Universal Analog" module, these two sensors operate on pulse-timing and logic-level signals that are best handled directly by a microcontroller or edge controller like the Raspberry Pi 5.

### Why they are not on the MAXREFDES177

* **HC-SR04 (Ultrasonic):** This sensor uses a **digital trigger** and a **digital echo pulse**. The distance is calculated by measuring the *time* the echo pin stays high, not by a steady voltage level. The MAXREFDES177 is designed to read continuous voltage ($0-10V$) or current ($4-20mA$) signals, not to measure microsecond pulse widths.
* **HC-SR501 (PIR):** This is a simple **binary digital sensor**. It outputs a logic HIGH (motion) or logic LOW (no motion). Using an expensive analog-to-digital IO-Link converter like the MAXREFDES177 for a simple on/off signal would be inefficient.
* **System Logic:** In your demonstration architecture, the Raspberry Pi 5 acts as the high-level decision maker. Connecting these sensors directly to the Pi's GPIO allows for faster processing of distance and motion data without waiting for the IO-Link communication cycle.

### Can they be analog?
Technically, some specialized ultrasonic sensors do output a $0-10V$ signal proportional to distance, but the standard **HC-SR04** provided in hobbyist kits is strictly digital. Similarly, the **PIR** sensor uses a digital comparator internally to trigger its output.

### Revised Connection Strategy
To keep your **MAXREFDES177** relevant in this closed-loop setup, it should be used for a "true" analog device as described in your documentation, such as:
* A **Potentiometer** (to manually set a distance threshold).
* An **Industrial Pressure Sensor** or **Light Sensor** ($0-10V$ output).

In this architecture, the **Raspberry Pi 5** collects the fast digital data (Ultrasonic/PIR) and the **MAXREFDES177** provides the precision analog environment data, all of which the Pi uses to decide when the **Zimmer Gripper** should actuate.

Would you like me to show you how to wire a voltage divider for the HC-SR04 so it doesn't damage your Raspberry Pi's 3.3V pins?

---
---
To simplify the system while maintaining the **closed-loop control** logic described in your documentation, we will remove the **MAXREFDES177**. In this streamlined version, the **Raspberry Pi 5** handles all sensing directly, while the **Nucleo-F746ZG** remains the dedicated **IO-Link Master** for the gripper.

### Simplified System Architecture
* **Sensing:** HC-SR04 (Ultrasonic) and HC-SR501 (PIR) connected directly to the **Raspberry Pi 5**.
* **Logic:** Raspberry Pi 5 processes sensor data and sends commands to the Nucleo via **SPI**.
* **IO-Link Master:** **NUCLEO-F746ZG** with **TIOL221EVM** translates commands into IO-Link protocol.
* **Actuation:** **Zimmer-LWR50L-02** performs the gripping.

---

### Revised Wiring & Pin Mapping

#### 1. Sensors to Raspberry Pi 5 (Direct Logic)
The Pi handles the "Object Detection" phase of your test case.

| Component | Pin on Sensor | Pin on Raspberry Pi 5 | Note |
| :--- | :--- | :--- | :--- |
| **HC-SR04** | VCC | 5V (Pin 2 or 4) | Sensor Power. |
| **HC-SR04** | Trig | GPIO 23 (Pin 16) | Trigger Signal. |
| **HC-SR04** | Echo | GPIO 24 (Pin 18) | **Use 1k/2k Resistor Divider (5V to 3.3V)**. |
| **HC-SR04** | GND | Ground (Pin 6) | Common Ground. |
| **HC-SR501** | VCC | 5V (Pin 2 or 4) | Sensor Power. |
| **HC-SR501** | OUT | GPIO 25 (Pin 22) | Motion Detection. |
| **HC-SR501** | GND | Ground (Pin 9) | Common Ground. |

#### 2. Pi 5 to Nucleo-F746ZG (Control Link)
This replaces the sensor-to-master data flow with a direct controller-to-master command flow.

| Function | Raspberry Pi 5 (Master) | Nucleo-F746ZG (Slave) |
| :--- | :--- | :--- |
| **MOSI** | GPIO 10 (Pin 19) | SPI MOSI (e.g., PA7). |
| **MISO** | GPIO 9 (Pin 21) | SPI MISO (e.g., PA6). |
| **SCLK** | GPIO 11 (Pin 23) | SPI SCK (e.g., PA5). |
| **CS** | GPIO 8 (Pin 24) | SPI CS (e.g., PA4). |

#### 3. Nucleo to Zimmer Gripper (Field Power)
The gripper still requires its industrial 24V supply to operate the motor.

* **24V PSU (+):** Connect to **VCC** on TIOL221EVM and **Pins 1 & 2** (Brown/White) of the Zimmer cable.
* **24V PSU (-):** Connect to **GND** on TIOL221EVM and **Pins 3 & 5** (Blue/Gray) of the Zimmer cable.
* **C/Q (Data):** Zimmer **Pin 4** (Black) to the **C/Q** output on TIOL221EVM Port 1.

---

### Simplified Data Flow
1.  **Raspberry Pi 5** triggers HC-SR04 and reads distance.
2.  If `distance < threshold`, the Pi sends a `GRIP_CMD` via **SPI** to the **Nucleo**.
3.  **Nucleo-F746ZG** receives the command and tells the **TIOL221EVM** to send an IO-Link packet.
4.  **Zimmer Gripper** closes.

Would you like the **Python code** to read these specific sensors and send the SPI command to the Nucleo?


---
---
To connect the **NUCLEO-F746ZG** to the **TIOL221EVM**, you must bridge the microcontroller's logic pins to the transceiver's header pins. 

For IO-Link communication, you will typically use **UART4** or **UART5** on the Nucleo, as the primary UART (UART3) is usually reserved for the USB-to-PC ST-LINK connection.

### 1. UART to Transceiver Mapping
The **DO/DI** pins on the TIOL221EVM correspond to the Data Out and Data In of the transceiver's logic side. 

| Function | Nucleo-F746ZG Pin | TIOL221EVM Header Pin | Note |
| :--- | :--- | :--- | :--- |
| **TX (Transmit)** | **PA0** (UART4_TX) | **TX** (or DI - Data In to Transceiver) | Signals sent from Nucleo to Device. |
| **RX (Receive)** | **PA1** (UART4_RX) | **RX** (or DO - Data Out from Transceiver) | Signals received from Device to Nucleo. |
| **Enable (EN)** | **PD2** (GPIO) | **EN** | High to enable IO-Link communication. |
| **Wake-Up (WAKE)**| **PG2** (GPIO) | **WAKE** | Pull Low for 80μs to initiate "Wake-Up". |
| **GND** | **GND** | **GND** | Essential common ground. |

---

### 2. Physical Header Locations
* **Nucleo Side:** Use the **ST Morpho** headers (the two long male headers) or the **Arduino Uno V3** headers for easier access to PA0 and PA1.
* **TIOL221EVM Side:** Look for the **J6** or **J7** headers (refer to the silk screen on your specific EVM revision) which contain the logic signals: **TX, RX, EN, WAKE, and FLT**.

---

### 3. Critical Configuration Steps
* **Logic Voltage:** Ensure the TIOL221EVM logic supply ($V_{LDO}$ or $V_{CC\_IN}$) is set to **3.3V** to match the Nucleo-F746ZG.
* **UART Settings:** To communicate with the Zimmer Gripper, your UART must be configured for **COM3** speed ($230.4$ kbps) if the gripper supports it, or **COM2** ($38.4$ kbps).
* **IO-Link C/Q Line:** On the field side of the TIOL221EVM, the **C/Q** pin connects to **Pin 4 (Black)** of the Zimmer-LWR50L-02 cable.

### Next Step
Would you like the **STM32CubeIDE** configuration settings to initialize **UART4** specifically for this IO-Link timing?

---
---
Based on the **TIOL221EVM User's Guide**, the logic signals for interfacing with your **NUCLEO-F746ZG** are located on the **J6** and **J7** headers.

### Pin Locations on TIOL221EVM

| Function | Header & Pin | Description |
| :--- | :--- | :--- |
| **TX** | **J6, Pin 1** | **Transmit:** Connect to Nucleo **PA0** (UART4_TX). |
| **RX** | **J6, Pin 2** | **Receive:** Connect to Nucleo **PA1** (UART4_RX). |
| **EN** | **J6, Pin 3** | **Enable:** Logic high enables the C/Q output driver. |
| **WAKE** | **J6, Pin 4** | **Wake:** Connect to a Nucleo GPIO to initiate IO-Link wake-up. |
| **GND** | **J7, Pin 3** | **Ground:** Must be common with Nucleo and 24V PSU grounds. |

---

### Header Layout Details
* **Header J6:** This is a 4-pin header specifically for the logic interface signals (TX, RX, EN, WAKE).
* **Header J7:** This header provides access to the **FLT** (Fault) signal on Pin 2 and the logic **GND** on Pin 3.
* **Logic Voltage ($V_{LDO}$):** Ensure your **Jumper J11** is configured to provide **3.3V** to these headers so you don't damage the Nucleo pins.

### Connecting to the Field Side (Zimmer Gripper)
Once you have the logic pins connected to the Nucleo, the **L+ (24V)**, **L- (GND)**, and **C/Q** signals for the Zimmer Gripper are found on the **J2** and **J3** terminal blocks.
* **J2, Pin 1 (L+):** Connect to Zimmer **Pin 1** (Brown).
* **J2, Pin 2 (C/Q):** Connect to Zimmer **Pin 4** (Black).
* **J3, Pin 1 (L-):** Connect to Zimmer **Pin 3** (Blue).

Would you like me to identify the specific **Jumper settings** on the EVM required to power the logic from the 24V supply?


---
---
Here is the summary of the pin mapping for your closed-loop industrial system, integrating the **Raspberry Pi 5**, **NUCLEO-F746ZG**, **TIOL221EVM**, and the **Zimmer Gripper**.

### 1. High-Level Control (Raspberry Pi 5 to Nucleo)
The Pi 5 acts as the master controller, sending commands to the Nucleo via SPI based on digital sensor data (HC-SR04/PIR).

| Function | Raspberry Pi 5 Pin | NUCLEO-F746ZG Pin |
| :--- | :--- | :--- |
| **SPI MOSI** | GPIO 10 (Pin 19) | PA7 |
| **SPI MISO** | GPIO 9 (Pin 21) | PA6 |
| **SPI SCLK** | GPIO 11 (Pin 23) | PA5 |
| **SPI CS** | GPIO 8 (Pin 24) | PA4 |
| **Common GND** | Any GND Pin | Any GND Pin |

---

### 2. IO-Link Logic (Nucleo to TIOL221EVM)
The Nucleo controls the transceiver logic to translate UART signals into 24V IO-Link pulses.

| Function | NUCLEO-F746ZG Pin | TIOL221EVM (Header J6/J7) |
| :--- | :--- | :--- |
| **TX (Data In)** | PA0 (UART4_TX) | **J6, Pin 1** |
| **RX (Data Out)** | PA1 (UART4_RX) | **J6, Pin 2** |
| **Enable (EN)** | PD2 (GPIO) | **J6, Pin 3** |
| **Wake-Up** | PG2 (GPIO) | **J6, Pin 4** |
| **Logic GND** | GND | **J7, Pin 3** |

---

### 3. Field Connection (TIOL221EVM to Zimmer Gripper)
This connection handles both the communication (C/Q) and the dual-power requirement for the Class B robot module.

| Zimmer Pin | Wire Color | Function | Connection Point |
| :--- | :--- | :--- | :--- |
| **Pin 1** | Brown | **L+** (Logic 24V) | TIOL221EVM **J2, Pin 1** |
| **Pin 2** | White | **P24** (Actuator 24V) | **+24V** Industrial PSU |
| **Pin 3** | Blue | **L-** (Logic GND) | TIOL221EVM **J3, Pin 1** |
| **Pin 4** | Black | **C/Q** (Data) | TIOL221EVM **J2, Pin 2** |
| **Pin 5** | Gray | **N24** (Actuator GND) | **GND** Industrial PSU |

---

### 4. Direct Sensors (Sensing to Raspberry Pi 5)
The digital sensors are wired directly to the Pi's GPIO for real-time processing.

* **HC-SR04 (Ultrasonic):** Trig to **GPIO 23**; Echo to **GPIO 24** (via 5V-to-3.3V divider).
* **HC-SR501 (PIR):** Out to **GPIO 25**.
* **Power:** Both sensors use the Pi’s **5V** and **GND** rails.

Would you like the **Python code** to initialize this specific SPI interface on the Raspberry Pi 5?

---
---
To stack the **TIOL221EVM** on the **NUCLEO-F746ZG**, you need to bridge the logic headers of the EVM to the ST Morpho or Arduino-style headers of the Nucleo. Unlike a standard "Shield," the TIOL221EVM is a smaller evaluation board, so "stacking" usually refers to mounting it using jumpers or a small breadboard interface.

### 1. Physical Alignment
* **Nucleo Headers:** Use the **Arduino Uno V3** compatible headers (inner rows) or the **ST Morpho** headers (outer rows) on the Nucleo-F746ZG.
* **EVM Logic Side:** Locate headers **J6** and **J7** on the TIOL221EVM.
* **Orientation:** Place the EVM over the Nucleo so that the J6/J7 pins align with the UART4 pins (PA0/PA1) on the Nucleo's right-side headers.

### 2. Required Jumper Wire Map
Since the pin spacing might not align perfectly for a direct "plug-in" stack, use female-to-male jumper wires to create the electrical stack:

| TIOL221EVM Pin | Nucleo-F746ZG Pin | Function |
| :--- | :--- | :--- |
| **J6 Pin 1 (TX)** | **PA0** | Transmit (UART4_TX) |
| **J6 Pin 2 (RX)** | **PA1** | Receive (UART4_RX) |
| **J6 Pin 3 (EN)** | **PD2** | Driver Enable (GPIO) |
| **J6 Pin 4 (WAKE)** | **PG2** | Wake-up Pulse (GPIO) |
| **J7 Pin 3 (GND)** | **GND** | Common Ground |

### 3. Jumper & Power Configuration
* **Logic Voltage (J11):** Ensure jumper **J11** on the TIOL221EVM is set to the **3.3V** position (shorting pins 2 and 3) to match the Nucleo's logic level.
* **VCC_IN Power:** You can take **3.3V** from the Nucleo’s `3V3` pin and connect it to the **VCC_IN** terminal on the EVM if you aren't using the internal LDO.
* **24V Isolation:** Ensure your 24V supply for the Zimmer Gripper is connected to the EVM's **J4/J5** terminals, but keep this high voltage physically away from the Nucleo's USB and logic pins.

### 4. Mechanical Mounting
* If you wish to secure them permanently, the TIOL221EVM has mounting holes that do not align with the Nucleo's. 
* It is recommended to use a **small adhesive breadboard** stuck to the top of the Nucleo's Ethernet jack or to use **nylon standoffs** to create a custom "mezzanine" mount.

Would you like me to find the **UART4 register configuration** needed in your C++ code to ensure the timing matches the IO-Link requirements for the Zimmer gripper?

---
---
Yes, you can use the **Micro-USB port** to flash firmware to the **NUCLEO-F746ZG**.

On this board, the Micro-USB connector is part of the integrated **ST-LINK/V2-1** debugger and programmer. When you connect it to your computer, it serves several functions:

* **Firmware Flashing**: It allows you to program the main STM32F746 microcontroller directly from your IDE (like STM32CubeIDE or Keil).
* **Mass Storage Interface**: The board appears as a USB drive on your computer; you can simply "drag and drop" a `.bin` or `.hex` file onto the drive to flash the firmware.
* **Virtual COM Port**: It provides a UART interface to your PC, which is useful for debugging and viewing serial terminal output.
* **Power Supply**: It provides the **5V** necessary to power the Nucleo's logic and the connected **TIOL221EVM** logic headers.

### Important Distinction
* **Micro-USB (ST-LINK)**: This is the port located at the top of the board (near the silk-screened "NUCLEO" logo). This is the one used for flashing.
* **User USB (Type-A/Micro)**: Depending on your specific board revision, there may be a second USB port (USB OTG) connected directly to the F746 chip. This port is for user applications (like making the Nucleo act as a mouse or keyboard) and **cannot** be used to flash the firmware via the standard debugger.

Would you like the specific **jumper settings** (specifically JP1) to ensure the board is drawing power correctly from that USB port during flashing?

---
---

Here is the summary of the pin mapping for the closed-loop industrial system, integrating the **Raspberry Pi 5**, **NUCLEO-F746ZG**, **TIOL221EVM**, and the **Zimmer Gripper**.

### 1. High-Level Control (Raspberry Pi 5 to Nucleo)

The Pi 5 acts as the master controller, sending commands to the Nucleo via SPI based on digital sensor data (HC-SR04/PIR).

| Function | Raspberry Pi 5 Pin | NUCLEO-F746ZG Pin |
| --- | --- | --- |
| **SPI MOSI** | GPIO 10 (Pin 19) | PA7 |
| **SPI MISO** | GPIO 9 (Pin 21) | PA6 |
| **SPI SCLK** | GPIO 11 (Pin 23) | PA5 |
| **SPI CS** | GPIO 8 (Pin 24) | PA4 |
| **Common GND** | Any GND Pin | Any GND Pin |

---

### 2. IO-Link Logic (Nucleo to TIOL221EVM)

The Nucleo controls the transceiver logic to translate UART signals into 24V IO-Link pulses.

| Function | NUCLEO-F746ZG Pin | TIOL221EVM (Header J6/J7) |
| --- | --- | --- |
| **TX (Data In)** | PA0 (UART4_TX) | **J6, Pin 1** |
| **RX (Data Out)** | PA1 (UART4_RX) | **J6, Pin 2** |
| **Enable (EN)** | PD2 (GPIO) | **J6, Pin 3** |
| **Wake-Up** | PG2 (GPIO) | **J6, Pin 4** |
| **Logic GND** | GND | **J7, Pin 3** |

---

### 3. Field Connection (TIOL221EVM to Zimmer Gripper)

This connection handles both the communication (C/Q) and the dual-power requirement for the Class B robot module.

| Zimmer Pin | Wire Color | Function | Connection Point |
| --- | --- | --- | --- |
| **Pin 1** | Brown | **L+** (Logic 24V) | TIOL221EVM **J2, Pin 1** |
| **Pin 2** | White | **P24** (Actuator 24V) | **+24V** Industrial PSU |
| **Pin 3** | Blue | **L-** (Logic GND) | TIOL221EVM **J3, Pin 1** |
| **Pin 4** | Black | **C/Q** (Data) | TIOL221EVM **J2, Pin 2** |
| **Pin 5** | Gray | **N24** (Actuator GND) | **GND** Industrial PSU |

---

### 4. Direct Sensors (Sensing to Raspberry Pi 5)

The digital sensors are wired directly to the Pi's GPIO for real-time processing.

* **HC-SR04 (Ultrasonic):** Trig to **GPIO 23**; Echo to **GPIO 24** (via 5V-to-3.3V divider).
* **HC-SR501 (PIR):** Out to **GPIO 25**.
* **Power:** Both sensors use the Pi’s **5V** and **GND** rails.


---
---
**M12 5-pin connector** is one of the most commonly used standard circular interfaces in industrial automation, sensors, actuators, and fieldbus communication. Its standard 12mm threaded design provides high vibration resistance and IP67-rated protection against water and dust. [1, 2, 3, 4, 5]

Below are the key technical specifications, standard pinout, and common coding types for the M12 5-pin connector:

## 📌 Most Common Standard A-Code Pinout
In industrial sensors (e.g., proximity switches, photoelectric sensors) and IO-Link devices, A-code is the most prevalent 5-pin standard. The standard wire color and pin assignments (compliant with IEC 61076-2-101) are as follows: [3, 5, 6]

| Pin Number [5, 7] | Standard Wire Color | Typical Function |
|---|---|---|
| Pin 1 | 🟤 Brown | +24V DC (positive power input) |
| Pin 2 | ⚪ White | I/O signal 2 / Normally Closed (NC) contact / Second channel signal |
| Pin 3 | 🔵 Blue | 0V / GND (power ground/negative) |
| Pin 4 | ⚫ Black | I/O signal 1 / Normally Open (NO) contact / Main switching output |
| Pin 5 | 🟢/🟡 Green-yellow or Grey (Gray) | FE / Shield (functional earth or shield) |

⚠️ **Layout note:** In A-code 5-pin connectors, Pins 1 through 4 are arranged on the four corners, while Pin 5 is in the exact center. Due to the addition of the center pin, the pin spacing becomes tighter; therefore, the rated operating voltage of A-code 5-pin versions is typically reduced to 60V AC/DC, whereas 3-pin or 4-pin versions can handle up to 250V. [8, 9, 10]

------------------------------

## 🔍 Common M12 5-pin Coding Types
M12 connectors use internal "keyways" to prevent mismating. Even with the same 5 pins, different codings serve completely different purposes: [7, 11]

* **A-Coded:** For standard DC sensors, actuators, IO-Link devices, and two-channel discrete signal transmission.
* **B-Coded:** Primarily for fieldbus applications, most notably Profibus-DP industrial networks.
* **L-Coded:** Specifically designed for DC high-power supply (typically 63V / 16A); now commonly used as the standard power interface for Profinet devices. [3, 4, 5, 9, 12, 13, 14, 15]

------------------------------

## ⚙️ Common Hardware Selection Specifications
If you are purchasing or replacing M12 5-pin connectors, pay attention to the following mechanical and electrical parameters:

- **Male vs. Female:** Male (with pins), female (with sockets). Typically, power outputs or sensor bodies use male connectors, while cables from control cabinets use female connectors.
- **Connection type:**
  - **Cordset (pre-molded):** Factory overmolded with a specified cable length (e.g., 2m, 5m).
  - **Field-wireable:** Housing can be opened, with screw terminals or solder terminals inside for on-site cable assembly.
- **Orientation:** Straight or 90° right angle; choose based on installation space constraints. [11, 16, 17, 18, 19, 20, 21]

------------------------------

What specific equipment are you using the M12 5-pin connector on (e.g., a particular brand of proximity sensor, IO-Link master, or Profibus network)? If you need a detailed wiring diagram or help finding a replacement part number, let me know and I can assist you further.

