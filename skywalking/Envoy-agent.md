# Envoy探针接入

如果你的服务使用envoy作为网关，可以参考该文档。本文讲解如何将 envoy 接入到分布式链路追踪。

## 前置条件

### 环境安装配置

*注意：最新版本的(envoy1.17)才支持接入skywalking*

* envoy 1.17，[envoy安装详细内容](https://www.envoyproxy.io/docs/envoy/latest/start/install#)：

```bash
sudo yum install getenvoy-envoy
```

* skywalking 8及以上

确保你的envoy是1.17版：

```
envoy --version
```

## 探针接入

* envoy-swapm-demo.yaml

```yaml
static_resources:
  listeners:
  - name: listener_0
    address:
      socket_address:
        address: 0.0.0.0
        port_value: 10000
    filter_chains:
    - filters:
      - name: envoy.filters.network.http_connection_manager
        typed_config:
          "@type": type.googleapis.com/envoy.extensions.filters.network.http_connection_manager.v3.HttpConnectionManager
          stat_prefix: ingress_http
          generate_request_id: true
          #从这里开始配置探针接入
          tracing:
            provider:
              name: envoy.tracers.skywalking
              typed_config:
                "@type": type.googleapis.com/envoy.config.trace.v3.SkyWalkingConfig
                grpc_service:
                  envoy_grpc:
                    cluster_name: skywalking
                  timeout: 0.250s
                client_config:
                  #service_name 以 @结尾代表该服务所在 DMP 租户
                  #instance_name 实例名字
                  service_name: front-envoy@devTenant
                  instance_name: front-envoy-1
          access_log:
          - name: envoy.access_loggers.file
            typed_config:
              "@type": type.googleapis.com/envoy.extensions.access_loggers.file.v3.FileAccessLog
              path: /dev/stdout
          http_filters:
          - name: envoy.filters.http.router
          route_config:
            name: local_route
            virtual_hosts:
            - name: local_service
              domains: ["*"]
              routes:
              - match:
                  prefix: "/"
                route:
                  host_rewrite_literal: www.envoyproxy.io
                  cluster: service_envoyproxy_io

  clusters:
  - name: skywalking
    connect_timeout: 1s
    type: strict_dns
    lb_policy: round_robin
    http2_protocol_options: {}
    load_assignment:
      cluster_name: skywalking
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address:
                # 你的skywaling opa地址与端口配置
                address: 172.0.0.1
                port_value: 11800
  - name: service_envoyproxy_io
    connect_timeout: 30s
    type: LOGICAL_DNS
    dns_lookup_family: V4_ONLY
    load_assignment:
      cluster_name: service_envoyproxy_io
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address:
                address: www.envoyproxy.io
                port_value: 443
    transport_socket:
      name: envoy.transport_sockets.tls
      typed_config:
        "@type": type.googleapis.com/envoy.extensions.transport_sockets.tls.v3.UpstreamTlsContext
        sni: www.envoyproxy.io
admin:
  access_log_path: /dev/null
  address:
    socket_address:
      address: 0.0.0.0
      port_value: 9901

```

启动envoy
```
envoy -c envoy-swapm-demo.yaml
```

