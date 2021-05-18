## Configuration parameters description
|  Environment variable name   | Default Value  |Description  | Is it necessary to  |
|  ----  | ----  |---- |----  |
| VEDFOLNIR_WS_DELAY  | 10 s | After the probe starts, the connection with vedfolnir-manager initializes with a delay  |no  |
| VEDFOLNIR_PROMETHEUS_PORT | 8888 | metrics Port number  | no  |
| VEDFOLNIR_IS_EXPOSE_PROMETHEUS  | false | Do you need to expose the metrics interface  | no  |
| DX_DMP_ACTUATOR_SERVER  | true | vedfolnir-manager url | no  |
| VEDFOLNIR_LOG_LEVEL  | DEBUG |  Probe Log Level| no  |
| VEDFOLNIR_LOG_PATH  | {vedfolnir-agent-path}/logs/vedfolnir-agent.log | Probe log file path  | no  |
| VEDFOLNIR_LOG_BACKUPS  | 50 | Number of probe log files retained  | no  |
| VEDFOLNIR_LOG_MAX_SIZE  | 50 * 1024 (KB) | Probe log file size  | no  |
