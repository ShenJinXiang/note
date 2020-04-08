# 配置MyBatis多数据源

## 数据库准备

### 数据库1，mysql

```sql
-- 创建表 student
CREATE TABLE `student` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(100) DEFAULT NULL,
  `age` int(3) DEFAULT NULL,
  `sex` tinyint(1) DEFAULT NULL,
  `desc` varchar(200) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

-- 初始数据
insert into student (`name`, `age`, `sex`, `desc`) values ("刘备", 46, 1, "蜀汉开国老大");
insert into student (`name`, `age`, `sex`, `desc`) values ("李清照", 24, 0, "女词人");
insert into student (`name`, `age`, `sex`, `desc`) values ("武则天", 22, 0, "一代女皇");
insert into student (`name`, `age`, `sex`, `desc`) values ("范仲淹", 32, 1, "先天下之忧而忧，后天下之乐而乐");
insert into student (`name`, `age`, `sex`, `desc`) values ("苏秦", 26, 1, "身挂六国相印");
```

### 数据库2， mysql

```sql
-- 创建表 emperor
CREATE TABLE `emperor` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(100) DEFAULT NULL,
  `dynasty` varchar(20) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=5 DEFAULT CHARSET=utf8;
insert into emperor (`name`, `dynasty`) values ('刘彻', '汉');
insert into emperor (`name`, `dynasty`) values ('杨广', '隋');
insert into emperor (`name`, `dynasty`) values ('李世民', '唐');
insert into emperor (`name`, `dynasty`) values ('朱厚照', '明');
```

### 数据库3， oracle

```sql
create table BOOK
(
  id     number not null,
  name   varchar2(200),
  author varchar2(100)
)
tablespace SPB
  storage
  (
    initial 64K
    minextents 1
    maxextents unlimited
  );
comment on column BOOK.name
  is '名称';
comment on column BOOK.author
  is '作者';

alter table BOOK
  add constraint PK_BOOK_ID primary key (ID);

-- Create sequence
create sequence SEQ_BOOK_ID
minvalue 0
maxvalue 99999999999
start with 0
increment by 1
cache 20;

insert into BOOK (ID, NAME, AUTHOR) VALUES (SEQ_BOOK_ID.NEXTVAL, '万万没想到', '万维刚');
insert into BOOK (ID, NAME, AUTHOR) VALUES (SEQ_BOOK_ID.NEXTVAL, '三体', '刘慈欣');
insert into BOOK (ID, NAME, AUTHOR) VALUES (SEQ_BOOK_ID.NEXTVAL, '盲人钟表匠', '理查德·道金斯');
insert into BOOK (ID, NAME, AUTHOR) VALUES (SEQ_BOOK_ID.NEXTVAL, '四夷居中国', '张经纬');
```

## 配置pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.2.4.RELEASE</version>
        <relativePath/>
    </parent>

    <groupId>com.shenjinxiang</groupId>
    <artifactId>Spring-Boot-MyBatis-MultiSource</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.1.1</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
            <optional>true</optional>
        </dependency>

        <dependency>
            <groupId>com.oracle</groupId>
            <artifactId>ojdbc6</artifactId>
            <version>11.2.0.3</version>
        </dependency>

        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
        </dependency>

    </dependencies>

</project>
```

## 项目配置 

**application.yml**

```yml
spring:
  datasource:
    db1:
      url: jdbc:mysql://localhost:3306/spb1?useUnicode=true&characterEncoding=utf-8&useSSL=true&serverTimezone=UTC
      username: root
      password: 123456
      driverClassName: com.mysql.cj.jdbc.Driver

    db2:
      url: jdbc:mysql://localhost:3306/spb2?useUnicode=true&characterEncoding=utf-8&useSSL=true&serverTimezone=UTC
      username: root
      password: 123456
      driverClassName: com.mysql.cj.jdbc.Driver

    db3:
      url: jdbc:oracle:thin:@localhost:1521:orcl
      username: SPB_001
      password: 123456
      driverClassName: oracle.jdbc.driver.OracleDriver

logging:
  level:
    com:
      shenjinxiang:
        spb:
          mapper: debug

```

## 编写数据源配置类

### 数据源配置

**bd1**

```java
@Component
@ConfigurationProperties(prefix = "spring.datasource.db1")
public class Db1Config {

    private String url;
    private String username;
    private String password;
    private String driverClassName;
	
    // getter and setter
}
```

**db2**

```java
@Component
@ConfigurationProperties(prefix = "spring.datasource.db2")
public class Db2Config {

    private String url;
    private String username;
    private String password;
    private String driverClassName;
    
    // getter and setter
}
```

**db3**

```java
@Component
@ConfigurationProperties(prefix = "spring.datasource.db3")
public class Db3Config {

    private String url;
    private String username;
    private String password;
    private String driverClassName;
     
    // getter and setter
}
```

### 配置Bean

**db1**

```java
@Configuration
@MapperScan(basePackages = Db1DataSourceConfig.MAPPER_PACKAGE, sqlSessionFactoryRef = "db1SqlSessionFactory")
public class Db1DataSourceConfig {

    static final String MAPPER_PACKAGE = "com.shenjinxiang.spb.mapper.db1";
    static final String MAPPER_LOCATION = "classpath:mapper/db1/*Mapper.xml";

    @Autowired
    private Db1Config db1Config;

    @Primary
    @Bean(name = "db1DataSource")
    public DataSource db1DataSource() {
        DataSource dataSource = DataSourceBuilder.create()
                .driverClassName(db1Config.getDriverClassName())
                .url(db1Config.getUrl())
                .username(db1Config.getUsername())
                .password(db1Config.getPassword())
                .build();
        return dataSource;
    }

    @Primary
    @Bean(name = "db1SqlSessionFactory")
    public SqlSessionFactory db1SqlSessionFactory(@Qualifier("db1DataSource") DataSource dataSource) throws Exception {
        final SqlSessionFactoryBean sessionFactoryBean = new SqlSessionFactoryBean();
        sessionFactoryBean.setDataSource(dataSource);
        sessionFactoryBean.setMapperLocations(new PathMatchingResourcePatternResolver().getResources(MAPPER_LOCATION));
        return sessionFactoryBean.getObject();
    }

    @Primary
    @Bean(name = "db1TransactionManager")
    public DataSourceTransactionManager db1TransactionManager(@Qualifier("db1DataSource") DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }
}
```

**db2**

```java
@Configuration
@MapperScan(basePackages = Db2DataSourceConfig.MAPPER_PACKAGE, sqlSessionFactoryRef = "db2SqlSessionFactory")
public class Db2DataSourceConfig {

    static final String MAPPER_PACKAGE = "com.shenjinxiang.spb.mapper.db2";
    static final String MAPPER_LOCATION = "classpath:mapper/db2/*Mapper.xml";

    @Autowired
    private Db2Config db2Config;

    @Bean(name = "db2DataSource")
    public DataSource db2DataSource() {
        DataSource dataSource = DataSourceBuilder.create()
                .driverClassName(db2Config.getDriverClassName())
                .url(db2Config.getUrl())
                .username(db2Config.getUsername())
                .password(db2Config.getPassword())
                .build();
        return dataSource;
    }

    @Bean(name = "db2SqlSessionFactory")
    public SqlSessionFactory db2SqlSessionFactory(@Qualifier("db2DataSource") DataSource dataSource) throws Exception {
        final SqlSessionFactoryBean sessionFactoryBean = new SqlSessionFactoryBean();
        sessionFactoryBean.setDataSource(dataSource);
        sessionFactoryBean.setMapperLocations(new PathMatchingResourcePatternResolver().getResources(MAPPER_LOCATION));
        return sessionFactoryBean.getObject();
    }

    @Bean(name = "db2TransactionManager")
    public DataSourceTransactionManager db2TransactionManager(@Qualifier("db2DataSource") DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }
}
```

**db3**

```java
@Configuration
@MapperScan(basePackages = Db3DataSourceConfig.MAPPER_PACKAGE, sqlSessionFactoryRef = "db3SqlSessionFactory")
public class Db3DataSourceConfig {

    static final String MAPPER_PACKAGE = "com.shenjinxiang.spb.mapper.db3";
    static final String MAPPER_LOCATION = "classpath:mapper/db3/*Mapper.xml";

    @Autowired
    private Db3Config db3Config;

    @Bean(name = "db3DataSource")
    public DataSource db3DataSource() {
        DataSource dataSource = DataSourceBuilder.create()
                .driverClassName(db3Config.getDriverClassName())
                .url(db3Config.getUrl())
                .username(db3Config.getUsername())
                .password(db3Config.getPassword())
                .build();
        return dataSource;
    }

    @Bean(name = "db3SqlSessionFactory")
    public SqlSessionFactory db3SqlSessionFactory(@Qualifier("db3DataSource") DataSource dataSource) throws Exception {
        final SqlSessionFactoryBean sessionFactoryBean = new SqlSessionFactoryBean();
        sessionFactoryBean.setDataSource(dataSource);
        sessionFactoryBean.setMapperLocations(new PathMatchingResourcePatternResolver().getResources(MAPPER_LOCATION));
        return sessionFactoryBean.getObject();
    }

    @Bean(name = "db3TransactionManager")
    public DataSourceTransactionManager db3TransactionManager(@Qualifier("db3DataSource") DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }
}
```

## 实体类

**Student**

```java
public class Student {

    private int id;
    private String name;
    private int age;
    private boolean sex;
    private String desc;
    
    // getter and setter
}
```

**Emperor**

```java
public class Emperor {

    private int id;
    private String name;
    private String dynasty;
    
    // getter and setter
}
```

**Book**

```java
public class Book {

    private int id;
    private String name;
    private String author;

    // getter and setter
}
```

## 编写mapper接口

**StudentMapper**

```java
@Component
@Mapper
public interface StudentMapper {

    List<Student> queryAllStudent();

    Student queryById(int id);

    int add(Student student);

    int upd(Student student);

    int delById(int id);

}
```

**EmperorMapper**

```java
@Component
@Mapper
public interface EmperorMapper {

    List<Emperor> queryAllEmperor();

    Emperor queryById(int id);

    int add(Emperor emperor);

    int upd(Emperor emperor);

    int delById(int id);

}

```

**BookMapper**

```java
package com.shenjinxiang.spb.mapper.db3;

import com.shenjinxiang.spb.domain.Book;
import com.shenjinxiang.spb.domain.Emperor;
import org.apache.ibatis.annotations.Mapper;
import org.springframework.stereotype.Component;

import java.util.List;

@Component
@Mapper
public interface BookMapper {

    List<Book> queryAllBook();

    Book queryById(int id);

    int add(Book book);

    int upd(Book book);

    int delById(int id);
    
}
```

## mapper对应的xml

**studentMapper.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.shenjinxiang.spb.mapper.db1.StudentMapper">

    <resultMap id="baseResult" type="com.shenjinxiang.spb.domain.Student">
        <result property="id" column="id" jdbcType="INTEGER"/>
        <result property="name" column="name" jdbcType="VARCHAR"/>
        <result property="age" column="age" jdbcType="INTEGER"/>
        <result property="sex" column="sex" jdbcType="INTEGER"/>
        <result property="desc" column="desc" jdbcType="VARCHAR"/>
    </resultMap>

    <select id="queryAllStudent" resultMap="baseResult" >
        SELECT `id`, `name`, `age`, `sex`, `desc`
        FROM student
        ORDER BY ID ASC
    </select>

    <select id="queryById" parameterType="int" resultMap="baseResult" >
        SELECT `id`, `name`, `age`, `sex`, `desc`
        FROM student
        <where>
            AND id = #{id}
        </where>
    </select>

    <insert id="add" parameterType="com.shenjinxiang.spb.domain.Student" >
        INSERT INTO student
        (`name`, `age`, `sex`, `desc` )
        VALUES
        (#{name}, #{age}, #{sex}, #{desc})
    </insert>

    <update id="upd" parameterType="com.shenjinxiang.spb.domain.Student">
        UPDATE student
        <set>
            `name` = #{name},
            `age` = #{age},
            `sex` = #{sex},
            `desc` = #{desc},
        </set>
        <where>
            AND `id` = #{id}
        </where>
    </update>

    <delete id="delById" parameterType="int">
        DELETE FROM student
        <where>
            AND `id` = #{id}
        </where>
    </delete>
</mapper>
```

**emperorMapper.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.shenjinxiang.spb.mapper.db2.EmperorMapper">

    <resultMap id="baseResult" type="com.shenjinxiang.spb.domain.Emperor">
        <result property="id" column="id" jdbcType="INTEGER"/>
        <result property="name" column="name" jdbcType="VARCHAR"/>
        <result property="dynasty" column="dynasty" jdbcType="VARCHAR"/>
    </resultMap>

    <select id="queryAllEmperor" resultMap="baseResult">
        SELECT `id`, `name`, `dynasty`
        FROM emperor
        ORDER BY id ASC
    </select>

    <select id="queryById" parameterType="int" resultMap="baseResult">
        SELECT `id`, `name`, `dynasty`
        FROM emperor
        <where>
            AND id = #{id}
        </where>
    </select>

    <insert id="add" parameterType="com.shenjinxiang.spb.domain.Emperor">
        INSERT INTO emperor
        (`name`, `dynasty`)
        VALUES
        (#{name}, #{dynasty})
    </insert>

    <update id="upd" parameterType="com.shenjinxiang.spb.domain.Emperor">
        UPDATE emperor
        <set>
            `name` = #{name},
            `dynasty` = #{dynasty},
        </set>
        <where>
            AND `id` = #{id}
        </where>
    </update>

    <delete id="delById" parameterType="int">
        DELETE FROM emperor
        <where>
            AND `id` = #{id}
        </where>
    </delete>
</mapper>
```

**bookMapper.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.shenjinxiang.spb.mapper.db3.BookMapper">

    <resultMap id="baseResult" type="com.shenjinxiang.spb.domain.Book">
        <result property="id" column="ID" jdbcType="INTEGER"/>
        <result property="name" column="NAME" jdbcType="VARCHAR"/>
        <result property="author" column="AUTHOR" jdbcType="VARCHAR"/>
    </resultMap>

    <select id="queryAllBook" resultMap="baseResult">
        SELECT ID, NAME, AUTHOR
        FROM BOOK
        ORDER BY ID ASC
    </select>

    <select id="queryById" parameterType="int" resultMap="baseResult">
        SELECT ID, NAME, AUTHOR
        FROM BOOK
        <where>
            AND ID = #{id}
        </where>
    </select>

    <insert id="add" parameterType="com.shenjinxiang.spb.domain.Book">
        <selectKey keyProperty="id" resultType="java.lang.Integer" order="BEFORE">
            SELECT SEQ_BOOK_ID.NEXTVAL FROM DUAL
        </selectKey>
        INSERT INTO BOOK
        (ID, NAME, AUTHOR)
        VALUES
        (#{id}, #{name}, #{author})
    </insert>

    <update id="upd" parameterType="com.shenjinxiang.spb.domain.Book">
        UPDATE BOOK
        <set>
            NAME = #{name},
            AUTHOR = #{author},
        </set>
        <where>
            AND ID = #{id}
        </where>
    </update>

    <delete id="delById" parameterType="int">
        DELETE FROM BOOK
        <where>
            AND ID = #{id}
        </where>
    </delete>

</mapper>
```

## service

### Student

**StudentService**

```java
public interface StudentService {

    List<Student> queryAllStudent();

    Student queryById(int id);

    int add(Student student);

    int upd(Student student);

    int delById(int id);
}
```

**StudentServiceImpl**

```java
@Service("studentService")
public class StudentServiceImpl implements StudentService {

    @Autowired
    private StudentMapper studentMapper;

    @Override
    public List<Student> queryAllStudent() {
        return this.studentMapper.queryAllStudent();
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
    public int upd(Student student) {
        return this.studentMapper.upd(student);
    }

    @Override
    public int delById(int id) {
        return this.studentMapper.delById(id);
    }
}
```

### Emperor

**EmperorService**

```java
public interface EmperorService {

    List<Emperor> queryAllEmperor();

    Emperor queryById(int id);

    int add(Emperor emperor);

    int upd(Emperor emperor);

    int delById(int id);

}
```

**EmperorServiceImpl**

```java
@Service("emperorService")
public class EmperorServiceImpl implements EmperorService {

    @Autowired
    private EmperorMapper emperorMapper;

    @Override
    public List<Emperor> queryAllEmperor() {
        return this.emperorMapper.queryAllEmperor();
    }

    @Override
    public Emperor queryById(int id) {
        return this.emperorMapper.queryById(id);
    }

    @Override
    public int add(Emperor emperor) {
        return this.emperorMapper.add(emperor);
    }

    @Override
    public int upd(Emperor emperor) {
        return this.emperorMapper.upd(emperor);
    }

    @Override
    public int delById(int id) {
        return this.emperorMapper.delById(id);
    }
}
```

### Book

**BookService**

```java
public interface BookService {

    List<Book> queryAllBook();

    Book queryById(int id);

    int add(Book book);

    int upd(Book book);

    int delById(int id);
}

```

**BookServiceImpl**

```java
@Service("bookService")
public class BookServiceImpl implements BookService {

    @Autowired
    private BookMapper bookMapper;

    @Override
    public List<Book> queryAllBook() {
        return this.bookMapper.queryAllBook();
    }

    @Override
    public Book queryById(int id) {
        return this.bookMapper.queryById(id);
    }

    @Override
    public int add(Book book) {
        return this.bookMapper.add(book);
    }

    @Override
    public int upd(Book book) {
        return this.bookMapper.upd(book);
    }

    @Override
    public int delById(int id) {
        return this.bookMapper.delById(id);
    }
}
```

## controller

**StudentController**

```java
@RestController
@RequestMapping("/students")
public class StudentController {

    @Autowired
    private StudentService studentService;

    @GetMapping("")
    public List<Student> queryAll() {
        return this.studentService.queryAllStudent();
    }

    @GetMapping("/{id}")
    public Student queryById(@PathVariable int id) {
        return this.studentService.queryById(id);
    }

    @PostMapping("")
    public int add(@RequestBody Student student) {
        return this.studentService.add(student);
    }

    @PutMapping("/{id}")
    public int upd(@PathVariable int id, @RequestBody Student student) {
        Student student1 = this.studentService.queryById(id);
        student1.setName(student.getName());
        student1.setAge(student.getAge());
        student1.setSex(student.isSex());
        student1.setDesc(student.getDesc());
        return this.studentService.upd(student1);
    }

    @DeleteMapping("/{id}")
    public int delById(@PathVariable int id) {
        return this.studentService.delById(id);
    }
}
```

**EmperorController**

```java
@RestController
@RequestMapping("/emperors")
public class EmperorController {

    @Autowired
    private EmperorService emperorService;

    @GetMapping("")
    public List<Emperor> queryAllEmperor() {
        return this.emperorService.queryAllEmperor();
    }

    @GetMapping("/{id}")
    public Emperor queryById(@PathVariable int id) {
        return this.emperorService.queryById(id);
    }

    @PostMapping("")
    public int add(@RequestBody Emperor emperor) {
        return this.emperorService.add(emperor);
    }

    @PutMapping("/{id}")
    public int upd(@PathVariable int id, @RequestBody Emperor emperor) {
        Emperor emperor1 = this.emperorService.queryById(id);
        emperor1.setName(emperor.getName());
        emperor1.setDynasty(emperor.getDynasty());
        return this.emperorService.upd(emperor1);
    }

    @DeleteMapping("/{id}")
    public int delById(@PathVariable int id) {
        return this.emperorService.delById(id);
    }
}
```

**BookController**

```java
@RestController
@RequestMapping("/books")
public class BookController {

    @Autowired
    private BookService bookService;

    @GetMapping("")
    public List<Book> queryAll() {
        return this.bookService.queryAllBook();
    }

    @GetMapping("/{id}")
    public Book queryById(@PathVariable int id) {
        return this.bookService.queryById(id);
    }

    @PostMapping("")
    public int postUser(@RequestBody Book book) {
        return this.bookService.add(book);
    }

    @PutMapping("/{id}")
    public int putUser(@PathVariable int id, @RequestBody Book book) {
        Book book1 = this.bookService.queryById(id);
        book1.setName(book.getName());
        book1.setAuthor(book.getAuthor());
        return this.bookService.upd(book1);
    }

    @DeleteMapping("/{id}")
    public int deleteUser(@PathVariable int id) {
        return this.bookService.delById(id);
    }
}
```

## 启动类

```java
@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication application = new SpringApplication(Application.class);
        application.setBannerMode(Banner.Mode.OFF);
        application.run(args);
    }
}
```



## 目录结构

```
Spring-Boot-MyBatis-MultiSource
|-- pom.xml
|-- src
    |-- main
        |-- java
        |   |-- com
        |       |-- shenjinxiang
        |           |-- spb
        |               |-- Application.java
        |               |-- controller
        |               |   |-- BookController.java
        |               |   |-- EmperorController.java
        |               |   |-- StudentController.java
        |               |
        |               |-- domain
        |               |   |-- Book.java
        |               |   |-- Emperor.java
        |               |   |-- Student.java
        |               |
        |               |-- ds
        |               |   |-- config
        |               |   |   |-- Db1Config.java
        |               |   |   |-- Db2Config.java
        |               |   |   |-- Db3Config.java
        |               |   |
        |               |   |-- Db1DataSourceConfig.java
        |               |   |-- Db2DataSourceConfig.java
        |               |   |-- Db3DataSourceConfig.java
        |               |
        |               |-- mapper
        |               |   |-- db1
        |               |   |   |-- StudentMapper.java
        |               |   |
        |               |   |-- db2
        |               |   |   |-- EmperorMapper.java
        |               |   |
        |               |   |-- db3
        |               |       |-- BookMapper.java
        |               |
        |               |-- service
        |                   |-- BookService.java
        |                   |-- EmperorService.java
        |                   |-- StudentService.java
        |                   |-- impl
        |                       |-- BookServiceImpl.java
        |                       |-- EmperorServiceImpl.java
        |                       |-- StudentServiceImpl.java
        |-- resource
            |-- application.yml
            |-- mapper
                |-- db1
                |   |-- studentMapper.xml
                |-- db2
                |   |-- emperorMapper.xml
                |-- db3
                    |-- bookMapper.xml
```

## 使用Druid

### 修改pom.xml

新增druid依赖

```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid-spring-boot-starter</artifactId>
    <version>1.1.21</version>
</dependency>
```

### 修改application.yml

```yml
spring:
  datasource:
    druid:
      # 数据库访问配置, 使用druid数据源
      db1:
        type: com.alibaba.druid.pool.DruidDataSource
        url: jdbc:mysql://localhost:3306/spb1?useUnicode=true&characterEncoding=utf-8&useSSL=true&serverTimezone=UTC
        username: root
        password: 123456
        driverClassName: com.mysql.cj.jdbc.Driver

      db2:
      	type: com.alibaba.druid.pool.DruidDataSource
        url: jdbc:mysql://localhost:3306/spb2?useUnicode=true&characterEncoding=utf-8&useSSL=true&serverTimezone=UTC
        username: root
        password: 123456
        driverClassName: com.mysql.cj.jdbc.Driver

      db3:
      	type: com.alibaba.druid.pool.DruidDataSource
        url: jdbc:oracle:thin:@localhost:1521:orcl
        username: SPB_001
        password: 123456
        driverClassName: oracle.jdbc.driver.OracleDriver

      # 连接池配置
      initial-size: 5
      min-idle: 5
      max-active: 20
      # 连接等待超时时间
      max-wait: 30000
      # 配置检测可以关闭的空闲连接间隔时间
      time-between-eviction-runs-millis: 60000
      # 配置连接在池中的最小生存时间
      min-evictable-idle-time-millis: 300000
      validation-query: select '1' from dual
      test-while-idle: true
      test-on-borrow: false
      test-on-return: false
      # 打开PSCache，并且指定每个连接上PSCache的大小
      pool-prepared-statements: true
      max-open-prepared-statements: 20
      max-pool-prepared-statement-per-connection-size: 20
      # 配置监控统计拦截的filters, 去掉后监控界面sql无法统计, 'wall'用于防火墙
      filters: stat,wall
      # Spring监控AOP切入点，如x.y.z.service.*,配置多个英文逗号分隔
      aop-patterns: com.springboot.servie.*


      # WebStatFilter配置
      web-stat-filter:
        enabled: true
        # 添加过滤规则
        url-pattern: /*
        # 忽略过滤的格式
        exclusions: '*.js,*.gif,*.jpg,*.png,*.css,*.ico,/druid/*'

      # StatViewServlet配置
      stat-view-servlet:
        enabled: true
        # 访问路径为/druid时，跳转到StatViewServlet
        url-pattern: /druid/*
        # 是否能够重置数据
        reset-enable: false
        # 需要账号密码才能访问控制台
        login-username: druid
        login-password: 123456
        # IP白名单
        # allow: 127.0.0.1
        #　IP黑名单（共同存在时，deny优先于allow）
        # deny: 192.168.1.218

      # 配置StatFilter
      filter:
        stat:
          log-slow-sql: true

logging:
  level:
    com:
      shenjinxiang:
        spb:
          mapper: debug
```

### 修改数据源配置类

删除`com.shenjinxiang.spb.ds`包下的所有类

**db1**

```java
@Configuration
@MapperScan(basePackages = Db1DataSourceConfig.MAPPER_PACKAGE, sqlSessionFactoryRef = "db1SqlSessionFactory")
public class Db1DataSourceConfig {

    static final String MAPPER_PACKAGE = "com.shenjinxiang.spb.mapper.db1";
    static final String MAPPER_LOCATION = "classpath:mapper/db1/*Mapper.xml";

    @Primary
    @Bean(name = "db1DataSource")
    @ConfigurationProperties("spring.datasource.druid.db1")
    public DataSource db1DataSource() {
        return DruidDataSourceBuilder.create().build();
    }

    @Primary
    @Bean(name = "db1SqlSessionFactory")
    public SqlSessionFactory db1SqlSessionFactory(@Qualifier("db1DataSource") DataSource dataSource) throws Exception {
        final SqlSessionFactoryBean sessionFactoryBean = new SqlSessionFactoryBean();
        sessionFactoryBean.setDataSource(dataSource);
        // 如果不使用xml的方式配置mapper，可以
        sessionFactoryBean.setMapperLocations(new PathMatchingResourcePatternResolver().getResources(MAPPER_LOCATION));
        return sessionFactoryBean.getObject();
    }

    @Primary
    @Bean(name = "db1TransactionManager")
    public DataSourceTransactionManager db1TransactionManager(@Qualifier("db1DataSource") DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }
}
```

**db2**

```java
@Configuration
@MapperScan(basePackages = Db2DataSourceConfig.MAPPER_PACKAGE, sqlSessionFactoryRef = "db2SqlSessionFactory")
public class Db2DataSourceConfig {

    static final String MAPPER_PACKAGE = "com.shenjinxiang.spb.mapper.db2";
    static final String MAPPER_LOCATION = "classpath:mapper/db2/*Mapper.xml";

    @Bean(name = "db2DataSource")
    @ConfigurationProperties("spring.datasource.druid.db2")
    public DataSource db2DataSource() {
        return DruidDataSourceBuilder.create().build();
    }

    @Bean(name = "db2SqlSessionFactory")
    public SqlSessionFactory db2SqlSessionFactory(@Qualifier("db2DataSource") DataSource dataSource) throws Exception {
        final SqlSessionFactoryBean sessionFactoryBean = new SqlSessionFactoryBean();
        sessionFactoryBean.setDataSource(dataSource);
        // 如果不使用xml的方式配置mapper，可以
        sessionFactoryBean.setMapperLocations(new PathMatchingResourcePatternResolver().getResources(MAPPER_LOCATION));
        return sessionFactoryBean.getObject();
    }

    @Bean(name = "db2TransactionManager")
    public DataSourceTransactionManager db2TransactionManager(@Qualifier("db2DataSource") DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }
}
```

**db3**

```java
@Configuration
@MapperScan(basePackages = Db3DataSourceConfig.MAPPER_PACKAGE, sqlSessionFactoryRef = "db3SqlSessionFactory")
public class Db3DataSourceConfig {

    static final String MAPPER_PACKAGE = "com.shenjinxiang.spb.mapper.db3";
    static final String MAPPER_LOCATION = "classpath:mapper/db3/*Mapper.xml";

    @Bean(name = "db3DataSource")
    @ConfigurationProperties("spring.datasource.druid.db3")
    public DataSource db3DataSource() {
        return DruidDataSourceBuilder.create().build();
    }

    @Bean(name = "db3SqlSessionFactory")
    public SqlSessionFactory db3SqlSessionFactory(@Qualifier("db3DataSource") DataSource dataSource) throws Exception {
        final SqlSessionFactoryBean sessionFactoryBean = new SqlSessionFactoryBean();
        sessionFactoryBean.setDataSource(dataSource);
        // 如果不使用xml的方式配置mapper，可以
        sessionFactoryBean.setMapperLocations(new PathMatchingResourcePatternResolver().getResources(MAPPER_LOCATION));
        return sessionFactoryBean.getObject();
    }

    @Bean(name = "db3TransactionManager")
    public DataSourceTransactionManager db3TransactionManager(@Qualifier("db3DataSource") DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }
}
```

### 修改后的目录结构

```
Spring-Boot-MyBatis-MultiSource-Druid
|-- pom.xml
|-- src
    |-- main
        |-- java
        |   |-- com
        |       |-- shenjinxiang
        |           |-- spb
        |               |-- Application.java
        |               |-- controller
        |               |   |-- BookController.java
        |               |   |-- EmperorController.java
        |               |   |-- StudentController.java
        |               |
        |               |-- domain
        |               |   |-- Book.java
        |               |   |-- Emperor.java
        |               |   |-- Student.java
        |               |
        |               |-- ds
        |               |   |-- Db1DataSourceConfig.java
        |               |   |-- Db2DataSourceConfig.java
        |               |   |-- Db3DataSourceConfig.java
        |               |
        |               |-- mapper
        |               |   |-- db1
        |               |   |   |-- StudentMapper.java
        |               |   |
        |               |   |-- db2
        |               |   |   |-- EmperorMapper.java
        |               |   |
        |               |   |-- db3
        |               |       |-- BookMapper.java
        |               |
        |               |-- service
        |                   |-- BookService.java
        |                   |-- EmperorService.java
        |                   |-- StudentService.java
        |                   |-- impl
        |                       |-- BookServiceImpl.java
        |                       |-- EmperorServiceImpl.java
        |                       |-- StudentServiceImpl.java
        |-- resource
            |-- application.yml
            |-- mapper
                |-- db1
                |   |-- studentMapper.xml
                |-- db2
                |   |-- emperorMapper.xml
                |-- db3
                    |-- bookMapper.xml
```

