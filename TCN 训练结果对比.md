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

### 实验一： 
```
python train.py   --config_path configs.hip_encoder_imu_config   --data_dir ../train_data   --device cuda   --batch_size 16 --learning_rate 0.0004 --early_stopping 30

```

![[Pasted image 20260415165730.png]]




python train_both_legs.py   --config_path configs.hip_encoder_only_config   --data_dir ../train_data   --device cuda --batch_size 16 --learning_rate 0.0008 --early_stopping 30



实验: 双腿版本 + encoder

```
python train_both_legs.py   --config_path configs.hip_encoder_only_config   --data_dir ../train_data   --device cuda --batch_size 16 --learning_rate 0.0008 --early_stopping 30
```

![[Pasted image 20260417095137.png]]✓ Model saved: checkpoints/final_20260416_152544.pth


```
cd ~/sk/exo/capsule-5421243/code
python example_save_both_legs.py \
  --config_path configs.hip_encoder_only_config \
  --model_path checkpoints/final_20260416_152544.pth \
  --device cpu \
  --out_csv prediction_final_20260416_152544.csv
```

![[Pasted image 20260417095518.png]]


``` 
# 转化为onnx 
python export_onnx.py  --checkpoint checkpoints/final_20260416_152544.pth  --output checkpoints/final_20260416_152544.onnx  --device cpu --opset 16
```


```
# 转换为rknn
python convert_rknn.py \
  --onnx_path checkpoints/final_20260416_152544.onnx \
  --rknn_path checkpoints/final_20260416_152544.rknn \
  --target rk3576 \
  --seq_len 300 \
  --input_channels 4 \
  --input_name input
```

```
cd ~/sk/exo/capsule-5421243/code
python example_rknn.py \
  --config_path configs.hip_encoder_only_config \
  --rknn_path checkpoints/final_20260416_152544.rknn \
  --target none \
  --seq_len 300
```




![[Pasted image 20260417135828.png]]



实验 ： 双腿数据输入 单腿输出
python train.py   --config_path configs.hip_encoders_left   --data_dir ../train_data   --device cuda   --batch_size 16 --learning_rate 0.0004 --early_stopping 20

✓ Model saved: checkpoints/best_20260420_115522.pth

```
python export_onnx.py  --checkpoint checkpoints/best_20260420_115522.pth --output checkpoints/best_20260420_115522.onnx --device cpu --opset 16
```


```
# 转换为rknn
python convert_rknn.py \
  --onnx_path checkpoints/best_20260420_115522.onnx \
  --rknn_path checkpoints/best_20260420_115522.rknn \
  --target rk3576 \
  --seq_len 300 \
  --input_channels 4 \
  --input_name input
```


右腿
```

python train.py   --config_path configs.hip_encoders_right   --data_dir ../train_data   --device cuda   --batch_size 16 --learning_rate 0.0004 --early_stopping 20
```

✓ Model saved: checkpoints/final_20260420_135102.pth



```
python export_onnx.py --checkpoint checkpoints/best_20260420_135102.pth --output checkpoints/best_20260420_135102.onnx --device cpu --opset 16
```

```
# 转换为rknn
python convert_rknn.py --onnx_path checkpoints/best_20260420_135102.onnx --rknn_path checkpoints/best_20260420_135102.rknn --target rk3576 --seq_len 300 --input_channels 4 --input_name input
```



实验： transformer版本
```
python train_transformer.py   --config_path configs.default_config   --data_dir ../train_data   --device cuda   --crop_len 1024


```

```
python example_transformer.py   --config_path configs.default_config   --model_path checkpoints/best_transformer_20260420_180741.pth   --device cuda

```



```

python example_save_transformer.py   --config_path configs.default_config   --model_path checkpoints/best_transformer_20260420_180741.pth   --out_csv pred_transformer_20260420_180741.csv
```
`


![[Pasted image 20260421094020.png]]




实验：脚踝

```
python train.py   --config_path configs.left_ankle_imu_config   --data_dir ../train_data   --device cuda   --batch_size 16 --learning_rate 0.0004 --early_stopping 30
```
✓ Model saved: checkpoints/best_20260421_112638.pth

![[Pasted image 20260421145823.png]]


转换为onnx
```
python export_onnx.py --checkpoint checkpoints/best_20260421_112638.pth --output onnx/best_20260421_112638.onnx --device cpu --opset 16
```





```
# 转换为rknn
python convert_rknn.py --onnx_path onnx/best_20260421_112638.onnx --rknn_path rknn/left_ankle.rknn --target rk3576 --seq_len 300 --input_channels 6 --input_name input
```



## 实验：小腿

```

python train.py  --config_path configs.left_shank_imu_config \
    --epochs 100 \
    --learning_rate 1e-4 \
    --save_dir checkpoints/left_shank_imu \
    --train_from_scratch
    
    
```


```
python export_onnx.py \

--checkpoint checkpoints/final_20260423_115407.pth \

--output onnx/shank_20260423_115407.onnx \

--seq-len 300 \

--device cpu

```

```
python convert_rknn.py   --onnx_path onnx/shank_20260423_115407.onnx   --rknn_path rknn/shank_20260423_115407_rk3576.rknn   --target rk3576   --seq_len 600   --input_channels 6   --input_name input
```

```

python example_save.py   --config_path configs.left_shank_imu_config   --model_path onnx/shank_20260423_115407.onnx   --out_csv pred_shank_20260423_115407.csv
```


实验二： 带有encoder

```
python train.py   --config_path configs.left_shank_imu_config    --data_dir ../train_data   --device cuda   --batch_size 16 --learning_rate 0.0004 --early_stopping 30
```


```
python export_onnx.py --checkpoint checkpoints/best_20260428_171653.pth --output onnx/shank_encoder_20260428_171653.onnx --device cpu

```

```
python convert_rknn.py   --onnx_path onnx/shank_encoder_20260428_171653.onnx   --rknn_path rknn/shank_encoder_20260428_171653.rknn   --target rk3576   --seq_len 600   --input_channels 7   --input_name input
```



```
python example_save.py \
  --config_path configs.left_shank_imu_vali_config \
  --model_path checkpoints/best_20260428_171653.pth \
  --out_csv pred_shank_encoder_20260424_185219_vali.csv
```


仿真数据和 真实数据对比

X 轴
![[Pasted image 20260429112242.png]]

y轴 

![[Pasted image 20260429112909.png]]
z轴 
![[Pasted image 20260429112657.png]]

coder
![[Pasted image 20260429110522.png]]