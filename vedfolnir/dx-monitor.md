## Docking DX-monitoring alarms

### Steps

Dock to the alerting platform and require the application to actively expose the metrics interface.

- First, in the configuration file, set the environment variable `VEDFOLNIR_IS_EXPOSE_PROMETHEUS` to `true` (default is true, port number 8888), if you want to specify the port, you can set `VEDFOLNIR_PROMETHEUS_PORT` to a specific port number.

- Open the port in the yaml documentation when deploying the application

### Arrangement document reference

```
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: dmp-ns1
  name: daoshop-order
  labels:
    app: daoshop-order
spec:
  selector:
    matchLabels:
      app: daoshop-order
  template:
    metadata:
      labels:
        app: daoshop-order
    spec:
      # refs: https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-initialization/
      initContainers:
        - name: dx-monitor-agent-sidecar
          image: registry.dx.io/daocloud-dmp/dx-monitor-agent-sidecar:release-2.4.0
          imagePullPolicy: IfNotPresent
          command: 
            - "sh"
            - "-c"
            - > 
              mv /sidecar/skywalking/agent/optional-plugins/apm-trace-ignore-plugin-8.3.0-SNAPSHOT.jar /sidecar/skywalking/agent/plugins;
              echo 'trace.ignore_path=${TRACE_IGNORE_PATH:/api/sail/**,/metrics/**}' >> /sidecar/skywalking/agent/config/apm-trace-ignore-plugin.config;
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
            - containerPort: 8888 ➊
              name: metrics
          env:
            - name: JAVA_OPTS
              value: "-javaagent:/sidecar/sidecar/skywalking/agent/skywalking-agent.jar -javaagent:/sidecar/sidecar/vedfolnir/vedfolnir-agent.jar"
           ··· #此处省略多个环境变量
            - name: VEDFOLNIR_IS_EXPOSE_PROMETHEUS
              value: "true"       #The default value is true, if the default value is used, the configuration can be omitted
            - name: VEDFOLNIR_PROMETHEUS_PORT
              value: "8888"       #The default value is 8888, if you use the default value, you can omit this configuration
          volumeMounts:
            - name: sidecar
              mountPath: /sidecar
      volumes:
        - name: sidecar  #Shared agent folder
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
    - port: 8888 ➋
      name: metrics
  selector:
    app: daoshop-order
```

➊、➋ Exposing the monitoring port in the orchestration file

For related environment variables, please refer to👉[Configuration parameters description](agent-settings.md)