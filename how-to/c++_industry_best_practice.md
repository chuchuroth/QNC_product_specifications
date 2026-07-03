# industry best practice
Excellent choice.
Here is an **Industrial C++ Style Cheat Sheet** blending:

* ✅ Google C++ Guide
* ✅ MISRA / AUTOSAR C++ (safety robotics)
* ✅ ROS2 + embedded real-time conventions

Used in **AGV, arm controllers, SLAM, safety PLC–ROS bridges**.

---

## 🧠 **Philosophy**

> Code must run **forever**, **safely**, and be debugged at 3 AM during robot downtime.

---

## ✅ **Namespaces**

### ❌ Never

```cpp
using namespace std;
using namespace rclcpp;
```

### ✅ Allowed

```cpp
using std::string;
namespace msg = sensor_msgs::msg;
```

> Import only what you *explicitly* need.

---

## ✅ **Headers**

* Avoid wildcard headers (`<bits/stdc++.h>` = banned)
* Standard ordering:

```cpp
// 1. Own header
#include "motor_controller.hpp"

// 2. STL
#include <memory>
#include <string>
#include <vector>

// 3. Third-party
#include <Eigen/Dense>
#include <rclcpp/rclcpp.hpp>

// 4. System libs
#include <unistd.h>
```

> Deterministic order = fewer hidden build errors.

---

## ✅ **Memory / Performance**

| Rule                                                            | Reason                          |
| --------------------------------------------------------------- | ------------------------------- |
| No `new` / `delete`                                             | fragmentation, leaks            |
| Prefer `std::unique_ptr`                                        | ownership explicit              |
| Use `std::shared_ptr` only when needed                          | lifetime clarity                |
| Reserve vector capacity                                         | avoid realloc in real-time loop |
| No exceptions in real-time loops                                | unpredictable latency           |
| Avoid STL algos that allocate (`std::function` in control loop) | jitter                          |

---

## ✅ **Classes & Objects**

### Class layout

```cpp
class MotorController {
public:
    explicit MotorController(double max_rpm);
    void update(double dt);

private:
    double max_rpm_;
    double current_rpm_{0.0};
};
```

**Rules**

* `explicit` single-arg ctors
* init members in initializer list
* trailing `_` on private vars

---

## ✅ **Const Correctness**

```cpp
double getSpeed() const;        // doesn’t modify state
void setSpeed(double rpm);      // modifies state
```

> Const everything until proven mutable.

---

## ✅ **Control Loop Safety**

| Rule                         | Why                           |
| ---------------------------- | ----------------------------- |
| No dynamic memory            | RT determinism                |
| No logging inside 1kHz loops | stalls motor thread           |
| Use `noexcept`               | avoid unwinding in motor code |
| Use bounded queues           | avoid memory blow-up          |

---

## ✅ **Error Handling**

### ✅ Robotics pattern

```cpp
if (!sensor_ok) {
    RCLCPP_ERROR(logger_, "IMU failure");
    enterSafeStop(); // brake motors
}
```

### ❌ Bad

```cpp
throw std::runtime_error("imu bad");
```

> Safety first — system must degrade gracefully.

---

## ✅ **Threading**

| Rule                             | Reason          |
| -------------------------------- | --------------- |
| No `std::mutex` in RT loop       | lock contention |
| Prefer lock-free / double buffer | deterministic   |
| Pin control thread to CPU        | low jitter      |
| Use `std::atomic` for flags      | fast, safe      |

---

## ✅ **Naming Conventions**

| Type       | Style                                        |
| ---------- | -------------------------------------------- |
| Classes    | `MotorController`                            |
| Functions  | `setTargetRpm`                               |
| Variables  | `motor_rpm_` (private) / `motor_rpm` (local) |
| Constants  | `kWheelRadius`                               |
| Namespaces | `robot::control`                             |

---

## ✅ **Logging**

Use ROS throttle / once:

```cpp
RCLCPP_WARN_THROTTLE(get_logger(), *clock_, 1000, "slip detected");
```

> Logging ≠ debugging tool in a real-time loop.

---

## ✅ **Safety Patterns**

| Pattern          | Purpose                  |
| ---------------- | ------------------------ |
| Watchdog timer   | detect lockups           |
| Dead-man switch  | mobile robots            |
| Brake on timeout | AGV / arm emergency stop |
| Heartbeat topics | monitor sensor health    |

---

## ✅ **Banned in industrial robotics**

❌ `using namespace std;`
❌ raw `new/delete`
❌ throwing from control loop
❌ global mutable state
❌ `std::cout` in runtime
❌ unbounded queue
❌ heap allocation in hard real-time

---

## 🛠 Best Practice Checklist

| Category        | Checklist                               |
| --------------- | --------------------------------------- |
| Performance     | pre-allocate, no dynamic memory in loop |
| Safety          | watchdog, error fallback                |
| Clarity         | explicit types, descriptive names       |
| Testing         | unit tests + hardware-in-loop           |
| Determinism     | lock-free queues, RT kernel             |
| Maintainability | short functions, single responsibility  |

---

## 🎯 **Summary**

> Industrial C++ = *predictability + clarity + safety*
> ROS2 robotics = real-time + low-jitter + recoverable faults

This style gets you hired at:
ABB · Boston Dynamics · NVIDIA Robotics · AGV factories · Autoware · MoveIt2

---

## Next step?

Reply with a number:

| Option                                                            | Output next |
| ----------------------------------------------------------------- | ----------- |
| 1️⃣ Real-time motor control thread template (1 kHz)               |             |
| 2️⃣ ROS2 executor + callback groups for multi-sensor fusion       |             |
| 3️⃣ Move semantics + zero-copy ROS2 pipeline (PointCloud / Image) |             |
| 4️⃣ C++ safety MISRA pitfalls list (with fixes)                   |             |
| 5️⃣ Industrial code review checklist (50+ rules)                  |             |

Which one?


---

---

### ✅ Why *not* `using namespace std;` (and also not `using namespace rclcpp;`) in robotics / production C++**, especially not in ROS2.

| Reason                      | Impact                                       |
| --------------------------- | -------------------------------------------- |
| Namespace pollution         | `string`, `map`, `bind`, `mutex` collisions  |
| Library collisions          | ROS2 has `Duration`, `Time`, `bind` too      |
| Hidden bugs                 | Hard to trace errors in multi-module systems |
| Bad for code reviews        | Harder to understand where types come from   |
| Industrial coding standards | Robotics, embedded, automotive all forbid it |

In robotics we combine:

* ROS2 (rclcpp, builtin_interfaces, sensor_msgs)
* STL (`std`)
* Eigen
* OpenCV
* DDS transport libs
* Real-time middleware

Names collide **everywhere**.

Example conflict:

```cpp
string name;     // from std
rclcpp::Duration d(1s);  // ROS Duration
```

If you `using namespace std`, sometimes `Duration` resolves to std::chrono incorrectly.

---

### ✅ What to do instead

### **Use selective `using` for local clarity**

```cpp
using std::string;
using std::vector;
using sensor_msgs::msg::LaserScan;
```

This avoids spam like `std::vector<std::string>` everywhere while staying safe.

---

### ✅ Modern C++ alternative

Use **aliases**:

```cpp
namespace msg = sensor_msgs::msg;
using LaserScanPtr = msg::LaserScan::SharedPtr;
```

Then code becomes:

```cpp
rclcpp::Subscription<msg::LaserScan>::SharedPtr sub_;
void callback(LaserScanPtr scan);
```

Nice, clean, safe.

---

### ✅ TL;DR

> In ROS2/robotics, **never** `using namespace std;` globally.
> Prefer **specific using + namespace aliases** instead.

---

### 🛠 Professional ROS2 style guide used by:

* Nav2
* MoveIt2
* Autoware
* Industrial-grade AMR/AGV stacks

---

### Want me to rewrite your earlier snippets in **clean namespace style**?

Reply:
`yes — and include Eigen + chrono + nav2 example`

Or tell me which you want:

| Choice                                                          | Output |
| --------------------------------------------------------------- | ------ |
| 1️⃣ Clean PID with Eigen & chrono                               |        |
| 2️⃣ ROS2 node with QoS & TF2 & parameters style example         |        |
| 3️⃣ Multi-threaded control node (RT safe)                       |        |
| 4️⃣ Move semantics + zero-copy ROS2 message style               |        |
| 5️⃣ Industrial C++ style cheat sheet (Google + MISRA-C++ blend) |        |

Which number?

---
---


