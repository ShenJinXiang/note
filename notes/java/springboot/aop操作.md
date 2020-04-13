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

