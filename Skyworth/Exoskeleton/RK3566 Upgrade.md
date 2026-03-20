
在 Ubuntu 上查看版本信息，有几种常用方法：

1. **使用 `lsb_release` 命令**（推荐，比较全面）：
    `lsb_release -a`

输出示例：

`No LSB modules are available. Distributor ID: Ubuntu Description:    Ubuntu 20.04.6 LTS Release:        20.04 Codename:       focal`

2. **查看 `/etc/os-release` 文件**（适用于大部分 Linux 发行版）：
    ```cat /etc/os-release```
    

输出示例：

```
NAME="Ubuntu" VERSION="20.04.6 LTS (Focal Fossa)" ID=ubuntu ID_LIKE=debian PRETTY_NAME="Ubuntu 20.04.6 LTS" VERSION_ID="20.04"
```

3. **查看内核版本**（显示内核信息，不是发行版信息）：
	`uname -a`

输出示例：

```
Linux your_hostname 5.15.0-76-generic #83-Ubuntu SMP Fri Jun 9 00:32:35 UTC 2023 x86_64 x86_64 x86_64 GNU/Linux
```







你这个报错是 **DNS 解析失败**，也就是你的系统无法解析 `mirrors.ustc.edu.cn` 的域名，导致无法下载升级包。这个问题和升级本身无关，而是网络或源配置问题。可以按以下步骤解决：

---

### **1️⃣ 测试网络和 DNS**

先测试能否 ping 通域名：

```bash
ping mirrors.ustc.edu.cn
ping 8.8.8.8
```

- 如果 `8.8.8.8` 能 ping 通，但 `mirrors.ustc.edu.cn` 不行 → DNS 问题。
    
- 如果都不通 → 网络可能断了，需要检查接口或路由。
    

---

### **2️⃣ 临时换 DNS**

可以先临时使用 Google 公共 DNS：

```bash
sudo nano /etc/resolv.conf
```

添加：

```
nameserver 8.8.8.8
nameserver 8.8.4.4


#or
sudo bash -c 'echo "nameserver 8.8.8.8" > /etc/resolv.conf'
sudo bash -c 'echo "nameserver 8.8.4.4" >> /etc/resolv.conf'

```

保存后再尝试：

```bash
sudo apt update
sudo apt upgrade
```

---


```
# 设置 DNS 并忽略自动分配的 DNS
sudo nmcli connection modify "exoskeleton" ipv4.dns "8.8.8.8 8.8.4.4"
sudo nmcli connection modify "exoskeleton" ipv4.ignore-auto-dns yes

# 重新激活网络连接
sudo nmcli connection down "exoskeleton"
sudo nmcli connection up "exoskeleton"

# 检查 DNS 是否生效
systemd-resolve --status
cat /etc/resolv.conf

```





你现在系统是 **Ubuntu 20.04.6 LTS (Focal)**，内核是 4.19，架构是 **aarch64（ARM64）**。如果你想升级到 **Ubuntu 22.04 LTS (Jammy)**，可以按照以下步骤操作，但要注意在 ARM 设备上升级可能有一些额外风险，比如第三方驱动或硬件适配问题。建议先 **备份重要数据**。

---

### **1️⃣ 安装更新管理工具**

先确保系统当前是最新：

```bash
sudo apt update
sudo apt upgrade -y
sudo apt dist-upgrade -y
sudo apt install update-manager-core -y
```

确认升级管理器允许 LTS 升级：

```bash
sudo nano /etc/update-manager/release-upgrades
```

将 `Prompt=lts`（如果不是的话）改成：

```
Prompt=lts
```

保存并退出。

---



明白了，你想用 **方案 B：把 `/boot` 移到根分区**，这是个长期、安全的解决办法，只要操作正确，系统启动不会有问题。我给你写一个 **详细、一步步的操作流程**：

---

## **方案 B：把 /boot 移到根分区**

### **步骤 1：准备新的 /boot 目录**

1. 创建一个新的目录在根分区：
    

```bash
sudo mkdir /boot_new
```

2. 将原 `/boot` 内容复制过去：
    

```bash
sudo rsync -aHAX /boot/ /boot_new/
```

> `rsync` 能保持权限、链接、属性完整。

---

### **步骤 2：修改 fstab 挂载**

1. 编辑 `/etc/fstab`：
    

```bash
sudo nano /etc/fstab
```

2. 注释掉原来的 `/boot` 分区挂载行：
    

```text
#/dev/mmcblk1p2  /boot  ext4  defaults  0  2
```

3. 保存退出。此时 `/boot_new` 仍然在 `/boot_new` 下，原来的 `/boot` 仍挂载原分区。
    

---

### **步骤 3：临时使用新的 /boot**

1. 卸载原来的 `/boot`：
    

```bash
sudo umount /boot
```

2. 临时挂载新的 `/boot`：
    

```bash
sudo mount --bind /boot_new /boot
```

3. 检查内容是否正确：
    

```bash
ls /boot
```

> 你应该看到和原来 `/boot` 相同的文件。

---

### **步骤 4：更新 grub**

1. 更新 grub 配置，让系统从新的 `/boot` 启动：
    

```bash
sudo update-grub
sudo grub-install /dev/mmcblk1  # 注意：安装到你的设备，不是分区
```

---

### **步骤 5：重启验证**

1. 重启系统：
    

```bash
sudo reboot
```

2. 系统启动后，确认 `/boot` 使用新的目录：
    

```bash
df -h /boot
```

> 应该显示根分区的空间，而不是原来小的 126 MB。

---

### **步骤 6（可选）：删除旧 /boot 分区**

1. 如果新 `/boot` 工作正常，可以考虑把原来的小 `/boot` 分区删掉，或者留作备用。
    

---

✅ **注意事项**

- `grub-install` 一定要安装到设备（如 `/dev/mmcblk1`），不要安装到分区。
    
- 执行前建议备份 `/boot` 整个目录，防止启动失败。
    
- 操作完成后，你可以用 `df -h /boot` 确认空间是否已经扩大。
    

---

我可以帮你写一条 **完整命令版操作脚本**，直接执行就能把 `/boot` 安全迁移到根分区，无需手动每步。

你希望我直接给你脚本吗？
### **2️⃣ 启动升级**

执行升级命令：

```bash
sudo do-release-upgrade
```

- 如果提示“没有新的发行版”，可以尝试加上 `-d`：
    

```bash
sudo do-release-upgrade -d
```

> 注意：`-d` 是强制升级到开发版本或最新稳定版本，但 22.04 已经是稳定 LTS，可以用这个强制检测。

---

### **3️⃣ 按提示操作**

升级过程中会提示：

- 是否保留当前配置文件
    
- 是否删除不再需要的软件包  
    建议仔细阅读每个提示，通常默认选择安全选项即可。
    

---

### **4️⃣ 升级完成后**

升级完成后重启：

```bash
sudo reboot
```

然后检查版本：

```bash
lsb_release -a
uname -a
```

---

### **⚠️ 注意事项**

1. **备份**：升级风险较高，尤其是 ARM 平台。
    
2. **内核**：20.04 默认内核是 4.19，22.04 会升级到 5.15 系列。某些驱动可能需要重新编译。
    
3. **第三方软件**：如 Docker、NVIDIA 驱动等，最好在升级前确认支持 22.04。
    
4. **Docker 容器**：升级后可能需要重新拉取与 rebuild 镜像。
    

---
吗？



明白 👍。要扩大 `/boot` 空间，你得 **调整分区**，这比清理要复杂一些，尤其在嵌入式设备（RK356x / Lubancat）上，/boot 往往在 eMMC 或 SD 卡的固定分区里。大概有三种方法：

---

## 方法 1：重新分区 + 调整大小（适合 PC/虚拟机，风险较大）

1. **备份数据**：  
    用 `dd` 或者复制整个 eMMC/SD 卡到一个镜像文件，以防止操作失败：
    
    ```bash
    sudo dd if=/dev/mmcblk0 of=backup.img bs=1M status=progress
    ```
    
    （假设你的 eMMC 是 `/dev/mmcblk0`）
    
2. **用 gparted 调整分区**
    
    - 在 PC 上插入 eMMC/SD 卡，运行 `gparted`
        
    - 把 `/boot` 分区后的空闲空间分给 `/boot`
        
    - 应用更改
        
3. **更新 fstab**  
    分区号变动的话，记得修改 `/etc/fstab` 里 `/boot` 的 UUID：
    
    ```bash
    sudo blkid
    sudo nano /etc/fstab
    ```
    

⚠️ 风险：如果操作失误，系统可能无法启动。

---

## 方法 2：把 `/boot` 挂到更大分区（推荐）

1. 在 root 分区里建个新目录，比如：
    
    ```bash
    sudo mkdir /newboot
    ```
    
2. 把旧的 `/boot` 内容拷贝过去：
    
    ```bash
    sudo rsync -avx /boot/ /newboot/
    ```
    
3. 修改 `/etc/fstab`，让 `/boot` 挂载到 `/newboot` 所在分区（比如根分区）。
    
    例如原来是：
    
    ```
    UUID=xxxxxx  /boot  ext4  defaults  0  2
    ```
    
    改成：
    
    ```
    # /boot moved into /
    ```
    
    或者直接取消单独挂载 `/boot`，让它跟 `/` 共用。
    
4. 更新引导：
    
    ```bash
    sudo update-initramfs -u -k all
    sudo update-grub
    ```
    

这样 `/boot` 就不再受限于独立分区的大小。

---

