
## 一、背景

在强化学习或控制任务中，我们经常使用 **PyTorch 训练的模型（.pt 文件）**，但在嵌入式 ARM 板（如 RK3588）上部署时，通常需要 **ONNX** 格式进行推理。

本流程整理了从 `.pt` 到 ONNX，再到 RK3588 推理的完整步骤。

---

## 二、知识点总结

1. **PyTorch `.pt` 文件结构**
   - `.pt` 可以是：
     - **完整模型对象**（`torch.nn.Module`）
     - **字典**（dict），包含 `models`, `optimizers`, `state_preprocessor` 等
   - SKRL 保存模型时通常是 **dict**：
     ```text
     root
     ├─ exo
     │  ├─ policy
     │  └─ value
     └─ humanoid
        ├─ policy
        ├─ value
        └─ discriminator
     ```
   - 关键：只需 **policy 部分** 进行推理。

2. **map_location**
   - `torch.load("model.pt", map_location="cpu")`
   - 将原本 GPU 的权重加载到 CPU 或指定设备
   - 必须在 CPU 环境或没有 GPU 时使用。

3. **GaussianMixin / RunningStandardScaler**
   - SKRL 的 policy 输出通常是高斯分布 `(mean, log_std)`，推理时只用 `mean`
   - 输入状态通常经过 RunningStandardScaler 标准化：
     ```
     obs_norm = (obs - running_mean) / sqrt(running_var + 1e-8)
     ```

---

## 三、流程步骤

### 1️⃣ 查看 `.pt` 文件结构

```python
import torch

ckpt = torch.load("best_agent.pt", map_location="cpu")
print(ckpt.keys())   # 查看顶层 key
print(ckpt["humanoid"].keys())  # 查看 humanoid agent