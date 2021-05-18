# Probe parameters configuration
The following explains in detail the parameters in the `agent/config/agent.config` configuration file：

| Configure Key | Environment Variables | Description | Default Value | For example |
| ----------- | ---------- | --------- | --------- |
| `agent.env_code` | `DX_ENV_ID ` |  **Environment Code**. | Take the `DX_ENV_ID` environment variable value from the deployment file.|`vay9f4-fhoayf9-f94gf8a-vakcbk3-fad33r3` |  
| `agent.properties[dx-application-name]`| `DX_APP_NAME` | **Custom created application name** | `Your_ApplicationName` | daoshop | 
| `agent.properties[dx-application-id]` | `DX_APP_ID` | **Custom created application ID** | `Your_ApplicationId` | `fafafa-fe3vaf-8vy8gi-f9ayf4`
| `agent.service_name` | `DX_SERVICE_NAME ` |The name of the service presented in the DMP Link Trace UI.Rules：**TenantCode::namespace(K8S)::serviceName**，via ```::``` link | `Your_ApplicationName` | daoshop-user-center | 
|`agent.instance_uuid` | `AGENT_INSTANCE_UUID` | Instance ID | `""` | daoshop-user-center-5c9644d98-765gj | 
| `collector.backend_service`| `DX_DMP_TRACING_SERVER` |The address of the back-end Collector collector, splitting the cluster address by a comma.| 127.0.0.1:10800 |`dmp-skywalking-oap-ng.dx-pilot.svc:11800`|
|`agent.sample_n_per_3_secs`| `SW_AGENT_SAMPLE` |**Default all**. The number of samples taken every 3 seconds. Set to a negative number for full sample take.|Not set|`agent.sample_n_per_3_secs=-1`|
|`agent.authentication`| `SW_AGENT_AUTHENTICATION` | **Not set by default**. Authentication token, consistent with the setting in the backend application.yml.|Not set| `agent.authentication = dangrous` |
|`agent.span_limit_per_segment`| `SW_AGENT_SPAN_LIMIT` |**Not set by default**. The maximum number of spans in a single segment. With this setting, you can save the application memory cost.|Not set |`agent.span_limit_per_segment=300`|
|`agent.ignore_suffix`| `SW_AGENT_IGNORE_SUFFIX` |**Not set by default**. Ignore segments that contain this prefix in the trace.|Not set|`agent.ignore_suffix=.jpg,.jpeg,.js,.css,.png,.bmp,.gif,.ico,.mp3,.mp4,.html,.svg`|
|`agent.is_open_debugging_class`| `SW_AGENT_OPEN_DEBUG` |` defaults to false`. If set to `true `, the probe will generate all enhanced class files in the `/debugging` directory.|Not set|`agent.is_open_debugging_class = true`|
| `logging.level`| `SW_LOGGING_LEVEL` |`Default is DEBUG`. Probe log level setting.|`DEBUG`|`INFO`|
| `logging.file_name`| `SW_LOGGING_FILE_NAME` |Log file name.|`skywalking-api.log`|`logging.file_name=skywalking-collector.log`|
| `logging.dir`| `SW_LOGGING_DIR` |The name of the log file directory. The default is to use `system.out` to output the log.|`""`| `myapp/` |
| `logging.max_file_size`| `SW_LOGGING_MAX_FILE_SIZE` |Log file size maximum. If the logs exceed this threshold, they will be shredded to a new log file.|`300 * 1024 * 1024`| `200 * 1024 * 1024` |