IK（Inverse Kinematics，逆运动学）本质是在**已知末端位姿，求关节变量**。常见方法可以分成三大类：解析法、数值优化法、学习法。你做机器人控制的话，基本都会碰到。

---

# 一、解析解（Analytical IK）

## 1. 几何法（Geometric）

适合：结构简单的机械臂（2~3 DOF、平面、球形腕）
思路：用几何关系直接推导角度。
例如二维2连杆：
$x = l_1 \cos\theta_1 + l_2 \cos(\theta_1 + \theta_2)\quad ,\quad y = l_1 \sin\theta_1 + l_2 \sin(\theta_1 + \theta_2)$
然后通过余弦定理解：
$\cos\theta_2 = \frac{x^2 + y^2 - l_1^2 - l_2^2}{2 l_1 l_2}$
优点：
- 快（实时控制友好）
- 精确（无迭代误差）
缺点：
- 只适合特定结构
- 一旦结构复杂基本写不动

---

## 2. 代数法（Algebraic）

思路：把 IK 变成多项式方程组，然后求解。
- 通过齐次变换矩阵展开
- 消元得到多项式
- 求根（可能多个解）

特点:
- 可得完整解集
- 但推导非常复杂
- 工程上不常用（除非论文）

---

## 3. 几何分解（典型工业方法）

典型6DOF机械臂：
- 前3轴：求位置（position IK）
- 后3轴：求姿态（orientation IK）

利用：
- 球形腕（spherical wrist）结构分解
- 解耦 position + orientation

---

# 二、数值解法（Numerical IK）⭐ 工程最常用
当机械臂复杂（humanoid / redundant / legged robot），基本都用这个。

---
## 1. Jacobian 逆法（最经典）
核心关系：
$$ \dot{x} = J(q) \dot{q} $$
目标误差：
$$e = x_{target} - x(q)$$
基本更新：
$$\Delta q = J^{+} e$$
其中：
- (J^{+})：伪逆
---

### 改进版本（稳定性更好）
#### Damped Least Squares (DLS)
J^{+} = J^T (J J^T + \lambda^2 I)^{-1}
优点：
- 避免奇异点爆炸
- 工业机器人常用
---
## 2. 牛顿-拉夫森法（你前面问过的）
本质：解非线性方程
q_{k+1} = q_k - J(q_k)^{-1} (x(q_k) - x_{target})
特点：
- 收敛快（局部二次收敛）
- 但对初值敏感    

---

## 3. 伪逆 + 阻尼 + 投影（Redundancy handling）
冗余机械臂（7DOF+）常用：
\dot{q} = J^{+} \dot{x} + (I - J^{+}J)z
- 第一项：完成任务
- 第二项：零空间优化（避障/关节限位）
---
## 4. 优化法（最强通用方法）⭐
把 IK 写成优化问题：
\min_q ; |x(q) - x_{target}|^2 + \lambda |q|^2
再用：
- Gauss-Newton
- Levenberg–Marquardt
- SQP
优点：
- 可以加约束（关节限位、碰撞）
- 最适合 humanoid / MPC / RL    
缺点：
- 计算慢一点
---
# 三、学习法（Learning-based IK）
## 1. 神经网络直接回归
- 输入：目标 pose
- 输出：关节角

优点：
- 超快
- 可泛化（训练好之后）

缺点：
- 不保证精确满足约束
- 需要大量数据
---
## 2. Residual IK（很常见）
- 网络预测初值
- 再用 Jacobian refine

👉 工业 + RL 常用组合

---
## 3. Diff IK（可微分 IK）
把 IK 放进 end-to-end training：
- robot policy
- differentiable kinematics

用于：
- RL policy learning
- VLA / robotics foundation models

---
# 四、实际工程怎么选？
结合你做 humanoid + MPC + RL，我直接给你结论：
### ✔ 工程推荐组合：
- **主力：Damped Least Squares IK**
- **冗余：Null-space optimization**
- **复杂约束：QP / MPC formulation**
- **学习增强：Residual IK 或 policy guidance**

---

# 五、一句话总结
- 简单机械臂 → 解析解
- 通用机器人 → Jacobian + DLS
- 带约束系统 → Optimization IK
- 大模型/强化学习 → Differentiable / learned IK

---

