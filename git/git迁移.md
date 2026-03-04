### ✅ 第一步：在 Gitee 上创建一个新的项目

1. 打开 [https://gitee.com/dashboard/projects](https://gitee.com/dashboard/projects)
    
2. 点击右上角的「新建项目」
    
3. 填写项目名称（例如 `humanoid_amp`）和相关信息，创建项目
    

---

### ✅ 第二步：将本地 Git 项目绑定到 Gitee

你已经有本地的 Git 仓库，现在需要添加 Gitee 仓库作为一个新的远程源，或者替换掉原有的 GitHub 源。

#### 选项 1：**添加为新的远程仓库（保留 GitHub）**

```bash
git remote add gitee https://gitee.com/你的用户名/humanoid_amp.git

git push gitee main  # 或 master，视你的分支名称而定`
```




```bash

#### 选项 2：**替换 GitHub 源为 Gitee 源**


git remote set-url origin https://gitee.com/你的用户名/humanoid_amp.git`



git push origin main  # 或 master

```

---

### ✅ 第三步：确认推送成功

在终端看到成功信息后，打开 Gitee 项目页面，确认代码已经上传。