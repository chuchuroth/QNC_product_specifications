# Key Code Snippets for "Local_Planner" Path Planning Algorithm (C++)

These C++ snippets illustrate the core logic for the "Local_Planner," assuming a ROS 2 environment and the use of the **Eigen** library for matrix operations, which is standard for robotics state estimation.

## 1. Sensor Fusion: Extended Kalman Filter (EKF)

The EKF is implemented using Eigen matrices for the state vector ($\mathbf{X}$) and covariance matrix ($\mathbf{P}$).

### 第一页 EKF Prediction Step (Motion Model) 

This snippet shows the EKF's prediction step, driven by motion inputs (linear and angular velocity).


### 第二页 EKF Update Step (Measurement Correction)

This snippet shows the EKF's update step, triggered by a new RTK-GPS measurement (position).





---

Today I want to focus on explaining a **code snippet from the EKF (Extended Kalman Filter)**. I believe this module best represents both the system-engineering strength of our project and the modern features of C++/ROS2.

## Why focus on the EKF?

**The EKF sits at the core of our system’s sensor-fusion pipeline**:

* Inputs: multiple sensors (IMU, wheel odometry, RTK-GPS)
* Output: a single reliable state estimate for the entire system
* Influence: navigation, planning, and control modules all depend on its output

---

## Key highlights from the EKF implementation

### 1. 🎯 **Modern C++ + high-performance computing**

```cpp
// Matrix operations using Eigen templates — zero-runtime-overhead abstractions
Eigen::MatrixXd K = P * H.transpose() * S.inverse();

// Contrast: traditional C++ might require manual loops like this
// for(int i=0; i<5; i++) {
//     for(int j=0; j<2; j++) {
//         K(i,j) = ... manual inverse and multiplication
//     }
// }
```

**Key point**: Eigen uses **expression templates** to optimize at compile time, avoiding temporary object creation. Performance is close to hand-written C but with much greater readability and safety.

### 2. 🔄 **ROS2 integration and real-time guarantees**

```cpp
void EKF::predict(double linear_velocity, double angular_velocity, double dt) {
    // Note: this may run in high-frequency callbacks (e.g., IMU at 100Hz)
    // Must ensure deterministic execution time
    double yaw = X(2);
    X(0) += linear_velocity * std::cos(yaw) * dt;  // No dynamic memory
    // ... all operations use compile-time sized matrices
}
```

**Design consideration**: we explicitly use fixed-size Eigen matrices (5x5) to avoid heap allocations in real-time callbacks — essential for meeting real-time performance requirements.

### 3. 🧮 **Numerical stability in practice**

```cpp
// Kalman gain — robust inverse
Eigen::MatrixXd S = H * P * H.transpose() + R_gps;
Eigen::MatrixXd K = P * H.transpose() * S.inverse();

// In production, we use a more stable solver:
    // Eigen::MatrixXd K = P * H.transpose() * S.ldlt().solve(I);
```

**Lesson learned**: early versions used `inverse()`, but unstable GPS signals could make S ill-conditioned. Switching to LDLT decomposition significantly improved robustness.

### 4. 📐 **Object-oriented design & testability**

```cpp
class EKF {
private:
    Eigen::VectorXd X; // State: [x, y, yaw, v, omega]
    Eigen::MatrixXd P; // Covariance
    
public:
    void predict(double v, double omega, double dt);      // Motion model
    void update_gps(double x, double y);                  // GPS update
    void update_imu(const ImuData& imu);                  // IMU update (extendable)

    const Eigen::VectorXd& getState() const { return X; }
    const Eigen::MatrixXd& getCovariance() const { return P; }
};
```

**Architectural advantages**:

* Unit testing per sensor update
* Simulated test-scenario injection
* Real-time filter consistency monitoring (NIS tests)

---

## Our system design philosophy reflected in EKF

### **Deterministic execution**

```cpp
Eigen::Matrix<double, 5, 5> P;  // Fixed size ensures known memory layout
```

### **Sensor fusion strategy**

```cpp
void update_high_frequency(const ImuData& imu);   // 100Hz — update angle
void update_medium_frequency(const Odom& odom);   // 50Hz  — update velocity
void update_low_frequency(const GpsFix& gps);     // 10Hz  — global correction
```

### **Fault detection**

```cpp
bool check_innovation(const Eigen::VectorXd& Y, const Eigen::MatrixXd& S) {
    double nis = Y.transpose() * S.inverse() * Y;
    return nis < chi_square_threshold;  // Chi-square test
}
```

---

## Key review points during code review

1. **✅ Matrix dimension consistency** — compile-time validation?
2. **✅ Numerical stability** — edge-case handling for singular matrices?
3. **✅ Real-time performance** — any dynamic memory allocation?
4. **✅ Thread safety** — concurrent sensor callbacks protected?
5. **✅ Test coverage** — covariance matrix positive-definiteness tests?

---

## Summary

Although the EKF module is small, it **embodies the core technical philosophy of our project**:

* **Eigen** — high-performance math
* **Modern C++** — zero-cost abstractions
* **ROS2** — real-time callback handling
* **System engineering** — reliability, testability, maintainability

Its successful implementation provides a solid foundation for our navigation and planning modules.

---
