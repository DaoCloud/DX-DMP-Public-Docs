# Customize to ignore the specified Trace Path

This method is used as an optional plugin to help us ignore specified Trace, such as common Eureka heartbeat messages, etc.

## Pre-requisites

Get the zip archive of the Skywalking probe or the image of the probe (the method used here). The directory structure after decompression is as follows.

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
    		apm-trace-ignore-plugin.jar ➊
    skywalking-agent.jar
    
```

Since it is an optional plugin, it is not activated by default. We need to copy or cut the jar package in ➊ to the `plugins` directory above and add the relevant configuration to activate the plugin, i.e.：

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
         apm-trace-ignore-plugin.jar ➊ Note that the folder here is plugins.
         .....
    +-- optional-plugins
    		apm-spring-annotation-plugin.jar 
    skywalking-agent.jar
    
```

Create the `apm-trace-ignore-plugin.config` file in the `config` directory and add the following configuration to activate the plugin：

```txt
trace.ignore_path=/your/path/1/**,/your/path/2/** ➊
```

The path configuration rules in ➊ support the `Ant Path` matching style, for example.
`/path/*`, `/path/**`, `/path/? `. Matching Path paths will not be logged.

## Best practices for [container Sidecar-based access](docker-sidecar.md)

Mainly explains to execute Shell command in Kubernetes Pod initContainer, copy `apm-trace-ignore-plugin.jar` plugin to `plugins` folder and configure paths to be ignored.

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: dmp-ns1
  name: daoshop-user-center
  labels:
    app: daoshop-user-center
spec:
  selector:
    matchLabels:
      app: daoshop-user-center
  template:
    metadata:
      labels:
        app: daoshop-user-center
    spec:
      # refs: https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-initialization/
      initContainers:
        - name: dx-monitor-agent-sidecar
          image: registry.dx.io/daocloud-dmp/dx-monitor-agent-sidecar:release-2.3.0-0b0cbd1
          imagePullPolicy: IfNotPresent
          command: 
            - "sh"
            - "-c"
            - > 
              mv /sidecar/skywalking/agent/optional-plugins/apm-trace-ignore-plugin-8.3.0-SNAPSHOT.jar /sidecar/skywalking/agent/plugins; ➊
              echo 'trace.ignore_path=${TRACE_IGNORE_PATH:/api/sail/**}' >> /sidecar/skywalking/agent/config/apm-trace-ignore-plugin.config; ➋
              cp -r /sidecar /target; 
          volumeMounts:
            - name: sidecar
              mountPath: /target
      containers:
        - image: {{ daoshop-user-center.image }}
          name: daoshop-user-center
          resources:
            requests:
              memory: "2048Mi"
              cpu: "500m"
            limits:
              memory: "2048Mi"
              cpu: "500m"
          ports:
            - containerPort: 18081
          env:
            - name: JAVA_OPTS
              value: "-javaagent:/sidecar/sidecar/skywalking/agent/skywalking-agent.jar" 
            - name: AGENT_NAME 
              valueFrom:
                fieldRef:
                  fieldPath: 'metadata.labels[''dx.daocloud.io/service-name'']'
            - name: AGENT_INSTANCE_UUID
              valueFrom:
                fieldRef:
                  fieldPath: 'metadata.name'
            - name: AGENT_INSTANCE_ID
              valueFrom:
                fieldRef:
                  fieldPath: 'metadata.name'
            - name: DMP_ENVIRONMENT_CODE
              valueFrom:
                fieldRef:
                  fieldPath: 'metadata.labels[''dx.daocloud.io/env-id'']'
            - name: DX_APPLICATION_NAME
              valueFrom:
                fieldRef:
                  fieldPath: 'metadata.labels[''dx.daocloud.io/application-name'']'
            - name: DX_APPLICATION_ID
              valueFrom:
                fieldRef:
                  fieldPath: 'metadata.labels[''dx.daocloud.io/application-id'']'
            - name: SW_AGENT_COLLECTOR_BACKEND_SERVICES  
              value: dx-skywalking-oap-ng.dx-pilot.svc:11800
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
  name: daoshop-user-center
spec:
  type: NodePort
  ports:
    - port: 18081
  selector:
    app: daoshop-user-center
```

- ➊ Copy the trace-ignore plugin to the `plugins` directory.
- ➋ Add the relevant configuration, here the trace-ignore URLs prefixed with `/api/sail/**` will be ignored.
