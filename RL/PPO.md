
# Algorithm

For each iteration do:

- Collect, in a rollout memory, a set of states $s$, actions $a$, rewards $r$, terminated $d_{end}$, truncated $d_{timeout}$, log probabilities $\text{log}p$ and values $V$ on policy using $\pi_\theta$ and $V_{\phi}$  
- Estimate returns $R$ and advantages $A$ using Generalized Advantage Estimation (GAE($\lambda$)) from the collected data $[r, d_{end} \lor d_{timeout}, V]$
- Compute the entropy loss $L_{entropy}$
- Compute the clipped surrogate objective (policy loss) with $ratio$ as the probability ratio between the action under the current policy and the action under the previous policy:  
  $$L_{\pi_\theta}^{clip} = \mathbb{E}[\min(A \cdot ratio, A \cdot \text{clip}(ratio, 1 - c, 1 + c))]$$
- Compute the value loss $L_{V_\phi}$ as the mean squared error (MSE) between the predicted values $V_{predicted}$ and the estimated returns $R$
- Optimize the total loss $L = L_{\pi_\theta}^{clip} - c_1 L_{V_\phi} + c_2 L_{entropy}$





在强化学习中，**“collect data”** 是指从与环境交互中收集一批用于训练的数据样本。你提到的这句：

> Estimate returns $R$ and advantages $A$ using Generalized Advantage Estimation (GAE($\lambda$)) from the collected data $[r, d_{end} \lor d_{timeout}, V]$

中的 **"collected data"** 是指通过 agent（智能体）与环境进行一段时间的交互，按照当前策略 $\pi_\theta$ 收集到的以下信息：

---

### 📦 Collected data 包含哪些内容？

1. **$r$ (reward)**：每个时间步环境返回的奖励。
    
2. **$d_{end}$ (termination flag)**：是否是因为成功或失败导致 episode 结束（比如摔倒、完成任务等）。
    
3. **$d_{timeout}$ (timeout flag)**：是否是因为达到最大步数限制而强制结束。
    
4. **$V$ (value)**：当前策略下预测的状态价值 $V(s)$。
    

---

### 举个例子：

假设你有一个机器人在地面上行走，agent 与环境交互生成如下轨迹：

|状态 $s_t$|动作 $a_t$|奖励 $r_t$|结束标志 $d_{end}$|超时标志 $d_{timeout}$|$V(s_t)$|
|---|---|---|---|---|---|
|$s_0$|$a_0$|0.1|False|False|0.5|
|$s_1$|$a_1$|0.3|False|False|0.6|
|$s_2$|$a_2$|0.5|True|False|0.0|

这些就是一条 collected trajectory 中的 **collected data**，也就是用于 GAE 或其他方法来估计 **returns** $R$ 和 **advantages** $A$ 的基础数据。

---

### 简单理解：

“Collect data” = 在环境中“跑一遍策略”，收集：

- 状态、动作、奖励、
    
- 是否终止、
    
- 预测的价值函数值 $V$。
    

这些数据随后用于策略更新阶段，比如使用 GAE 计算优势函数 $A$。

---

如果你想了解具体怎么用这些数据来算 $A$ 和 $R$，我也可以详细展开 Generalized Advantage Estimation 的公式和实现过程。要吗？










## PPO Update Pseudocode (with Math)

---

### 1. Compute returns and advantages

$$
V' \leftarrow V_\phi(s')
$$

$$
R, A \leftarrow f_{GAE}(r, d_{end}, V, V'_{last})
$$

---

### 2. Sample mini-batches from memory

$$
[s, a, \text{logp}, V, R, A] \leftarrow \text{states, actions, log\_prob, values, returns, advantages}
$$

---

### 3. Learning epochs

For each learning epoch up to **learning\_epochs**:

- For each mini-batch up to **mini\_batches**:

  1. Compute new log prob:

  $$
  \text{logp}' \leftarrow \pi_\theta(s, a)
  $$

  2. Approximate KL divergence:

  $$
  \text{ratio} \leftarrow \text{logp}' - \text{logp}
  $$

  $$
  KL_{divergence} = \frac{1}{N} \sum_{i=1}^N \Big( (e^{\text{ratio}} - 1) - \text{ratio} \Big)
  $$

  - Early stopping with KL divergence:

  $$
  \text{If} \quad KL_{divergence} > KL_{threshold} \quad \text{then BREAK LOOP}
  $$

---

### 4. Compute entropy loss

If entropy computation is enabled:

$$
L_{entropy} = \text{entropy\_loss\_scale} \cdot \frac{1}{N} \sum_{i=1}^N \pi_{\theta, entropy}
$$

Else:

$$
L_{entropy} = 0
$$

---

### 5. Compute policy loss

$$
\text{ratio} = e^{\text{logp}' - \text{logp}}
$$

$$
L_{surrogate} = A \cdot \text{ratio}
$$

$$
L_{clipped\_surrogate} = A \cdot \text{clip}(\text{ratio}, 1 - c, 1 + c) 
\quad (\text{with } c = \text{ratio\_clip})
$$

$$
L_{\pi_\theta} = \frac{1}{N} \sum_{i=1}^N \min(L_{surrogate}, L_{clipped\_surrogate})
$$

---

### 6. Compute value loss

$$
V_{predicted} = V_\phi(s)
$$

If **clip\_predicted\_values** is enabled:

$$
V_{predicted} = V + \text{clip}(V_{predicted} - V, -c, c)
\quad (\text{with } c = \text{value\_clip})
$$

$$
L_{V_\phi} = \text{value\_loss\_scale} \cdot \frac{1}{N} \sum_{i=1}^N (R - V_{predicted})^2
$$

---

### 7. Optimization step

- Reset optimizer:

  ```
  reset optimizer_{θ, ϕ}
  ```

- Compute gradient:

  $$
  \nabla_{\theta, \phi} (L_{\pi_\theta} + L_{entropy} + L_{V_\phi})
  $$

- Clip gradients:

  $$
  \text{clip}(\|\nabla_{\theta, \phi}\|) \quad \text{with grad\_norm\_clip}
  $$

- Optimizer step:

  ```
  step optimizer_{θ, ϕ}
  ```

---

### 8. Update learning rate

If there is a **learning\_rate\_scheduler**:

```
step scheduler_{θ, ϕ}(optimizer_{θ, ϕ})
```