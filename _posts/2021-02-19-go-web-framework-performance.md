---
layout: post
title: Go Framework performance test
categories: [go]
tags: [go]
sidebar: []
---
## 框架性能测试

说明：

- 测试工具wrk
- wrk版本3.1.0

测试框架
- Gin
- Fiber
- GoFrame
- Beego
- Echo

硬件：
- Windows 4核 16G
- Docker虚拟机

```bash
Tasks:  22 total,   1 running,  21 sleeping,   0 stopped,   0 zombie
%Cpu0  :  0.0 us,  0.7 sy,  0.0 ni, 99.3 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu1  :  0.3 us,  0.0 sy,  0.0 ni, 99.7 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu2  :  0.7 us,  1.3 sy,  0.0 ni, 98.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu3  :  0.3 us,  0.3 sy,  0.0 ni, 99.3 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem : 12989752 total, 10216648 free,   620332 used,  2152772 buff/cache
KiB Swap:  4194304 total,  4194304 free,        0 used. 11680228 avail Mem 
```

测试指标
- QPS

测试报告
- WRK： 线程数=80 并发=1000 持续时间=10秒


- Gin:
```bash
[root@eff147d04140 ~]# wrk -t80 -c1000 -d10 http://localhost:9981
Running 10s test @ http://localhost:9981
  80 threads and 1000 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency    27.55ms   49.62ms 257.29ms   89.06%
    Req/Sec     1.43k     2.47k   40.00k    95.32%
  1114357 requests in 9.87s, 153.03MB read
Requests/sec: 112869.06
Transfer/sec:     15.50MB
```

- Fiber
```bash
[root@eff147d04140 ~]# wrk -t80 -c1000 -d10 http://localhost:9983
Running 10s test @ http://localhost:9983
  80 threads and 1000 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency    22.58ms   54.64ms 245.46ms   92.87%
    Req/Sec     2.19k     3.31k   61.77k    95.21%
  1692207 requests in 9.89s, 208.18MB read
Requests/sec: 171139.64
Transfer/sec:     21.05MB
```

- Beego
```bash
[root@eff147d04140 ~]# wrk -t80 -c1000 -d10 http://localhost:9984
Running 10s test @ http://localhost:9984
  80 threads and 1000 c s   68.78ms 308.13ms   91.32%
    Req/Sec     1.30k     2.22k   33.89k    95.58%
  1008239 requests in 9.93s, 138.46MB read
Requests/sec: 101491.03
Transfer/sec:     13.94MB
```

- GoFrame
```bash
[root@eff147d04140 ~]# wrk -t80 -c1000 -d10 http://localhost:9982
Running 10s test @ http://localhost:9982
  80 threads and 1000 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   327.14ms  108.07ms 480.05ms   88.64%
    Req/Sec     0.86k     1.85k   24.56k    95.38%
  651566 requests in 9.87s, 125.52MB read
Requests/sec:  65989.86
Transfer/sec:     12.71MB 
```

- Echo
```bash
[root@eff147d04140 ~]# wrk -t80 -c1000 -d10 http://localhost:9989
Running 10s test @ http://localhost:9989
  80 threads and 1000 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency    30.29ms   54.49ms 267.83ms   89.08%
    Req/Sec     1.48k     2.78k   42.44k    95.48%
  1154911 requests in 9.87s, 159.70MB read
Requests/sec: 117046.02
Transfer/sec:     16.19MB
```

测试代码
- Gin
```bash
package main

import (
	"github.com/gin-gonic/gin"
	"net/http"
)

func main() {

	app := gin.New()
	app.GET("/", func(ctx *gin.Context) {
		ctx.JSON(http.StatusOK, gin.H{"msg": "successful."})
	})
	app.Run(":9981")
}
```

- Echo
```bash
package main

import (
	"github.com/labstack/echo/v4"
	"net/http"
)

func main() {
	app := echo.New()
	app.GET("/", func(c echo.Context) error {
		return c.JSON(http.StatusOK, echo.Map{
			"msg": "successful.",
		})
	})
	app.Start(":9989")
}
```

- Fiber
```bash
package main

import "github.com/gofiber/fiber/v2"

func main() {
	app := fiber.New()
	app.Get("/", func(ctx *fiber.Ctx) error {
		return ctx.JSON(fiber.Map{"msg": "successful."})
	})
	app.Listen(":9983")
}
```

- Beego
```bash
package main

import (
	"github.com/beego/beego/v2/server/web"
	"github.com/beego/beego/v2/server/web/context"
)

func main() {
	app := web.NewHttpSever()
	app.Get("/", func(ctx *context.Context) {
		ctx.Resp(web.M{
			"msg": "successful.",
		})
	})
	app.Run(":9984")
}
```

- GoFrame
```bash
package main

import (
	"github.com/gogf/gf/v2/frame/g"
	"github.com/gogf/gf/v2/net/ghttp"
)

func main() {
	s := g.Server()
	s.SetPort(9982)
	s.Group("", func(group *ghttp.RouterGroup) {
		group.GET("/", func(r *ghttp.Request) {
			r.Response.WriteJson(g.Map{
				"msg": "successful.",
			})
		})
	})
	s.Run()
}
```
