
## 1. Docker 容器命令

- **查看正在运行的容器**
  ```bash
  docker ps
  ```

- **查看所有容器（包括停止的）**
  ```bash
  docker ps -a
  ```

- **启动一个容器**
  ```bash
  docker start <container_id或container_name>
  ```

- **停止一个容器**
  ```bash
  docker stop <container_id或container_name>
  ```

- **重启一个容器**
  ```bash
  docker restart <container_id或container_name>
  ```

- **进入容器（交互模式）**
  ```bash
  docker exec -it <container_id或container_name> bash
  ```

- **删除一个容器**
  ```bash
  docker rm <container_id或container_name>
  ```

- **查看容器日志**
  ```bash
  docker logs <container_id或container_name>
  ```

## 2. Docker 镜像命令

- **查看所有镜像**
  ```bash
  docker images
  ```

- **拉取镜像**
  ```bash
  docker pull <image_name>
  ```

- **删除镜像**
  ```bash
  docker rmi <image_name或image_id>
  ```

- **构建镜像**
  ```bash
  docker build -t <image_name> <path_to_dockerfile>
  ```

## 3. Docker 容器和镜像操作

- **运行容器**
  ```bash
  docker run <options> <image_name>
  ```

  示例：运行一个交互式容器并启动 bash
  ```bash
  docker run -it <image_name> bash
  ```

- **在后台运行容器**
  ```bash
  docker run -d <image_name>
  ```

- **查看容器的 IP 地址**
  ```bash
  docker inspect <container_id或container_name> | grep "IPAddress"
  ```

- **查看容器的资源使用情况**
  ```bash
  docker stats <container_id或container_name>
  ```

## 4. Docker 网络命令

- **查看所有网络**
  ```bash
  docker network ls
  ```

- **创建一个自定义网络**
  ```bash
  docker network create <network_name>
  ```

- **连接容器到指定网络**
  ```bash
  docker network connect <network_name> <container_name>
  ```

- **从网络断开容器**
  ```bash
  docker network disconnect <network_name> <container_name>
  ```

## 5. Docker 数据卷命令

- **查看所有数据卷**
  ```bash
  docker volume ls
  ```

- **创建数据卷**
  ```bash
  docker volume create <volume_name>
  ```

- **删除数据卷**
  ```bash
  docker volume rm <volume_name>
  ```

## 6. Docker Compose 常用命令

- **启动 Compose 项目**
  ```bash
  docker-compose up
  ```

- **后台启动 Compose 项目**
  ```bash
  docker-compose up -d
  ```

- **停止 Compose 项目**
  ```bash
  docker-compose down
  ```

- **查看 Compose 服务状态**
  ```bash
  docker-compose ps
  ```

- **查看 Compose 容器日志**
  ```bash
  docker-compose logs
  ```

### **6. 在 Linux 上重启 Docker**

如果你使用的是基于 **systemd** 的 Linux 发行版（如 Ubuntu、CentOS、Debian 等），可以使用以下命令：

- **重启 Docker 服务**
```bash
sudo systemctl restart docker
```
- **查看 Docker 服务状态**
```bash

sudo systemctl status docker

```

### **7. Docker Prune 常用命令总结

#### 1. 删除未使用的容器、网络、镜像和卷

- **清理所有未使用的 Docker 容器、网络、镜像和卷**
  ```bash
  docker system prune
  ```
  这会删除所有不再使用的容器、网络、镜像和卷，并释放空间。执行时会提示确认。

- **强制删除（不提示确认）**
  ```bash
  docker system prune -f
  ```

#### 2. 删除未使用的镜像

- **删除所有未被容器使用的镜像**
  ```bash
  docker image prune
  ```
  这将删除所有不再被任何容器使用的镜像。

- **删除所有未被任何容器使用的镜像（不提示确认）**
  ```bash
  docker image prune -f
  ```

#### 3. 删除未使用的容器

- **删除所有停止的容器**
  ```bash
  docker container prune
  ```

- **强制删除停止的容器**
  ```bash
  docker container prune -f
  ```

#### 4. 删除未使用的卷

- **删除所有未使用的数据卷**
  ```bash
  docker volume prune
  ```

- **强制删除未使用的数据卷**
  ```bash
  docker volume prune -f
  ```

#### 5. 删除未使用的网络

- **删除所有未使用的网络**
  ```bash
  docker network prune
  ```

- **强制删除未使用的网络**
  ```bash
  docker network prune -f
  ```

---