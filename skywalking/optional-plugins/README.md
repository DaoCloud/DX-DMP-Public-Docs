# Using the optional plug-in
The plug-in package contains an `optional-plugins` directory, which contains some plug-ins that are not required in specific scenarios and may affect performance. The directory structure is as follows:

```
+-- agent
    +-- activations
         apm-toolkit-log4j-1.x-activation.jar
         apm-toolkit-log4j-2.x-activation.jar
         apm-toolkit-logback-1.x-activation.jar
         ...
    +-- config
         agent.config  
    +-- plugins
         apm-dubbo-plugin.jar
         apm-feign-default-http-9.x.jar
         apm-httpClient-4.x-plugin.jar
         .....
    +-- optional-plugins
    		apm-spring-annotation-plugin.jar 
    		apm-lettuce-5.x-plugin.jar ➊
    skywalking-agent.jar
    
```

## How to use

Here is an example of the `Lettuce 5.x` plugin for `Resis Client`.

It's very easy to use: "Just copy the relevant plugins from the `optional-plugins` directory to the `plugins` directory`"😄.

## Add CMD to the initialization container for operation

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: dmp-ns1
  name: daoshop-order
  labels:
    app: daoshop-order
    dx.daocloud.io/trace-agent: "true"
    dx.daocloud.io/monitor-agent: "true"
spec:
  selector:
    matchLabels:
      app: daoshop-order
  template:
    metadata:
      labels:
        app: daoshop-order
        dx.daocloud.io/trace-agent: "true"
        dx.daocloud.io/monitor-agent: "true"
    spec:
      # refs: https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-initialization/
      initContainers:
        - name: dx-monitor-agent-sidecar
          image: registry.dx.io/dx-pilot/dx-monitor-agent-sidecar:2.4.0-3cdaa67
          imagePullPolicy: Always
          command: 
            - "sh"
            - "-c"
            - > 
              mv /sidecar/skywalking/agent/optional-plugins/apm-trace-ignore-plugin-6.6.0-SNAPSHOT.jar /sidecar/skywalking/agent/plugins;
              echo 'trace.ignore_path=${TRACE_IGNORE_PATH:/api/sail/**,/metrics/**,/resource/v0.1/**}' >> /sidecar/skywalking/agent/config/apm-trace-ignore-plugin.config;
              mv /sidecar/skywalking/agent/optional-plugins/apm-lettuce-5.x-plugin.jar /sidecar/skywalking/agent/plugins;   ➊
              cp -r /sidecar /target;
          volumeMounts:
            - name: sidecar
              mountPath: /target
      containers:
        - image: {{ daoshop-order.image }}
          name: daoshop-order
          resources:
            requests:
              memory: "2048Mi"
              cpu: "500m"
            limits:
              memory: "2048Mi"
              cpu: "500m"
          ports:
            - containerPort: 8080
              name: web
            - containerPort: 8888
              name: metrics
          env:
            - name: JAVA_OPTS
              value: "-javaagent:/sidecar/sidecar/skywalking/agent/skywalking-agent.jar -javaagent:/sidecar/sidecar/vedfolnir/vedfolnir-agent.jar"
            - name: DX_APP_NAME
              value: "test-app"
            - name: DX_ENV_ID
              value: "test"
            - name: DX_DMP_TRACING_SERVER
              value: dmp-skywalking-oap-ng.dmp-2-4-x.svc:11800
            - name: DX_DMP_ACTUATOR_SERVER
              value: ws://dx-fate:9056
          volumeMounts:
            - name: sidecar
              mountPath: /sidecar
      volumes:
        - name: sidecar  #Shareagent folder
          emptyDir: {}
---
apiVersion: v1
kind: Service
metadata:
  name: daoshop-order
spec:
  type: NodePort
  ports:
    - port: 8080
      name: web
    - port: 8888
      name: metrics
  selector:
    app: daoshop-order

```

- ➊ Copy the `lettuce-5.x` plugin to the `plugins` directory



