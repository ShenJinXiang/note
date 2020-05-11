# Spring Boot项目打包成war包

## 修改配置

pom.xml修改打包方式：

```xml
<packaging>war</packaging>
```

添加tomcat依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-tomcat</artifactId>
    <scope>provided</scope>
</dependency>
```

配置打包名称

```
<build>
	<finalName>spb</finalName>
</build>
```

## 启动类

```java
public class ServletInitializer extends SpringBootServletInitializer {

    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
        return application.sources(Application.class);
    }
}
```

