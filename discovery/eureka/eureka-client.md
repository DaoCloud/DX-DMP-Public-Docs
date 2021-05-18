# Use Eureka Client to get registered services

Some non-`Spring Boot` class `JVM` applications choose to use the `Eureka Client` directly to get information about the service, which is a different behavior than `Spring Cloud & Spring Boot`, which requires its own integration with the `Http Client client`.

### Eureka Client

```java
EurekaClient eurekaClient = DiscoveryManager.getEurekaClient();
eurekaClient.getApplication("bookmark-service").getInstances((InstanceInfo instanceInfo)->{
    // InstanceInfo here is the real instance object
    // the port number of the service instance
    int port = instanceInfo.getPort();
    // The address of the service instance (Hostname/IP) is determined by the registered address
    String address = instanceInfo.getIPAddr();
    System.out.println(ToStringBuilder.reflectionToString(s));
})
```

## Appendix

1. [EurekaClient Example](https://github.com/Netflix/eureka/blob/master/eureka-examples/src/main/java/com/netflix/eureka/ExampleEurekaClient.java)
