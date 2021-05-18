# Envoy Probe Access

If your service uses envoy as a gateway, you can refer to this document. This article explains how to connect envoy to distributed link tracing.

## Pre-requisites

### Environment installation configuration

*Note: access to skywalking is only supported by the latest version (envoy 1.17)*

* envoy 1.17，[envoy installation details](https://www.envoyproxy.io/docs/envoy/latest/start/install#)：

```bash
sudo yum install getenvoy-envoy
```

* skywalking 8 and above

Make sure your envoy is version 1.17:

```
envoy --version
```

## Probe access

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
          #Start configuring probe access from here
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
                  #service_name ends with @ for the DMP tenant where the service is located
                  #instance_name Instance name
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
                # Your skywaling opa address and port configuration
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

Start envoy
```
envoy -c envoy-swapm-demo.yaml
```

