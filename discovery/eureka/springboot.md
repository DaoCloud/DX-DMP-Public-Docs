# [Recommended] SpringBoot & SpringCloud Access

## Introducing dependencies

In `pom.xml` add

```java
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

## Modify startup class (optional)

This step can be ignored when there is no conflicting configuration within the application, and the application will enable `eureka-client` on its own as `dependencies` are added

```java
@SpringBootApplication
@EnableDiscoveryClient
public class XXXApplication {

    public static void main(String[] args) {
        SpringApplication.run(XXXApplication.class, args);
    }
}
```

## Add registration configuration

Add `eureka-client` related configuration to `application.yml` with the following code：

```yaml
eureka:
  instance:
    # Note that this configuration may be prefer-ip-address: true depending on the version, as follows
    preferIpAddress: true
    metadata-map:
    # This configuration is for the application in Application Management and can be passed in Java Options via -Deureka.instance.metadata-map.dmp-application=DaoShop.
      dmp-application: DaoShop
  client:
    serviceUrl:
      defaultZone: ${EUREKA_SERVER:http://localhost:8761/eureka/}
```
