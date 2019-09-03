## go框架解析--beego
1. 配置加载：如何加载配置文件的
2. **路由**：分析框架如何通过URI执行对应业务的
3. ORM：ORM如何实现的

### 代码分析
go有自己实现的http包，大多go框架也是基于这个http包，所以看beego之前我们先补充或者复习下这个知识点。如下：

Go如何启动一个HTTP SERVER
```
package main

import (
	"net/http"
)

func main() {
	// --- 使用http包启动一个http服务 方式一 -------
	// *http.Request http请求内容实例的指针
	// http.ResponseWriter 写http响应内容的实例
	http.HandleFunc("/demo", func(w http.ResponseWriter, r *http.Request) {
		// 写入响应内容
		w.Write([]byte("Hello World!\n"))
	})
	// 启动一个http服务并监听8888端口 这里第二个参数可以指定handler
	http.ListenAndServe(":8888", nil)
	
	// 测试我们的服务
	// --------------------
	// 启动：bee run
	// 访问：curl "http://127.0.0.1:8888/demo"
	// 响应结果： Hello World!
}
```

ListenAndServe是对http.Server的进一步封装，除了上面的方式，还可以使用http.Server直接启服务，这个需要设置Handler，这个Handler要实现Server.Handler这个接口。当请求来了会执行这个Handle的`ServeHTTP`方法，如下：
```
package main

// 导入net/http包
import (
	"net/http"
)

// DemoHandle server handle示例
type DemoHandle struct {}

// ServeHTTP 匹配到路由后执行的方法
func (DemoHandle) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	w.Write([]byte("Hello TIGERB !\n"))
}

func main() {
	// ------------------ 使用http包的Server启动一个http服务 方式二 ------------------
	// 初始化一个http.Server
	server := &http.Server{}
	// 初始化handler并赋值给server.Handler
	server.Handler = DemoHandle{}
	// 绑定地址
	server.Addr = ":8888"

	// 启动一个http服务
	server.ListenAndServe()

}

// 测试我们的服务
// --------------------
// 启动：bee run
// 访问： curl "http://127.0.0.1:8888/v1/demo"
// 响应结果：Hello TIGERB !
```

### BEEGO路由分析
接下来我们开始看beego的代码。拿访问`"http://127.0.0.1:8080"/`来说，对于beego代码来说有三个关键点，分别如下：
1. 启动：main.go -> `beego.Run()`
2. 注册路由：routers\router.go -> `beego.Router("/",&controllers.MainController{})`
3. 控制器：controllers\default.go -> `Get()`

下面来看3个关键点的详细分析：
`BEEGO.RUN()`主要的工作：
```

```