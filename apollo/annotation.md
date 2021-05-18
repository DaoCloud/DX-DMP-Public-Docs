# Open by annotation
Apollo supports annotations to access the configuration center. This is also the most recommended approach.
The source code for the `daoshop-admin` service in DaoShop used here is at 👉[Github](https://github.com/DaoCloud-Labs/daoshop-admin)

To use Apollo's annotations, you need to introduce the relevant dependencies in maven or gradle, e.g.

## 1 Introducing dependencies
- Maven way, add the following dependency to the `pom.xml` file：

```
        <!--import apollo-client-->
        <dependency>
            <groupId>com.ctrip.framework.apollo</groupId>
            <artifactId>apollo-client</artifactId>
            <version>2.4.0.DMP.RELEASE</version>
        </dependency>
        
        <repository>
            <id>maven-public</id>
            <name>maven-public</name>
            <url>https://nexus.daocloud.io/repository/maven-public/</url>
        </repository>
```

- Gradle method, add the following dependencies to the `build.gradle` file：

```
repositories {
    maven {
        url "https://nexus.daocloud.io/repository/maven-public/"
    }
}
dependencies {
    compile group: 'com.ctrip.framework.apollo', name: 'apollo-client', version: '2.4.0.DMP.RELEASE'
}
```

## 2 Annotation configuration
For Java applications written using the Spring Boot approach, add the following annotation to the application startup class：

```java
@Configuration
@EnableApolloConfig({"application", "my-another-namespace", "application.yml"})
public class AnotherAppConfig {
	//······
}
```

**Note: ** The `application` and `my-another-namespace` in the `@EnableApolloConfig` annotation are the Namespace (namespace) you create in the Configuration Center, fill in as appropriate.
All namespaces should be suffixed, except for the namespace in properties format, which does not need to be suffixed. For example, .yml, .yaml, .xml, etc.

## 3 Start Passing Parameters
When pulling configuration from the configuration center, you need to tell your service or `apollo-client` which configuration group to pull configuration from, and what address to pull configuration from.

The easiest way to do this is to pass in parameters via Vm Options when running the application:

```bash
app.id = ${Environment code}. ${AppId created in the configuration center}
apollo.configService = http://192.168.2.96:8080 （Here is the address of Apollo-ConfigService.）
```
Of course, you can also pass in parameters to override the parameter values when running the Jar package：

```bash
java -Dapp.id=dmp -Dapollo.configService=http://192.168.2.96:8080 -jar your-app.jar
Or to override the configuration via environment variables:
APOLLO_CONFIGSERVICE=http://192.168.2.96:8080 APOLLO_APP_ID=test240.dmp java -jar your-app.jar
```
