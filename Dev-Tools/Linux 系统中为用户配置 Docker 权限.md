

本文介绍如何在 Linux 系统中为当前用户配置 Docker 使用权限，避免每次运行 Docker 命令都需要使用 `sudo`。

---

## 🔍 一、查看 `docker` 用户组及其成员

```bash
grep docker /etc/group
```

- 示例输出：
  ```bash
  docker:x:998:your_username
  ```
- 若无输出，说明尚未创建 `docker` 用户组。

---

## 🧱 二、创建 `docker` 用户组（如不存在）

```bash
sudo groupadd docker
```

- 如果提示 `groupadd: group 'docker' already exists`，说明该组已存在，可跳过此步骤。

---

## 👤 三、将当前用户添加到 `docker` 用户组

```bash
sudo gpasswd -a $USER docker
```

- 或使用 `usermod` 命令（作用相同）：
  ```bash
  sudo usermod -aG docker $USER
  ```

📌 **注意**：执行该命令后需重新登录或运行：
```bash
newgrp docker
```
使用户组变更生效。

---

## 🛠️ 四、（可选）增加 Docker 套接字访问权限

```bash
sudo chmod a+rw /var/run/docker.sock
```

⚠️ **注意**：该命令将 `/var/run/docker.sock` 的读写权限开放给所有用户，存在安全风险，不建议在生产环境使用。

更推荐通过用户组的方式进行授权。

---

## ✅ 五、验证是否成功

重新登录当前用户或重新打开终端，执行以下命令：

```bash
docker ps
```

如果可以成功运行且无权限错误，则配置成功。

---

## 📎 原文链接及声明

原文链接：https://blog.csdn.net/jason_src/article/details/87862124  
版权声明：本文为博主原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。
