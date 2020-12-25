# Python 探针接入

如果你的服务涉及到 Python，可以参考该文档。本文以 Flask 应用为例讲解如何将 Flask 的endpoints 接入到分布式链路追踪。

## 前置条件

### 依赖包与环境安装

* 环境 Python 3.7.9 

* Python agent包被发布在[Pypi](https://pypi.org/project/apache-skywalking/), 你可以通过`pip`安装：

```bash
#安装最新的版本并使用gRPC协议传输数据到OPA
pip install "apache-skywalking"

#安装最新的版本并使用http协议传输数据到OPA
pip install "apache-skywalking[http]"

#安装最新的版本并使用kafka协议传输数据到OPA
pip install "apache-skywalking[kafka]"

#安装特定的版本 x.y.z
pip install apache-skywalking==0.1.0
```

本文档示例中需要的库：

```
Flask==1.1.2
redis==3.5.3
```

## 目前支持的Python库

|  库名    |   版本   | 插件名字     |
| ---- | ---- | ---- |
| [http.server](https://docs.python.org/3/library/http.server.html) | Python 3.5 ~ 3.8 | `sw_http_server` |
| [urllib.request](https://docs.python.org/3/library/urllib.request.html) | Python 3.5 ~ 3.8 | `sw_urllib_request` |
| [requests](https://requests.readthedocs.io/en/master/) | >= 2.9.0 < 2.15.0, >= 2.17.0 <= 2.24.0 | `sw_requests` |
| [Flask](https://flask.palletsprojects.com/en/1.1.x/) | >=1.0.4 <= 1.1.2 | `sw_flask` |
| [PyMySQL](https://pymysql.readthedocs.io/en/latest/) | 0.10.0 | `sw_pymysql` |
| [Django](https://www.djangoproject.com/) | >=2.0 <= 3.1 | `sw_django` |
| [redis-py](https://github.com/andymccurdy/redis-py/) | 3.5.3 | `sw_redis` |
| [kafka-python](https://kafka-python.readthedocs.io/en/master/) | 2.0.1 | `sw_kafka` |
| [tornado](https://www.tornadoweb.org/en/stable/) | 6.0.4 | `sw_tornado` |
| [pika](https://pika.readthedocs.io/en/stable/) | 1.1.0 | `sw_rabbitmq` |
| [pymongo](https://pymongo.readthedocs.io/en/stable/) | 3.11.0 | `sw_pymongo` |
| [elasticsearch](https://github.com/elastic/elasticsearch-py) | 7.9.0 | `sw_elasticsearch` |
| [urllib3](https://urllib3.readthedocs.io/en/latest/) | >= 1.25.9 <= 1.25.10 | `sw_urllib3` |
| [sanic](https://sanic.readthedocs.io/en/latest/) | >= 20.3.0 <= 20.9.1 | `sw_sanic` |
| [aiohttp](https://sanic.readthedocs.io/en/latest/) | >= 3.7.3 | `sw_aiohttp` |
| [pyramid](https://trypyramid.com) | >= 1.9 | `sw_pyramid` |

目前只有以上列出的库能够被自动接入探针并接入到分布式链路追踪。

## Demo项目的探针接入

* demo结构

```
Flask-Agent
├── app.py
├── service.py
└── userRepo.py
```

其中 **userRepo.py** 示例代码如下：
```python
import json
import time
import redis
class redisConfig():
    def __init__(self,_host,_port,_db):
        self.r = redis.StrictRedis(host=_host, port=_port, db=_db)
        self.r.set('message', 'the message from redis')
        self.r.set('minquan','{"name":"minquan","passward":"123456","message":"redis cache data"}')
    def getConncet(self):
        return self.r

redisClient = redisConfig("localhost", 6379, 0)
class UserRepo:
    def __init__(self):
        pass
    def getUser(self,username):
        user = redisClient.r.get(username)
        user = json.loads(user)
        time.sleep(0.5)
        return user
```

其中 **service.py** 示例代码如下：

```python
from UserRepo import UserRepo
class UserServce:
    def __init__(self):
        self.userRepo = UserRepo()
    def getUser(self,userName):
        user = self.userRepo.getUser(username=userName)
        return user
```

其中 **app.py** 示例代码如下：

```python
from flask import Flask, url_for, request
import time
import json
from service import UserServce
app = Flask(__name__)
@app.route('/')
def hello_world():
    time.sleep(0.5)
    return "Hello_world"
    # return 'Hello World!'

@app.route("/user")
def getUser():
    username = request.args.get("username") or None
    if username ==None:
        return "no message"
    else:
        userService = UserServce()
        user = userService.getUser(username)
        userString = json.dumps(user)
        return userString
"""
导入依赖库 Python agent SDK 需要SkyWalking 8.0+ and Python 3.5+
参数说明：
@ collector :SkyWalking 后端收集器地址
@ service :服务名字, 以 @结尾代表该服务所在 DMP 租户。
"""
from skywalking import agent, config
config.init(collector='127.0.0.1:11800', service='python-flask@devTenant')
agent.start()
if __name__ == '__main__':
    app.run()
```

python flask 探针的接入需要随着 flask 应用一起启动，如上面代码的展示，此外还可以通过环境变量的方式进行配置信息设置。

## 支持的环境变量

|      环境变量| 介绍     | 默认值     |
| ---- | ---- | ---- |
| PSW_AGENT_NAME                        | 在 DMP 链路追踪 UI 中展示的服务名。以 @结尾代表该服务所在 DMP 租户(比如  **python-flask@devTenant** 代表 **devTenant** 租户).                     | `Python Service Name` |
| `SW_AGENT_INSTANCE` |DMP 链路追踪 UI 中展示的实例名。 | Randomly generated |
| `SW_AGENT_COLLECTOR_BACKEND_SERVICES` | 后端Collector收集器的地址，通过逗号分割集群地址。 | `127.0.0.1:11800` |
| `SW_AGENT_PROTOCOL` | 与后端OAP进行数据传输的协议： `http`, `grpc` or `kafka`,我们强烈建议在生产环境中使用`grpc`协议，它与`http`协议相比有更好的性能。 `kafka` 协议作为最后的选择使用 | `grpc` |
| `SW_AGENT_AUTHENTICATION              | 认证token，与后端application.yml中的设置保持一致。           | unset |
| `SW_AGENT_LOGGING_LEVEL` | 探针日志级别设置。可以是它们中的任意一个： `CRITICAL`, `FATAL`, `ERROR`, `WARN`(`WARNING`), `INFO`, `DEBUG` | `INFO` |
| `SW_AGENT_DISABLE_PLUGINS` | 禁用的插件，使用逗号分隔名字 | `''` |
| `SW_MYSQL_TRACE_SQL_PARAMETERS` | 是否收集SQL参数 | `False` |
| `SW_MYSQL_SQL_PARAMETERS_MAX_LENGTH` | 最大的SQL参数长度，超过的将会被截断                          | `512` |
| `SW_PYMONGO_TRACE_PARAMETERS` | 是否收集the filters of pymongo | `False` |
| `SW_PYMONGO_PARAMETERS_MAX_LENGTH` | The maximum length of the collected filters, filters longer than the specified length will be truncated |  `512` |
| `SW_IGNORE_SUFFIX` | 如果第一个span的操作名包含在这个集合中，这个segement将会被忽视 | `.jpg,.jpeg,.js,.css,.png,.bmp,.gif,.ico,.mp3,.mp4,.html,.svg` |
| `SW_FLASK_COLLECT_HTTP_PARAMS`| 是否允许flask插件收集中的请求参数 | `false` |
| `SW_DJANGO_COLLECT_HTTP_PARAMS`| 是否允许Django插件收集请求参数 | `false` |
| `SW_HTTP_PARAMS_LENGTH_THRESHOLD`| 当`COLLECT_HTTP_PARAMS` 启用时, 这个参数限制多少个字符会发送给后端,如果使用了负数，表示发送全部数据, NB. 添加此配置项是为了提高性能. | `1024` |
| `SW_CORRELATION_ELEMENT_MAX_NUMBER`| 在相关联的context中最大数量的element                         | `3` |
| `SW_CORRELATION_VALUE_MAX_LENGTH`| 在相关联的context中的element的最大长度                       | `128` |
| `SW_TRACE_IGNORE`| 是否需要忽略trace | `false` |
| `SW_TRACE_IGNORE_PATH`| 可以配置多个URL模板，去匹配不想被追踪的endpoint. 当前的匹配规则使用Ant Path match style , 比如 /path/*, /path/**, /path/?. | `''` |
| `SW_ELASTICSEARCH_TRACE_DSL`| 如果true, 追踪所有接入ElasticSearch的 DSL(Domain Specific Language) | `false` |
| `SW_KAFKA_REPORTER_BOOTSTRAP_SERVERS` | 与kafka集群建立连接的 列表. 具体格式： host1:port1,host2:port2,... | `localhost:9092` |
| `SW_KAFKA_REPORTER_TOPIC_MANAGEMENT` | 为服务实例报告和注册指定Kafka主题名 | `skywalking-managements` |
| `SW_KAFKA_REPORTER_TOPIC_SEGMENT` | 指定用于跟踪数据的Kafka主题名 | `skywalking-segments` |
| `SW_KAFKA_REPORTER_CONFIG_key` | 初始化Kafka生产端的配置. 支持基本的参数 ( `str`, `bool`, or `int`) 具体请看 [这里](https://kafka-python.readthedocs.io/en/master/apidoc/KafkaProducer.html#kafka.KafkaProducer) | unset |