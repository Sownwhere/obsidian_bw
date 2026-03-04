## Algorithm

**For each iteration do:**  
	**For each agent do:**

-  Collect, in a rollout memory, a set of states $s$, actions $a$, rewards $r$, terminated $d_{end}$, truncated $d_{timeout}$, log probabilities $\text{logp}$ and values $V$ on policy using $\pi_\theta$ and $V_\phi$
- Estimate returns $R$ and advantages $A$ using Generalized Advantage Estimation (GAE($\lambda$)) from the collected data $[r, d_{end} \vee d_{timeout}, V]$
- Compute the entropy loss $L_{entropy}$
- Compute the clipped surrogate objective (policy loss) with $ratio$ as the probability ratio between the action under the current policy and the action under the previous policy:  
  $$L_{\pi_\theta}^{clip} = \mathbb{E}[\min(A \cdot ratio, A \cdot \text{clip}(ratio, 1 - c, 1 + c))]$$
- Compute the value loss $L_{V_\phi}$ as the mean squared error (MSE) between the predicted values $V_{predicted}$ and the estimated returns $R$
- Optimize the total loss  
  $$L = L_{\pi_\theta}^{clip} - c_1 L_{V_\phi} + c_2 L_{entropy}$$
