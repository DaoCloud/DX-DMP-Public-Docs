# Access by way of container-based Sidecar (best practice)

You can refer to this document if your service is deployed with containers. This document explains how to access distributed link tracking by means of a kubernetes deployment YAML file for `daoshop-user-center` that explains Sidecar. This article uses the Sidecar image package released by DaoCloud as an example.

## Prerequisites

- The ability to pull/download the DaoCloud published link tracking agent Sidecar image.

## Steps

#### Pull the image

### Orchestration file reference

The main use of the Kubernetes initContainer mechanism, see more: [https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-initialization/](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-initialization/)

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
          image: registry.dx.io/dx-pilot/dx-monitor-agent-sidecar:2.4.0-3cdaa67
          imagePullPolicy: IfNotPresent
          command: ["cp", "-r", "/sidecar", "/target"] ➊
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
              value: "-javaagent:/sidecar/sidecar/skywalking/agent/skywalking-agent.jar" ➋
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
  name: daoshop-user-center
spec:
  type: NodePort
  ports:
    - port: 18081
  selector:
    app: daoshop-user-center

```

- ➊ Copy the probes from the image with the Agent to the shared directory.
- ➋ Use the `-javaagent` parameter to specify the path to the Skywalking probe.

Attention⚠️：The relevant environment variables required by the probe are passed into the container during DX deployment.

### More environment variables

 You can refer to [probe parameters configuration](agent-settings.md), [Dockerfile](https://github.com/DaoCloud-Labs/daoshop-product/blob) accessed in DaoShop `daoshop-product` service, [Dockerfile](https://github.com/DaoCloud-Labs/daoshop-product/blob /master/Dockerfile), [Dockerfile] accessed in DaoShop `daoshop-order` service (https://github.com/DaoCloud-Labs/daoshop-order/blob/master/ Dockerfile).
 
