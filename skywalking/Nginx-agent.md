# Nginx 探针接入(容器模式)

如果你的应用使用到了Nginx，可以参考该文档。本文以 OpenResty为例讲解如何将经过nginx的endpoints接入到分布式链路追踪。

## 前置条件

### 依赖包与环境安装

* docker 19及以上

* Nginx agent以lua脚本形式实现的，通过git下载需要的lua module：

```bash
git clone https://github.com/apache/skywalking-nginx-lua.git
```

## 探针接入

你需要构建的目录结构：

```
your folder
├── skywalking-nginx-lua			#git拉取的lua gen仓库
├── nginx.conf
├── startup.sh
└── docker-compose.yaml
```

具体的三个文件：

* docker-compose.yaml

```yaml
version: '2.2'
services:
  openresty:
    image: openresty/openresty
    container_name: openresty
    environment:
      - "COLLECTOR=172.0.0.0"	#你的skywalking后端地址
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

* nginx.conf（**探针接入配置**）

```
worker_processes  1;
daemon off;
error_log /dev/stdout debug;

events {
    worker_connections 1024;
}
http {
	#你放置nginx探针库的位置
    lua_package_path "/usr/local/skywalking-nginx-lua/lib/?.lua;;";
    # Buffer represents the register inform and the queue of the finished segment
    lua_shared_dict tracing_buffer 100m;

    # Init is the timer setter and keeper
    # Setup an infinite loop timer to do register and trace report.
    init_worker_by_lua_block {
        local metadata_buffer = ngx.shared.tracing_buffer
        
        #参数说明：
        #@serviceName :服务名字
        #
        metadata_buffer:set('serviceName', 'openresty-lua')
        -- Instance means the number of Nginx deployment, does not mean the worker instances
        metadata_buffer:set('serviceInstanceName', 'openresty-lua-instanceA')

        require("skywalking.util").set_randomseed()
        #参数说明：
        #填入skywalking后端地址
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

