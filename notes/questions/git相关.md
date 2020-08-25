# git 相关

## 本地项目初始化并推送远程

```shell
$ git remote add origin 项目地址
$ git branch --set-upstream-to=origin/远程分支名称
```

从远程获取代码报`fatal: refusing to merge unrelated histories`

```shell
$ git pull origin master --allow-unrelated-histories
```

之后正常提交