

---

# 一、核心概念

## 1. 什么是 TCN

TCN（Temporal Convolutional Network）是一种用于**序列建模**的卷积神经网络，用来替代传统的 RNN / LSTM。

核心思想：

> 用卷积（CNN）来建模时间序列，而不是递归（RNN）

---

## 2. TCN 的三大核心结构

### （1）Causal Convolution（因果卷积）

- 只能使用过去信息
    
- 不允许“看到未来”
    

作用：保证时间因果性

---

### （2）Dilated Convolution（空洞卷积）

- 扩大感受野
    
- 指数增长：1, 2, 4, 8...
    

作用：

> 用较少层数捕捉长时间依赖

---

### （3）Residual Connection（残差连接）

- 类似 ResNet
    
- 防止梯度消失
    

---

## 3. TCN vs RNN

|特性|TCN|RNN/LSTM|
|---|---|---|
|并行计算|✅|❌|
|长时依赖|✅（更稳定）|⚠️（理论强，实际弱）|
|训练稳定性|✅|❌|
|速度|快|慢|

---

# 二、TCN 模型结构（代码）

```python
class TCN(nn.Module):
    def __init__(self, input_size, output_size, num_channels, kernel_size, dropout):
        super(TCN, self).__init__()
        self.tcn = TemporalConvNet(input_size, num_channels, kernel_size, dropout=dropout)
        self.linear = nn.Linear(num_channels[-1], output_size)
        self.sig = nn.Sigmoid()

    def forward(self, x):
        output = self.tcn(x.transpose(1, 2)).transpose(1, 2)
        output = self.linear(output)
        return self.sig(output)
```

---

# 三、代码解析

## 1. **init**（结构定义）

作用：定义模型包含哪些层

```python
self.tcn = TemporalConvNet(...)
self.linear = nn.Linear(...)
self.sig = nn.Sigmoid()
```

---

## 2. forward（数据流）

作用：定义数据如何流动

```python
x → transpose → TCN → transpose → Linear → Sigmoid
```

---

## 3. 输入输出维度

输入：

```
(N, L, C)
```

TCN需要：

```
(N, C, L)
```

输出：

```
(N, L, output_size)
```

---

# 四、任务理解（Polyphonic Music）

## 1. 数据形式

- 每个时间步：88维（钢琴键）
    
- 值：0/1（是否按下）
    

---

## 2. 训练目标

```
x(t-历史) → 预测 x(t)
```

代码实现：

```python
x = data[:-1]
y = data[1:]
```

---

# 五、训练流程

## 1. 前向传播

```python
output = model(x)
```

---

## 2. 损失函数（本质）

Binary Cross Entropy：

```
loss = y log(p) + (1-y) log(1-p)
```

---

## 3. 反向传播

```python
loss.backward()
optimizer.step()
```

---

# 六、关键工程技巧

## 1. Gradient Clipping

```python
torch.nn.utils.clip_grad_norm_(model.parameters(), clip)
```

作用：防止梯度爆炸

---

## 2. 学习率衰减

```python
lr /= 10
```

---

## 3. 保存最优模型

```python
torch.save(model, path)
```

推荐改为：

```python
torch.save(model.state_dict(), path)
```

---

# 七、常见问题

## 1. 为什么不用 CrossEntropyLoss？

因为这是：

> 多标签问题（multi-label）

不是分类问题

---

## 2. sigmoid 是否必须？

|任务|是否需要|
|---|---|
|二分类|✅|
|多分类|❌（用 softmax）|
|回归|❌|

---

## 3. 推荐改进

更稳定写法：

```python
criterion = nn.BCEWithLogitsLoss()
```

同时：

```python
去掉 sigmoid
```

---

# 八、在机器人中的应用

TCN 可用于：

## 1. 状态估计

```
state history → TCN → latent state
```

---

## 2. 步态预测

```
历史关节 → 预测 gait phase
```

---

## 3. 控制策略（RL）

```
state history → TCN → action
```

---

# 九、核心总结

TCN 的本质：

> 用卷积代替递归进行时间建模

优势：

- 更稳定
    
- 更易训练
    
- 更强长时记忆
    

---

# 十、一句话总结

TCN = 因果卷积 + 空洞卷积 + 残差结构

用于：

> 高效、稳定的时间序列建模

---