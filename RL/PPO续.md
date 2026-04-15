PPO 分类
	PPO penalty 和  PPO clip 
  $$L_{\pi_\theta}^{clip} = \mathbb{E}[\min(A \cdot ratio, A \cdot \text{clip}(ratio, 1 - c, 1 + c))]$$


TRPO： 带不等式约束的最大化问题，计算复杂开销高
PPO penalty 将TRPO 的硬约束更改为 惩罚性质的软约束
PPO clip： clipped surrogate  Objective function


