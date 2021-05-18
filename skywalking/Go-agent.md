# Go Probe Access

If you have a service that involves Go, you can refer to this document. This article uses a Go gin application as an example to explain how to access Gin's endpoints to a distributed link trace.

## Prerequisites

### Dependency packages and environment installation

* Environment go 1.15.4

* The Go agent package can be installed in the following way.

```bash
go get -u github.com/SkyAPM/go2sky
```

Other library installations in this document example.

```
//go2sky's gin plugin
go get -u github.com/SkyAPM/go2sky-plugins/gin/v3
go get -u github.com/gin-gonic/gin
```

## Probe access

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
	Parameter Description：
	@serverAddr:SkyWalking back-end collector address
	*/
	re, err := reporter.NewGRPCReporter("172.16.200.201:11800")
	if err != nil {
		log.Fatalf("new reporter error %v \n", err)
	}
	defer re.Close()
	/*
	Parameter description.
	@service service name, rule: tenantCode::namespace(K8S)::service name, linked via ::.
	@opts Fixed format, an instance of Reporter
	*/
	tracer, err := go2sky.NewTracer("devTenant::dmp-t::go-gin-test", go2sky.WithReporter(re))
	if err != nil {
		log.Fatalf("create tracer error %v \n", err)
	}
	gin.SetMode(gin.ReleaseMode)
	r := gin.New()
    
    //gin loads go2sky's middleware (plugin) to implement path tracing
	r.Use(v3.Middleware(r, tracer))
	r.GET("/ping", func(c *gin.Context) {
		c.JSON(200, gin.H{
			"message": "hi gin",
		})
	})
	r.Run()
}
```

## Other plugins that have been implemented：

1. [http server & client](http/README.md)
1. [gear](gear/README.md)
1. [go-resty](resty/README.md)
1. [go-micro](micro/README.md)
1. [go-restful](go-restful/README.md)

With the go agent plug-in, endpoints developed using the above go web framework can be automatically connected to the distributed link tracking.