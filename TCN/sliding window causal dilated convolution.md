下面给你整理成一份**结构清晰、可以直接保存的 `.md` 笔记**（偏你现在做外骨骼 / TCN 的工程视角）：

---

# TCN vs Sliding Window Dataset 解析（外骨骼时序建模）

## 1. 问题背景

在传统时间序列建模中，常使用 **sliding window** 构建数据集：

- 手动切分时间序列
- 每个样本是一个固定长度窗口

但在 TCN（Temporal Convolutional Network）中：

> ❗ 不需要显式构造 sliding window  
> ❗ 整段 trial 直接输入模型

---

## 2. 数据输入形式

训练数据：

```

x: [num_trials, C_in, max_seq_len]  
y: [num_trials, C_out, max_seq_len]

```

含义：

- 每个 trial = 一整段时间序列（例如完整步态）
- 没有窗口切分
- 输入输出是 **时间对齐的 sequence-to-sequence**

---

## 3. 传统 Sliding Window 方法

### 数据构造方式

原始序列：
```

[x1 x2 x3 x4 x5 x6]

```

切分为：
```

[x1 x2 x3]  
[x2 x3 x4]  
[x3 x4 x5]  
...

```

### 特点

优点：
- 简单直观
- 易于控制窗口长度

缺点：
- 数据重复严重（overlap）
- 数据量膨胀
- 只能建模固定长度依赖
- 破坏长时间结构（例如 gait cycle）

---

## 4. TCN 的核心思想

### 4.1 Causal Convolution（因果卷积）

```

y_t 只依赖 x_{<=t}

```

特点：
- 不使用未来信息
- 符合真实控制系统（外骨骼 / 机器人）

---

### 4.2 Dilated Convolution（空洞卷积）

普通卷积（kernel=3）：
```

[t-2, t-1, t]

```

dilation=2：
```

[t-4, t-2, t]

```

dilation=4：
```

[t-8, t-4, t]

```

---

### 4.3 Receptive Field（感受野）

TCN 通过多层 dilated convolution：

> ✅ 自动形成“时间窗口”  
> ✅ 无需手动切分

---

## 5. 本质区别

| 方法 | 窗口来源 |
|------|--------|
| Sliding Window | 人工定义 |
| TCN | 模型结构（receptive field） |

👉 关键思想：

> **窗口设计从“数据工程问题” → 变成“模型设计问题”**

---

## 6. 在外骨骼任务中的意义

你的任务：

- Joint moment estimation
- Gait modeling
- IMU + encoder → torque

### 数据特点

- 强时间依赖
- 存在周期结构（gait cycle）
- 序列长度不固定

---

### Sliding Window 的问题

- 固定窗口可能截断 gait cycle
- 无法捕捉长依赖（heel strike → push-off）
- 需要人工调 window size

---

### TCN 的优势

- 自动学习 temporal dependency
- 同时建模：
  - short-term（瞬时动态）
  - long-term（步态周期）
- 支持任意长度序列
- 更适合 real-time control

---

## 7. 实际训练过程（工程视角）

```

for each trial:  
input = sensor sequence (IMU / encoder)  
output = joint moment (每个时刻)

```

👉 本质：

> **Sequence-to-sequence regression（逐时间步预测）**

---

## 8. Receptive Field 计算（非常关键）

TCN 实际“窗口大小” = receptive field

近似公式：

```

receptive_field ≈ (kernel_size - 1) * (2^num_layers - 1) + 1

```

---

### 举例

假设：

- kernel_size = 3
- num_layers = 6

```

RF ≈ (3-1)*(2^6 -1)+1  
= 2 * 63 + 1  
= 127

```

---

## 9. 为什么这个很重要？

你的数据：

- 采样率：200 Hz
- gait cycle ≈ 1s → 200 steps

👉 要求：

```

receptive_field >= 200

```

否则：

❌ 模型看不到完整步态周期  
❌ 学不到关键动力学结构  

---

## 10. 核心总结

### 一句话理解

> ❗ TCN 不是没有窗口  
> 👉 而是“窗口由网络自动学习”

---

### 更深一层

> ❗ 你把问题从：
> - “怎么切数据”
>
> 变成：
>
> - “模型能看多远（receptive field）”

---

## 11. 实战建议（针对你当前项目）

### 必做检查

- [ ] 当前 TCN 的 receptive field 多大？
- [ ] 是否 ≥ gait cycle？
- [ ] 是否覆盖多个 phase？

---

### 可优化方向

1. 增大 receptive field
   - 增加 layer
   - 增加 dilation

2. 多尺度建模
   - small RF → 局部动态
   - large RF → gait cycle

3. 进一步升级（进阶）
   - TCN + Attention
   - Transformer encoder
   - Temporal Fusion

---

## 12. 最关键一句（必须记住）

> **Sliding window 是“人为裁剪时间”**  
> **TCN 是“让模型自己理解时间”**
