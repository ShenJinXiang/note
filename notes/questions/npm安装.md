# npm安装

运行`npm install` 时，卡在sill idealTree buildDeps没有反应：

```bash
# 设置淘宝镜像源
npm config set registry https://registry.npm.taobao.org
# 使用如下命令检验是否成功
npm config get registry
```

`Cannot find module 'webpack'` 问题解决办法

```bash
# 卸载
npm uninstall webpack-dev-server
# 安装对应版本
npm install webpack-dev-server@2.9.6
# 运行
npm run dev
```

