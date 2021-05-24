# General JVM access

## Introduce dependencies

In `pom.xml` add

```java
<dependency>
    <groupId>com.netflix.eureka</groupId>
    <artifactId>eureka-client</artifactId>
    <version>${eureka-version}</version>
</dependency>
```

## Initializing the Eureka Client

```java
// Initialize the Eureka Client using the configuration
DiscoveryManager.getInstance().initComponent(
                new CloudInstanceConfig(),
                new DefaultEurekaClientConfig());
```

## Create configuration file

Create the `eureka-client.properties` configuration file in the `Resources` directory, which is read by default when the `Eureka Client` is initialized.

```yaml
eureka.name=${YOUR_Application_Name}
eureka.port=${YOUR_Application Port}
eureka.region=default
eureka.serviceUrl.default=${YOUR_EUREKA_ADDRESS}
```

Compared to the `Eureka Client` provided by `SpringCloud`, the native `Eureka Client` does not provide parameters like `eureka.instance.preferIpAddress` and will use the local `Hostname` to register to the server So if the network in the company is not accessible via `Hostname`, this solution will not work.

## Appendix

1. [Configuring Eureka](https://github.com/Netflix/eureka/wiki/Configuring-Eureka)
