# Spring Boot中使用MyBatis
## 配置pom.xml
### 引入mybatis-spring-boot-starter
```xml
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.1.1</version>
</dependency>
```
不同版本的Spring Boot和MyBatis版本对应不一样，具体可查看[官方文档](http://www.mybatis.org/spring-boot-starter/mybatis-spring-boot-autoconfigure/)。
### 引入驱动

**mysql**
```xml
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
</dependency>
```

**oracle**

由于版权的原因，我们需要将`ojdbc6.jar`依赖安装到本地的maven仓库，下载`ojdbc6.jar`，执行命令：
```bash
mvn install:install-file -DgroupId=jar包的groupId -DartifactId=jar包的artifactId -Dversion=jar包的版本号 -Dpackaging=jar -Dfile=jar包所在本地目录
```
安装到本地maven仓库以后，在pom中引入：

```xml
<dependency>
    <groupId>com.oracle</groupId>
    <artifactId>ojdbc6</artifactId>
    <version>11.2.0.3</version>
</dependency>
```

## 配置application.yml
```yml
spring:
  datasource:
    username: root
    password: pass#word1
    url: jdbc:mysql://localhost:3306/spb?useUnicode=true&characterEncoding=utf-8&useSSL=true&serverTimezone=UTC
    driver-class-name: com.mysql.cj.jdbc.Driver
```

## 准备数据
创建表：
```sql
CREATE TABLE `student` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(100) DEFAULT NULL,
  `age` int(3) DEFAULT NULL,
  `sex` tinyint(1) DEFAULT NULL,
  `desc` varchar(200) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8
```

导入初始数据：

```sql
insert into student (`name`, `age`, `sex`, `desc`) values ("刘备", 46, 1, "蜀汉开国老大");
insert into student (`name`, `age`, `sex`, `desc`) values ("李清照", 24, 0, "女词人");
insert into student (`name`, `age`, `sex`, `desc`) values ("武则天", 22, 0, "一代女皇");
insert into student (`name`, `age`, `sex`, `desc`) values ("范仲淹", 32, 1, "先天下之忧而忧，后天下之乐而乐");
insert into student (`name`, `age`, `sex`, `desc`) values ("苏秦", 26, 1, "身挂六国相印");
```

## 使用注解方式的mapper
编写StudentMapper.java
```java
package com.shenjinxiang.spb.mapper;

import com.shenjinxiang.spb.bean.Student;
import org.apache.ibatis.annotations.*;
import org.springframework.stereotype.Component;

import java.util.List;

@Component
@Mapper
public interface StudentMapper {

    @Select("select `id`, `name`, `age`, `sex`, `desc` from student")
    @Results(id = "queryAllStudent", value = {
        @Result(property = "id", column = "id", javaType = Integer.class),
        @Result(property = "name", column = "name", javaType = String.class),
        @Result(property = "age", column = "age", javaType = Integer.class),
        @Result(property = "sex", column = "sex", javaType = Integer.class),
        @Result(property = "desc", column = "desc", javaType = String.class)
    })
    List<Student> queryAll();

    @Select("select `id`, `name`, `age`, `sex`, `desc` from student where `id` = #{id}")
    @Results(id = "queryById", value = {
            @Result(property = "id", column = "id", javaType = Integer.class),
            @Result(property = "name", column = "name", javaType = String.class),
            @Result(property = "age", column = "age", javaType = Integer.class),
            @Result(property = "sex", column = "sex", javaType = Integer.class),
            @Result(property = "desc", column = "desc", javaType = String.class)
    })
    Student queryById(int id);

    @Insert("insert into student (`name`, `age`, `sex`, `desc`) values (#{name}, #{age}, #{sex}, #{desc})")
    int add(Student student);

    @Update("update student set `name` = #{name}, `age` = #{age}, `sex` = #{sex}, `desc` = #{desc} where `id` = #{id}")
    int update(Student student);

    @Delete("delete from student where `id` = #{id}")
    int delById(int id);
}
```

## 编写service

StudentService.java
```java
public interface StudentService {

    List<Student> queryAll();

    Student queryById(int id);

    int add(Student student);

    int update(Student student);

    int delById(int id);
}
```

实现类StudentServiceImp.java
```java
@Service("studentService")
public class StudentServiceImp implements StudentService {

    @Autowired
    private StudentMapper studentMapper;

    @Override
    public List<Student> queryAll() {
        return this.studentMapper.queryAll();
    }

    @Override
    public Student queryById(int id) {
        return this.studentMapper.queryById(id);
    }

    @Override
    public int add(Student student) {
        return this.studentMapper.add(student);
    }

    @Override
    public int update(Student student) {
        return this.studentMapper.update(student);
    }

    @Override
    public int delById(int id) {
        return this.studentMapper.delById(id);
    }
}
```

## 编写controller
StudentController.java
```java
@RestController
@RequestMapping("/student")
public class StudentController {

    @Autowired
    private StudentService studentService;

    @GetMapping("/queryAll")
    public List<Student> queryAll() {
        return this.studentService.queryAll();
    }

    @GetMapping("/queryById")
    public Student queryById(int id) {
        return this.studentService.queryById(id);
    }
}
```

## 测试
启动项目，访问`http://localhost:8080/student/queryAll`:
![](./images/20200314184801.png)
