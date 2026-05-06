###  Target: 用 TCN 从外骨骼/人体传感器时间序列估计生物关节力矩，之后可以把估计力矩作为外骨骼辅助控制的 feedforward reference。

# **1. Problem Formulation**
该代码库解决的是 biological joint moment estimation，也就是从可穿戴传感器估计人体关节力矩。

典型任务的输入输出：
```
输入: IMU / encoder / insole / COP （center of Pressure）等传感器时间序列
输出: hip / knee / ankle biological joint moment

```

我们的项目 ankle checkpoint 实际输入是 7 通道，见 predict_real_csv.py (line 16)：
```
shank_imu_l_gyro_x/y/z
shank_imu_l_accel_x/y/z
ankle_angle_l

```
输出是
```
predicted_ankle_angle_l_moment
```

更准确地说，模型输出是归一化关节力矩估计：
```
ŷ_t ≈ τ_human(t) / body_mass
```



# **2. Data Pipeline**

训练数据加载在 dataloader.py (line 8)。每个 trial 需要：
```
Exo.csv                # 传感器输入
Joint_Moments_Filt.csv # 力矩标签
```
传感器类型由 config 决定。默认配置 default_config.py (line 13) 包括：
```
foot / shank / thigh IMU: gyro_x/y/z, accel_x/y/z
insole: cop_x, cop_z, force_y
encoder: hip/knee angle, filtered velocity
```

当前 ankle 模型更简化：小腿 IMU 6 轴 + ankle angle，共 7 通道。

数据张量格式统一是：

`[trial, channel, time]`

加载代码在 dataloader.py (line 127)：

`df[self.input_names].values # [T, C] .transpose(0, 1).unsqueeze(0) # [1, C, T]`

不同 trial 长度不一样时，代码用 zero padding 补齐，见 dataloader.py (line 142)。NaN timestep 会被删除，见 dataloader.py (line 159)。

采样率在配置注释和处理记录里按 200 Hz 使用，例如 model_delays = [10] 表示 50 ms。低通处理记录里也用 fs = 200.0。

这里不是传统 sliding window dataset。训练时整段 trial 直接输入 TCN：

`x: [num_trials, C_in, max_seq_len] y: [num_trials, C_out, max_seq_len]`

TCN 自己通过 causal dilated convolution 形成时间窗口。

# **3. TCN Architecture**

模型实现在 tcn.py (line 95)。结构是：

`Input [B, C_in, T] TCN blocks Linear per timestep Output [B, C_out, T]`

你的 checkpoint 参数是：

`input_size = 7 output_size = 1 num_channels = [80, 80, 80, 80, 80] kernel_size = 5 dropout = 0.2 eff_hist = 50`

每个 TemporalBlock 有两个 Conv1d，每层后面接：

`Chomp1d -> ReLU -> Dropout`

并且有 residual connection，见 tcn.py (line 70)：

`out = self.net(x) res = x if self.downsample is None else self.downsample(x) return self.af(out + res)`

dilation 是指数增长，见 tcn.py (line 81)：

`dilation_size = 2 ** i`

所以 5 层 block 的 dilation 是：

`1, 2, 4, 8, 16`

理论 receptive field 是：

`RF = 1 + 2 * (kernel_size - 1) * sum(dilations) = 1 + 2 * 4 * (1+2+4+8+16) = 249 samples`

如果采样率是 200 Hz：

`249 / 200 ≈ 1.245 s`

这里有个重要细节：checkpoint 里 eff_hist = 50，训练评估时会跳过前 50 个点，但从卷积结构理论上看，该网络的最大历史感受野约是 249 点。也就是说 eff_hist 更像“评估 warm-up 忽略长度”，不完全等于真实 receptive field。
