	
```bash
cd ~/RL/humanoid-gym/humanoid/scripts
conda activate humanoid_gym
export LD_LIBRARY_PATH=/home/bowen/anaconda3/pkgs/python-3.8.20-he870216_0/lib:$LD_LIBRARY_PATH
export LD_PRELOAD=/usr/lib/x86_64-linux-gnu/libstdc++.so.6



```


```bash 
python train.py --task=bw_ppo --headless
```

```bash

python scripts/play.py --task=bw_ppo --run_name v1 --load_run=

```



```bash
cd exoskeleton/exoskeleton_rl/humanoid
conda activate myenv
export LD_LIBRARY_PATH=/home/bowen/anaconda3/pkgs/python-3.8.20-he870216_0/lib:$LD_LIBRARY_PATH
export LD_PRELOAD=/usr/lib/x86_64-linux-gnu/libstdc++.so.6
```

