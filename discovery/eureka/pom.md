# Introducing Dependencies

This article explains how to introduce dependencies to access Eureka's registry

The main dependencies are

```xml
		<!-- Eureka Client Related Dependencies -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <!-- Enable web services -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
```

The startup class code is as follows：

```java
@SpringBootApplication
@EnableDiscoveryClient
public class ConsumerApplication {

    public static void main(String[] args) {
        SpringApplication.run(ConsumerApplication.class, args);
    }
}
```

Add eureka-client related configuration to application.yml with the following code：

```
server:
  port: 8762
spring:
  application:
    name: eureka-client
eureka:
  instance:
    preferIpAddress: true
  client:
    serviceUrl:
      defaultZone: ${EUREKA_SERVER:http://localhost:8761/eureka/}
```

Just start the application after the configuration is done

For more details, see [daoshop-user-center](https://github.com/DaoCloud-Labs/daoshop-user-center), [daoshop-product](https://github.com/) in Dao_Shop. DaoCloud-Labs/daoshop-product) service demo.