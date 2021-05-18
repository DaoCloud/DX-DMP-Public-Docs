
# DotNet Probe access

If your service involves DotNet, you can refer to this document. This article uses the [ASP.NET Core](https://github.com/dotnet/aspnetcore) application as an example to explain how to connect ASP.NET Core endpoints to distributed link tracing. By default, you use the [official tutorial to build a Web API application for ASP.NET Core](https://docs.microsoft.com/zh-cn/aspnet/core/tutorials/first-web-api?view=aspnetcore- 3.1&tabs=visual-studio)

## Pre-requisites

### Dependency packages and environment installation

* netcoreapp2.0/netframework4.6.1 or above

* The SkyAPM .NET Core Agent library can be installed in the following way (package manager console execution) or using Visual Studio's NuGet package manager：

```shell
dotnet add package SkyAPM.Agent.AspNetCore
```

* NET Core Agent v1.0.0, it only supports SkyWalking 8.0 or higher, if you are using Skywalking 7.0 or other low version, please use SkyAPM .NET Core Agent v0.9.0

## Probe access

1. Download the profile generation tool：`SkyAPM.DotNet.CLI`

   ```
   dotnet tool install -g SkyAPM.DotNet.CLI
   ```

2. Use the tool to generate the `skyapm.json` file `dotnet skyapm config [your_service_name] [your_servers]` and execute the example：

   ```
   dotnet skyapm config sample_app 192.168.0.1:11800
   ```

   The command will generate a file named `skyapm.json` in the current directory after execution：

   ```
   {
     "SkyWalking": {
       "ServiceName": "your service name",
       "Namespace": "",
       "HeaderVersions": [
         "sw8"
       ],
       "Sampling": {
         "SamplePer3Secs": -1,
         "Percentage": -1.0
       },
       "Logging": {
         "Level": "Information",
         "FilePath": "logs\\skyapm-{Date}.log"
       },
       "Transport": {
         "Interval": 3000,
         "ProtocolVersion": "v8",
         "QueueSize": 30000,
         "BatchSize": 3000,
         "gRPC": {
           "Servers": "172.0.0.0:11800",
           "Timeout": 10000,
           "ConnectTimeout": 10000,
           "ReportTimeout": 600000,
           "Authentication": ""
         }
       }
     }
   }
   ```

   Configuration file explanation: [**Important configuration explanation**](#Important configuration explanation)

3. Set environment variables

   ```
   ASPNETCORE_HOSTINGSTARTUPASSEMBLIES=SkyAPM.Agent.AspNetCore
   ```

4. Launch Application

   After starting the application, you can view the probe usage in the `logs` folder

## Libraries now supported

- [Support List](https://github.com/SkyAPM/SkyAPM-dotnet/blob/master/docs/Supported-list.md)

Together with dotnet agent, the above libraries can be automatically connected to distributed link tracing.

## Important configurations explained

| Name           | Explanation                                                  | Default                |
| -------------- | ------------------------------------------------------------ | ---------------------- |
| ServiceName    | The name of the service displayed in the DMP link tracking UI. Ends with @ for the DMP tenant where the service is located | your_service_name |
| SamplePer3Secs | Number of samples per 3 seconds                              | -1                     |
| Percentage     | Sampling Percentage                                          | -1.0                   |
| Level          | Log level                                                    | Information            |
| FilePath       | path to logs/skyapm-{Date}.log |
| Interval       | How many milliseconds to refresh                             | 3000                   |
| Servers        | The address of the back-end Collector collector, split by a comma.  | localhost:11800 |
| Timeout        | The timeout to create a gRPC link, in milliseconds           | 10000                  |
| ConnectTimeout | The maximum link time for gRPC, in milliseconds              | 10000                  |