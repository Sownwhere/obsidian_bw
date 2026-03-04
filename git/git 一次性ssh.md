## ✅ 临时使用指定 SSH Key 的方法

你可以在一次 `git push` 操作中**临时指定使用哪个私钥文件**，而不用修改全局配置文件。

### 方法一：使用 `GIT_SSH_COMMAND` 临时推送


```bash
GIT_SSH_COMMAND='ssh -i ~/.ssh/id_rsa_gitee' git push origin main
```

解释：

- `-i ~/.ssh/id_rsa_gitee` 表示使用你刚才生成的新 SSH key。
    
- 这个方式只对当前这一条命令生效，不影响你其他 SSH 或 Git 设置。
    

---

### 方法二：使用 `ssh-agent`（会话内有效）

适用于你要连续用几次 Gitee：

```bash
# 启动 ssh-agent eval "$(ssh-agent -s)"  # 添加你的 Gitee 私钥 ssh-add 

~/.ssh/id_rsa_gitee  # 然后你就可以正常使用 git push 了 git push origin main

```



注意：`ssh-add` 添加的 key **只在当前 shell 会话中生效**，关闭终端后需要重新添加。