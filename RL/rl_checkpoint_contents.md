# 强化学习训练中的 Checkpoint 内容详解

在强化学习（Reinforcement Learning, RL）训练中，**checkpoint** 文件用于**保存训练过程中模型的状态**，以便：

- 后续恢复训练（resume training）
- 进行推理（inference / evaluation）
- 分析模型行为
- 迁移学习（fine-tune）

---

## ✅ Checkpoint 中通常包含的内容：

### 1. 策略网络（Policy Network）参数

- 保存策略网络（actor）的权重和结构状态。
- 决定 agent 如何根据环境状态选择动作。

```python
checkpoint["policy"] = policy.state_dict()
```

---

### 2. 值函数网络（Value Function / Critic）参数

- 对状态或状态-动作对的“价值”进行估计。
- 用于指导策略优化，特别是 PPO、A2C、DDPG 等算法中。

```python
checkpoint["value"] = value_function.state_dict()
```

---

### 3. 判别器（Discriminator）参数（如使用 AMP 或 GAIL）

- 用于模仿学习或奖励重构（如 Adversarial Motion Priors）。
- 判别器会区分 expert 和 agent 行为。

```python
checkpoint["discriminator"] = discriminator.state_dict()
```

---

### 4. 优化器状态（Optimizers）

- 包括学习率、动量、梯度累积等。
- 关键在于继续训练时可保持学习状态。

```python
checkpoint["optimizer_policy"] = optimizer_policy.state_dict()
checkpoint["optimizer_value"] = optimizer_value.state_dict()
```

---

### 5. 调度器（Scheduler）状态（如果有）

- 如使用学习率调度器（`torch.optim.lr_scheduler`），保存其状态。

```python
checkpoint["scheduler"] = scheduler.state_dict()
```

---

### 6. Replay Buffer（如果是 off-policy）

- 包含环境交互的数据：状态、动作、奖励、下一个状态。
- 对于 DDPG、TD3、SAC 等算法至关重要。

```python
checkpoint["replay_buffer"] = replay_buffer.dump()  # 自定义序列化
```

---

### 7. 当前训练进度信息

- 当前的迭代数或 timestep
- 总 reward、loss 等指标
- 有助于训练恢复和日志对齐

```python
checkpoint["iteration"] = current_iteration
checkpoint["timestep"] = global_step
checkpoint["stats"] = {
    "loss": last_loss,
    "reward": avg_reward,
    ...
}
```

---

### 8. 环境信息（可选）

- 包括环境种子、物理参数、机器人配置等。
- 特别是仿真中（如 MuJoCo、IsaacGym）很重要。

---

## 💡 示例结构：

```python
{
    "policy": actor.state_dict(),
    "value": critic.state_dict(),
    "discriminator": discriminator.state_dict(),
    "optimizer_policy": optimizer_actor.state_dict(),
    "optimizer_value": optimizer_critic.state_dict(),
    "replay_buffer": replay_buffer.dump(),
    "iteration": 100000,
    "timestep": 500000,
    "stats": {"reward": 123.4, "loss": 0.002},
    "env_params": {"mass": 2.0, "gravity": 9.8}
}
```

---

## 📦 保存格式

大多数 RL 框架使用如下方式保存：

```python
torch.save(checkpoint, "checkpoint.pt")
```

---

## 🔄 恢复（load checkpoint）

```python
checkpoint = torch.load("checkpoint.pt")

policy.load_state_dict(checkpoint["policy"])
optimizer.load_state_dict(checkpoint["optimizer_policy"])
# 其余按需加载
```

---

> 💬 如果你告诉我你用的是什么框架（如 `skrl`, `Stable-Baselines3`, `RLlib`, 自定义 PyTorch），我可以给出更具体的 checkpoint 内容和结构。

