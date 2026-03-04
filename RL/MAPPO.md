
# MAPPO Update Pseudocode (with Math)



### Update dataset of reference motions
	Collect reference motions of size amp_batch_size → append(M)


 Compute combined rewards
$$
r_D \leftarrow -\log(\max(1 - \hat{y}(D_\psi(s_{AMP})), 10^{-4})) \quad \text{with} \quad \hat{y}(x) = \frac{1}{1 + e^{-x}}
$$

$$
r \leftarrow task\_reward\_weight \cdot r + style\_reward\_weight \cdot discriminator\_reward\_scale \cdot r_D
$$

### Compute AMP returns and advantages
$$
R, A \leftarrow f_{GAE}(r, d_{end} \vee d_{timeout}, V, V')
$$


### AMP Sample mini-batches from memory
$$
[s, a, \log p, V, R, A, s_{AMP}] \leftarrow \text{states, actions, log\_prob, values, returns, advantages, AMP states}
$$

From memory buffers:
- $s_{AMP}^{M}$ ← AMP states from $M$
- If $B$ is not empty: $s_{AMP}^{B}$ ← AMP states from $B$
- Else: $s_{AMP}^{B}$ ← ∅ PPO

### PPO Compute returns and advantages

$$
V' \leftarrow V_\phi(s')
$$

$$
R, A \leftarrow f_{GAE}(r, d_{end}, V, V'_{last})
$$

---

###  PPO Sample mini-batches from memory

$$
[s, a, \text{logp}, V, R, A] \leftarrow \text{states, actions, log\_prob, values, returns, advantages}
$$


### if agent class == [[PPO]]
For each learning epoch up to **learning\_epochs**:
	For each mini-batch up to **mini\_batches**:
		Compute new log prob
		Approximate KL divergence
		Early stopping with KL divergence
		Compute entropy loss
		Compute policy loss
		Compute value loss
		Optimization step
	  Update learning rate

### if agent class == [[RL/AMP|AMP]]
For each learning epoch up to `learning_epochs`:
	For each mini-batch $[s, a, \log p, V, R, A, s_{AMP}^{B}, s_{AMP}^{M}]$ up to `mini_batches`:
		$logp' ← π_θ(s, a)$
		 1.Compute entropy loss
		 2.Compute policy loss
		 3.Compute value loss
		 4.Compute discriminator loss
		 5.Discriminator prediction loss
		 6.Discriminator logit regularization
		 7.Discriminator gradient penalty
		 8.Discriminator weight decay
		 9.Optimization step
	  Update learning rate
Update AMP replay buffer


# 总结 解决MAPPO（Multi-Agent PPO）问题技巧

B因此，MARL 领域几乎已经达成共识：**与 on-policy 算法（例如 PPO[3]）相比，在计算资源有限的情况下，off-policy 算法（例如 MADDPG[5]，QMix[6]）因其更高的采样效率更适合用来训练智能体，并且也演化出一系列解决某些具体问题（domain-specific）的 SOTA 算法（例如 SAD[9]，RODE[7]）。**


但是，来自清华大学与 UC 伯克利的研究者在一篇论文中针对这一传统认知提出了不同的观点：MARL 算法需要综合考虑数据样本效率（sample efficiency）和算法运行效率（wall-clock runtime efficiency）。**在有限计算资源的条件下，与 off-policy 算法相比，on-policy 算法 --MAPPO（Multi-Agent PPO）具有显著高的算法运行效率和与之相当（甚至更高）的数据样本效**