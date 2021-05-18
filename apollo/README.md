# Introduction to Configuration Center User Access
This section will be covered in the following subsections, or you can just pick and choose the parts that interest you.

* [Development model in K8S environment](Apollo-ConfigSerivce-In-Docker-k8s.md)
* [Open by annotation](annotation.md)
* [Enabled by configuration file](bootstrap.md)
* [Using public profiles](Apollo-Public-Config.md)
* [Grayscale Release](Apollo-GrayRule.md)
* [Jar package containerization](docker.md)
* [Json/xml type configuration file get method](json-and-xml-configFile.md)

For more use you can refer to[Official Website](https://github.com/ctripcorp/apollo/wiki/Java%E5%AE%A2%E6%88%B7%E7%AB%AF%E4%BD%BF%E7%94%A8%E6%8C%87%E5%8D%97#3213-spring-boot%E9%9B%86%E6%88%90%E6%96%B9%E5%BC%8F%E6%8E%A8%E8%8D%90)

## Introduction
With the popularity of microservices, the number of applications and machines has increased dramatically, and the program configuration has become more and more complicated: the switch of various functions, the configuration of parameters, the address of servers, etc. At the same time, we expect more and more from the program configuration: real-time effect after modification, gray release, sub-environment and sub-cluster management, perfect permission and audit mechanism, etc.

At the same time, our expectations of program configuration are also getting higher and higher: real-time effect after configuration modification, grayscale release, sub-environment and sub-cluster management, perfect permission and audit mechanism, and so on.

In this environment, the traditional way through the configuration file, database, etc. has become increasingly unable to meet our needs for configuration management.

The Configuration Center was born!

Through the configuration center, we can easily manage the configuration of microservices in different environments, so that we can dynamically adjust the service behavior at runtime, and truly achieve the goal of "control" of configuration.

So, to a certain extent, the configuration center becomes the brain of microservices, and how to use this brain well to make microservices more 'intelligent' becomes a more important issue.

The configuration center in DMP is developed by customizing the Apollo configuration center of Ctrip open source. Also, in the process of Apollo transformation, it will contribute its own source code to the community.

Through the visual operation interface of the configuration center, it can centralize and manage the configuration of different environments and clusters of the application, and the configuration can be pushed to the application side in real time after modification, and has standardized permissions, process governance and other features, which is suitable for microservice configuration management scenarios.

The server side is developed based on Spring Boot and Spring Cloud, and can be run directly after packaging, without the need for additional installation of application containers such as Tomcat.

Java client does not depend on any framework and can run on all Java runtime environments, and also has good support for Spring/Spring Boot environments.

.Net client does not rely on any framework , can run on all .


## Configuration Center Features：

* **Configuration changes take effect in real time (hot release)**
  * After a user modifies the configuration and publishes it in Apollo, the client receives the latest configuration in real time (1 second) and is notified to the application.

* **Release Management**
  * All configuration releases have the concept of versioning so that rollback of the configuration can be easily supported.

* **Grayscale Release**
  * Support grayscale release of the configuration, for example, after you click release, it only takes effect for some application instances, and then push it to all application instances after observing no problem for a period of time.

* **Permission management, release review, operation audit**
  * The management of applications and configurations are well managed with permission management mechanisms, and the management of configurations is also divided into two parts: editing and publishing, thus reducing human errors.
  * Audit logs are available for all operations, allowing easy tracking of problems.

* **Unified management of configurations for different environments and clusters**
  * Provides a unified interface to centrally manage the configuration of different environments, clusters, and namespaces.
  * The same code is deployed in different clusters and can have different configurations, such as the zk address, etc.
  * Namespace makes it easy to support multiple applications sharing the same configuration, and also allows applications to override the shared configuration

* **Provide open platform API**
  * Itself provides a relatively complete unified configuration management interface, supporting multiple environments, multi-data center configuration management, permissions, process governance and other features.
  * However, for reasons of generality, changes to the configuration will not be overly restrictive and can be saved as long as they conform to the basic format.
  * In our research, we found that for some users, their configurations may have more complex formats, such as xml, json, and need to do validation on the format.
  * There are also some users such as DAL, which not only have a specific format, but also require checks on the entered values before they can be saved, such as checking if the database, username and password match.
  * For such applications, support the application side to modify and publish the configuration through the open interface, and have perfect authorization and permission control

For more information you can refer to：

* [Introduction to Apollo Configuration Center](https://github.com/ctripcorp/apollo/wiki/Apollo%E9%85%8D%E7%BD%AE%E4%B8%AD%E5%BF%83%E4%BB%8B%E7%BB%8D) 
* [Design and implementation of Apollo, Ctrip's open source configuration center](http://www.itdks.com/dakalive/detail/3420) , [Slides](https://myslide.cn/slides/10168)
* [Configuration Center, making microservices more 'intelligent'](https://2018.qconshanghai.com/presentation/799) , [Slides](https://myslide.cn/slides/10035)
* [Github](https://github.com/ctripcorp/apollo)



