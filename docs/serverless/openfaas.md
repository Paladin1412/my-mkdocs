# 目录

- OpenFaaS Stack 
- HightLights
- OpenFaaS Architecture
- 示例演示
- 基础组件
  - Gateway
  - Provider
  - Watchdog
  - Queue-Worker & NATS
  - Promethus & AlertManager & Faas-idler
- Gateway 源码分析
- Provider 源码分析
- Watchdog 源码分析
- 总结
- Landscape 2.0

# 正文

## OpenFaaS Stack 

![a](http://pic3.iqiyipic.com/common/ivp/20200426/4eccf9f385e544d8bc0514afac30c8c8.png)  



## HightLights

- UI 支持
- 多种编程语言
- Faas-cli，丰富的 CLI 命令
- YAML 配置
- 按需自动伸缩
- 函数商店
- 另外：支持 CRD

> 如今看来，这些 hightlights 其实是 serverless 框架的标配。但其架构，也值得研究。



##  OpenFaaS Architecture

![aaa](http://pic2.iqiyipic.com/common/ivp/20200426/7b9f70902c7a4bb18bbad862be78a45b.png) 

## 示例演示

- Hello-world.

- 看一下 templates 里的 Dockerfile

## 基础组件

![zz](http://pic3.iqiyipic.com/common/ivp/20200427/8edd9b9d2bf84d5eb0c2c27764206f5f.png) 

### Gateway

- 为函数调用提供路由，起到一个基础转发的功能
- Promethus 收集监控指标
- 接受AlterManager通知，自动扩缩容
- 内置 UI 界面

![aa](http://pic3.iqiyipic.com/common/ivp/20200427/ad60fe1c92204a83911286468b4c7928.png)

图中，

- 用户通过`faas-cli`、UI 或 REST API 请求访问 Gateway网关;
-  `/function/`请求会被转发到 Provider，Provider 调用函数（通过Watchdog调用 forked process）；
- Prometheus 通过 /metrics 收集监控数据；
- AlertManager 根据设置的报警规则，通知 Gateway `/system/alert`自动扩缩容。

### Provider

- 提供函数的 CRUD API 能力
- 提供函数的 invoke 能力
- 设置/获取函数副本数

![Conceptual diagram](http://pic2.iqiyipic.com/common/ivp/20200427/6d14e94231e846bca6dfdbd67f4248dd.png)  

### Watchdog

集成到每个容器，responsible for starting and monitoring functions in OpenFaaS.

Classic Watchdog，如下，每个请求都会fork一个函数进程执行。

![watchdog](http://pic3.iqiyipic.com/common/ivp/20200427/e584eac9c4c64d6ba1d603fa6b4492c6.png)  

新的 of-watchdog 项目，性能更高，默认http。

![aa](http://pic1.iqiyipic.com/common/ivp/20200427/5d46c64f28714c5e9e599c4427eb0d5d.png)

有点像数据库连接池概念，The primary difference is the ability to keep the function process warm between invocations。ame process can be re-used repeatedly to offset the latency of forking.

### Queue-Worker & NATS

Queue-worker 组件提供消息队列服务，用于[**异步**](https://docs.openfaas.com/reference/async/)地处理事件。

nats是一个开源的，云原生的消息系统。使程序可以轻松地跨不同环境，语言，云提供商和内部部署系统进行通信。

### Promethus & AlertManager & Faas-idler

Promethus 收集监控指标；

接受AlterManager通知，自动扩缩容。

Faas-idler： 监控Prometheus 指标，决定是否一个函数可以被 scaled to zero.



## Gateway 源码分析

https://github.com/openfaas/faas

https://github.com/openfaas/nats-queue-worker

**gateway/main.go** 

1. ReadConfig (读取环境配置信息)
2. BasicAuthCredentials (基本的身份认证)

3. MakeForwardingProxyHandler (和函数相关的代理转发，net/http上抽象了一层) 
   - List: faasHandlers.ListFunctions
   - Deploy: faasHandlers.DeployFunction
   - Delete: faasHandlers.DeleteFunction
   - Update: faasHandlers.UpdateFunction 
   - Query: faasHandlers.QueryFunction
   - Info：faasHandlers.InfoHandler
   - Secret：faasHandlers.SecretHandler
   - Namespace: faasHandlers.NamespaceListerHandler
4. if ScaleFromZero : NewFunctionCache （参考 [zero-scale](https://www.openfaas.com/blog/zero-scale/)） 

5. If UseNATS : CreateNATSQueue (异步函数启用NATS，消息发布者角色)

6. NewPrometheusQuery（设置Prometheus监控）

7. Scale：faasHandlers.ScaleFunction（设置自动扩缩容）

8. mux.NewRouter()（注册各种路由）
9. go runMetricsServer()（启动Prometheus指标收集）
10. &http.Server{} => ListenAndServe （启动服务）

可见，Gateway 是 OpenFaaS 最为重要的一个组件。整个项目的基本脉络都在这个组件中，主要就是代理转发。



## Provider 源码分析

https://github.com/openfaas/faas-provider

- **serve.go**

  1. r = mux.NewRouter() （注册各种路由）

  2. Serve

     1. if EnableBasicAuth: handlers.FunctionReader = auth.DecorateWithBasicAuth (鉴权) 

     2. r.HandleFunc("/system/***")  (处理 HTTP 请求等路由)

        r.HandleFunc("/function/***")

        r.HandleFunc("/healthz/***")

  3. ListenAndServe()

- **types/serve.go**

```
type FaaSHandlers struct {
	// FunctionProxy provides the function invocation proxy logic.  Use proxy.NewHandlerFunc to
	// use the standard OpenFaaS proxy implementation or provide completely custom proxy logic.
	FunctionProxy http.HandlerFunc

	FunctionReader http.HandlerFunc
	DeployHandler  http.HandlerFunc

	DeleteHandler  http.HandlerFunc
	ReplicaReader  http.HandlerFunc
	ReplicaUpdater http.HandlerFunc
	SecretHandler  http.HandlerFunc
	// LogHandler provides streaming json logs of functions
	LogHandler http.HandlerFunc

	// UpdateHandler an existing function/service
	UpdateHandler http.HandlerFunc
	// HealthHandler defines the default health endpoint bound to "/healthz
	// If the handler is not set, then the "/healthz" path will not be configured
	HealthHandler        http.HandlerFunc
	InfoHandler          http.HandlerFunc
	ListNamespaceHandler http.HandlerFunc
}
```

声明各种 handler 的接口，实现该接口的平台（kubernetes or DockerSwarm）都具备了FaaS Provider 的能力。

- **proxy/proxy.go**

```
// proxyRequest 处理调用函数服务的实际请求
func proxyRequest(...) {
	...
	proxyReq = buildProxyRequest(originalReq, functionAddr, pathVars["params"])
}

// 拼装 Watchdog 的 Url，并代理过去
func buildProxyRequest(...) {
	f baseURL.Port() == "" {
		host = baseURL.Host + ":" + watchdogPort
	}

	url := url.URL{
		Scheme:   baseURL.Scheme,
		Host:     host,
		Path:     extraPath,
		RawQuery: originalReq.URL.RawQuery,
	}

	upstreamReq, err := http.NewRequest(originalReq.Method, url.String(), nil)
	if err != nil {
		return nil, err
	}
  ...
	return upstreamReq
}
```

https://github.com/openfaas/faas-netes

- **main.go**

```
main() {
	if !operator {
		runController(kubeconfig, masterURL) // 运行 k8s 的核心控制器
	} else {
		runOperator(kubeconfig, masterURL) // 操作 k8s 的CRD 资源，新增能力
	}
}
// masterURL： address of the Kubernetes API server
// operator：Use the operator mode instead of faas-netes
```

> The faas-netes controller for OpenFaaS is the most mature, well tested and supported. You may also use the OpenFaaS Operator if you would like to use a Function CRD. If you are not sure which to use, then use faas-netes.
>
> See also: [faas-netes](https://github.com/openfaas/faas-netes), [openfaas-operator](https://github.com/openfaas-incubator/openfaas-operator).

```
func runController(kubeconfig, masterURL string) {
	// 1. k8s namespace
	functionNamespace := "default"
	
	// 2. 获取 k8s, 探针，镜像拉取策略等配置信息
	deployConfig := k8s.DeploymentConfig{}
	
	// 3. k8s 客户端
	factory := k8s.NewFunctionFactory(clientset, deployConfig)
	
	// 4. 核心： k8s 操作句柄
	bootstrapHandlers := providertypes.FaaSHandlers{
			FunctionProxy:        proxy.NewHandlerFunc(cfg.FaaSConfig, functionLookup),
      DeleteHandler:        handlers.MakeDeleteHandler(functionNamespace, clientset),
      DeployHandler:        handlers.MakeDeployHandler(functionNamespace, factory),
      FunctionReader:       handlers.MakeFunctionReader(functionNamespace, clientset),
      ReplicaReader:        handlers.MakeReplicaReader(functionNamespace, clientset),
      ReplicaUpdater:       handlers.MakeReplicaUpdater(functionNamespace, clientset),
      UpdateHandler:        handlers.MakeUpdateHandler(functionNamespace, factory),
      HealthHandler:        handlers.MakeHealthHandler(),
      InfoHandler:          handlers.MakeInfoHandler(version.BuildVersion(), version.GitCommit),
      SecretHandler:        handlers.MakeSecretHandler(functionNamespace, clientset),
      LogHandler:           logs.NewLogHandlerFunc(k8s.NewLogRequestor(clientset, functionNamespace), cfg.FaaSConfig.WriteTimeout),
      ListNamespaceHandler: handlers.MakeNamespacesLister(functionNamespace, clientset),
	}
	
	// 5. 运行服务器
	faasProvider.Serve(&bootstrapHandlers, &cfg.FaaSConfig)
}
```

- **handlers/delete.go**

```
// 返回 delete handler
func MakeDeleteHandler() {
	return func{
			// ...
    deployment, findDeployErr := clientset.AppsV1().
      Deployments(lookupNamespace).
      Get(request.FunctionName, getOpts)
    if isFunction(deployment) {
        err := deleteFunction(lookupNamespace, clientset, request, w)
    }
    // ...	
	}
}

// 执行删除
func deleteFunction(..., clientset *kubernetes.Clientset, ...) {
	// 删除 deploy
	clientset.AppsV1().Deployments(functionNamespace).
		Delete(request.FunctionName, opts)
		
	// 删除 service
	clientset.CoreV1().
		Services(functionNamespace).
		Delete(request.FunctionName, opts); 
}
```

- **handles/replica.go**

```
// 调用 Kubernetes API 进行的扩缩容，代码都差不多，不再赘述
```

因此，provider 做了两件事，

1. 代理转发请求到 watchdog
2. 操作 Kubernetes

## Watchdog 源码分析

https://github.com/openfaas/faas

- **watchdog/main.go**

```
func main() {
	// 1. 读取配置
	readConfig := ReadConfig{}
	config := readConfig.Read(osEnv)
	
	// 2. 处理请求
	http.HandleFunc("/_/health", makeHealthHandler())
	// 核心 makeRequestHandler
	http.HandleFunc("/", metrics.InstrumentHandler(makeRequestHandler(&config), httpMetrics))
	
	// 3. Promethus 服务
	go metricsServer.Serve(cancel)
	
	// 4. 函数服务
	listenUntilShutdown(shutdownTimeout, s, config.suppressLock)
}

// listen for HTTP 直到听到信号 SIGTERM 且等待超时后
func listenUntilShutdown() {
	// Run the HTTP server in a separate go-routine.
	go func() {
		if err := s.ListenAndServe(); err != http.ErrServerClosed {
			log.Printf("Error ListenAndServe: %v", err)
			close(idleConnsClosed)
		}
	}()
}
```

- **watchdog/handler.go**

```
// 处理 Http 请求
func makeRequestHandler(config *WatchdogConfig) http.HandlerFunc {
	switch r.Method {
		case
			http.MethodPost,
			http.MethodPut,
			http.MethodPatch,
			http.MethodDelete,
			http.MethodGet:
			pipeRequest(config, w, r, r.Method)
			break
	}
}

// 核心：pipeRequest 
// 为每个请求 Fork 一个 fprocess
func pipeRequest() {
	// ...
	parts := strings.Split(config.faasProcess, " ") // faasProcess: "node index.js"
	// ...
	targetCmd := exec.Command(parts[0], parts[1:]...) // 执行函数
	// ...
	requestBody, buildInputErr = buildFunctionInput(config, r)
	// ...
}

// 格式化输入
buildFunctionInput(config *WatchdogConfig, r *http.Request) {
	//...
	return res, err
}
```

- **watchdog/readconfig.go** 

```
func (ReadConfig) Read(hasEnv HasEnv) WatchdogConfig {
	// ... 
	cfg.faasProcess = hasEnv.Getenv("fprocess")
	// ... 
}
```



## 总结

- 通过 faas-cli 如何维护一个 Serveless 应用
- 看 Templetes 的 Dockerfile 知道这个镜像是怎么构建
- 读源码知道 openFaas 都做了什么，如何使用 kubernets 的能力
- 此外，读内置UI 源码可定制化及开发公司UI及内部商店等。
- ...



## Landscape 2.0

![2](http://pic0.iqiyipic.com/common/ivp/20200426/56b59bf83a69427faeec26786c48fd5e.png) 