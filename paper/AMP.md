AMP: Adversarial Motion Priors for Stylized Physics-Based Character Control




![[Pasted image 20250609101156.png]]


## Goal-Conditioned Reinforcement learning

![[Pasted image 20250825114547.png]]
![[Pasted image 20250825114639.png]]



## Generative Adversarial Imitation Learning

![[Pasted image 20250825145526.png]]![[Pasted image 20250825145646.png]]


## ADVERSARIAL MOTION PRIOR

![[Pasted image 20250825151806.png]]

	We propose to model the style-reward with a learned discriminator, which we refer to as an adversarial motion prior(AMP), by analogy to the adversarial pose priors that were previously proposed for vision-based pose estimation tasks











# Sigmoid Cross-Entropy Loss 解释

## 1. Sigmoid 函数回顾

Sigmoid 把任意实数压缩到 (0,1) 之间：

$$
\sigma(x) = \frac{1}{1 + e^{-x}}
$$

它常用来表示二分类问题中的概率预测。

---

## 2. 交叉熵（Cross-Entropy）

交叉熵衡量预测分布和真实分布之间的差异。  
在二分类中，真实标签 $y \in \{0,1\}$，预测概率 $\hat{y} \in (0,1)$，交叉熵为：

$$
H(y, \hat{y}) = - \Big[ y \log(\hat{y}) + (1-y)\log(1-\hat{y}) \Big]
$$

---

## 3. Sigmoid Cross-Entropy Loss

如果模型的输出是一个“原始分数” $z$（还没经过 Sigmoid），则先用 Sigmoid 得到预测概率：

$$
\hat{y} = \sigma(z) = \frac{1}{1+e^{-z}}
$$

再代入交叉熵：

$$
L(z, y) = - \Big[ y \log(\sigma(z)) + (1-y)\log(1-\sigma(z)) \Big]
$$

这就是 **Sigmoid Cross-Entropy Loss**。

---

## 4. 数值稳定写法

直接计算可能会因为 $\log(\sigma(z))$ 或 $\log(1-\sigma(z))$ 溢出。  
所以框架（TensorFlow / PyTorch）一般实现为：

$$
L(z, y) = \max(z, 0) - z \cdot y + \log(1+e^{-|z|})
$$

这样可以避免数值不稳定。

---

## 5. 直观理解

- 如果真实标签 $y=1$，loss 惩罚预测概率 $\hat{y}$ 偏小；  
- 如果真实标签 $y=0$，loss 惩罚预测概率 $\hat{y}$ 偏大；  

本质上就是在逼近：

- $z \gg 0 \Rightarrow \hat{y} \approx 1 \Rightarrow loss$ 很小  
- $z \ll 0 \Rightarrow \hat{y} \approx 0 \Rightarrow loss$ 很小  
- 预测错时 loss 会变大

---

## 👉 总结

**Sigmoid Cross-Entropy Loss** 是二分类任务里常用的损失函数：  
它把网络输出 $z$ 通过 Sigmoid 映射成概率，再用交叉熵来衡量预测和真实的差异。










