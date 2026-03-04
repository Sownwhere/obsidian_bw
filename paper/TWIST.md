


We first derive tracking targets—humanoid joint positions and root velocities—by retargeting arbitrary human motions captured by motion capture (MoCap) devices. We then train a single policy with reinforcement learning (RL) in large-scale simulated environments, combined with human motion data. The resulting controller can robustly and accurately track target robot joint positions and root velocities at each timestep while maintaining whole-body balance.


we propose a two-stage teacher-student framework: the teacher policy is trained with privileged access to future motion frames to learn smoother behaviors, and subsequently guides the student policy, which tracks only a single frame.

Offline human motion datasets are usually high-quality and smooth, while the real-time human motions and the real-time retargeting are not that stable and smooth, causing a distribution shift for online teleoperation. Therefore, we collect a small-scale MoCap human dataset (150 clips) using the online MoCap and retargeting settings, combined with 15K offline motion clips as the training set for training the RL controller. Surprisingly, despite only a small set of online motions we use, the controller performs significantly better and more stably on unseen test motions and in real-world teleoperation.



During offline retargeting of human motions, we can ensure high-quality motion data through many iterations of optimization. However, for online retargeting during teleoperation, fast inference is critical, often at the expense of smoothness. We find that jointly optimizing 3D joint positions and orientations helps mitigate this offline-to-online gap, compared to optimizing orientations alone


As the learning objective of the controller is simply motion tracking, tasks requiring force exertion (e.g., lifting a box) rather than reaching target positions represent out-of-distribution scenarios, causing the controller to produce jittery behaviors occasionally. To enable the controller to learn to apply force, we propose to train controllers with large end-effector perturbations, which significantly improves robustness in tasks requiring contact and force.


![[Pasted image 20250530101316.png]]


