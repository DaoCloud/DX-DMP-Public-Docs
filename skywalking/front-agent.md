# Browser-side monitoring access

The browser-side probe access requires some changes to the project source code, here is an example of Daoshop-UI project.
Daoshop-UI is developed using Vue.

## Steps
- 1 Install skywalking-client-js sdk via npm: ``bash
```bash
npm install skywalking-client-js --save
```

- Afterwards, introduce the ClientMonitor in main.js and initialize
```js
ClientMonitor.register({
  // eslint-disable-next-line quotes
  service: `DX_ENV_ID::SW_AGENT_KUBE_NS::DX_SERVICE_NAME`, // Three environment variables are defined here, representing the DMP Environment Code, the corresponding Kubenetes Namespace, and the service name. They will be replaced later in the script.
  pagePath: 'index.html',
  serviceVersion: 'v1.0.0',
  vue: Vue,
  useFmp: true,
});
```

- Configuring Nginx
Since skywalking-client-js will request the /browser and /v3/segments interfaces by default, we need to forward them to the OAP Server via Nginx. nginx.conf is configured as follows:
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
Among them, OAP_SERVER will also be replaced by subsequent scripts to the real backend address.

- By replacing the border variables with scripts, we can achieve the effect of dynamically configuring Skywalking Client related configuration to DaoShop-UI during deployment
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

- Finally, to run the script when the Docker image starts:
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

At this point, we can run through Docker and configure：
```bash
docker run -e DX_ENV_ID=test -e SW_AGENT_KUBE_NS=dmp-ns -e DX_SERVICE_NAME=daoshop-ui -e OAP_SERVER="dmp-skywalking-oap.dmp-m.svc:12800" -d daoshop-ui 
```

Finally, you can visit Daoshop-ui and see if the DMP interface has links and monitoring associated with daoshop-ui.

## Reference
- [skywalking-client-js](https://github.com/apache/skywalking-client-js)
- [daoshop-ui Source Code](https://gitlab.daocloud.cn/bda/daoshop/shop-ui)