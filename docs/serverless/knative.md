# 目录

- Why Not Kubernetes
- Why Knative
- Serving (Core)
  - Serving 模型
  - Serving 资源
  - Serving 架构
  - 自动扩缩容
  - 一个 service 
- Eventing (Core)
  - Eventing 实现的功能
  - Eventing 资源
  - Source
  - Sink
  - Eventing 模型
    - 简单投递
    - 通道与订阅
    - 序列
    - 并列
    - 黑盒模型
- Observability
- Build
  - Tekton Pipeline Core
  - Tekton Pipeline CRDs 
  - 一个 Pipeline 流程
- DevOps
- 总结

# 正文

## Why Not Kubernetes

Most popular Container Management project today, Deloy and manage your containerized application.

- Components

  - Apiserver
  - Proxy
  - Etcd
  - Kubelet
  - Controller
  - Sheduler
  - CNI

- Resources

  - Pod
  - Node
  - Namespace
  - Service
  - Volume
  - PersistentVolume
  - Deployment
  - Secret
  - StatefulSet
  - DaemonSet
  - ServiceAccount
  - ReplicationController
  - ReplicaSet
  - Job
  - CronJob
  - SecurityContext
  - ResourceQuota
  - LimitRange
  - HorizontalPodAutoscaling
  - Ingress
  - ConfigMap
  - Label
  - CustomResourceDefinition
  - Role
  - ClusterRole

- Tools

  - Yaml

  - Spec

  - helm

  - Kubectl

  - istio

  - promethus

    ...

如果你想托管你开发的 app，你得成为一个基础设施专家。事与愿违。



## Why Knative

- 简化的 application/container 管理
- 让开发者关注 coding
  - 提供 PaaS 和 Severless 概念的抽象
- Kubernetes 的扩展
  - Kubernetes 作为底层
  - 随时可用 Kubernetes 原生能力



## Knative 的受众

![aa](http://pic2.iqiyipic.com/common/ivp/20200427/5f7992c58a7e400eb45fe33da629c8d1.png) 

> Knative 跟 Istio 的关系：
>
> 起初，Istio 在 Knative 下边；
>
> 后来，Knative 在 Istio 上边
>
> 如今，没有 Istio。

## Knative 组件介绍

- 核心组件 Serving

  服务系统，用来配置应用的路由、升级策略、自动扩缩容等功能。

- 核心组件 Eventing

  事件系统，用来自动完成事件的绑定和触发

- Observability

  可视化插件 Prometheus& Grafana，ELK，Jaeger/Zipkin 等。

- Build

  Teton，构建系统，从 Knative 项目独立出去。主要完成两步： source 到 image，deploy on Knative。

## Serving (Core)

### Serving 模型

![11](http://pic3.iqiyipic.com/common/ivp/20200427/7122adbcecdd4d26ab62713521aab48c.png) 

1. 部署 app as Pod/Revision
2. 部署完成后，自动建立网络，暴露 endpoint
3. Revision 自动扩缩容（可以 scale to 0）
4. 当更新 Revision 时，自动迁移到新的
5. Traffice Splitting, 流量根据配置分发到某个Revision。
6. 专用 URLs 针对 Revision

### Serving 资源

- [Service](https://knative.dev/docs/serving/spec/knative-api-specification-1.0#service):  `service.serving.knative.dev` 管理服务的生命周期。确保每次创建都有 Route, Configuration 和 Revision。
- [Route](https://knative.dev/docs/serving/spec/knative-api-specification-1.0#route):  `route.serving.knative.dev` 定义网络端口，映射到一个或多个Revision，提供流量分发管理。
- [Configuration](https://knative.dev/docs/serving/spec/knative-api-specification-1.0#configuration): `configuration.serving.knative.dev`  维护 deployment 的期望状态，并发数, Min/Max/Target Scale 等。 修改 Configuration 就会创建一个 Revision。
- [Revision](https://knative.dev/docs/serving/spec/knative-api-specification-1.0#revision):  `revision.serving.knative.dev` 每次修改代码和配置的快照。Revisions 不可更改，且根据流量自动扩缩。 

![aaa](http://pic3.iqiyipic.com/common/ivp/20200427/1c61752c182b45399dbe68f81eb54d5a.png) 

### Serving 架构

 $ kubectl apply -f serving-core.yaml， serving 的核心组件如图。

![aaa](http://pic0.iqiyipic.com/common/ivp/20200428/7e01eeb5601a4b929b35040ae6005f5d.png)

可以看到 再 namespace knative-serving 下，有  controller  、webhook 、autoscaler、activator 这四个核心组件；

其实，还有一个 queue, 运行在每个应用的 pod 里，作为 sidecar 存在。

![bb](http://pic2.iqiyipic.com/common/ivp/20200428/269cea24ed0d49b99e27e18fc5433fef.png) 

1. Controller 负载 Service 整个生命周期的管理，涉及、Configuration、Route、Revision 等的 CURD。
2. Webhook 主要负责创建和更新的参数校验。
3.  Autoscaler 根据应用的请求并发量对应用扩缩容。
4. Activator 负责从 0 到 1。在应用缩容到 0 后，拦截用户的请求，通知 autoscaler 启动相应应用实例，等待启动后将请求转发。
5.  Queue 负载拦截转发给 Pod 的请求，用于统计 Pod 的请求并发量等，autoscaler 会访问 queue 获取相应数据对应用扩缩容。

### 自动扩缩容

**如何发生**

-  1 -> n：任何访问应用的请求在进入 Pod 后都会被 Queue 拦截，统计当前 Pod 的请求并发数，同时 Queue 会开放一个 metric 接口，autoscalor 通过访问该端口去获取 Pod 的请求并发量并计算是否需要扩缩容。当需要扩缩容时，autoscalor  会通过修改 Revision 下的 deployment 的实例个数达到扩缩容的效果。

- 0 -> 1：在应用长时间无请求访问时，实例会缩减到 0。这个时候，访问应用的请求会被转发到 activator，并在请求在转发到  activator 之前会被标记请求访问的 Revision 信息（由 controller 修改 VirtualService  实现）。activator 接收到请求后，会将改 Revision 的并发量加 1，并将 metric 推送给 autoscalor，启动  Pod。同时，activator 监控 Revision 的启动状态，Revision 正常启动后，将请求转发给相应的 Pod。

- 当然，在 Revision 正常启动后，应用的请求将不会再发送到 activator，而且直接发送至应用的 Pod（由 controller 修改 VirtualService 实现）。

**监控指标**

- KPA 根据并发数进行伸缩，`kpa.autoscaling.knative.dev`。
- HPA 根据制CPU进行伸缩，`hpa.autoscaling.knative.dev`，直接利用 k8s 的HPA能力。

### 一个 service 

```yaml
# helloworld-go.yaml
apiVersion: serving.knative.dev/v1 # Current version of Knative
kind: Service
metadata:
  name: helloworld-go
  namespace: default
spec:
  template:
   	metadata:
  		name: helloworld-go-1
    spec:
      containers:
        - image: gcr.io/knative-samples/helloworld-go
          env:
            - name: TARGET
              value: "Go Sample v1"
```

kubectl apply -f  helloworld-go.yaml 一下会发生什么。

```
Service 
	| -> Route
	| -> Configuration -> Revision
```

```
Revision
	| -> Deployment / PodAutoscaler
	| -> ServerlessService
```

```
Route -> ClusterIngress -> VitualService(istio)
```

- Traffic-spitting

```yaml
apiVersion: serving.knative.dev/v1 # Current version of Knative
kind: Service
metadata:
  name: helloworld-go
  namespace: default
spec:
  template:
  	metadata:
  		name: helloworld-go-2
    spec:
      containers:
        - image: gcr.io/knative-samples/helloworld-go
          env:
            - name: TARGET
              value: "Go Sample v2"
  traffic:
  - tag: current
    revisionName: helloworld-go-1
    percent: 50
  - tag: candidate
    revisionName: helloworld-go-2
    percent: 50
  - tag: latest
    latestRevision: true
    percent: 0
```



## Eventing (Core)

> Knative Eventing is a system that is designed to address a common need for cloud native development and provides composable primitives to enable late-binding event sources and event consumers.

### Eventing 实现的功能

![zzz](http://pic2.iqiyipic.com/common/ivp/20200428/6f5062e521464e258cfb52854a118c23.png) 

- 可靠的消息通道
- 订阅转发机制

### Eventing 资源

- Event Source：把时间生产者接入 Knative Eventing 平台，并把事件传到消费者。

- Channel： 实现事件转发和存储，支持 Kafka 和 NATS Streaming
- Subscription：事件订阅者。
- Squence: 事件依次转发给多个订阅者的一种机制
- Parallel：事件同时转发给多个订阅者的一种机制

- Broker：接受事件，并将事件转发到订阅者。
- Trigger：事件订阅。
- Event Registry 注册中心，事件类型的集合，可查看。

 ### Source

- Knative 中有个资源叫 EventSource。用于把事件生产者接入 Knative 事件平台。

- 事件生产者在 Knative 事件平台的抽象

- Spec

  - 参数
  - Sink

- Knative 中已有的预定义资源类型：

  - APIServerSource

  - GithubSource
  - AwsSource
  - **ContainerSource**
  - CronJobSource
  - ...

### Sink

- Sink 表示事件消息传送的目的地，暴露一个 HTTP 端口。不是一个具体资源。
- 在 Kubernets，只要带着属性：status.address.hostname 的都可以看作 sink。
- 两种类型：
  - Addressable: 暴露一个HTTP 端口，接受消息，返回成功或失败。
  - Callable: 暴露一个HTTP 端口，接受消息，返回 0 到 1 个新事件。
- Knative 中的 Sink 类型的资源： Service, Channel, Broker, Parallel 及 Sequence

### Eventing Models

#### 模型一

Simple Dlivery 简单投递

- 定义事件源
- 通过 Sink 指定消费者地址

![a](http://pic3.iqiyipic.com/common/ivp/20200428/14b8436196f64ea0975f332af6344e46.png) 

```
# event-source.yaml
apiVersion: sources.knative.dev/v1alpha2
kind: CronJobSource
metadata:
  name: test-heartbeats
spec:
  template:
    spec:
      containers:
        - image: <heartbeats_image_uri>
          name: heartbeats
          args:
            - --period=1
          env:
            - name: POD_NAME
              value: "mypod"
            - name: POD_NAMESPACE
              value: "event-test"
      sink:
        ref:
          apiVersion: serving.knative.dev/v1
          kind: Service
          name: event-display
```

#### 模型二

Channel & Subscription 通道与订阅

- 通道 Channel, 代表事件的转发和持久化。
- 订阅描述通道与消费者之间的关系，也能连接消费者与第二个通道，用于处理消费者返回的事件。
- 订阅 Subscription，支持不同技术的通道，如 Kafka, NATS Streaming 等。
- 通道与订阅，一对多。

![a](http://pic2.iqiyipic.com/common/ivp/20200428/5d1c40414a174289b1ff2c7ded04073c.png) 

```
# channel.yaml
apiVersion: messaging.knative.dev/v1alpha1
kind: Channel
metadata:
  name: my-channel
```

```
# event-source.yaml
...	
	spec:
		...
		sink:
			apiVersion: messaging.knative.dev/v1
			Kind: Channel
			name: my-channel
```

```
# subscription.yaml
apiVersion: eventing.knative.dev/v1alpha2
kind: Subscription
metadata:
  name: test-subscription
spec:
  channel:
  	apiVersion: messaging.knative.dev/v1
  	kind: Channel
  	name: my-channel
  subscriber:
  	apiVersion: serving.knative.dev/v1
  	kind: Service
  	name: event-display
```

![a](http://pic1.iqiyipic.com/common/ivp/20200428/6f734b8e5a4f4a7b88bb559e10c65f62.png) 



#### 模型三

Sequence 序列

- Sequence 定义一系列依次执行事件
- 事件发送给服务
- 服务可以修改 or 创建新的事件

![a](http://pic1.iqiyipic.com/common/ivp/20200428/69d94aee66604efaacbaff4e075c7565.png) 

```
# event-source.yaml
...	
	spec:
		...
		sink:
			apiVersion: messaging.knative.dev/v1
			Kind: Sequence
			name: my-sequence
```

```
# sequence.yaml
apiVersion: eventing.knative.dev/v1alpha2
kind: Sequence
metadata:
  name: my-sequence
spec:
	steps:
  -	ref:
      apiVersion: serving.knative.dev/v1
      kind: Service
      name: step-1
  -	ref:
      apiVersion: serving.knative.dev/v1
      kind: Service
      name: step-2
  -	ref:
      apiVersion: serving.knative.dev/v1
      kind: Service
      name: step-3
```



#### 模型四

Parallel 并列

- Parallel 定义一系列并行事件
- 每个服务接受到同一事件
- 每个分支定义 filter

![a](http://pic2.iqiyipic.com/common/ivp/20200428/25ebc84741a34bf6a1b13eba8d6a3b0c.png)  

```yaml
# event-source.yaml
...	
	spec:
		...
		sink:
			apiVersion: messaging.knative.dev/v1
			Kind: Parallel
			name: my-parallel
```

```yaml
# parallel.yaml
apiVersion: eventing.knative.dev/v1alpha2
kind: Parallel
metadata:
  name: my-parallel
spec:
	branches:
  	-	filter
  			ref:
          apiVersion: serving.knative.dev/v1
          kind: Service
          name: filter-1
      subscriber:
        ref:
          apiVersion: serving.knative.dev/v1
          kind: Service
          name: mysevice-1
   -	filter
  			ref:
          apiVersion: serving.knative.dev/v1
          kind: Service
          name: filter-2
      subscriber:
        ref:
          apiVersion: serving.knative.dev/v1
          kind: Service
          name: mysevice-2
```



#### 模型五

Tigger & Broker 黑盒模型

- 搭建黑盒，隐藏具体实现，用户不关心
- Broker 事件桶，接入各种事件，事件具有各种属性用来区分
- Tigger 过滤器，只有通过过滤器的条件才能传给消费者

![aaa](http://pic3.iqiyipic.com/common/ivp/20200427/de791f8ac77d4bedab19e02a42dc6085.png) 

```yaml
# Broker: knative 自己实现，安装即可。
broker-filter
broker-ingress
```

```yaml
# Trigger.yaml
apiVersion: eventing.knative.dev/v1beta1
kind: Trigger
metadata:
  name: my-trigger
spec:
  filter:
    attributes:
      type: dev.knative.sources.ping
  subscriber:
    ref:
      apiVersion: serving.knative.dev/v1beta1
      kind: Service
      name: event-display
```



## Observability

- Logging: ELK
- Monitoring: Promethus & Grafana
- Tracing: Jaeger/Zipkin

## Build

### Tekton Pipeline Core 

- pipeline-controller
- Pipeline-webhook

### Tekton Pipeline CRDs

Tekton 为 Kubernetes 提供了多种 CRD 资源对象，可用于定义我们的流水线，主要有以下几个资源对象：

- Task：定义了一组构建步骤，如编译代码、运行测试以及构建和部署镜像。表示执行命令的一系列步骤，task 里可以定义一系列的 steps，例如编译代码、构建镜像、推送镜像等，每个 step 实际由一个 Pod 执行。
- TaskRun：task 只是定义了一个模版，taskRun 才真正代表了一次实际的运行，当然你也可以自己手动创建一个 taskRun，taskRun 创建出来之后，就会自动触发 task 描述的构建任务。
- Pipeline：定义了构成流水线的 **Tasks**。一组任务，表示一个或多个 task、PipelineResource 以及各种定义参数的集合。
- PipelineRun：定义了流水线的执行。它引用要运行的 **Pipeline** 以及要用作输入和输出的 **PipelineResources**。类似 task 和 taskRun 的关系，pipelineRun 也表示某一次实际运行的 pipeline，下发一个 pipelineRun CRD 实例到 Kubernetes后，同样也会触发一次 pipeline 的构建。
- PipelineResource：流水线的输入（例如 git 存储库）或输出（例如 docker 镜像）。表示 pipeline 输入资源，比如 github 上的源码，或者 pipeline 输出资源，例如一个容器镜像或者构建生成的 jar 包等。

### 一个 Pipeline 流程

![Screen Shot 2020-04-29 at 11.46.12 AM](http://pic3.iqiyipic.com/common/ivp/20200429/33145c4893d84c84b2512930fb55ce17.png) 



## DevOps

![11](http://pic1.iqiyipic.com/common/ivp/20200429/173bb03850bc47279cee342f7018e223.png) 



## 总结

- Cloud Native/Serverless/Faas 概念

- Stack: Kubernetes, Istio, microservices, logging/monitoring/tracing, 

- OpenFaas 和 Knative

  