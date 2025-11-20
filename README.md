[![Open in MATLAB Online](https://www.mathworks.com/images/responsive/global/open-in-matlab-online.svg)](https://matlab.mathworks.com/open/github/v1?repo=mschiavo93/exercise_pendulum_adrc&file=https://github.com/mschiavo93/exercise_pendulum_adrc/blob/main/pendulum_exercise.mlx)

# Pendulum exercise
# 1. Introduction & System Modeling
## 1.1 System Description and Parameters

This exercise demonstrates Active Disturbance Rejection Control (ADRC) for a pendulum system. The controller must regulate the pendulum angle to a setpoint while rejecting disturbances and handling sensor noise.


System Parameters:

-  Pendulum tip mass: M = 0.4 kg 
-  Rod length: l = 0.3 m 
-  Viscous friction coefficient: c = 0.1 
-  Gravitational acceleration: g = 9.81 m/s² 

Control Specifications:

-  Settling time (10% of setpoint): < 5 seconds (initial), < 2 seconds (final) 
-  Disturbance rejection: oscillations within 10% of setpoint 
-  Control effort: torque oscillations within 1 Nm peak\-to\-peak 
-  Setpoint: θ\_setpoint = 1 rad 

Disturbance Configuration:

-  Eccentric mass: m\_e = 0.2 kg at radius r\_e = 0.02 m 
-  Disturbance frequency: ω\_m = 20 rad/s 
-  Centrifugal force: Fc = m\_e \* r\_e \* ω\_m² 
# 2. ADRC Design & Implementation
## 2.1 Controller Tuning Parameters

The ADRC controller consists of a state\-feedback controller and an Extended State Observer (ESO) that estimates both system states and total disturbance.

## **2.2 Debug Mode: Validation with Ideal Plant**

Before controlling the actual pendulum, we validate our ADRC implementation on a simplified double\-integrator plant where the ESO model perfectly matches the plant dynamics.


**Objective:** Verify that the simulated response matches the theoretical second\-order response, confirming correct implementation of the ESO and control law.


**Expected Result:** Perfect overlap between simulated and theoretical responses, with ESO accurately estimating all states and zero total disturbance (no model mismatch).

![figure_0.png](./pendulum_exercise_media/figure_0.png)

![figure_1.png](./pendulum_exercise_media/figure_1.png)

**Observation:** The responses match perfectly. Furthermore, the ESO's state estimates (`theta_hat`, `theta_hat_dot`) align exactly with the plant's true states, and the estimated total disturbance `f_hat` is zero. This is expected because the plant model and the ESO's internal model are identical, leaving no "unmodeled" dynamics to reject.

# 3. Pendulum Model Validation
## 3.1 Open\-Loop Response Verification
### **Test 1:** Free Oscillation from Non\-Zero Angle
-  Initial condition: θ = π/2 rad 
-  Control: Manual mode with u_manual = 0 
-  Disturbance: Enabled at t = 5s 

**Expected Behavior:** Damped oscillations settling to vertical position (θ = 0) due to gravity and friction, with visible disturbance effects after t = 5s.

![figure_2.png](./pendulum_exercise_media/figure_2.png)

**Observation:** The pendulum swings and settles as expected, confirming the basic dynamics (gravity, inertia, friction) are implemented correctly. The disturbance at t=5s creates a clear oscillation.

### **Test 2:** Static Torque Balance
-  Initial condition: θ = π/2 rad 
-  Control: Constant torque u_manual = M*g*l 
-  Disturbance: Disabled 

**Expected Behavior:** Pendulum remains steady at π/2 rad because the applied torque balances gravitational torque.

![figure_3.png](./pendulum_exercise_media/figure_3.png)
### Test 3: Dynamic Torque Balance
-  Initial condition: θ = 0 rad 
-  Control: Constant torque u_manual = M*g*l/2 
-  Disturbance: Disabled 

**Expected Behavior:** Pendulum settles at non\-zero angle where applied torque balances gravitational torque.

![figure_4.png](./pendulum_exercise_media/figure_4.png)

**Observation:** The pendulum moves and settles at a non\-zero angle, confirming that the input (motor torque) correctly influences the plant model.


We also check the states estimated by the ESO.

![figure_5.png](./pendulum_exercise_media/figure_5.png)

![figure_6.png](./pendulum_exercise_media/figure_6.png)

 **Observation:** Note that now the total disturbance term is different from zero because the actual plant differes from the ideal double integrator plant. However, the ESO is doing a good job in properly estimating the states of the actual plant.

# 4. Closed\-Loop Performance Analysis
## 4.1 Nominal Case with ADRC Control

**Configuration:**

-  Control mode: ADRC enabled 
-  Initial conditions: θ = 0 rad, θ_dot = 0 rad/s 
-  Disturbance: Enabled 
-  Observer bandwidth: `keso = 20`  

**Performance Assessment:**

-  Verify settling within 5 seconds to 10% of setpoint 
-  Check disturbance rejection within 10% bounds 
-  Monitor control torque within 1 Nm peak\-to\-peak 

```matlabTextOutput
keso = 20
```

![figure_7.png](./pendulum_exercise_media/figure_7.png)

![figure_8.png](./pendulum_exercise_media/figure_8.png)

![figure_9.png](./pendulum_exercise_media/figure_9.png)
## **4.2 Effect of Sensor Noise**

Real sensors introduce measurement noise that affects control performance, particularly through the high\-gain observer.


 **Problem:** High observer bandwidth (`keso=20`) amplifies measurement noise into the control signal, causing excessive torque oscillations that violate the 1 Nm specification.

![figure_10.png](./pendulum_exercise_media/figure_10.png)

![figure_11.png](./pendulum_exercise_media/figure_11.png)

![figure_12.png](./pendulum_exercise_media/figure_12.png)
## **4.3 Noise\-Robust Tuning Trade\-off**

**Solution:** Reduce observer bandwidth to filter high\-frequency noise while maintaining adequate disturbance rejection.


**Trade\-off Analysis:**

-  Lower bandwidth (`keso=7`): Smoother control action, better noise rejection 
-  Higher bandwidth (`keso=20`): Faster state estimation, better disturbance rejection 
-  Compromise: `keso=7` provides acceptable performance with compliant control effort 

```matlabTextOutput
keso = 7
```

![figure_13.png](./pendulum_exercise_media/figure_13.png)

![figure_14.png](./pendulum_exercise_media/figure_14.png)

![figure_15.png](./pendulum_exercise_media/figure_15.png)
# 5. Performance Specification Change
## 5.1 Faster Set\-Point Tracking

**New Specification:** Settling within 10% of setpoint in less than 2 seconds.


**Approach:** Increase controller bandwidth while maintaining observer ratio for noise immunity.


**Validation:** Verify that the faster response meets the new settling time requirement while maintaining acceptable control effort and disturbance rejection.


```matlabTextOutput
keso = 5
```

![figure_16.png](./pendulum_exercise_media/figure_16.png)

![figure_17.png](./pendulum_exercise_media/figure_17.png)

![figure_18.png](./pendulum_exercise_media/figure_18.png)
# 6. Conclusion
## 6.1 Summary of Results

The ADRC controller successfully meets all control specifications:

1.  Setpoint Tracking: Achieves required settling times (5s and 2s versions)
2. Disturbance Rejection: Maintains oscillations within 10% of setpoint despite eccentric mass disturbance
3. Control Effort: Maintains torque oscillations within 1 Nm peak\-to\-peak specification
4. Noise Robustness: Proper observer tuning provides immunity to sensor noise
## **6.2 Key Insights**
-  Observer Bandwidth Trade\-off: Critical balance between estimation speed and noise sensitivity 
-  Model Independence: ADRC effectively handles unmodeled dynamics and disturbances 
-  Practical Tuning: Systematic approach to meeting multiple competing specifications 
## **6.3 Final Controller Parameters**

For nominal performance with noise:

-  Controller bandwidth: ω_c = 3 rad/s 
-  Observer ratio: `keso = 7` 
-  Settling time: ~2 seconds 

For enhanced set\-point racking performance:

-  Controller bandwidth: ω_c = 4 rad/s 
-  Observer ratio: `keso = 5` 

-  Settling time: ~1.5 seconds 

