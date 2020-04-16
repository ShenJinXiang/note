# logback配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <property name="CONSOLE_LOG_PATTERN" value="%d{yyyy-MM-dd HH:mm:ss.SSS} [%-5level]-[Thread: %34t] %-40.40logger{39} : %m%n" />
    <property name="FILE_LOG_PATH" value="./logs" />
    <property name="FILE_LOG_NAME" value="xm" />
    <appender name="console" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>${CONSOLE_LOG_PATTERN}</pattern>
        </encoder>
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>info</level>
        </filter>
    </appender>
    <appender name="dailyRollingFile" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${FILE_LOG_PATH}/${FILE_LOG_NAME}_log.log</file>
        <append>true</append>
        <encoder>
            <pattern>${CONSOLE_LOG_PATTERN}</pattern>
        </encoder>
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy" >
            <fileNamePattern>${FILE_LOG_PATH}/${FILE_LOG_NAME}_log.%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <maxFileSize>20MB</maxFileSize>
            <maxHistory>7</maxHistory>
        </rollingPolicy>
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>INFO</level>
        </filter>
    </appender>
    <root level="INFO">
        <appender-ref ref="console"/>
        <appender-ref ref="dailyRollingFile"/>
    </root>
</configuration>

```

