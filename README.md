[![Build Status](https://travis-ci.org/gmsec/gmsec.svg?branch=master)](https://travis-ci.org/gmsec/gmsec)
[![Go Report Card](https://goreportcard.com/badge/github.com/gmsec/gmsec)](https://goreportcard.com/report/github.com/gmsec/gmsec)
[![codecov](https://codecov.io/gh/gmsec/gmsec/branch/master/graph/badge.svg)](https://codecov.io/gh/gmsec/gmsec)
[![GoDoc](https://godoc.org/github.com/gmsec/gmsec?status.svg)](https://godoc.org/github.com/gmsec/gmsec)


# [gmsec](https://github.com/gmsec/gmsec)

### 特点

- 打通grpc + gin，同时支持grpc 跟 restful 模式
- grpc , gin 公用端口
- gorm 嵌入，自动数据库代码生成

### golang 微服务集成框架 

- [grpc](https://github.com/grpc/grpc-go)
- [gorm 自动构建(gormt)](https://github.com/xxjwxc/gormt)
- [gin 参数自动绑定(ginrpc)](https://github.com/xxjwxc/ginrpc)
- [dns 注册发现(mdns)](https://github.com/asim/mdns)
- [markdown/mindoc 文档自动导出](https://github.com/grpc)
- [支持etcd/nacos服务注册于发现](https://github.com/gmsec/goplugins)

## 安装

- install

- proto环境安装

```
 make install 
```

- 本地环境搭建(gmsec为例)

```
make gen
```

- 新建一个服务
```
make new service=[服务名]
```

## proto定义

```
syntax = "proto3"; // 指定proto版本
package proto;     // 指定包名
option go_package = ".;proto"; // 指定路径

// 定义Hello服务
service Hello {
    // 定义SayHello方法
    rpc SayHello(HelloRequest) returns (HelloReply) {}
}
// HelloRequest 请求结构
message HelloRequest {
    string name = 1; // 名字
}
// HelloReply 响应结构
message HelloReply {
    string message = 1; // 消息
}
```

## 服务端代码示例

``` go
package main

import (
	"context"
	"fmt"
	proto "gmsec/rpc/gmsec"

	"github.com/gmsec/goplugins/api"
	"github.com/gin-gonic/gin"
	"github.com/gmsec/goplugins/plugin"
	"github.com/gmsec/micro"
	"github.com/xxjwxc/ginrpc"
)

func main() {
	// grpc 相关 初始化服务
	service := micro.NewService(
		micro.WithName("lp.srv.eg1"),
	)
	h := new(hello)
	proto.RegisterHelloServer(service.Server(), h) // 服务注册
	// ----------- end

	// gin 相关
	base := ginrpc.New(ginrpc.WithCtx(api.NewAPIFunc), ginrpc.WithDebug(dev.IsDev()))
	router := gin.Default()
	v1 := router.Group("/xxjwxc/api/v1")
	base.Register(v1, h) // 对象注册
	// ------ end

	plg, _ := plugin.Run(plugin.WithMicro(service),// grpc 入口
		plugin.WithGin(router), // http 入口
        plugin.WithAddr(":8080")) // 开始服务(公用端口)
    
	plg.Wait() // 等待结束(ctrl+c)
    
	fmt.Println("done")
}

// Ctx gin.Context 到 context.Context 的转换
func Ctx(c *gin.Context) interface{} {
	return context.Background()
}
```

## 客户端代码:

``` go
package main

import (
	"context"
	"fmt"
	proto "gmsec/rpc/gmsec"

	"github.com/gmsec/micro"
)

func main() {
    // reg := registry.NewDNSNamingRegistry()
	// grpc 相关 注册服务发现等
	micro.NewService(
        micro.WithName("lp.srv.eg1"),
        // micro.WithRegisterTTL(time.Second*30),      //指定服务注册时间
        // micro.WithRegisterInterval(time.Second*15), //让服务在指定时间内重新注册
        // micro.WithRegistryNaming(reg),
	)
	// ----------- end

	say := proto.GetHelloClient()
	ctx := context.Background()
	resp, _ := say.SayHello(ctx, &proto.HelloRequest{Name:"xxjwxc"})
	fmt.Println("result:", resp)
}
```

## 更多示例 => [传送门](https://github.com/gmsec/gmsec/tree/master/example)

## 正在做
- etcdv3 


## [传送门](https://github.com/gmsec)
