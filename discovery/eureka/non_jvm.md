# Non-JVM Application Access

If the application is a `non-JVM` language, we can use the `Sidecar` approach to plug the application into the `Eureka Server`.
! [](http://cdn.jared-says.cn/eddc95fcgy1g0sx2c19waj20y80hoq36.jpg)

## Adding a health check interface

Because `Eureka Server` needs to perform `health checks` on the application, a `Restful` `health check` interface must be added to the proxy application. This interface needs to return a `JSON` object and must contain the `status` field.

```json
{
  "status": "UP"
}
```

`status` can be set to 2 values **UP**,**DOWN**. If set to other values, it will appear as `UNKOWN` on `Eureka Server`, see <sup><a href='#Appendix' style="font-size:12px;">Appendix 2</a></sup>

## Create a new Sidecar project

1. Quickly create a `Spring Boot` application using [start.spring.io](https://start.spring.io/)
2. Adding `Sidecar` dependencies

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-netflix-sidecar</artifactId>
    <version>${NETFLIX_SIDECAR_VERSION}</version>
</dependency>
```

## Modifying the Sidecar Project Configuration

Modify the `application.yml` file

```yaml
server:
  port: ${YOUR_SERVER_PORT}
spring:
  application:
    name: ${YOUR_SIDECARD_APPLICATION_NAME}
sidecar:
  # Port number of the proxy application
  port: 8000
  # Addresses for monitoring checks: http://localhost:8000/health.json
  health-uri: ${YOUR_HEALTH_URI}
eureka:
  client:
    serviceUrl:
      defaultZone: ${YOUR_EUREKA_ADDRESS}
```

## Deploying Sidecar

`Sidecar` only supports deployment on the same network layer as the application, i.e. `Sidecar` only supports access to the proxied application by way of `localhost`.

- Virtual machine deployment: `Sidecar` and `App` are deployed in the same virtual machine
- Container Cloud Platform deployment: `Sidecar` and `App` are deployed in the same `Container Group (Pod)`

## Appendix

1. [Polyglot support with Sidecar](https://cloud.spring.io/spring-cloud-netflix/multi/multi__polyglot_support_with_sidecar.html)
2. [Expose Health Check API](https://github.com/xetys/microservices-sidecar-example/blob/master/RailsApplication/app/controllers/application_controller.rb)
