# 修复 Docker 中 `nvidia` runtime 无法识别的问题

## 📌 问题描述

在运行以下命令时报错：

```bash
docker: Error response from daemon: unknown or invalid runtime name: nvidia
```

## ✅ 解决方案

以下为在 **Ubuntu 实体机（非 WSL）** 上的完整修复流程：

---

### 1. 检查 NVIDIA 驱动是否正常

运行：

```bash
nvidia-smi
```

若未能正常显示显卡信息，说明驱动未安装或异常：

```bash
sudo ubuntu-drivers autoinstall
sudo reboot
```

---

### 2. 安装 NVIDIA Container Toolkit

#### 2.1 添加 NVIDIA Docker 的 GPG key 和源

```bash
 distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
distribution=ubuntu22.04
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -

curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | \
  sudo tee /etc/apt/sources.list.d/nvidia-docker.list
```

#### 2.2 安装 nvidia-docker2 并重启 Docker

```bash
sudo apt update
sudo apt install -y nvidia-docker2
sudo systemctl restart docker
```

---

### 3. 修复 Docker 配置文件 `/etc/docker/daemon.json`


#### 请将其修改为如下内容：

```json
{
  "runtimes": {
    "nvidia": {
      "path": "nvidia-container-runtime",
      "runtimeArgs": []
    }
  }
}


{
  "registry-mirrors": [
    "https://do.nark.eu.org",
    "https://dc.j8.work",
    "https://docker.m.daocloud.io",
    "https://dockerproxy.com",
    "https://docker.mirrors.ustc.edu.cn",
    "https://docker.nju.edu.cn"
  ],
  "runtimes": {
    "nvidia": {
      "path": "nvidia-container-runtime",
      "runtimeArgs": []
    }
  }
}

```

编辑命令：

```bash
sudo nano /etc/docker/daemon.json
```

编辑完成后，重启 Docker：

```bash
sudo systemctl restart docker
```

---

### 4. 验证是否成功

运行：

```bash
sudo docker run --rm --runtime=nvidia nvidia/cuda:12.2.0-base-ubuntu20.04 nvidia-smi
```

若能正确显示 GPU 信息，则说明配置成功。

---

## 🎉 成功！

现在你就可以在 Docker 中使用 GPU 运行深度学习或仿真程序了。