	# 安装 NVIDIA Docker（nvidia-docker2）在 Ubuntu 24.04 上

Ubuntu 24.04 当前尚未被 NVIDIA 官方 `nvidia-docker` 明确支持，但你可以手动使用 Ubuntu 22.04 的源以实现兼容安装。

---

## 步骤 1：设置兼容的发行版变量

```bash
distribution=ubuntu22.04
```

---

## 步骤 2：添加 NVIDIA Docker 软件源

```bash
curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list
```

---

## 步骤 3：添加 GPG 密钥（推荐方式）

```bash
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/nvidia-docker.gpg > /dev/null
```

---

## 步骤 4：更新软件源并安装 `nvidia-docker2`

```bash
sudo apt update
sudo apt install -y nvidia-docker2
```

---

## 步骤 5：重启 Docker 服务

```bash
sudo systemctl restart docker
```

---

## 步骤 6：验证安装是否成功

运行以下命令来测试是否可以在容器中访问 GPU：

```bash
sudo docker run --rm --gpus all nvidia/cuda:12.2.0-base-ubuntu22.04 nvidia-smi
```

如果一切正常，你应该会看到类似如下的 NVIDIA GPU 状态输出：

```
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 535.54.03    Driver Version: 535.54.03    CUDA Version: 12.2     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|===============================+======================+======================|
|   0  NVIDIA RTX XXXX     Off  | 00000000:01:00.0 Off |                  N/A |
+-------------------------------+----------------------+----------------------+
```

---

## 注意事项

- 尽管 Ubuntu 24.04 尚未被官方支持，但目前使用 Ubuntu 22.04 的源大多数情况是可以正常工作的。
- 后续若 NVIDIA 正式支持 Ubuntu 24.04，请更换为对应的源以获得更好兼容性。

---

## 参考资料

- [NVIDIA Docker 官方文档](https://nvidia.github.io/nvidia-docker)
