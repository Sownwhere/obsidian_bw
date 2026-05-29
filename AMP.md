AMP 里 **Discriminator 的更新**，本质就是一个二分类训练：

> 给它一批 reference motion，让它输出接近 1；  
> 给它一批 policy 生成的 motion，让它输出接近 0。

也就是让判别器学会区分：

```text
真实专家动作 / motion dataset
vs
当前 policy 生成的动作
```

---

## 1. Discriminator 的输入是什么？

在 AMP 里，Discriminator 一般不直接看完整 observation，而是看 **motion state**。

例如人形机器人可能输入：

```text
root height
root rotation
root velocity
joint positions
joint velocities
end-effector positions
```

也可以是两帧之间的状态：

```text
s_t, s_{t+1}
```

因为 AMP 关心的是“运动模式”，不是单独某一帧姿态。

所以可以记作：

```text
m = motion_feature(s_t, s_{t+1})
```

---

## 2. Discriminator 要判断什么？

Discriminator 输出一个分数：

```text
D(m)
```

含义是：

```text
D(m) 越接近 1，越像 reference motion
D(m) 越接近 0，越像 policy motion
```

所以训练目标是：

```text
reference motion -> label = 1
policy motion    -> label = 0
```

---

## 3. Discriminator loss

最常见的是二分类交叉熵 loss：

L_D = -\mathbb{E}_{m \sim M_{ref}}[\log D(m)] - \mathbb{E}_{m \sim M_{policy}}[\log(1-D(m))]

解释一下：

第一项：

```text
- log D(reference)
```

如果 reference motion 的 D 输出接近 1，loss 小。

第二项：

```text
- log(1 - D(policy))
```

如果 policy motion 的 D 输出接近 0，loss 小。

---

## 4. 更新流程

一次完整的 Discriminator 更新大概是这样：

```python
# 1. 从 motion dataset 采样真实动作
real_motion = sample_reference_motion()

# 2. 从 replay buffer 或 rollout buffer 采样 policy 生成的动作
fake_motion = sample_policy_motion()

# 3. 输入 discriminator
real_logits = discriminator(real_motion)
fake_logits = discriminator(fake_motion)

# 4. 计算二分类 loss
real_loss = BCEWithLogitsLoss(real_logits, torch.ones_like(real_logits))
fake_loss = BCEWithLogitsLoss(fake_logits, torch.zeros_like(fake_logits))

disc_loss = real_loss + fake_loss

# 5. 反向传播更新 discriminator
disc_optimizer.zero_grad()
disc_loss.backward()
disc_optimizer.step()
```

注意：实际代码里通常用 `BCEWithLogitsLoss`，也就是说 discriminator 最后一层输出 **logits**，不手动加 sigmoid。

---

## 5. 为什么 policy motion 要 detach？

更新 Discriminator 的时候，只更新 Discriminator，不更新 Policy。

所以 policy 生成的数据要当成普通样本，不让梯度传回 policy。

例如：

```python
fake_motion = fake_motion.detach()
```

否则 Discriminator 的 loss 会错误地影响 policy 网络。

正确理解是：

```text
更新 Discriminator：
只让 Discriminator 学会判断真假

更新 Policy：
再用 Discriminator 给出的 reward 去优化 policy
```

这两个阶段要分开。

---

## 6. 加 gradient penalty

AMP 里经常会加一个 regularization，防止 Discriminator 过强或者梯度爆炸。

常见形式类似：

```text
gradient penalty
```

它的作用是让 Discriminator 对输入不要太敏感。

AMP 论文里常用的是对 reference motion 加 gradient penalty：

```text
L_grad = λ * ||∇D(m_ref)||²
```

所以总 loss 变成：

```text
L_total = L_D + L_grad
```

代码形式大概是：

```python
real_motion.requires_grad_(True)

real_logits = discriminator(real_motion)

grad = torch.autograd.grad(
    outputs=real_logits,
    inputs=real_motion,
    grad_outputs=torch.ones_like(real_logits),
    create_graph=True,
    retain_graph=True,
    only_inputs=True
)[0]

grad_penalty = grad.pow(2).sum(dim=-1).mean()

disc_loss = real_loss + fake_loss + lambda_gp * grad_penalty
```

---

## 7. Discriminator 更新几次？

常见做法是：

```text
每次 PPO update 前后，更新 discriminator 若干次
```

比如：

```text
policy rollout 一批数据
更新 discriminator 1~5 次
用 discriminator 计算 AMP reward
用 PPO 更新 policy
```

如果 Discriminator 学太快，会出现问题：

```text
D(reference) ≈ 1
D(policy) ≈ 0
```

这时 policy 得到的 AMP reward 可能很小，学不到东西。

如果 Discriminator 太弱，也会有问题：

```text
D(reference) ≈ 0.5
D(policy) ≈ 0.5
```

这时 AMP reward 没有区分能力。

比较健康的状态是：

```text
D(reference) 高于 D(policy)
但不要完全碾压 policy
```

---

## 8. Discriminator 更新和 Policy 更新的区别

你可以这样记：

```text
Discriminator 更新：
让 D 更会分辨 reference 和 policy

Policy 更新：
让 policy 产生的动作更像 reference，从而骗过 D
```

对应关系：

```text
D 想让：
D(reference) -> 1
D(policy) -> 0

Policy 想让：
D(policy) -> 1
```

这就是对抗关系。

---

## 9. AMP reward 怎么从 Discriminator 来？

更新完 Discriminator 后，用它给 policy motion 打分。

常见 reward 形式之一是：

```text
r_amp = -log(1 - D(policy_motion))
```

如果 D 认为 policy motion 很像真实动作：

```text
D(policy_motion) 高
1 - D(policy_motion) 小
-log(1 - D) 大
reward 高
```

所以 policy 会被鼓励产生更像 reference 的动作。

---

## 10. 一个完整训练循环

```python
for iteration in range(num_iterations):

    # =========================
    # 1. Policy rollout
    # =========================
    rollout_data = collect_rollout(env, policy)

    policy_motion = rollout_data["amp_obs"]

    # =========================
    # 2. Sample reference motion
    # =========================
    reference_motion = motion_dataset.sample(batch_size)

    # =========================
    # 3. Update Discriminator
    # =========================
    for _ in range(num_disc_updates):

        real_motion = motion_dataset.sample(batch_size)
        fake_motion = replay_buffer.sample(batch_size)

        fake_motion = fake_motion.detach()

        real_logits = discriminator(real_motion)
        fake_logits = discriminator(fake_motion)

        real_loss = bce_loss(real_logits, torch.ones_like(real_logits))
        fake_loss = bce_loss(fake_logits, torch.zeros_like(fake_logits))

        disc_loss = real_loss + fake_loss

        disc_optimizer.zero_grad()
        disc_loss.backward()
        disc_optimizer.step()

    # =========================
    # 4. Compute AMP reward
    # =========================
    with torch.no_grad():
        policy_logits = discriminator(policy_motion)
        policy_prob = torch.sigmoid(policy_logits)
        amp_reward = -torch.log(1 - policy_prob + 1e-6)

    # =========================
    # 5. PPO update policy
    # =========================
    total_reward = task_reward + amp_weight * amp_reward

    ppo_update(policy, value_function, total_reward, rollout_data)
```

---

## 11. 面试里可以这样说

你可以这样回答：

> In AMP, the discriminator is updated as a binary classifier. We sample motion features from the reference motion dataset as positive examples and motion features generated by the current policy as negative examples. The discriminator is trained with a binary cross-entropy loss, often with gradient penalty regularization. During discriminator update, the policy samples are detached so gradients only update the discriminator. After that, the discriminator output is converted into an AMP reward, which is combined with task rewards and used by PPO to update the policy.

中文意思是：

> AMP 中的 Discriminator 被当作二分类器训练。reference motion 是正样本，policy 生成的 motion 是负样本。使用二分类交叉熵 loss 更新 Discriminator，并经常加入 gradient penalty。更新 Discriminator 时，policy 生成的数据要 detach，避免梯度传回 policy。更新完成后，Discriminator 的输出会被转换成 AMP reward，再和 task reward 一起用于 PPO 更新策略。

---

## 你最需要记住的一句话

**Discriminator 的更新就是：reference motion 标 1，policy motion 标 0，用 BCE loss 训练一个二分类器；更新它时不更新 policy，之后再把它的输出变成 AMP reward 去训练 policy。**




因为 **Discriminator 同时有两个学习目标**：

1. 看到真实 reference motion，要判断成 **real / 1**
    
2. 看到 policy 生成的 motion，要判断成 **fake / 0**
    

所以总损失要把这两部分都算进去：

```python
disc_loss = real_loss + fake_loss
```

---

## 1. real_loss 在惩罚什么？

```python
real_loss = BCEWithLogitsLoss(real_logits, torch.ones_like(real_logits))
```

意思是：

> 真实动作的 label 是 1，希望 Discriminator 对真实动作输出高分。

如果真实动作输入进去，Discriminator 却输出很低，比如接近 0，那么 `real_loss` 就会很大。

它逼着网络学会：

```text
D(real_motion) -> 1
```

---

## 2. fake_loss 在惩罚什么？

```python
fake_loss = BCEWithLogitsLoss(fake_logits, torch.zeros_like(fake_logits))
```

意思是：

> policy 生成的动作 label 是 0，希望 Discriminator 对假动作输出低分。

如果 policy motion 输入进去，Discriminator 却输出很高，比如接近 1，那么 `fake_loss` 就会很大。

它逼着网络学会：

```text
D(fake_motion) -> 0
```

---

## 3. 为什么要相加？

因为 Discriminator 不是只学会“识别真”，也不是只学会“识别假”。

它要同时学会：

```text
真实的像真
假的像假
```

所以总目标就是：

```text
总错误 = 判断真实样本的错误 + 判断假样本的错误
```

也就是：

```python
disc_loss = real_loss + fake_loss
```

如果你只用 `real_loss`，Discriminator 只会被训练成：所有东西都输出 1。

如果你只用 `fake_loss`，Discriminator 只会被训练成：所有东西都输出 0。

这两个都没意义。

---

## 4. 用数学形式看

BCE 的二分类损失是：

L_D = -\mathbb{E}_{m \sim M_{ref}}[\log D(m)] - \mathbb{E}_{m \sim M_{policy}}[\log(1-D(m))]

第一项对应：

```python
real_loss
```

第二项对应：

```python
fake_loss
```

所以代码里就是：

```python
disc_loss = real_loss + fake_loss
```

---

## 5. 更直观的例子

假设一个 batch 里：

```text
real_motion 有 64 条
fake_motion 有 64 条
```

Discriminator 要完成的任务其实是一个 128 条数据的二分类问题：

```text
前 64 条 label = 1
后 64 条 label = 0
```

你可以把代码写成两种形式。

### 写法 1：分开算

```python
real_loss = bce(real_logits, ones)
fake_loss = bce(fake_logits, zeros)
disc_loss = real_loss + fake_loss
```

### 写法 2：拼起来算

```python
logits = torch.cat([real_logits, fake_logits], dim=0)
labels = torch.cat([
    torch.ones_like(real_logits),
    torch.zeros_like(fake_logits)
], dim=0)

disc_loss = bce(logits, labels)
```

本质是一样的。

---

## 6. 为什么不是 `real_loss - fake_loss`？

这是很多人会疑惑的点。

因为 Discriminator 自己的目标不是“让 fake loss 变大”，而是要**正确分类 fake 为 0**。

对于 fake 样本，正确分类的方式就是让：

```text
D(fake_motion) -> 0
```

而 `fake_loss` 本身已经在惩罚 “D(fake_motion) 不接近 0”。

所以要最小化：

```python
real_loss + fake_loss
```

不是相减。

真正想让 fake motion 被判断成 real 的，是 **Policy**，不是 Discriminator。

也就是说：

```text
Discriminator 更新：
D(real) -> 1
D(fake) -> 0

Policy 更新：
D(fake) -> 1
```

这两个优化目标是相反的。

---

一句话记住：

**`real_loss + fake_loss` 是因为 Discriminator 要同时减少“把真看错”和“把假看错”这两种错误。**