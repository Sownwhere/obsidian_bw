env_isaaclab


```bash
cd ~/RL/LeggedLab
conda activate env_isaaclab

```

```bash
python legged_lab/scripts/train.py --task=bw_flat --logger=tensorboard --num_envs=4096 --headless
```
	

```bash
python legged_lab/scripts/play.py --task=bw_flat --load_run=2025-06-11_18-13-07

```