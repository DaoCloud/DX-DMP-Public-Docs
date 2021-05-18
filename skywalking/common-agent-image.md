# Build a generic probe image

## Load the probe into the base image via Dockerfile

```txt

FROM openjdk:8-jre-alpine

TODO:Modify download address
ENV TZ=Asia/Shanghai \
    AGENT_REPO_URL="http://nexus.mschina.io/nexus/service/local/repositories/labs/content/io/daocloud/mircoservice/skywalking/agent/2.0.1/agent-2.0.1.gz"

# Install required packages
RUN apk add --no-cache \
    bash

RUN set -ex; \
    ln -sf /usr/share/zoneinfo/$TZ  /etc/localtime; \
    echo $TZ > /etc/timezone;

ADD $AGENT_REPO_URL /

RUN set -ex; \
    tar -zxf /agent-2.0.1.gz; \
    rm -rf agent-2.0.1.gz;

```

## Build Mirror

For example:

```bash
docker build -t my-common-base-agent-image:latest .
```

## How to use

There are two ways to use it.

- As a base image for a business service: for example, use this image in the business service Dockerfile `FROM my-common-base-agent-image:latest`.
- Use it through Agent Sidecar, see [Container Sidecar-based access](docker-sidecar.md) for more information.