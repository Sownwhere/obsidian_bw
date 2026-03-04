	## 1. Install git 
```bash
sudo apt update
sudo apt install git
git config --global user.name "Bowen"
git config --global user.email "Bowen@lbw.com"

```
 1.生成 SSH 公钥

```bash
ssh-keygen -t ed25519 -C "Gitee SSH Key"
```
- 中间通过三次**回车键**确定

2. 查看生成的 SSH 公钥和私钥：

```
ls ~/.ssh/
```

输出：

```
id_ed25519  id_ed25519.pub
```

- 私钥文件 `id_ed25519`
- 公钥文件 `id_ed25519.pub`

3. 读取公钥文件 `~/.ssh/id_ed25519.pub`：

```
cat ~/.ssh/id_ed25519.pub
```

输出，如：

```
ssh-ed25519 AAAA***5B Gitee SSH Key
```

复制终端输出的公钥。


## 2.中文输入法
https://zhuanlan.zhihu.com/p/132558860

## 3.安装 flameshot
```bash 
sudo apt install flameshot
```
## 4. Chinese 

