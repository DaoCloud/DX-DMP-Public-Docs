# Nginx Probe Access (Container Mode)

If your application uses Nginx, you can refer to this document. This article uses OpenResty as an example to explain how to access endpoints that go through nginx to distributed link tracing.

## Pre-requisites

### Dependency packages and environment installation

* docker 19 and above

* Nginx agent is implemented as a lua script, download the required lua module via git：

```bash
git clone https://github.com/apache/skywalking-nginx-lua.git
```

## Probe access

The directory structure you need to build：


```
your folder
├── skywalking-nginx-lua			#The lua gen repository pulled by git
├── nginx.conf
├── startup.sh
└── docker-compose.yaml

```


Three specific documents：

* docker-compose.yaml

```yaml
version: '2.2'
services:
  openresty:
    image: openresty/openresty
    container_name: openresty
    environment:
      - "COLLECTOR=172.0.0.0"	#Your skywalking backend address
    volumes:
      - $PWD/startup.sh:/opt/bin/startup.sh
      - $PWD/skywalking-nginx-lua/lib:/usr/local/skywalking-nginx-lua/lib
      - $PWD/nginx.conf:/var/nginx/conf.d/nginx.conf
    command: /bin/bash /opt/bin/startup.sh
    ports:
      - 8080:8080
    networks:
      - swnet

networks:
  swnet:
```

* nginx.conf（**Probe access configuration**）

```
worker_processes  1;
daemon off;
error_log /dev/stdout debug;

events {
    worker_connections 1024;
}
http {
	#The location where you put the nginx probe library
    lua_package_path "/usr/local/skywalking-nginx-lua/lib/?.lua;;";
    # Buffer represents the register inform and the queue of the finished segment
    lua_shared_dict tracing_buffer 100m;

    # Init is the timer setter and keeper
    # Setup an infinite loop timer to do register and trace report.
    init_worker_by_lua_block {
        local metadata_buffer = ngx.shared.tracing_buffer
        
        #Parameter Description：
        #@serviceName :Service Name
        #
        metadata_buffer:set('serviceName', 'openresty-lua')
        -- Instance means the number of Nginx deployment, does not mean the worker instances
        metadata_buffer:set('serviceInstanceName', 'openresty-lua-instanceA')

        require("skywalking.util").set_randomseed()
        #Parameter Description：
        #Fill in the skywalking backend address
        require("skywalking.client"):startBackendTimer("http://${collector}:12800")
    }

    server {
        listen 8080;

        location /ingress {
            default_type text/html;

            rewrite_by_lua_block {
                require("skywalking.tracer"):start("openresty-lua:upstream_ip:port")
            }
            
            proxy_pass http://127.0.0.1:8080/tier2/lb;

            body_filter_by_lua_block {
                require("skywalking.tracer"):finish()
            }

            log_by_lua_block {
                require("skywalking.tracer"):prepareForReport()
            }
        }
    }
}
```

* startup.sh

```bash
sed -e "s%\${collector}%${COLLECTOR}%g" /var/nginx/conf.d/nginx.conf > /var/run/nginx.conf
/usr/bin/openresty -c /var/run/nginx.conf
```

