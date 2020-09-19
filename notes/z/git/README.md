# git

## 初始设置

### 设置邮箱和用户名

```bash
$ git config --global user.name "Firstname Lastname"
$ git config --global user.email "your_email@example.com"
```

设置后会保存在`~/.gitconfig`中

```bash
$ cat ~/.gitconfig
[user]
	name = shenjinxiang
	email = sjx-sword@163.com
[push]
	default = simple
[core]
	autocrlf = false
[color]
	ui = auto
```

### 提高命令输出可读性

```bash
 git config --global color.ui auto
```

### 设置SSH Key

```bash
$ ssh-keygen -t rsa -C "your_email@example.com"
```

## 常规操作

```bash
# 本地初始化仓库
$ git init
# 查看状态
$ git status
# 克隆远程仓库
$  git clone 仓库地址
# 添加文件到暂存区
$ git add 文件路径
# 暂存区到本地仓库
$ git commit -m "提交注释"
# 修改提交信息
$ git commit --amend
# 推送到远程仓库
$ git push
# 查看提交日志
$ git log
# 查看当前仓库执行过的操作的日志
$ git reflog
# 查看更改前后的差异
$ git diff 文件路径
```

### 分支操作

```bash
# 查看分支列表
$ git branch
# 创建、切换分支
$ git checkout -b 分支名称
# 创建、切换分支两步走
$ git branch 分支名称
$ git checkout 分支名称
# 切换回上一分支
$ git checkout -
# 合并峰值
$ git merge --no-ff 分支名称
# 图表形式查看分支
$ git log --graph

```

### 回溯操作

```bash
# 跳转到某次提交的状态
$ git reset --hard 提交id
```

