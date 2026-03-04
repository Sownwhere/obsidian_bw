俄
# Linux 系统中安装 Anaconda 或 Miniconda 的步骤

## 步骤 1：更新系统软件包

```bash
sudo apt update
sudo apt upgrade -y
```

---

## 步骤 2：下载 Anaconda 或 Miniconda 安装脚本

您可以选择安装完整的 Anaconda 发行版，或者更精简的 Miniconda。

### 下载 Anaconda：

前往 Anaconda 官方网站获取最新版本的下载链接：[Anaconda 下载页面](https://www.anaconda.com/products/distribution#download-section)

或者使用 `wget` 命令直接下载（请根据最新版本更新链接）：

```bash
wget https://repo.anaconda.com/archive/Anaconda3-2023.07-1-Linux-x86_64.sh
```

### 下载 Miniconda：

前往 Miniconda 官方网站获取最新版本的下载链接：[Miniconda 下载页面](https://docs.conda.io/en/latest/miniconda.html)

或者使用 `wget` 命令直接下载：

```bash
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
```

---

## 步骤 3：运行安装脚本

### 给安装脚本添加执行权限

**Anaconda：**

```bash
chmod +x Anaconda3-2023.07-1-Linux-x86_64.sh
```

**Miniconda：**

```bash
chmod +x Miniconda3-latest-Linux-x86_64.sh
```

### 运行安装脚本

**安装 Anaconda：**

```bash
./Anaconda3-2023.07-1-Linux-x86_64.sh
```

**安装 Miniconda：**

```bash
./Miniconda3-latest-Linux-x86_64.sh
```

---

## 步骤 4：按照提示完成安装

安装过程中，您需要：

- 阅读并同意许可协议（输入 `yes`）
- 选择安装路径（默认情况下安装在您的主目录下）
- 选择是否初始化 Conda（建议选择 `yes`）

---

## 步骤 5：激活 Conda 环境

如果您在安装时选择了初始化 Conda，那么重新启动终端或运行以下命令使更改生效：

```bash
source ~/.bashrc
```

---

## 步骤 6：验证安装是否成功

```bash
conda --version
```

如果安装成功，您将看到类似 `conda 23.7.2` 的输出。

---

## 步骤 7：更新 Conda 到最新版本

```bash
conda update conda
```

---

## 可能的注意事项

- **防火墙或代理设置**：在公司或学校网络环境下，可能需要配置代理才能正常下载和更新。
- **环境变量**：如果 `conda` 命令无法识别，请确保环境变量已正确配置，或手动添加 Conda 路径到 `~/.bashrc` 中。

---

## 卸载 Conda（如果需要）

删除安装目录：

```bash
rm -rf ~/anaconda3  # 或 ~/miniconda3，取决于您的安装目录
```

然后编辑 `~/.bashrc`，删除与 Conda 相关的初始化代码。
