
---
## Ubuntu 中终端分屏工具 Terminator 的安装与使用指南

Terminator 是一款强大的终端分屏工具，支持在一个窗口中打开多个终端，并以网格方式进行分屏、重排和标签管理，非常适合开发者和系统管理员。

---

### 一、安装 Terminator

在终端中执行以下命令进行安装：

```bash
sudo apt update
sudo apt install terminator
```

安装完成后，可以在终端中输入 `terminator` 启动，或从应用菜单中打开。

---

### 二、基本使用方法

#### 1. 启动 Terminator

```bash
terminator
```

或从应用程序菜单中启动。

#### 2. 分屏操作

|操作|快捷键|
|---|---|
|垂直分屏（左右）|Ctrl + Shift + E|
|水平分屏（上下）|Ctrl + Shift + O|
|关闭当前分屏|Ctrl + Shift + W|
|移动焦点|Ctrl + Shift + 方向键|

#### 3. 标签页操作

|操作|快捷键|
|---|---|
|新建标签页|Ctrl + Shift + T|
|关闭标签页|Ctrl + Shift + W（标签页）|
|切换标签页|Ctrl + Page Up/Page Down|

#### 4. 右键菜单功能

右键点击终端区域，可以访问以下功能：

- 分屏（垂直/水平）
    
- 关闭终端
    
- 设置编码
    
- 打开 Preferences 设置界面（外观、字体、快捷键等）
    

---

### 三、自定义与高级用法

#### 1. 配置文件路径

配置文件位于：

```bash
~/.config/terminator/config
```

可手动编辑或通过 Preferences 图形界面进行修改。

#### 2. 使用布局

- 保存布局：右键 → Preferences → Layouts
    
- 启动时加载布局：
    

```bash
terminator -l <布局名>
```

#### 3. 启动时最大化窗口

```bash
terminator --maximize
```

---

Terminator 是一个高效、易用且功能强大的终端管理工具，非常适合同时处理多个终端任务的开发者和运维人员。