# Go 探针接入

如果你的服务涉及到 Go，可以参考该文档。本文以 Go gin应用为例讲解如何将 Gin 的endpoints 接入到分布式链路追踪。

## 前置条件

### 依赖包与环境安装

* 环境 go 1.15.4

* Go agent包可以通过一下方式安装：

```bash
go get -u github.com/SkyAPM/go2sky
```

本文档示例中其他库安装：

```
//go2sky的gin插件
go get -u github.com/SkyAPM/go2sky-plugins/gin/v3
go get -u github.com/gin-gonic/gin
```

## 探针接入

```go
package main

import (
	"log"
	"github.com/SkyAPM/go2sky"
	v3 "github.com/SkyAPM/go2sky-plugins/gin/v3"
	"github.com/SkyAPM/go2sky/reporter"
	"github.com/gin-gonic/gin"
)

func main() {
	/*
	参数说明：
	@serverAddr:SkyWalking 后端收集器地址
	*/
	re, err := reporter.NewGRPCReporter("172.16.200.201:11800")
	if err != nil {
		log.Fatalf("new reporter error %v \n", err)
	}
	defer re.Close()
	/*
	参数说明：
	@service 服务名字, 规则：租户Code::namespace(K8S)::服务名，通过 :: 链接。
	@opts 固定格式，一个Reporter的实例
	*/
	tracer, err := go2sky.NewTracer("devTenant::dmp-t::go-gin-test", go2sky.WithReporter(re))
	if err != nil {
		log.Fatalf("create tracer error %v \n", err)
	}
	gin.SetMode(gin.ReleaseMode)
	r := gin.New()
    
    //gin加载go2sky的中间件(插件)实现路径追踪
	r.Use(v3.Middleware(r, tracer))
	r.GET("/ping", func(c *gin.Context) {
		c.JSON(200, gin.H{
			"message": "hi gin",
		})
	})
	r.Run()
}
```

## 其他已经实现的插件：

1. [http server & client](http/README.md)
1. [gear](gear/README.md)
1. [go-resty](resty/README.md)
1. [go-micro](micro/README.md)
1. [go-restful](go-restful/README.md)

配合go agent的插件，使用以上go web框架开发的endpoints可以自动接入到分布式链路追踪。