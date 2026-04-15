官方提供的模型结果


























## 对全数据进行训练:

```
python train.py --config_path configs.default_config --data_dir ../train_data --device cuda
```

![[Pasted image 20260413165323.png]]

```

python example_save.py   --config_path configs.default_config.py   --model_path checkpoints/final_20260413_163615.pth  --device cuda   --out_csv merged_prediction_all_input_1.csv
```


![[Pasted image 20260413165011.png]]



```
python train.py   --config_path configs.default_config  --data_dir ../train_data   --device cuda   --batch_size 16 --learning_rate 0.0004 --early_stopping 30

```


## 实验：输入hip incoder 
### 实验1
![[Pasted image 20260413165829.png]]


### 实验二：
训练结果：
```
python train.py --config_path configs.hip_encoder_only_config --data_dir ../train_data --device cuda --seed 42

```
![[Pasted image 20260413173457.png]]

验证结果：
![[Pasted image 20260413174407.png]]

### 实验三：
```
python train.py   --config_path configs.hip_encoder_only_config   --data_dir ../train_data   --device cuda   --batch_size 4
```
![[Pasted image 20260414134046.png]]
```
python example_save.py   --config_path configs.hip_encoder_only_config   --model_path checkpoints/final_20260413_181202   --device cuda   --out_csv prediction_0413_181202.csv
```

![[Pasted image 20260414134538.png]]

![[Pasted image 20260414163609.png]]python ros2_publish_merged_csv.py   --csv_path prediction_0413_181202.csv   --topic_prefix /exo   --rate 100   --loop

### 实验四：
python train.py   --config_path configs.hip_encoder_only_config   --data_dir ../train_data   --device cuda   --batch_size 16 --learning_rate 0.0004

![[Pasted image 20260414191607.png]]

```
python example_save.py   --config_path configs.hip_encoder_only_config   --model_path checkpoints/final_20260414_171401.pth   --device cuda   --out_csv prediction_final_20260414_171401.csv
```

![[Pasted image 20260415091214.png]]


```
python ros2_publish_merged_csv.py   --csv_path prediction_final_20260414_171401.csv   --topic_prefix /exo   --rate 1000   --loop
```



### 实验五：
```
python train.py   --config_path configs.hip_encoder_only_config   --data_dir ../train_data   --device cuda   --batch_size 16 --learning_rate 0.0004 --early_stopping 30
```


![[Pasted image 20260415092404.png]]

```
python example_save.py   --config_path configs.hip_encoder_only_config   --model_path checkpoints/final_20260414_172524.pth   --device cuda   --out_csv prediction_final_20260414_172524.csv
```

![[Pasted image 20260415092603.png]]



## 实验: 输入 为imu 和 encoder

