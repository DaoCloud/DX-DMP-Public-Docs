# 浏览器端监控接入

浏览器端探针接入需要改动一些项目源码，这里以 Daoshop-UI 项目为例。
Daoshop-UI 采用了 Vue 进行开发。

## 步骤
- 1 通过 npm 安装 skywalking-client-js sdk：
```bash
npm install skywalking-client-js --save
```

- 随后，在 main.js 中引入 ClientMonitor 并初始化
```js
ClientMonitor.register({
  // eslint-disable-next-line quotes
  service: `DX_ENV_ID::SW_AGENT_KUBE_NS::DX_SERVICE_NAME`, // 这里定义了三个环境变量，分别代表 DMP 环境Code，对应的 Kubenetes Namespace，服务名。后续会在脚本里面对其替换。
  pagePath: 'index.html',
  serviceVersion: 'v1.0.0',
  vue: Vue,
  useFmp: true,
});
```

- 配置 Nginx
由于 skywalking-client-js 会默认请求 /browser、/v3/segments 这两个接口，因此，我们需要通过 Nginx 转发到 OAP Server， nginx.conf 配置如下：
```bash
user  nginx;
worker_processes  1;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;

events {
    worker_connections  1024;
}

http {

    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    rewrite_log on;
    proxy_http_version 1.1;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    gzip on;
    gzip_min_length 1k;
    gzip_buffers 4 16k;
    gzip_http_version 1.0;
    gzip_comp_level 6;
    gzip_types text/plain application/javascript application/x-javascript text/css application/xml text/javascript application/x-httpd-php image/jpeg image/gif image/png;
    gzip_vary off;

    sendfile        on;

    keepalive_timeout  65;

    server {
        listen       80;
        server_name  localhost;
        location /browser {
            proxy_pass http://OAP_SERVER;
        }

        location /v3/segments {
            proxy_pass http://OAP_SERVER;
        }

        location / {
            root   /usr/share/nginx/html;
            try_files $uri $uri/ /index.html =404;
        }
        
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   /usr/share/nginx/html;
        }
    }
}
```
其中，OAP_SERVER 也会通过后续的脚本替换到真实的后端地址。

- 通过脚本替换边境变量，达到部署时动态给 DaoShop-UI 配置 Skywalking Client 相关配置的效果
```shell
#!/bin/bash

set -x
set -e

absolute_path="/etc/nginx/nginx.conf"

OAP_SERVER=${OAP_SERVER:="dmp-skywalking-oap.dmp-m.svc:12800"}
DX_SERVICE_NAME=${DX_SERVICE_NAME:="daoshop-ui"}
DX_ENV_ID=${DX_ENV_ID:="test"}
SW_AGENT_KUBE_NS=${SW_AGENT_KUBE_NS:="dmp-t"}
APP_NAME=${APP_NAME:="demo"}

mv ${absolute_path} ${absolute_path}.old
cat ${absolute_path}.old | 
    sed s#DX_SERVICE_NAME#${DX_SERVICE_NAME}#g   |
    sed s#DX_ENV_ID#${DX_ENV_ID}#g   |
    sed s#SW_AGENT_KUBE_NS#${SW_AGENT_KUBE_NS}#g   |
    sed s#OAP_SERVER#${OAP_SERVER}#g  >  ${absolute_path}

cat ${absolute_path}

filename=`ls /usr/share/nginx/html/js/ | grep "app.*.js$"`
absolute_path2='/usr/share/nginx/html/js/'${filename}

mv ${absolute_path2} ${absolute_path2}.old
cat ${absolute_path2}.old |
    sed s#DX_SERVICE_NAME#${DX_SERVICE_NAME}#g   |
    sed s#DX_ENV_ID#${DX_ENV_ID}#g   |
    sed s#SW_AGENT_KUBE_NS#${SW_AGENT_KUBE_NS}#g   |
    sed s#OAP_SERVER#${OAP_SERVER}#g  >  ${absolute_path2}

```

- 最后，在Docker 镜像启动的时候运行脚本：
```bash
FROM node:8 AS node-builder

COPY . /app/
RUN cd /app/ \
    && npm install -g cnpm --registry=https://registry.npm.taobao.org \
    && cnpm install node-sass \
    && cnpm install \
    && cnpm run build


FROM nginx:1.14
COPY --from=node-builder /app/ /app/
RUN mv /app/dist/* /usr/share/nginx/html/ \
    && mv /app/nginx.conf /etc/nginx/nginx.conf \
    && mv /app/run.sh /usr/share/ \
    && chmod +x /usr/share/run.sh  \
    && cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime \
    && echo "Asia/Shanghai" > /etc/timezone \
    && rm -rf /app
EXPOSE 80
CMD /usr/share/run.sh && nginx -g 'daemon off;'
```

至此，我们就可以通过 Docker Run并配置：
```bash
docker run -e DX_ENV_ID=test -e SW_AGENT_KUBE_NS=dmp-ns -e DX_SERVICE_NAME=daoshop-ui -e OAP_SERVER="dmp-skywalking-oap.dmp-m.svc:12800" -d daoshop-ui 
```

最后，可以访问 Daoshop-ui 并查看 DMP 界面是否有 daoshop-ui 的相关链路与监控。

## 参考
- [skywalking-client-js](https://github.com/apache/skywalking-client-js)
- [daoshop-ui源码](https://gitlab.daocloud.cn/bda/daoshop/shop-ui)