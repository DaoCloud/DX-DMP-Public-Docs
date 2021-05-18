# By way of extending PatternLayout
In the process of integrating custom formatted logs, it is usually done using filters or interceptors. However, for the `Integration of Spring Boot service AppName and Eureka instance information into the logging system` explained here, the application name and instance Id generally do not change after the application is started, so it feels a bit redundant to go through filters and interceptors every time.

Source Code Reference：[Github](https://git.mschina.io/microservice/demo/demo-project/tree/master/dmp-producer)
## Ideas

- Add a listener to the `ApplicationEnvironmentPreparedEvent` event during Spring Boot startup in the application
and then put the AppName and InstanceId into a static variable.
- After inheriting `PatternLayout`, the above two messages are filled in during the log output.

## Steps：
1、Create custom format converters`AppInfoConverter`:
This class inherits from`ch.qos.logback.classic.pattern.ClassicConverter`

```java
/**
 * @author jian.tan
 */
public class AppInfoConverter extends ClassicConverter {
    public static String appName;
    public static String instanceId;

    @Override public String convert(ILoggingEvent event) {
        return "App_Name: " + appName + "," + " Instance_Id: " + instanceId;
    }
}
```

2、Add the converter to the custom `PatternLayout`:
This class inherits from`ch.qos.logback.classic.PatternLayout`:

```java

import ch.qos.logback.classic.PatternLayout;

public class AppInfoPatternLayout extends PatternLayout {
    static {
        defaultConverterMap.put("app_info", AppInfoConverter.class.getName());
    }
}
```

3、Add a listener for the Spring Boot start event `ApplicationEnvironmentPreparedEvent`:
- This class inherits from`org.springframework.context.ApplicationListener`

```java
import org.springframework.boot.context.event.ApplicationEnvironmentPreparedEvent;
import org.springframework.context.ApplicationListener;
import org.springframework.core.env.Environment;
import static com.example.webclientdemo.logback.AppInfoConverter.*;

public class ApplicationStartedEventListener implements ApplicationListener<ApplicationEnvironmentPreparedEvent> {

    @Override
    public void onApplicationEvent(ApplicationEnvironmentPreparedEvent event) {
        Environment environment = event.getEnvironment();
        appName = environment.getProperty("spring.application.name");
        instanceId = environment.getProperty("eureka.instance.instanceId");
    }
}
```

- Add the following description file under `resources/META-INF` in accordance with the official SpringBoot documentation：

`spring.factories`

```text
org.springframework.context.ApplicationListener=com.example.webclientdemo.MyApplicationStartedEventListener
```

4、Add `logback.xml` configuration file
Add the following under `resources/logback.xml`：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration scan="true" scanPeriod="60 seconds" debug="false">
    <!--Output to console-->
    <appender name="console" class="ch.qos.logback.core.ConsoleAppender">
        <!-- <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
             <level>ERROR</level>
         </filter>-->
        <layout class="io.daocloud.apollodemo.logback.AppInfoPatternLayout">
            <pattern>%d{HH:mm:ss.SSS} %contextName [%app_info] [%thread] %-5level %logger{36} -%msg%n</pattern>
        </layout>
    </appender>

    <!--Output to file-->
    <appender name="file" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- Log paths can be absolute or relative-->
        <file>logs/logback-producer.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!-- Defines how the logs are sliced -->
            <fileNamePattern>logs/crn/apollo.%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <!-- Indicates that only the last 30 days of logs are kept -->
            <maxHistory>30</maxHistory>
            <!-- Used to specify the maximum size of the log file, for example, if it is set to 1GB, then the old logs will be deleted when it reaches this value-->
            <totalSizeCap>1GB</totalSizeCap>
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <!-- maxFileSize:This is the size of the active file, the default value is 10MB -->
                <maxFileSize>10MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
        </rollingPolicy>
        <encoder>
            <pattern>%d{HH:mm:ss.SSS} %contextName [%app_info] [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>
    <!-- Used to specify the most basic log output level, with only one level attribute. -->
    <root level="info">
        <appender-ref ref="console"/>
        <appender-ref ref="file"/>
    </root>
</configuration>

```

5、Finally start the program to see the results：
Found the words `[App_Name: dmp-producer, Instance_Id: null]`, where `Instance_Id: null` is the reason why there is no access to Eureka in the demo.

```bash
18:13:34.415 default [App_Name: dmp-producer, Instance_Id: null] [main] INFO  i.d.apollodemo.ProducerApplication -No active profile set, falling back to default profiles: default
18:13:35.166 default [App_Name: dmp-producer, Instance_Id: null] [main] WARN  c.c.f.f.i.p.DefaultApplicationProvider -app.id is not available from System Property and /META-INF/app.properties. It is set to null
18:13:35.178 default [App_Name: dmp-producer, Instance_Id: null] [main] INFO  c.c.f.f.i.p.DefaultServerProvider -Environment is set to null. Because it is not available in either (1) JVM system property 'env', (2) OS env variable 'ENV' nor (3) property 'env' from the properties InputStream.
18:13:35.331 default [App_Name: dmp-producer, Instance_Id: null] [main] INFO  o.s.cloud.context.scope.GenericScope -BeanFactory id=7342d45a-9f38-3e75-b931-411c5d464b8f
```