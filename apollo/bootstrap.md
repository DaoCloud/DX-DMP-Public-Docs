# I Turn on the configuration center via configuration file

To inject configuration in the initial bootstrap phase of Spring Boot, you need to enable this feature in applocation.properties or application.yml and tag the Namespace where you need to inject configuration in advance (the configuration with the same key is based on the preceding namespace).

```
# Inject the configuration of the default application namespace in the bootstrap phase
apollo.bootstrap.enabled = true
# Specify the namespace to be loaded
apollo.bootstrap.namespaces = application,my-another-namespace
```
Note: You don't need to add the @EnableApolloConfig annotation to the startup class if you use the configuration file to open it.

demo地址 [apollodemo2](https://github.com/DaoCloud-Labs/DMP-Demo/tree/master/apollo/apollo-demo%202)

# II Using apollo configuration center to implement changes to the logging level and dynamically take effect

**Configure the logging configuration in the Configuration Center to take effect**

Starting with version 1.2.0, i.e. starting with DMP version 1.5, if you want to change the logging-related configuration (
such as logging.level.root=info or the parameters in logback-spring.xml)
in Apollo, then you can additionally configure apollo.bootstrap.eagerLoad.enabled=true
to put Apollo loading order before logging system loading, but this will cause the Apollo startup process
cannot be logged (because when Apollo is loaded, the logging system is not ready at all!
The logging system is not ready when Apollo is loaded! So the log output in the Apollo code using Slf4j will have no content). The reference configuration example is as follows.

Example configuration.
```properties
# Put aopllo initialization before logging system initialization
apollop.bootstrap.eagerLoad.enabled=true
```

demo project address [apollodemo3](https://github.com/DaoCloud-Labs/DMP-Demo/tree/master/apollo/apollo-demo3)

# III Dynamically adjusting logging levels via the configuration center

**Client use spring boot's own LoggingSystem api to dynamically set the logging level**

code into the following：
```java
@Service
public class DynamicLoggersConfig{
	Logger logger= LoggerFactory.getLogger(getClass());
	@ApolloConfig
	private Config config;
	private final static String LoggerTag="logging.level.";
	private final LoggingSystem loggingSystem;
	public DynamicLoggersConfig(LoggingSystem loggingSystem) {
		Assert.notNull(loggingSystem, "LoggingSystem must not be null");
		this.loggingSystem = loggingSystem;
	}
	@ApolloConfigChangeListener
	private void configChangeListter(ConfigChangeEvent changeEvent){
		SetkeyNames=config.getPropertyNames();
		for (String key:keyNames){
			if (StringUtils.containsIgnoreCase(key,LoggerTag)){
				String strLevel=config.getProperty(key,"info");
				LogLevel level = LogLevel.valueOf(strLevel.toUpperCase());
				loggingSystem.setLogLevel(key.replace(LoggerTag,""),level);
				logger.info("{}:{}",key,strLevel);
			}
		}
	}
}
```

Configuration and spring environment normal good configuration logging level configuration can be, such as
> logging.level.org.springframework = info  
logging.level.org.hibernate = info

After publishing the changes in apollo, wait a while for the logs to take effect dynamically

demo project address [apollodemo4](https://github.com/DaoCloud-Labs/DMP-Demo/tree/master/apollo/apollo-demo4)
