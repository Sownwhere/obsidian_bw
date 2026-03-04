

## AMP Update Algorithm

```python
_update(...)
```

### Update dataset of reference motions
Collect reference motions of size `amp_batch_size` → `append(M)`

### Compute combined rewards
$$
r_D \leftarrow -\log(\max(1 - \hat{y}(D_\psi(s_{AMP})), 10^{-4})) \quad \text{with} \quad \hat{y}(x) = \frac{1}{1 + e^{-x}}
$$

$$
r \leftarrow task\_reward\_weight \cdot r + style\_reward\_weight \cdot discriminator\_reward\_scale \cdot r_D
$$

### Compute returns and advantages
$$
R, A \leftarrow f_{GAE}(r, d_{end} \vee d_{timeout}, V, V')
$$

### Sample mini-batches from memory
$$
[s, a, \log p, V, R, A, s_{AMP}] \leftarrow \text{states, actions, log\_prob, values, returns, advantages, AMP states}
$$

From memory buffers:
- $s_{AMP}^{M}$ ← AMP states from $M$
- If $B$ is not empty: $s_{AMP}^{B}$ ← AMP states from $B$
- Else: $s_{AMP}^{B}$ ← ∅[[PPO]]

---

### Learning epochs
For each learning epoch up to `learning_epochs`:

#### Mini-batch loop
For each mini-batch $[s, a, \log p, V, R, A, s_{AMP}^{B}, s_{AMP}^{M}]$ up to `mini_batches`:

$$logp' ← π_θ(s, a) $$

#### Compute entropy loss
If entropy computation is enabled:
$$
L_{entropy} \leftarrow -entropy\_loss\_scale \cdot \frac{1}{N} \sum_{i=1}^N \pi_\theta^{entropy}
$$

Else:
$$
L_{entropy} \leftarrow 0
$$

#### Compute policy loss
$$
ratio \leftarrow \exp(\log p' - \log p)
$$

$$
L_{surrogate} \leftarrow A \cdot ratio
$$

$$
L_{clipped\_surrogate} \leftarrow A \cdot \text{clip}(ratio, 1 - c, 1 + c) \quad \text{(with } c = ratio\_clip\text{)}
$$

$$
L_{\pi_\theta}^{clip} \leftarrow -\frac{1}{N} \sum \min(L_{surrogate}, L_{clipped\_surrogate})
$$

#### Compute value loss
$$
V_{predicted} \leftarrow V_\phi(s)
$$

If value clipping is enabled:
$$
V_{predicted} \leftarrow V + \text{clip}(V_{predicted} - V, -c, c) \quad \text{(with } c = value\_clip\text{)}
$$

$$
L_{V_\phi} \leftarrow value\_loss\_scale \cdot \frac{1}{N} \sum_{i=1}^N (R - V_{predicted})^2
$$

---

### Compute discriminator loss
```python
logit_AMP^M ← D_ψ(s_AMP^M)  # size: discriminator_batch_size
logit_AMP^B ← D_ψ(s_AMP^B)  # size: discriminator_batch_size
logit_AMP^s ← D_ψ(s_AMP)    # size: discriminator_batch_size
```

#### Discriminator prediction loss
$$
L_{D_\psi} \leftarrow \frac{1}{2} (BCE(logit_{AMP}^B, 0) + BCE(logit_{AMP}^M, 1))
$$

Binary Cross Entropy:
$$
BCE(x, y) = -\frac{1}{N} \sum_{i=1}^N y \log(\hat{y}) + (1 - y) \log(1 - \hat{y}), \quad \hat{y} = \frac{1}{1 + e^{-x}}
$$

#### Discriminator logit regularization
$$
L_{D_\psi} \leftarrow L_{D_\psi} + discriminator\_logit\_regularization\_scale \cdot \sum_{i=1}^N \text{flatten}(\psi_{w}[:-1])^2
$$

#### Discriminator gradient penalty
$$
L_{D_\psi} \leftarrow L_{D_\psi} + discriminator\_gradient\_penalty\_scale \cdot \frac{1}{N} \sum_{i=1}^N \sum (\nabla_\psi logit_{AMP}^M)^2
$$

#### Discriminator weight decay
$$
L_{D_\psi} \leftarrow L_{D_\psi} + discriminator\_weight\_decay\_scale \cdot \sum_{i=1}^N \text{flatten}(\psi_{w})^2
$$

---

### Optimizer step
```python
reset optimizer_{θ, φ, ψ}
∇_{θ, φ, ψ} ← ∇(L_πθ^{clip} + L_{entropy} + L_{Vφ} + L_{Dψ})
clip(∇_{θ, φ, ψ}, grad_norm_clip)
step optimizer_{θ, φ, ψ}
```

If there is a `learning_rate_scheduler`:
```python
step scheduler_{θ, φ, ψ}(optimizer_{θ, φ, ψ})
```

---

### Update AMP replay buffer

$$s_{AMP} → append(B)$$





在 AMP（Adversarial Motion Prior）中，**replay buffer** 是用来存储训练数据（经验）的关键组件，目的是帮助训练过程中的 **discriminator（判别器）** 和 **policy（策略网络）** 更高效、更稳定。

---

### ✅ AMP 中的 Replay Buffer 是什么？

在 AMP 中，Replay Buffer 存储的是来自两个来源的数据：

1. **专家动作片段（Expert motion clips）**：也叫 reference motion，来自 Motion Capture 数据或其他高质量动作。
    
2. **agent 执行产生的动作片段（Policy-generated motion clips）**：当前策略执行出来的片段。
    

这些数据被用于训练 **判别器（Discriminator）** 来区分 agent 的动作和专家动作。
    

---

### 🗃️ Replay Buffer 的结构

通常包含：

|类型|内容|用途|
|---|---|---|
|Expert buffer|多个来自 expert 的动作片段|判别器正样本|
|Agent buffer|策略执行时采样的动作片段|判别器负样本|

---

### 🔁 为什么要用 Replay Buffer？

1. **数据高效再利用**：避免每次都重新采样，提高训练效率。
    
2. **样本多样性**：增加训练稳定性，避免判别器和策略过拟合到当前数据分布。
    
3. **异步更新**：判别器和策略可以不必完全同步更新，有助于平稳训练。
    

---

### 🚀 总结一句话：

> **AMP 的 replay buffer 是一个用于存储专家和 agent 动作片段的数据池，目的是辅助判别器训练，使策略学习到更自然、更真实的动作。**

如果你是在用如 `skrl`、`IsaacGymEnvs` 等框架，也可以贴出配置文件我帮你具体解释它的结构和用法。