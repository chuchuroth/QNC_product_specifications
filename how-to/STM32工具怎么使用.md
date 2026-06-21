**STM32CubeMX** is a graphical software configuration tool from STMicroelectronics that generates C initialization code for STM32 microcontrollers. It allows you to configure the hardware pins and peripherals of your **NUCLEO-F746ZG** without writing low-level register code manually.

### How to Configure Peripherals in CubeMX

To set up your industrial automation system, follow these steps within the CubeMX interface:

#### 1. Project Initialization
* **Select Board:** Start a new project and select the **NUCLEO-F746ZG** from the "Board Selector" to automatically pre-configure the high-speed external clocks and the ST-LINK pins.
* **Clear Pinout:** If prompted, you can choose to clear the default pinout to start fresh for your specific mapping.

#### 2. Configure IO-Link Communication (UART4)
For the **TIOL221EVM** connection:
* **Enable Peripheral:** In the "Connectivity" category, select **UART4**.
* **Mode:** Set the Mode to **Asynchronous**.
* **Parameters:** Set the **Baud Rate** to 38400 (COM2) or 230400 (COM3) to match the Zimmer gripper's requirements.
* **Pin Assignment:** Ensure **PA0** is set as `UART4_TX` and **PA1** is set as `UART4_RX`.

#### 3. Configure Controller Link (SPI1)
For the **Raspberry Pi 5** communication:
* **Enable Peripheral:** Select **SPI1** from the "Connectivity" menu.
* **Mode:** Set the Mode to **Full-Duplex Slave**.
* **Pin Assignment:** Verify the pins are mapped as **PA5 (SCK)**, **PA6 (MISO)**, and **PA7 (MOSI)**.
* **NSS Signal:** Set the Hardware NSS Signal to **Input Hardware** (mapped to **PA4**).

#### 4. Configure Control GPIOs
For the **TIOL221EVM** logic pins:
* **Pin Mode:** Click on **PD2** and **PG2** in the chip diagram and set them to **GPIO_Output**.
* **Labeling:** Right-click the pins to enter "User Labels" like `IO_LINK_EN` and `IO_LINK_WAKE` for cleaner code generation.

#### 5. Code Generation
* **Project Manager:** Go to the "Project Manager" tab, name your project, and select your Toolchain (e.g., **STM32CubeIDE**).
* **Generate:** Click **GENERATE CODE** in the top right corner. CubeMX will create a project folder with all drivers and the `main.c` file ready for your logic.

---

### Summary Table for CubeMX Settings

| Peripheral | Mode | Required Pins |
| :--- | :--- | :--- |
| **UART4** | Asynchronous | PA0, PA1 |
| **SPI1** | Full-Duplex Slave | PA4, PA5, PA6, PA7 |
| **GPIO Output** | Push-Pull | PD2, PG2 |

Would you like me to walk through the **Clock Configuration** tab to ensure your UART timings are precise enough for IO-Link?


---
---
Since you are a beginner, the best way to do this is using **STM32CubeMX** (or the Device Configuration tool inside **STM32CubeIDE**). 

Follow these exact steps to set up your NUCLEO-F746ZG.

---

### Step 1: Start the Project
1. Open **STM32CubeIDE**.
2. Go to **File > New > STM32 Project**.
3. In the **Board Selector** tab, type `NUCLEO-F746ZG` in the search box.
4. Select the board and click **Next**.
5. Name your project (e.g., `IOLink_Project`) and click **Finish**.
6. **Important:** When it asks "Initialize all peripherals with their default mode?", click **No**. We want a clean slate.

---

### Step 2: Configure SPI1 (Communication with RPi5)
1. On the left sidebar, click **Connectivity** and select **SPI1**.
2. In the **Mode** dropdown at the top, select **Full-Duplex Slave**.
3. For **Hardware NSS Signal**, select **Input Hardware**.
4. In the **Configuration** panel (bottom center), click the **Parameter Settings** tab:
   - **Data Size:** 8 Bits
   - **First Bit:** MSB First
   - **CPOL:** Low (CPOL = 0)
   - **CPHA:** 1 Edge (CPHA = 0)
5. Click the **NVIC Settings** tab:
   - Check the box for **SPI1 global interrupt** to **Enabled**.

---

### Step 3: Configure UART4 (IO-Link to TIOL221EVM)
1. In the sidebar under **Connectivity**, select **UART4**.
2. Set **Mode** to **Asynchronous**.
3. In **Parameter Settings**:
   - **Baud Rate:** 38400
   - **Word Length:** 8 Bits (including parity)
   - **Parity:** None
   - **Stop Bits:** 1
4. Look at the **Pinout view** (the chip diagram on the right). Ensure **PA0** shows `UART4_TX` and **PA1** shows `UART4_RX`.

---

### Step 4: Configure GPIOs (Control Pins)
1. Locate **PD2** on the chip diagram.
2. Left-click **PD2** and select **GPIO_Output**.
3. Right-click **PD2**, select **Enter User Label**, and type `IOLINK_EN`.
4. Locate **PG2** on the chip diagram.
5. Left-click **PG2** and select **GPIO_Output**.
6. Right-click **PG2**, select **Enter User Label**, and type `IOLINK_WAKE`.
7. (Optional) Go to **System Core > GPIO** in the sidebar to verify:
   - Both should have **GPIO output level** set to **Low**.

---

### Step 5: System Core (Timing & Debug)
1. Go to **System Core > SYS**.
2. Set **Timebase Source** to **Timer 1** (or any timer except SysTick). *This is a best practice for RTOS or complex HAL projects.*
3. Ensure **Debug** is set to **Serial Wire**.

---

### Step 6: Generate Code
1. Press **Ctrl + S** (or click the Cog icon in the toolbar) to save and generate code.
2. Click **Yes** when it asks if you want to generate code.
3. Click **Yes** when it asks if you want to open the "C/C++ Perspective."

---

### Step 7: Adding the µs-accurate DWT Timing (Manual Code)
CubeMX cannot configure the DWT (Data Watchpoint and Trace) automatically, so you must add this small snippet in your `main.c` to handle the **IO-Link Wake-Up pulse** (which requires exactly 80µs).

1. Open `Core/Src/main.c`.
2. Scroll down to **USER CODE BEGIN 2** (inside the `main` function) and paste this:

```c
/* USER CODE BEGIN 2 */
// Enable DWT for microsecond delays
CoreDebug->DEMCR |= CoreDebug_DEMCR_TRCENA_Msk;
DWT->CYCCNT = 0;
DWT->CTRL |= DWT_CTRL_CYCCNTENA_Msk;
/* USER CODE END 2 */
```

3. To use it for a delay, you can add this simple function later in your code:
```c
void delay_us(uint32_t us) {
    uint32_t startTick = DWT->CYCCNT;
    uint32_t delayTicks = us * (SystemCoreClock / 1000000);
    while (DWT->CYCCNT - startTick < delayTicks);
}
```

### Final Result
Your hardware is now logically mapped:
* **RPi5** sends data to **SPI1** pins.
* **Nucleo** sends IO-Link data through **UART4**.
* **PD2** and **PG2** act as the "Master Enable" switches for your **TIOL221EVM**.

---
---
Since you are using **STM32CubeIDE**, the "Saving" and "Project Update" steps are usually handled automatically. Here is the exact workflow to move from the graphical interface to your code:

### 1. The "Save & Generate" Shortcut
Once you have finished selecting your SPI, UART, and GPIO settings in the Device Configuration Tool (the `.ioc` file):
* **Press `Ctrl + S`** (Windows/Linux) or **`Cmd + S`** (Mac).
* Alternatively, click the **Yellow Cog Icon** (Generate Code) in the top toolbar.

### 2. The Generation Dialogs
After you save, the IDE will pop up two important windows. **Click "Yes" to both:**
1.  **"Do you want to generate code?"** → **Yes**. This writes the initialization C-code into your `main.c` and creating the necessary library folders (`Drivers`, `Inc`, `Src`).
2.  **"This action is associated with the C/C++ perspective. Do you want to open this perspective now?"** → **Yes**. This switches the view from the chip diagram back to the code editor.

### 3. Where to find your configuration in the project
After the code is generated, your folder structure on the left (Project Explorer) will update. Look for these specific files:

* **`Core/Src/main.c`**: This is where your code lives. 
    * Scroll down to find `MX_SPI1_Init();`, `MX_UART4_Init();`, and `MX_GPIO_Init();`. These are the functions CubeMX just wrote for you based on your settings.
* **`Core/Inc/main.h`**: This is where your **User Labels** (like `IOLINK_EN`) are defined as macros (e.g., `#define IOLINK_EN_Pin GPIO_PIN_2`).

### 4. Critical Rules for Beginners (The "User Code" Blocks)
**This is the most important part of using CubeMX/IDE:** If you ever need to change a pin setting later and re-generate the code, the IDE will **delete** any code you wrote unless it is placed inside the special "User Code" comments.

**Always write your code here:**
```c
/* USER CODE BEGIN 2 */
// Your code here will survive if you change CubeMX settings later
/* USER CODE END 2 */
```

**Never write your code here:**
```c
/* Generated initialization code */
MX_SPI1_Init();
// Your code here WILL BE DELETED if you click "Save" in CubeMX again!
```

### 5. Troubleshooting: If it doesn't generate
If clicking save doesn't do anything:
* Go to **Project > Generate Code** in the top menu.
* Check the **Console** tab at the bottom of the screen. If there is a "Conflict" (e.g., you assigned two things to one pin), it will list the error there.

**Next Step:** Once your code is generated, are you ready for the `HAL_UART_Transmit` command to send your first message to the Zimmer Gripper?


---
---
In a standard **STM32CubeIDE** project, the `.ioc` file is located in the **root directory** of your project folder.

### 1. Visual Location in the Project Explorer
Look at the **Project Explorer** (the list of folders on the left side of your screen). You will see it listed near the bottom of the project tree, just above the `.mxproject` or `.project` files.
* **Filename:** `[YourProjectName].ioc`
* **Icon:** It has a blue and white "MX" cube icon.

### 2. Physical Location on your Computer
If you are looking for it in Windows Explorer or macOS Finder, go to your workspace folder:
`C:\Users\[Username]\STM32CubeIDE\workspace_1.x.x\[YourProjectName]\[YourProjectName].ioc`

---

### Why the `.ioc` file is important:
* **It is your "Source of Truth":** It stores all the graphical settings you just made (UART4, SPI1, GPIO labels). 
* **Double-Click to Edit:** If you ever realize you need to change a pin or a baud rate, simply double-click this file inside the IDE. It will re-open the chip diagram and configuration menus.
* **Trigger Code Generation:** Every time you modify this file and save it, the IDE will ask to update your `main.c`.

### Pro-Tip for Beginners
If you accidentally close the graphical pinout view, don't panic! Just find that `.ioc` file in the list on the left and **double-click it** to bring the interface back.

**Would you like me to show you how to write the first few lines of code in `main.c` to pull the `IOLINK_EN` pin high and start the transceiver?**


---
---
That is a common point of confusion for beginners! If you only see the `.ld` (Linker Script) file, it usually means your **Project Explorer** view is "filtered" or the project structure hasn't fully expanded.

Here is how to fix it and find your files:

### 1. Check the "Filters and Customization"
The Project Explorer sometimes hides files starting with a dot (like `.project`).
* In the **Project Explorer** tab, look for the **three vertical dots** (or a small downward arrow) in the top right corner of that specific sub-window.
* Click it and select **Filters and Customization...** (or just *Filters*).
* Uncheck **".* resources"**. 
* Now, files like `.project` and `.mxproject` should appear.

### 2. The "Hidden" Folder Structure
Actually, STM32CubeIDE is designed to keep your workspace clean. You don't usually need to touch the `.project` or `.mxproject` files—they are system files used by the IDE to manage your project. 

If you are looking for your **IOC file**, it should be at the same level as the `Core`, `Drivers`, and `Middlewares` folders. 
* **If you don't see any folders:** Click the small **">" arrow** next to your Project Name to expand the tree.

### 3. If you truly cannot find the `.ioc` file
If you created the project but the `.ioc` file is missing, the project may have been created as a "Standard C" project instead of an "STM32 Project." 
* **The Fix:** Look at the top menu. Go to **File > New > STM32 Project**. 
* When you name it and click finish, the IDE *must* generate the `.ioc` file automatically.

---

### Where to go from here?
Since you see the `.ld` file, you are definitely in the right folder. Look for the folder named **`Core`**.
1.  Inside **`Core`**, you will find **`Src`** (Source files).
2.  Inside **`Src`**, you will find **`main.c`**. 

**Would you like me to give you the `HAL_GPIO_WritePin` code to put into `main.c` so you can test if your TIOL221EVM "Power" LED turns on?**

