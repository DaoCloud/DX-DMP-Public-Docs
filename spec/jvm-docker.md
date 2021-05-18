# Making the JVM aware of Docker's parameters
## Reason
> The JVM in Docker detects the memory information of the host, it can't sense the resource limit of the container, which may lead to unexpected situations. For example, we usually start the container with the container resources set, but the Java application container will still somehow be killed by the `OOM` Killer during operation.

### What is the root of the problem？
- For the JVM, if the Heap Size is not set, it will set its maximum heap size by default according to the memory size of the host environment.
- The Docker container uses CGroup to limit the resources used by the process, while the JVM in the container still uses the memory size and CPU cores of the host environment for default settings, which leads to miscalculation of the JVM Heap.

Similarly, the default number of GC and JIT compilation threads for the JVM depends on the number of host CPU cores. If we run multiple Java applications on a node, even if we set the CPU limit, there is still a chance that the application performance will suffer due to GC thread preemption switching between applications.

Understanding the root cause of the problem, we can solve the problem very simply

### How to solve？
**Turn on CGroup resource awareness！！！**

At the same time, the Java community was concerned about this issue and supported the ability to automatically sense container resource limits in JavaSE8u131+ and JDK9.

After JDK 8u191 and JDK 10, the community made further optimizations and enhancements to the JVM running in containers. the JVM can automatically sense the CPU and memory resource limits inside the Docker container. the number of CPU cores available to a Java process is calculated from parameters such as cpu sets, cpu shares and cpu quotas.

Related articles can be found in👇：

- [Java SE support for Docker CPU and memory limits.](https://blogs.oracle.com/java-platform-group/java-se-support-for-docker-cpu-and-memory-limits)
- [Improved container integration 10](https://blog.docker.com/2018/04/improved-docker-container-integration-with-java-10/)

This is done by adding parameters such as：
`java -XX:+UnlockExperimentalVMOptions -XX:+UseCGroupMemoryLimitForHeap -jar your-app.jar`

#### So, how to use it in the container？
Here is an example of the commonly used `jdk8`. Of course, we know from the article above that `JavaSE8u131+` version is the only one that supports this approach.
So we choose `openjdk:8-jre-alpine` as our base image，**Basic Dockerfile Template**：

```bash
FROM openjdk:8-jre-alpine

LABEL maintainer="hao.cheng@daocloud.io"

ENV DIST_NAME=demo-service

COPY target/"$DIST_NAME.jar" /"$DIST_NAME.jar"

RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime \
    && echo "Asia/Shanghai" > /etc/timezone

EXPOSE 8080

ENTRYPOINT java  -XX:+PrintFlagsFinal \
 -XX:+UnlockExperimentalVMOptions \
 -XX:+UseCGroupMemoryLimitForHeap \
 $JAVA_OPTS -jar /$DIST_NAME.jar
 
```

## Summary
Containers are different from virtual machines in that their resource limits are implemented through CGroup. And processes inside the container that do not perceive CGroup's limits for memory and CPU allocation may lead to resource conflicts and problems.
If you can't upgrade your Java version, please use `-Xmx` to set your own limits.

