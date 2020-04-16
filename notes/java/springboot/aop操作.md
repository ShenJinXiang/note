# AOP操作

## 项目配置

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

## 切面类

```java
@Component
@Aspect
public class WebLogAspect {

    private static final Logger log = LoggerFactory.getLogger(WebLogAspect.class);

    @Pointcut("execution(public * com.shenjinxiang.spb.controller..*(..))")
    public void controllerLog() {}

    @Before("controllerLog()")
    public void logBeforeController(JoinPoint joinPoint) {
//        log.info("logBeforeController before...");
    }

    @Around("controllerLog()")
    public Object aroundController(ProceedingJoinPoint point) {
        String className = point.getTarget().getClass().getName();
        MethodSignature signature = (MethodSignature) point.getSignature();
        Object[] args = point.getArgs();
        log.info("请求方法：" + className + "." + signature.getName());
        log.info("请求参数：" + JSON.toJSONString(args));
        Object result = null;
        try {
            result = point.proceed();
        } catch (Throwable e) {
        }
        log.info("返回结果：" + JSON.toJSONString(result));
        return result;
    }
}
```

## 步骤

### 声明切面类

```java
@Aspect
@Component
public class WebLogAspect {
}
```

### 声明切点

```java
@Pointcut("execution(public * com.shenjinxiang.spb.controller..*(..))")
public void controllerLog() {}
```

### 实现增强

```java
@Around("controllerLog()")
public Object aroundController(ProceedingJoinPoint point) {
// ...
}
```

增强类型：

* before(前置通知)： 在方法开始执行前执行
* after(后置通知)： 在方法执行后执行
* afterReturning(返回后通知)： 在方法返回后执行
* afterThrowing(异常通知)： 在抛出异常时执行
* around(环绕通知)： 在方法执行前和执行后都会执行

增强顺序：

> around > before > around > after > afterReturning