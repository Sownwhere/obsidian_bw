




| 参数名                                                                                                                                                               | 作用                                                                 |
| ----------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------ |
| **`actor_critic`**                                                                                                                                                | 一个包含 policy 和 value 网络的对象，负责选择动作和估计状态值（V(s)），即 Actor-Critic 架构的核心。 |
| **`num_learning_epochs`**                                                                                                                                         | 每个 iteration 中对 rollout 数据训练多少轮（epochs），相当于数据的重复利用次数。              |
| **`num_mini_batches`**                                                                                                                                            | 每个 epoch 中将 rollout 数据分成多少个 mini-batch（小批量），用于梯度下降。                |
| **`clip_param`**                                                                                                                                                  | PPO中的剪切系数 ε，用于限制策略更新的幅度，避免训练不稳定。通常是0.1~0.3之间。用于计算 surrogate loss：  |
| $$LCLIP(θ)=min⁡(rt(θ)A^t,clip(rt(θ),1−ϵ,1+ϵ)A^t)L^{CLIP}(\theta)$$ $$ = \min(r_t(\theta) \hat{A}_t, \text{clip}(r_t(\theta), 1-\epsilon, 1+\epsilon) \hat{A}_t)$$ |                                                                    |
| **`gamma`**                                                                                                                                                       | 折扣因子 γ，用于奖励的时间折扣，数值越接近 1，考虑未来奖励越多。                                 |
| **`lam`**                                                                                                                                                         | GAE（Generalized Advantage Estimation）中的 λ，用于平衡 bias 和 variance。    |
| **`value_loss_coef`**                                                                                                                                             | 值函数（Critic）的损失项权重（MSE），控制其在总 loss 中的影响大小。                          |
| **`entropy_coef`**                                                                                                                                                | 熵项系数，用于鼓励策略的探索性。值越大，策略越随机。                                         |
| **`learning_rate`**                                                                                                                                               | 优化器的学习率。控制参数更新的步长。                                                 |
| **`max_grad_norm`**                                                                                                                                               | 梯度裁剪的最大范数，用于防止梯度爆炸，提升训练稳定性。                                        |
| **`use_clipped_value_loss`**                                                                                                                                      | 是否对值函数 loss 也使用 clipping，控制 V 的更新不要变化太大。用于提升稳定性。                   |
| **`schedule`**                                                                                                                                                    | 学习率调度方式，常见如 `"fixed"` 固定不变、`"linear"` 线性下降、`"cosine"` 等。           |
| **`desired_kl`**                                                                                                                                                  | PPO 中目标的 KL 散度阈值，用于 early stopping 或动态调整 clip range / 学习率。         |
| **`device`**                                                                                                                                                      | 模型训练使用的设备，比如 `"cpu"` 或 `"cuda"`。                                   |



## 🔁 `num_learning_epochs`

**定义：**  
表示每次从环境中收集完一批 trajectory（通常叫一个 rollout）后，用这批数据训练多少轮（epochs）。

**举个例子：**  
假设你一次 rollout 得到了 `N = 32,000` 个 transition 样本（例如 `obs, action, reward` 等），如果：

```python
num_learning_epochs = 4
```

那么你会用这 32,000 个数据样本进行 **4 次重复训练**，也就是整个训练过程中你重复使用这批数据 4 次。这种方式提升了样本利用率，因为 RL 中采样代价昂贵。

---

## 📦 `num_mini_batches`

**定义：**  
指的是将上面 rollout 的全部数据在每个 epoch 中分成多少个 mini-batch，用于 mini-batch SGD（随机梯度下降）训练。

**继续举例：**  
假设我们仍然有 32,000 条样本，如果：

``` python 
num_mini_batches = 8


```
则每个 epoch 会把 rollout 数据划分为 8 个 mini-batch（每个 mini-batch 大小为 `32,000 / 8 = 4,000`），你对这 8 个小批量逐个进行前向传播、反向传播和优化。


---

## 🧠 综合理解：

这两个参数一起控制了每轮训练的更新强度和计算量：

|参数组合|意义|说明|
|---|---|---|
|`num_learning_epochs = 1`|数据训练一遍|用完即弃，效率低但训练快|
|`num_learning_epochs = 4`|数据训练四遍|提高样本利用率，提高学习效果|
|`num_mini_batches = 1`|整个数据一次训练|内存占用高，训练震荡大|
|`num_mini_batches = 8`|分成小批量训练|更稳定、适合大 batch，但训练稍慢|

---

## ✅ 在实际项目中怎么选？

|情况|推荐设置|
|---|---|
|小模型 + 小 batch|`epochs = 3~5`, `mini_batches = 1~4`|
|大模型 / batch 很大|`epochs = 4~10`, `mini_batches = 8~32`|
|追求稳定训练|增加 `num_mini_batches`，避免过拟合|
|追求训练速度|减小 `num_learning_epochs`，加快迭代|