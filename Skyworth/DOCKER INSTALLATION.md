	# Install Docker Engine using the apt Repository
Before installing Docker Engine for the first time on a new host machine, you need to set up the Docker apt repository. Afterward, you can install and update Docker from the repository.

---

## 1. Set up Docker's apt repository

### Add Docker's official GPG key:

```bash
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

### Add the repository to Apt sources:

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```
------
## 2. Install Docker Packages

To install the latest version of Docker Engine, run:
```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

```

---
## 3. Verify the Installation

Run the following command to verify that Docker Engine is installed correctly:
```bash
sudo docker run hello-world
```

This command downloads a test image and runs it in a container. When the container runs, it prints a confirmation message and exits.

---


# Docker Installation Verification Issue and Fix

## ❗ Problem: Unable to Download `hello-world` Image

When running:

```bash
sudo docker run hello-world
```

You might encounter the following error:

```
Unable to find image 'hello-world:latest' locally
docker: Error response from daemon: Get "https://registry-1.docker.io/v2/": net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers).
See 'docker run --help'.
```

---

## ✅ Solution: Set Up Docker Registry Mirrors

Create or edit the Docker daemon configuration file:

```bash
sudo mkdir -p 
sudo tee /etc/docker/daemon.json <<-'EOF'
{
    "registry-mirrors": [
        "https://do.nark.eu.org",
        "https://dc.j8.work",
        "https://docker.m.daocloud.io",
        "https://dockerproxy.com",
        "https://docker.mirrors.ustc.edu.cn",
        "https://docker.nju.edu.cn"
    ]
}
EOF
```

Then reload and restart the Docker service:

```bash
sudo systemctl daemon-reload
sudo systemctl restart docker
systemctl status docker
```

---

## 🎉 Retry Verification

Run the following again:

```bash
sudo docker run hello-world
```

It should now complete successfully by downloading the image from a mirror and printing the confirmation message.

---

✅ Problem resolved!