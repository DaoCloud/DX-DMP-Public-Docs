# Python Probe access

If your service involves Python, you can refer to this document. This article uses a Flask application as an example to explain how to connect Flask's endpoints to distributed link tracing.

## Pre-requisites

### Dependency packages and environment installation

* Environment Python 3.7.9  

* The Python agent package is distributed at [Pypi](https://pypi.org/project/apache-skywalking/), which you can install via `pip`：

```bash
#Install the latest version and transfer data to OPA using the gRPC protocol
pip install "apache-skywalking"

#Install the latest version and transfer data to OPA using the http protocol
pip install "apache-skywalking[http]"

#Install the latest version and transfer data to OPA using the kafka protocol
pip install "apache-skywalking[kafka]"

#Install a specific version x.y.z
pip install apache-skywalking==0.1.0
```

Libraries needed for the examples in this document：

```
Flask==1.1.2
redis==3.5.3
```

## Currently supported Python libraries

|  Library name    |   Versions   |      Plugin Name     |
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

Currently only the libraries listed above can be automatically accessed by the probe and connected to the distributed link trace.

## Probe access for Demo projects

* demo structure

```
Flask-Agent
├── app.py
├── service.py
└── userRepo.py
```

where **userRepo.py** sample code is as follows：
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
        if user is None:
            user = "no message";
        else:
            user = json.loads(user)
        time.sleep(0.5)
        return user
```

where **service.py** sample code is as follows：

```python
from UserRepo import UserRepo
class UserServce:
    def __init__(self):
        self.userRepo = UserRepo()
    def getUser(self,userName):
        user = self.userRepo.getUser(username=userName)
        return user
```

where **app.py** sample code is as follows：

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
Importing the dependency library Python agent SDK requires SkyWalking 8.0+ and Python 3.5+
Parameter description.
@ collector :SkyWalking backend collector address
@ service :Service name, ending with @ for the DMP tenant where the service is hosted.
"""
from skywalking import agent, config
config.init(collector='127.0.0.1:11800', service='python-flask@devTenant')
agent.start()
if __name__ == '__main__':
    app.run()
```

Access to the python flask probe needs to be started along with the flask application, as shown in the code above, in addition to the configuration information set by way of environment variables.

## Supported Environment Variables

| Environment Variables | Introduction | Defaults |
| ---- | ---- | ---- |
| PSW_AGENT_NAME | The name of the service presented in the DMP link tracking UI. Rule: tenantCode::namespace(K8S)::serviceName, via :: link. (e.g. **devTenant::system::python-flask** stands for **devTenant** tenant, **system** namespace).                     | `Python Service Name` |
| `SW_AGENT_INSTANCE` | The name of the instance displayed in the DMP link tracking UI. | `Python Service Name` | `SW_AGENT_INSTANCE`
| `SW_AGENT_COLLECTOR_BACKEND_SERVICES` | The address of the back-end Collector collector, split by a comma. | `127.0.0.1:11800` |
| `SW_AGENT_PROTOCOL` | Protocols for data transfer with back-end OAP: `http`, `grpc` or `kafka`, we strongly recommend using the `grpc` protocol in production environments, it has better performance compared to the `http` protocol. The `kafka` protocol is used as the last option | `grpc` |
| `SW_AGENT_AUTHENTICATION | authentication token, consistent with the settings in the back-end application.yml.           | unset |
| `SW_AGENT_LOGGING_LEVEL` | The probe logging level setting. Can be any of them: `CRITICAL`, `FATAL`, `ERROR`, `WARN`(`WARNING`), `INFO`, `DEBUG` | `INFO` |
| `SW_AGENT_DISABLE_PLUGINS` | Disabled plugins, use comma-separated names | `''` |
| `SW_MYSQL_TRACE_SQL_PARAMETERS` | Whether to collect SQL parameters | `False` |
| `SW_MYSQL_SQL_PARAMETERS_MAX_LENGTH` | Maximum SQL parameter length, anything longer than that will be truncated | `512` |
| `SW_PYMONGO_TRACE_PARAMETERS` | Whether to collect the filters of pymongo | `False` |
| `SW_PYMONGO_PARAMETERS_MAX_LENGTH` | The maximum length of the collected filters, filters longer than the specified length will The maximum length of the collected filters, filters longer than the specified length will be truncated | `512` |
| `SW_IGNORE_SUFFIX` | If the operation name of the first span is included in this collection, this segement will be ignored | `.jpg,.jpeg,.js,.css,.png,.bmp,.gif,.ico,.mp3,.mp4,.html,.svg` |
| `SW_FLASK_COLLECT_HTTP_PARAMS`| Whether to allow request parameters in flask plugin collection | `false` |
| `SW_DJANGO_COLLECT_HTTP_PARAMS`| Whether to allow the Django plugin to collect request parameters | `false` |
| `SW_HTTP_PARAMS_LENGTH_THRESHOLD`| When `COLLECT_HTTP_PARAMS` is enabled, this parameter limits how many characters will be sent to the backend, and if a negative number is used, it means that all data is sent, NB. This configuration item is added to improve performance. | `1024` |
| `SW_CORRELATION_ELEMENT_MAX_NUMBER`| The maximum number of elements in the associated context | `3` |
| `SW_CORRELATION_VALUE_MAX_LENGTH`| the maximum length of elements in the associated context | `128` |
| `SW_TRACE_IGNORE`| whether to ignore traces | `false` |
| `SW_TRACE_IGNORE_PATH`| Multiple URL templates can be configured to match endpoints that do not want to be traced. The current match rule uses the Ant Path match style , e.g. /path/*, /path/**, /path/? | `''` |
| ``SW_ELASTICSEARCH_TRACE_DSL`` | if true, tracks all DSLs (Domain Specific Language) that access ElasticSearch | ``false`` |
| `SW_KAFKA_REPORTER_BOOTSTRAP_SERVERS` | A list of connections to the kafka cluster. The specific format: host1:port1,host2:port2,... | `localhost:9092` |
| `SW_KAFKA_REPORTER_TOPIC_MANAGEMENT` | Specify Kafka topic names for service instance reporting and registration | `skywalking-managements` |
| `SW_KAFKA_REPORTER_TOPIC_SEGMENT` | Specify the Kafka topic name for tracking data | `skywalking-segments` |
| `SW_KAFKA_REPORTER_CONFIG_key` | Initialize the Kafka production side configuration. Basic parameters are supported ( `str`, `bool`, or `int`) See [here](https://kafka-python.readthedocs.io/en/master/apidoc/KafkaProducer.html#kafka. KafkaProducer) | unset |

