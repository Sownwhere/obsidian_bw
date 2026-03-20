```bash

	ssh -X  gdp@192.168.3.10

cd ~/bowen/humanoid_gym/humanoid/scripts/

conda activate Bw_humanoid_gym

python train.py --task=bw_ppo --headless


```


## 使用方法

要实时监控训练，您需要：

1. **启动训练**：运行训练脚本，系统会自动创建 TensorBoard 日志
2. **启动 TensorBoard**：在另一个终端中运行：
    

```bash
cd ~/RL/humanoid-gym
tensorboard --logdir=logs/Bw_ppo --port=6010

```

3. **实时查看**：在浏览器中打开 TensorBoard 界面，数据会每10秒自动更新



```bash
python train.py --task=bw_ppo --resume --load_run=May28_10-35-37_  --checkpoint=300 --headless

```