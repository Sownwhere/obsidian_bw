# Franks Collins Hip Controller
## Description
Provides Hip assistance based on a series of splines that have an extension and flexion portion.
Uses percent gait, requiring heel FSR.
Based on:
P.W. Franks, G.M. Bryan, R.M. Martin, R. Reyes, A.C. Lakmazaheri, & S.H. Collins (2021).
[Comparing optimized exoskeleton assistance of the hip, knee, and ankle in single and multi-joint configurations](https://www.cambridge.org/core/journals/wearable-technologies/article/comparing-optimized-exoskeleton-assistance-of-the-hip-knee-and-ankle-in-single-and-multijoint-configurations/9FBC1580F11614B388BE621D716800AD).in Wearable Technologies, vol. 2, pp. e16, doi: 10.1017/wtc.2021.14.

  

## Parameters
Parameter index order can be found in [ControllerData.h](/ExoCode/src/ControllerData.h).
- mass - User mass in kg used for denormalizing torque, currently not used in the controller but easy to implement if desired, defaults to zero as a result.
- trough_normalized_torque_Nm_kg - Largest extension torque (Nm). Note: Currently not noramlized by user mass.
- peak_normalized_torque_Nm_kg - Largest flexion torque (Nm). Note: Currently not noramlized by user mass.
- start_percent_gait - Percent gait to start the torque pattern so it doesn't have a discontinuity at heel strike.
- trough_onset_percent_gait - Percent gait to start applying extension torque.
- trough_percent_gait - Percent gait to apply the largest extension torque.
- mid_time - Transition point between extension and flexion to apply 0 torque.
- mid_duration - The duration of the transition pause as a percent of the gait cycle.
- peak_percent_gait - Percent gait to apply the largest flexion torque.
- peak_offset_percent_gait - Percent gait to stop applying flexion torque.
- use_pid - This flag turns PID on(1) or off(0)
- p_gain - Proportional gain for closed loop control
- i_gain - Integral gain for closed loop control
- d_gain - Derivative gain for closed loop control
---

## 1️⃣ 控制结构总体

这个算法核心思想是：

> **“预定义助力力矩曲线 + 时间/步态移位 + 可选 PID 闭环修正 + 三次样条插值平滑”**

- **底层控制**：通过 `_spline_generation` 生成光滑的助力力矩（feedforward torque）。
    
- **高层控制**：根据步态百分比计算当前阶段，并动态调整曲线起点以避免不连续（heel strike 时）。
    
- **闭环可选**：通过 PID 控制修正力矩偏差（根据传感器读取的关节力矩）。
    

---

## 2️⃣ 步态百分比与时间管理

- **步态百分比计算**：
    
    ```cpp
    float percent_gait = _get_percent_gait(simulate);
    ```
    
    - **真实机器人**：直接读取 `_side_data->percent_stance`。
        
    - **仿真模式**：通过 `_t_helper->tick()` 累加仿真时间，计算 0~100% 的步态循环。
        
- **步态移位处理（shifted_percent_gait）**：
    
    - 避免在 heel strike（脚跟着地）出现力矩不连续。
        
    - 当步态超过设定的起点 `start_percent_gait` 时，用 `millis() - last_start_time` 计算移位后的百分比：
        
        ```cpp
        shifted_percent_gait = (millis() - last_start_time) / expected_duration * 100
        ```
        

---

## 3️⃣ 力矩曲线生成逻辑

- **用户参数**：
    - 伸展力矩（extension）峰值、屈曲力矩（flexion）峰值。
    - 力矩峰值在步态周期的百分比位置。
    - 中间零力矩区域（mid_time, mid_duration）。
        
- **节点计算**：
    - 伸展和屈曲曲线各用 **3 个关键节点**：
        - `node1` → 力矩开始上升或下降
        - `node2` → 力矩峰值
        - `node3` → 力矩下降到零
    - `_spline_generation(node1, node2, node3, torque, gait)` 生成平滑曲线。
        
- **三次样条插值**：
    - 保证曲线平滑，无力矩突变：
        ```cpp
	    torque_cmd = spline(extension_nodes) + spline(flexion_nodes)
        ```

---

## 4️⃣ 力矩输出逻辑
1. **未开始步态循环**：
    - `last_start_time == -1` → 返回 0，等待完整步态循环开始。
2. **Feedforward 力矩**：
    - 基于步态百分比和三次样条曲线计算。
3. **可选 PID 调节**：
    ```cpp
    if (use_pid) cmd = torque_cmd + pid_correction
    else cmd = torque_cmd
    ```
    
    - `_pid()` 根据力矩测量和期望值计算闭环修正。
4. **力矩存储**：
    - `ff_setpoint` → feedforward 力矩
    - `desired_torque` → 绘图/调试用

---

## 5️⃣ 算法特点

- **周期性与平滑性**：
    - 三次样条保证力矩曲线连续且光滑。
    - 移位步态百分比避免 heel strike 力矩跳变。
- **模块化与可调节性**：
    - 用户可调参数：力矩峰值、起始点、中间零力矩区域、PID 增益。
- **仿真/真实兼容**：
    - `_get_percent_gait` 支持仿真累加时间和真实机器人状态读取。
- **混合开环 + 闭环**：
    - 默认开环 feedforward，必要时可加 PID 闭环修正。

---

✅ **一句话总结**：

> 这个算法通过根据步态百分比生成平滑的伸展/屈曲力矩曲线，并结合步态移位与可选 PID 修正，实现髋关节外骨骼的周期性助力输出，保证动作连续、自然且可调节。





# Proportional Hip Moment Controller
## Description
Provides Hip assistance based on real-time estimate of the user's hip joint moment
Requires both toe and heel FSRs.
NOTE: THIS CONTROLLER IS STILL UNDER DEVELOPMENT (Stay tuned for future updates).
Based on:
S.S.P.A. Bishe, L. Liebelt, Y. Fang, & Z.F. Lerner (2022).[A Low-Profile Hip Exoskeleton for Pathological Gait Assistance](https://ieeexplore.ieee.org/document/9812300).2022 IEEE ICRA.
## Parameters
Parameter index order can be found in [ControllerData.h](/ExoCode/src/ControllerData.h).
- extension_setpoint - Magnitude of peak extension torque (Nm).
- flexion_setpoint - Magnitude of peak flexion torque (Nm).



你贴的这段 `ProportionalHipMoment::calc_motor_cmd()` 代码实现的是一种基于 **步态阶段（State Machine） + 足底压力（FSR）反馈 + 比例助力** 的髋关节外骨骼力矩控制策略。我帮你总结一下核心控制原理和逻辑：

---
# Proportional Hip Moment Controller Summary

## 控制原理总览

该控制器实现了一种基于 **步态阶段 (State Machine) + 足底压力反馈 (FSR) + 比例助力** 的髋关节外骨骼力矩控制策略。主要特点如下：

1. **状态机 (State Machine) 区分步态阶段**
    
    - State 1：Late Swing（后摆期）
        
    - State 2：Stance（支撑期）
        
    - State 3：Early Swing（早摆期）
        
2. **足底传感器判断阶段切换**
    
    - 支撑 → 摆动：前脚和后跟 FSR 信号低于阈值
        
    - 摆动 → 支撑：前脚或后跟 FSR 信号高于阈值，并连续多次检测去噪
        
3. **设定点计算**
    
    - 每个阶段有对应的力矩设定点 (extension_setpoint/flexion_setpoint)
        
    - 通过线性插值、比例因子和步态时间归一化系数动态计算当前力矩
        
4. **安全保护**
    
    - 限制最大输出力矩（abs(setpoint) > 20）
        
    - 防止设定点超过阶段上限
        
    - 阶段切换去噪（连续多次检测才切换）
        

## 状态机逻辑

### State 1：Late Swing（后摆期）
- 初始化计数器
- 线性插值计算 setpoint
- 脚接地检测连续 3 次 → 进入 State 2
- 限制最大力矩

### State 2：Stance（支撑期）
- 初始化摆动计数器
- 计算 fs = heel_fsr - 0.25 * toe_fsr
- 根据 fs 计算 hip_ratio
    - Early stance：逐渐从 0.6 → 1
    - Late stance：根据压力和上一次摆动时长平滑过渡
- setpoint = hip_ratio * extension_setpoint / flexion_setpoint
- 脚离地连续 3 次 → 进入 State 3
    

### State 3：Early Swing（早摆期）
- 初始化支撑计数器
- 计算过渡系数 t = Alpha_counter / Alpha
- setpoint = (4_fs_min_0.5 - 0.5*(9_t^2 - 9_t)/(3*t-4)) * flexion_setpoint
- 脚触地 → 直接跳到 State 2
- Early Swing 结束 → 进入 State 1

## 助力计算核心公式

### 支撑期 (State 2)

$$ \begin{aligned} fs &= heel\_fsr - 0.25 \times toe\_fsr \\ \\ hip\_ratio &= \begin{cases} 0.6 + fs \times 0.4, & \text{early stance} \\ 4 \times fs_{min} \times 0.5 - 0.5 \times \frac{9t^2 - 9t}{3t - 4}, & \text{late stance} \\ fs, & \text{其他情况} \end{cases} \\ \\ setpoint &= hip\_ratio \times \frac{extension\_setpoint}{flexion\_setpoint} \end{aligned} $$
### 摆动期 (State 1/3)

setpoint=插值/过渡系数∗flexionsetpointsetpoint = 插值/过渡系数 * flexion_setpoint
### 安全限制

∣setpoint∣>20⇒setpoint=0|setpoint| > 20 \Rightarrow setpoint = 0

## 控制特点
- 基于步态分阶段控制，针对不同阶段使用不同策略
- 使用 FSR 信号判断阶段切换和比例因子
- 线性 + 非线性插值平滑生成力矩
- 开环力矩输出，可扩展为闭环 PID 控制
- 阶段切换去噪，保证稳定性
    

## 总结

> 该控制器通过 **状态机划分步态阶段 + 足底压力检测 + 线性/非线性比例插值** 来动态生成髋关节助力力矩，并在各阶段使用安全限制和平滑过渡保证稳定、自然的人机交互。