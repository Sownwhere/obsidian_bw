### Symbolic Links
```
ln -s ~/RL/IsaacLab/source/isaaclab_tasks/isaaclab_tasks/direct/humanoid_amp ~RL/IsaacLab/

ln -s ~/IsaacLab/scripts/reinforcement_learning/skrl ~/IsaacLab/
```
### Train
```
cd RL/IsaacLab/
./isaaclab.sh -p ~/RL/IsaacLab/skrl/train.py --task Isaac-Bw-AMP-Walk-Direct-v0 --headless

./isaaclab.sh -p ~/RL/IsaacLab/skrl/train.py --task exo
```
or
```
./isaaclab.sh -p ~/IsaacLab/skrl/train.py --task Isaac-G1-AMP-Dance-Direct-v0 --headless
```
### Eval
```
./isaaclab.sh -p ~/RL/IsaacLab/skrl/play.py --task Isaac-Bw-AMP-Walk-Direct-v0 --num_envs 32 
```
### TensorBoard
```
./isaaclab.sh -p -m tensorboard.main --logdir logs/skrl/
```

The parameters of the code in this repository have not been fine-tuned. Currently, the walk performance is acceptable, but the dance performance is quite poor. Due to personal bussiness, I will not begin to debug until summer.
### Dataset & URDF
~~The dataset and URDF files are from [Hugging Face Unitree Robotics](https://huggingface.co/datasets/unitreerobotics/LAFAN1_Retargeting_Dataset).~~

**Note**: The original dataset and URDF files from [Unitree Robotics](https://huggingface.co/datasets/unitreerobotics/LAFAN1_Retargeting_Dataset) have been removed by the official source.

If you're still looking for the dataset, a third-party mirror is currently available here:  
[lvhaidong/LAFAN1_Retargeting_Dataset](https://huggingface.co/datasets/lvhaidong/LAFAN1_Retargeting_Dataset)

*Use at your own discretion, as it is not officially maintained. qwq*

### Others
Video: [Bilibili](https://www.bilibili.com/video/BV19cRvYhEL8/?vd_source=5159ce41348cd4fd3d83ef9169dc8dbc)

DeepWiki: [humanoid_amp](https://deepwiki.com/linden713/humanoid_amp)

**Contributions**, **discussions**, and stars are all welcome! ❥(^_-)