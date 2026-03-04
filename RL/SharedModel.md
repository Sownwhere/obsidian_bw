## ✅ SharedModel 是什么？

`SharedModel` 是一个自定义的 PyTorch `nn.Module` 类，它的作用通常是：

> **将一个神经网络骨干（backbone）用于多个输出任务（如 policy 和 value）**。

---

### 🔧 一般结构

根据你最初贴的模型结构，这个类大致像这样：


```
class SharedModel(nn.Module):     def __init__(self, observation_space, action_space, device, ...):         super().__init__()                  # Shared feature extractor         self.net_container = nn.Sequential(             nn.Linear(in_features=2, out_features=32),             nn.ELU(),             nn.Linear(32, 32),             nn.ELU()         )                  # Separate heads         self.policy_layer = nn.Linear(32, 1)  # 输出动作的分布参数或动作本身         self.value_layer = nn.Linear(32, 1)   # 输出 state 的 value 值      def forward(self, x):         feat = self.net_container(x)         policy_out = self.policy_layer(feat)         value_out = self.value_layer(feat)         return policy_out, value_out

```

---

## 🧠 为什么叫 “Shared”？

### ✅ 因为它“共享”了特征提取网络 `net_container`：

```
self.net_container = ...
```

然后将这个共享特征送入两个分支：

|分支|作用|
|---|---|
|`policy_layer`|输出 policy（动作或动作分布）|
|`value_layer`|输出 value（状态值）|

---

## 🧠 用于哪些算法？

`SharedModel` 是典型的 **Actor-Critic** 架构核心组件，常用于：

- **PPO**
    
- **A2C / A3C**
    
- **SAC (带共享 encoder 的版本)**
    
- **IPPO / MAPPO**
    
- 甚至在 AMP 等模仿学习中也可以用到共享 encoder
    

---

## ✅ 举个例子（在你项目中）

假设你配置：


```
"models": {     "policy": {         "class": "SharedModel",         "hidden_units": 64     },     "value": {         "class": "SharedModel",         "hidden_units": 64     } }
```

那你的 `SharedModel` 就被调用两次分别用于 policy/value，但结构其实是相同的，参数不共享，因为是两个实例（前提是 separate=True）。

---

## ❓SharedModel 和 NonSharedModel 有何区别？

|类型|特征|参数共享？|使用场景|
|---|---|---|---|
|`SharedModel`|一个网络干道+两个输出头|是（内部共享）|PPO, A2C 等|
|`NonSharedModel`|policy 和 value 是完全独立的网络|否|某些对策略和值要求不同的算法|
|`SharedModel` x 多 agent|结构相同但每个 agent 拥有自己的模型实例|否（全模型不共享）|IPPO 默认行为|
|`SharedModel` 同一实例给所有 agent 用|是（全模型共享）|Parameter Sharing PPO||

---

## 📌 总结一句话：

> `SharedModel` 是一种在 actor-critic 算法中常用的神经网络结构，它将 policy 和 value 两个任务共享一个特征提取器（backbone），但输出分支独立。