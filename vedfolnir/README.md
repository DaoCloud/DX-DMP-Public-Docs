# Introduction

### What is Vedfolnir

> In Norse mythology, Vedfolnir is an eagle that occupies the top of the world tree and is given the power to observe the world.
>
> ​																																							——Wikipedia

Vedfolnir is a real-time data collection tool for instances, which allows you to observe jvm, gc, threads, logs and other important information at runtime to facilitate real-time troubleshooting. The communication link uses websocket, which is a combination of the original collector-server and collector-manager.



#### Version Requirements

###### vedfolnir-manager

- Jdk 8+

###### vedfolnir-agent

- Jdk 7+
- Spring 3+



#### Supported Features

- Real-time collection of monitoring data
- Support for application docking prometheus



#### Supported monitoring content

- Runtime data such as cpu usage, gc, memory, etc.
- Basic information about the instance, and the jvm parameters
- metrics Metrics
- Threads
- Environment
- Beans
- Log management, including modification of log levels (requires the introduction of spring-boot-actuator)
