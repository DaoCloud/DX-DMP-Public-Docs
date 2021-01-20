
# DotNet 探针接入

如果你的服务涉及到 DotNet，可以参考该文档。本文以 [ASP.NET Core](https://github.com/dotnet/aspnetcore)应用为例讲解如何将 ASP.NET Core的endpoints 接入到分布式链路追踪。本文档默认你使用了[官方教程搭建一个了ASP.NET Core 的Web API 应用](https://docs.microsoft.com/zh-cn/aspnet/core/tutorials/first-web-api?view=aspnetcore-3.1&tabs=visual-studio)

## 前置条件

### 依赖包与环境安装

* netcoreapp2.0/netframework4.6.1 或以上

* SkyAPM .NET Core Agent库可以通过一下方式安装(程序包管理器控制台执行)或使用Visual Studio的NuGet包管理器安装：

```shell
dotnet add package SkyAPM.Agent.AspNetCore
```

* 对于SkyAPM .NET Core Agent v1.0.0版本，它只支持SkyWalking 8.0或者更高版本，如果你使用Skywalking 7.0或者其他低版本，请使用SkyAPM .NET Core Agent v0.9.0

## 探针接入

1. 下载配置文件生成工具：`SkyAPM.DotNet.CLI`

   ```
   dotnet tool install -g SkyAPM.DotNet.CLI
   ```

2. 使用工具生成`skyapm.json`文件  `dotnet skyapm config [your_service_name] [your_servers]` ，执行示例：

   ```
   dotnet skyapm config sample_app 192.168.0.1:11800
   ```

   命令执行后会在当前目录下生成名叫`skyapm.json`的文件：

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

   配置文件解释：[**重要配置解释**](#重要配置解释)

3. 设置环境变量

   ```
   ASPNETCORE_HOSTINGSTARTUPASSEMBLIES=SkyAPM.Agent.AspNetCore
   ```

4. 启动应用

   启动应用后，你可以在`logs`文件夹下查看探针的运用情况

## 现在已经支持的库

- [支持列表](https://github.com/SkyAPM/SkyAPM-dotnet/blob/master/docs/Supported-list.md)

配合dotnet agent，以上库可以自动接入到分布式链路追踪。

## 重要配置解释

| 名字           | 解释                                                         | 默认值                 |
| -------------- | ------------------------------------------------------------ | ---------------------- |
| ServiceName    | 在 DMP 链路追踪 UI 中展示的服务名。以 @结尾代表该服务所在 DMP 租户 | your_service_name      |
| SamplePer3Secs | 每3秒采样数                                                  | -1                     |
| Percentage     | 采样百分比                                                   | -1.0                   |
| Level          | 日志级别                                                     | Information            |
| FilePath       | 日志保存路径                                                 | logs/skyapm-{Date}.log |
| Interval       | 每多少毫秒刷新                                               | 3000                   |
| Servers        | 后端Collector收集器的地址，通过逗号分割集群地址。            | localhost:11800        |
| Timeout        | 创建gRPC链接的超时时间，毫秒                                 | 10000                  |
| ConnectTimeout | gRPC最长链接时间，毫秒                                       | 10000                  |