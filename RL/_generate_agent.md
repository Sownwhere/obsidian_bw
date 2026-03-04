


# 1. 获取数据
```python 
multi_agent = isinstance(env, MultiAgentEnvWrapper) # 判断是否是MARL

device = env.device
num_envs = env.num_envs
possible_agents = env.possible_agents if multi_agent else ["agent"]
state_spaces = env.state_spaces if multi_agent else {"agent": env.state_space}
observation_spaces = env.observation_spaces if multi_agent else {"agent": env.observation_space}
action_spaces = env.action_spaces if multi_agent else {"agent": env.action_space}
```

# 2. 生成memory

```python 
# instantiate memory

if cfg["memory"]["memory_size"] < 0:

cfg["memory"]["memory_size"] = cfg["agent"]["rollouts"] # memory_size is the agent's number of rollouts

for agent_id in possible_agents:

memories[agent_id] = memory_class(num_envs=num_envs, device=device, **self._process_cfg(cfg["memory"]))
```



# 3. AMP agent


```python
if agent_class in ["amp"]:
	agent_id = possible_agents[0]
	try:
		amp_observation_space = env.amp_observation_space
	except Exception as e:

	amp_observation_space = observation_spaces[agent_id]	
	agent_cfg = self._component(f"{agent_class}_DEFAULT_CONFIG").copy()	
	agent_cfg.update(self._process_cfg(cfg["agent"]))	
	agent_cfg["state_preprocessor_kwargs"].update({"size": observation_spaces[agent_id], "device": device})	
	agent_cfg["value_preprocessor_kwargs"].update({"size": 1, "device": device})
	agent_cfg["amp_state_preprocessor_kwargs"].update({"size": amp_observation_space, "device": device})
	
	  
	
	motion_dataset = None
	if cfg.get("motion_dataset"):
		motion_dataset_class = cfg["motion_dataset"].get("class")
	
		if not motion_dataset_class:
			raise ValueError(f"No 'class' field defined in 'motion_dataset' cfg")
		del cfg["motion_dataset"]["class"]
	
		motion_dataset = self._component(motion_dataset_class)(
			device=device, **self._process_cfg(cfg.get("motion_dataset", {}))
	
	)
	
	reply_buffer = None
	
	if cfg.get("reply_buffer"):
	
	reply_buffer_class = cfg["reply_buffer"].get("class")
	
	if not reply_buffer_class:
	
	raise ValueError(f"No 'class' field defined in 'reply_buffer' cfg")
	
	del cfg["reply_buffer"]["class"]
	
	reply_buffer = self._component(reply_buffer_class)(
	
	device=device, **self._process_cfg(cfg.get("reply_buffer", {}))
	
	)
	
	  
	
	agent_kwargs = {
	
	"models": models[agent_id],
	
	"memory": memories[agent_id],
	
	"observation_space": observation_spaces[agent_id],
	
	"action_space": action_spaces[agent_id],
	
	"amp_observation_space": amp_observation_space,
	
	"motion_dataset": motion_dataset,
	
	"reply_buffer": reply_buffer,
	
	"collect_reference_motions": lambda num_samples: env.collect_reference_motions(num_samples),

}
```