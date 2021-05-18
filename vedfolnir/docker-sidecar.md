## Access based on the container Sidecar

You can refer to this document if your services are deployed in containers. This document explains how to access application monitoring using Sidecar through kubernetes deployment YAML files. This article uses the Sidecar image package released by DaoCloud as an example.

## Pre-requisites

- Ability to pull/download DaoCloud published application monitoring agent Sidecar images.

## Steps

### Pull Mirror

### Arrangement document reference

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: dmp-dev
  name: ns-daoshop-admin
  labels:
    app: ns-daoshop-admin
spec:
  selector:
    matchLabels:
      app: ns-daoshop-admin
  template:
    metadata:
      labels:
        app: ns-daoshop-admin
    spec:
      imagePullSecrets:
      - name: registry-pull
      initContainers:
      - name: agent-sidecar
        image: registry.dx.io/dx-pilot/dx-monitor-agent-sidecar:2.4.0
        command: ["cp", "-r", "/sidecar", "/target"]  ➊
        volumeMounts:
        - name: sidecar
          mountPath: /target
      containers:
      - image: {{ ns-daoshop-admin.image }}
        name: ns-daoshop-admin
        resources:
          requests:
            memory: "1000Mi"
            cpu: "500m"
          limits:
            memory: "1000Mi"
            cpu: "500m"
        ports:
        - containerPort: 18083
        env:
        - name: VEDFOLNIR_SERVER_URL
          value: 'ws://dmp-vedfolnir-manager.dmp-dev:8002'
        ···
        - name: JAVA_OPTS
          value: "-javaagent:/sidecar/sidecar/vedfolnir/vedfolnir-agent.jar" ➋
        volumeMounts:
        - name: host-time
          mountPath: /etc/localtime
          readOnly: true
        - name: sidecar
          mountPath: /sidecar
      volumes:
      - name: host-time
        hostPath:
          path: /etc/localtime
      - name: sidecar  #Shared agent folder
        emptyDir: {}
      restartPolicy: Always
---
apiVersion: v1
kind: Service
metadata:
  namespace: dmp-dev
  name: ns-daoshop-admin
spec:
  type: NodePort
  ports:
  - port: 18083
    nodePort: 30083
  selector:
    app: ns-daoshop-admin
```

➊ Copy the probes from the image with the Agent to the shared directory.

➋ Use the -javaagent parameter to specify the path to the Vedfolnir probe

For related environment variables, please refer to👉[Configuration parameters description](agent-settings.md)



