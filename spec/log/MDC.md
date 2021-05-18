# Implemented through interceptors and MDCs
This method differs from the above in that this method is implemented through an interceptor. This scenario is suitable for e.g. if you want to inherit the Ip of each request into the logging system, and for some dynamic integration of values.

This section explains how to implement the functions achieved by `PatternLayout`.
Reference source code ：[Github](https://git.mschina.io/microservice/demo/demo-project/tree/master/dmp-consumer)
## 思路
1. Make a global interceptor, logback's MDC class to inject appName, instanceId

2. The logback output pattern tag configures the parameters for getting MDC injections, such as<pattern>[%X{appName}]  [%X{instanceId}]</pattern>

## Steps:
1、Inheritance`org.springframework.web.servlet.HandlerInterceptor` class：

The values to be written to the log at each request interception are written through the MDC interface to:

```java
@Component
@Slf4j
public class LogInterceptor implements HandlerInterceptor {

    private final static String REQUEST_ID = "requestId";

    @Value("${spring.application.name}")
    private String appName;

    @Value("${eureka.instance.instanceId}")
    private String instanceId;

    @Override
    public boolean preHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o) throws Exception {
        MDC.put("appName", appName);
        MDC.put("instanceId", instanceId);
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, ModelAndView modelAndView) throws Exception {
        MDC.remove("appName");
        MDC.remove("instanceId");
    }

    @Override
    public void afterCompletion(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, Exception e) throws Exception {

    }
}
```

2、Register this interceptor：

Inheritance`org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter`class：

```java
@Configuration
public class WebMvcConfigurer extends WebMvcConfigurerAdapter {
    @Autowired
    private LogInterceptor logInterceptor;

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(logInterceptor);
        super.addInterceptors(registry);
    }
}
```

3、Add `logback.xml` configuration

```xml
······
        <Console name="Console" target="SYSTEM_OUT" follow="true">
            <PatternLayout pattern="${LOG_PATTERN}"/>
        </Console>
······
```

4、Start the program to view the log output：

```bash
2018-12-27 19:01:13.218 [dmp-consumer] [localhost:dmp-2018-12-27 19:01:10.835 [dmp-consumer] [localhost:dmp-consumer:1235] |-INFO  [http-nio-1235-exec-1] com.netflix.config.ChainedDynamicProperty [115] -| Flipping property: dmp-consumer.ribbon.ActiveConnectionsLimit to use NEXT property: niws.loadbalancer.availabilityFilteringRule.activeConnectionsLimit = 2147483647
2018-12-27 19:01:10.853 [dmp-consumer] [localhost:dmp-consumer:1235] |-INFO  [http-nio-1235-exec-1] com.netflix.util.concurrent.ShutdownEnabledTimer [58] -| Shutdown hook installed for: NFLoadBalancer-PingTimer-dmp-consumer
2018-12-27 19:01:10.862 [dmp-consumer] [localhost:dmp-consumer:1235] |-INFO  [http-nio-1235-exec-1] com.netflix.loadbalancer.BaseLoadBalancer [192] -| Client: dmp-consumer instantiated a LoadBalancer: DynamicServerListLoadBalancer:{NFLoadBalancer:name=dmp-consumer,current list of Servers=[],Load balancer stats=Zone stats: {},Server stats: []}ServerList:null
2018-12-27 19:01:10.871 [dmp-consumer] [localhost:dmp-consumer:1235] |-INFO  [http-nio-1235-exec-1] com.netflix.loadbalancer.DynamicServerListLoadBalancer [222] -| Using serverListUpdater PollingServerListUpdater
2018-12-27 19:01:10.877 [dmp-consumer] [localhost:dmp-consumer:1235] |-INFO  [http-nio-1235-exec-1] com.netflix.loadbalancer.DynamicServerListLoadBalancer [150] -| DynamicServerListLoadBalancer for client dmp-consumer initialized: DynamicServerListLoadBalancer:{NFLoadBalancer:name=dmp-consumer,current list of Servers=[],Load balancer stats=Zone stats: {},Server stats: []}ServerList:org.springframework.cloud.netflix.ribbon.eureka.DomainExtractingServerList@1e790ee6
2018-12-27 19:01:10.991 [dmp-consumer] [localhost:dmp-consumer:1235] |-ERROR [http-nio-1235-exec-1] org.apache.catalina.core.ContainerBase.[Tomcat].[localhost].
```