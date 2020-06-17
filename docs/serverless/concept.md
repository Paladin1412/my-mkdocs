# 目录

- Why Cloud Native
- Cloud Native 历史

- 什么是 Serverless

- 对比 Serverful

- IaaS/PaaS/Caas/FaaS 等名词
- 什么不是 Serverless
- 一个例子
- Serverless 的技术特点

- Serverless 的优势

- Serverless 的局限性

- Serverless 应用场景

- Serverless 开源框架

- CNCF Serverless WG 

# 正文

## Why Cloud Native

- Break up monolith
  - 提升资源利用率
  - 节省成本
- Abstraction of Infrastructure
  - 个人：开发者关注 Code 而非 Infrastructure
  - 公司：更快的开拓市场（Money）

![a855d5d4dca64584b9785e7c649c764c.png](https://wx2.sbimg.cn/2020/06/17/a855d5d4dca64584b9785e7c649c764c.png)


## Cloud Native 历史

![0df650e1683b4ebab8698feb6ed1f74f-1.md.jpg](https://wx2.sbimg.cn/2020/06/17/0df650e1683b4ebab8698feb6ed1f74f-1.md.jpg)

![NAtUvd.png](https://s1.ax1x.com/2020/06/17/NAtUvd.png)

因此， Serverless 是云原生技术发展的高级阶段。

> 上一家单位还处在 Data Centre 阶段，而今我在研究 Serverless, 看来我进化了。



## 什么是 Serverless

- 开发者不关注服务器，只关注代码及业务逻辑。
- **Serverless 指的是服务端逻辑由开发者实现，运行在无状态的计算容器中，由事件触发，完全被第三方管理，而业务层面的状态则记录在数据库或存储资源中。**

关键词：no server，stateless，event-driven



## 对比 Serverful

- 弱化了存储和计算之间的联系
- 代码的执行不再需要手动分配资源
- 按使用量计费



## IaaS/PaaS/Caas/FaaS 等名词

- IaaS：Infrastructure as a Service
- PaaS：Platform as a Service 
- CaaS：Container as a Service 
- Faas： Function as a Service
- AaaS
- ...



## 什么不是 Serverless

> 如果你的PaaS可以将以前半秒启动的应用在20ms内启动，就叫它Serverless。——Adrian Cockcroft

大部分 PaaS 应用不会为了每个请求都启动并结束整个应用，而 this is what FaaS does。



## Serverless ≠ FaaS

> 从外延来看，Serverless 比 FaaS 的外延要广，FaaS 主要解决的是用户自定义的代码逻辑如何做到 Serverless，可以叫做 Serverless Compute。

![aaa](http://pic2.iqiyipic.com/common/ivp/20200426/9438f869c2d3414b819887597a5254ee.png) 



有张老图找不到了，下边找了一张 Spring Serverless 官网介绍页的图代替，意思差不多。

![aa](http://pic0.iqiyipic.com/common/ivp/20200426/3d2b7f52865b4a5281fbf0f107861e8a.png) 

目前业界的各类 Serverless 实现按功能而言，主要为应用服务提供了两个方面的支持：函数即服务（Function as a Service，FaaS）以及后台即服务（Backend as a Service，BaaS）。

- **FaaS**: FaaS 提供了一个计算平台，在这个平台上，应用以一个或多个函数的形式开发、运行和管理。

  Faas 的主要概念是，将使用云服务的过程变得和传统编程一样：一个应用由很多个函数组成，每个函数有输入输出，在函数内完成必要的逻辑。开发者只需要编写好函数代码，将代码注册到云端，将这些函数组成一个完整的应用即可。

- **BaaS**: 通过 BaaS 平台将应用所依赖的第三方服务，如数据库、消息队列及存储等服务化并发布出来，用户通过向 BaaS 平台申请所需要的服务进行消费，而不需要关心这些服务的具体运维。



## 一个例子

- 一个传统的三层 C/S 架构

![a](http://pic3.iqiyipic.com/common/ivp/20200426/1f0c0bc09335449ca49f4c3e242a11bb.png)

- 改造成 Serverless 架构

![b](http://pic1.iqiyipic.com/common/ivp/20200426/9d35d07019b24789b4c1ed4463de3086.png) 

	1. 一个第三方的 BaaS 服务，身份验证逻辑
	2. 另一个 BaaS 示例，允许客户端直接访问一部分数据库内容
	3. 先前服务器端的部分逻辑已经转移到了客户端
	4. 服务端，搜索功能可以从持续运行的服务端中拆分出来，以 FaaS 的方式实现
	5. 服务端，购买功能改写为另一个 FaaS 函数


## Serverless 的技术特点

- 按需加载
- 事件驱动
- 状态非本地持久化
- 非会话保持
- 自动弹性伸缩
- 应用函数化



## Serverless 的优势

- 提升人力成本
- 降低风险
- 减少资源开销
- 增加缩放的灵活性
- 缩短创新周期



## Serverless 的局限性

- 状态管理
- 冷启动延迟
- 本地测试



## Serverless 应用场景

Serverless 的优势和局限性决定，适合以下场景：

- 异步的并发，组件可独立部署和扩展
- 应对突发或服务使用量不可预测（主要是为了节约成本，因为 Serverless 应用在不运行时不收费）
- 短暂、无状态的应用，对冷启动时间不敏感
- 需要快速开发迭代的业务（因为无需提前申请资源，因此可以加快业务上线速度）

示例：

- ETL
- 机器学习及 AI 模型处理
- 图片处理
- IoT 传感器数据分析
- 流处理
- 聊天机器人



## Serverless 开源框架

- OpenFaas: 成熟易用，核心项目不够活跃； 

- Knative: 起步较晚，Google & IBM 大厂支持, 新鲜，真香。

  

## CNCF Serverless WG 

- [CloudEvents 规范](https://cloudevents.io)：定义最小化的事件通用属性。
- [Workflow 规范](https://github.com/cncf/wg-serverless/blob/master/workflow/spec/spec.md)：函数编排。
- [Whitepaper](https://gw.alipayobjects.com/os/basement_prod/24ec4498-71d4-4a60-b785-fa530456c65b.pdf) 
