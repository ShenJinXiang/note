# linux系统Tomcat无法访问软连接的处理

## 配置软连接

> ln -s 源文件 目标文件

示例:

```bash
ln -s /usr/file/images /usr/local/apache-tomcat-8.5.51/webapps/ROOT/imgs
```

## 配置tomcat

配置tomcat中conf目录下`context.xml`文件：

```xml
<Context>
    <Resources allowLinking="true" />
</Context>
```

