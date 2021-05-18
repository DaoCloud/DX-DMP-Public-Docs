# Jar package containerized use

If you want to run your application through a Docker container. You can refer to the following to make an image by writing a Dockerfile and set the parameters needed to access the configuration center via environment variables.

## Writing a Dockerfile

The prerequisite is that you package your application, for example: `mvn clean package -U -DskipTests`.
Then, we just type the package inside the image, assuming the jar package name ends up being `query-service.jar`.

- Dockerfile example：

```dockerfile
FROM openjdk:8-jre-alpine

LABEL maintainer="jian.tan@daocloud.io"

ENV TZ=Asia/Shanghai \
    DIST_NAME=query-service

# Install required packages
RUN apk add --no-cache \
    bash

RUN ln -sf /usr/share/zoneinfo/$TZ /etc/localtime \
    && echo $TZ > /etc/timezone

COPY target/"$DIST_NAME.jar" /"$DIST_NAME.jar"

EXPOSE 12801

ENTRYPOINT -XX:+PrintFlagsFinal -XX:+UnlockExperimentalVMOptions -XX:+UseCGroupMemoryLimitForHeap \
           $JAVA_OPTS -jar /$DIST_NAME.jar 
```

## Build Docker images

```bash
docker build -t my-query-service-image .
```

## Run the container

```bash
docker run -e APOLLO_CONFIGSERVICE=http://192.168.2.96:8080 ➊ \
-e APOLLO_APP_ID="zzzGEKiRSd.test11" ➋ \
-d my-query-service-image
```

- ➊ The address of the Config Center Config Server.
- ➋ Configuration Group ID

For more usage, please refer to.

* [Open by annotation](annotation.md)
* [Open by configuration file](bootstrap.md)
