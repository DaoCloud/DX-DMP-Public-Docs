# Alarm Module

The basic principle of sending alerts by Skywalking is to poll the link trace data collected by skywalking-oap at regular intervals, and then send an alert message if the threshold is reached according to the configured alert rules (e.g. service response time, service response time percentage).

Sending alert messages is done by calling the webhook interface in an asynchronous way with a thread pool (the specific webhook interface can be defined by yourself and processed according to the received data), so that developers can write various alert methods in the specified webhook interface by themselves, and the default support for nailing, enterprise WeChat, Slack and other docking. If you need to alert by other means such as email, you need to interface to webhook.

The core of alerting is driven by a set of rules, which are defined in the `config/alarm-settings.yml` file inside the unpacked installation, and opened as follows: rules is the list of alerting rules that need to be configured; the first rule, 'endpoint_percent_rule', is the rule name, which cannot be The first rule, 'endpoint_percent_rule', is a rule name that cannot be repeated and must end with '_rule'.

```yaml
rules:
  # Rule unique name, must be ended with `_rule`.（The alert rule name should be unique and must end with `_rule`.）
  endpoint_percent_rule:
    # Metrics value need to be long, double or int. （The value of the indicator needs to be long, double or int）
    metrics-name: endpoint_percent
    threshold: 75
    op: <
    # The length of time to evaluate the metrics（Alarm check period: how often to check whether the current indicator data conforms to the alarm rules, in minutes）
    period: 10
    # How many times after the metrics match the condition, will trigger alarm（How many times the alarm is triggered after the accumulated alarm value is reached）
    count: 3
    # How many times of checks, the alarm keeps silence after alarm triggered, default as same as period.（Ignore the period of the same alarm message, the default is the same as the alarm check period）
    silence-period: 10
    
  service_percent_rule:
    metrics-name: service_percent
    # [Optional] Default, match all services in this metrics（Optional, matches all services by default）
    include-names:
      - service_a
      - service_b
    threshold: 85
    op: <
    period: 10
    count: 4
```

The definition of alert rules is divided into two parts.
1. Alert rules. They define how metric alerts should be triggered and what conditions should be considered.
2. Web hooks. Which service endpoints need to be informed when a warning is triggered, you can receive alert messages via hooks and then push them to other alerting platforms on demand.

## Alarm rules

The main warning rules are as follows：
- **Rule name**. A unique name to be displayed in alert messages. Must end with `_rule`. The specified rule (different from the rule name, here is the map of the rule in the corresponding alarm, see [backend-alarm.md](https://github.com/apache/skywalking/blob/master/docs/en/setup/backend/) backend-alarm.md#list-of-all-potential-metrics-name), some of the common ones, endpoint_percent_rule - endpoint corresponding half-percent ratio alerts, service_ percent_rule - service corresponding percentage alert)
- **Metrics name**. Also the name of the metric in the OAL script. Only long,double and int types are supported. See [List of all possible metric names](#List of all possible metric names) for details.
- **Include names**. List of services alerted using this rule. For example, service name, endpoint name.
- **Threshold**. Threshold, matching the metrics-name and the following comparison symbols
- **OP**. Operators, support `>`, `<`, `=`. Feel free to contribute all operators. For example, metrics-name: endpoint_percent, threshold: 75, op: < , means if the corresponding duration is less than 75% of the average, an alert will be sent
- **Period**. The how-long-alarm rule needs to be verified. This is a time window that matches the backend deployment environment time.    
- **Count**. In a Period window, if **value**s exceeds the Threshold value (by op) and reaches the Count value, an alert needs to be sent.
- **Silence period**. After an alarm is triggered in Time N, no alarm is sent in the **TN -> TN + period** phase. By default, it is the same as **Period**, which means that the same alarm (having the same Id in the same Metrics name) will be triggered only once in the same Period.


## Default Alarm Rules
For convenience, we provide the default `alarm setting.yml` file in the distribution, including the following rules
1. Service response time averaged more than 1 second in the last 3 minutes.
1. Service success rate is less than 80% in the last 2 minutes.
1. Service 90% response time was less than 1000 ms in the last 3 minutes.
1. Service instance average response time exceeded 1 second within the last 2 minutes.
1. Endpoint average response time exceeded 1 second in the past 2 minutes.

## List of all possible metric names
These metric names are defined in the [OAL script](https://github.com/apache/skywalking/blob/master/oap-server/server-starter/src/main/resources/official_analysis .oal), now the metrics from **Service**, **Service Instance**, **Endpoint** can be used for alerting, we will extend them in a later version.

## Webhook
SkyWalking's alerting webhook requires the recipient to be a web container. The URL to accept the request needs to be configured in the `config/alarm-settings.yml` file at deployment startup：

```yaml
# Sample alarm rules.
rules:
  service_resp_time_rule:
    metrics-name: service_resp_time
    # [Optional] Default, match all services in this metrics（可选项，默认匹配所有服务）
    include-names:
      - dubbox-provider
      - dubbox-consumer
    threshold: 1000
    op: ">"
    period: 10
    count: 1

👇 Configure webhook to receive the URL of the alert rule。
webhooks: 
  - http://127.0.0.1:8090/notify/
  - http://127.0.0.1:8888/go-wechat/
```

Alarm messages are sent via HTTP requests with the request method `POST` and `Content-Type` of `application/json`,
JSON format can be found in `List<org.apache.skywalking.oap.server.core.alarm.AlarmMessage`, containing the following information.
- **scopeId**, **scope**. For all available Scopes, see `org.apache.skywalking.oap.server.core.source.DefaultScopeDefine`.
- **name**. Entity name of the target Scope.
- **id0**. ID of the Scope entity.
- **id1**. Not used.
- **ruleName**. Name of the alarm rule triggered.
- **alarmMessage**. Content of the alarm message.
- **startTime**. The time of the alarm.

Here is an example of an alert message pushed out by POST：
```json
[{
	"scopeId": 1, 
    "scope": "SERVICE",
    "name": "serviceA", 
	"id0": 12,  
	"id1": 0,  
    "ruleName": "service_resp_time_rule",
	"alarmMessage": "alarmMessage xxxx",
	"startTime": 1560524171000
}, {
	"scopeId": 1,
    "scope": "SERVICE",
    "name": "serviceB",
	"id0": 23,
	"id1": 0,
    "ruleName": "service_resp_time_rule",
	"alarmMessage": "alarmMessage yyy",
	"startTime": 1560524171000
}]
```

The following is a reference alarm configuration：

```yml
# Sample alarm rules.
rules:
  # Rule unique name, must be ended with `_rule`.
  service_resp_time_rule:
    metrics-name: service_resp_time
    op: ">"
    threshold: 500
    period: 10
    count: 1
    silence-period: 5
    message: Response time of service {name} is more than 1000ms in last 10 minutes.
  service_sla_rule:
    # Indicator value need to be long, double or int
    metrics-name: service_sla
    op: "<"
    threshold: 8000
    # The length of time to evaluate the metric
    period: 10
    # How many times after the metric match the condition, will trigger alarm
    count: 2
    # How many times of checks, the alarm keeps silence after alarm triggered, default as same as period.
    silence-period: 3
    message: Successful rate of service {name} is lower than 80% in last 10 minutes.
  service_p90_sla_rule:
    # Indicator value need to be long, double or int
    metrics-name: service_p90
    op: ">"
    threshold: 500
    period: 10
    count: 1
    silence-period: 5
    message: 90% response time of service {name} is lower than 1000ms in last 10 minutes
  service_instance_resp_time_rule:
    metrics-name: service_instance_resp_time
    op: ">"
    threshold: 500
    period: 10
    count: 1
    silence-period: 5
    message: Response time of service instance {name} is more than 1000ms in last 10 minutes.
  endpoint_avg_rule:
    metrics-name: endpoint_avg
    op: ">"
    threshold: 500
    period: 10
    count: 1
    silence-period: 5
    message: Response time of endpoint {name} is more than 1000ms in last 10 minutes.

👇 Webhook for custom logic
webhooks:
#  - http://127.0.0.1/notify/
#  - http://127.0.0.1/go-wechat/

👇 Docking enterprise wechatHook
wechatHooks:
  textTemplate: |-
    {
      "msgtype": "text",
      "text": {
        "content": "Apache SkyWalking Alarm: \n %s."
      }
    }
  webhooks:
#    - https://qyapi.weixin.qq.com/cgi-bin/webhook/send?key=dummy_key

👇 Docking enterprise dingtalkHook
dingtalkHooks:
  textTemplate: |-
    {
      "msgtype": "text",
      "text": {
        "content": "Apache SkyWalking Alarm: \n %s."
      }
    }
  webhooks:
#    - url: https://oapi.dingtalk.com/robot/send?access_token=dummy_token
#      secret: dummysecret

```