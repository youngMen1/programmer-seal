# 1.Dubbo框架设计介绍
## 1.1.基本介绍
### 整体设计
![](/static/image/12744765-157dd7b682851d26.webp)

官网图例说明：

* 图中左边淡蓝背景的为服务消费方使用的接口，右边淡绿色背景的为服务提供方使用的接口，位于中轴线上的为双方都用到的接口。
* 图中从下至上分为十层，各层均为单向依赖，右边的黑色箭头代表层之间的依赖关系，每一层都可以剥离上层被复用，其中，Service 和 Config 层为 API，其它各层均为 SPI。
* 图中绿色小块的为扩展接口，蓝色小块为实现类，图中只显示用于关联各层的实现类。
* 图中蓝色虚线为初始化过程，即启动时组装链，红色实线为方法调用过程，即运行时调时链，紫色三角箭头为继承，可以把子类看作父类的同一个节点，线上的文字为调用的方法。

### 各层说明
* **service 服务层：**这一层是用户自己编写的，不管是服务的接口还是服务的实现。从整体设计图可以看到，接口是提供方和消费方共同使用的，而实现则只在提供方，并不暴露给消费方。这也与我们平时的使用认知相符，服务提供方将接口单独打成一个jar包让消费方使用。

config 配置层：对外配置接口，以 ServiceConfig, ReferenceConfig 为中心，可以直接初始化配置类，也可以通过 spring 解析配置生成配置类。dubbo-config下面包含了config-api和config-spring两个子模块，分别对应了两种配置生成方式。配置类会包含协议、url等信息，是一个比较重的对象。

proxy 服务代理层：服务接口透明代理，生成服务的客户端 Stub 和服务器端 Skeleton, 以 ServiceProxy 为中心，扩展接口为 ProxyFactory。可以看整体设计图，Invoker其实就是负责调用服务提供方真实服务的一个代理，而Proxy则是在消费方负责调用提供方服务的一个代理。

registry 注册中心层：封装服务地址的注册与发现，以服务 URL 为中心，扩展接口为 RegistryFactory, Registry, RegistryService。

cluster 路由层：封装多个提供者的路由及负载均衡，并桥接注册中心，以 Invoker 为中心，扩展接口为 Cluster, Directory, Router, LoadBalance。相当于将集群封装为单个节点，这样外部就可以像单体调用一样处理，不需要考虑集群的情况(比如多个提供方的情况)

monitor 监控层：RPC 调用次数和调用时间监控，以 Statistics 为中心，扩展接口为 MonitorFactory, Monitor, MonitorService。会有一些服务调用信息的统计。

protocol 远程调用层：封装 RPC 调用，以 Invocation, Result 为中心，扩展接口为 Protocol, Invoker, Exporter。对协议的封装，dubbo支持多种协议，默认是dubbo协议。

exchange 信息交换层：封装请求响应模式，同步转异步，以 Request, Response 为中心，扩展接口为 Exchanger, ExchangeChannel, ExchangeClient, ExchangeServer。

transport 网络传输层：抽象 mina 和 netty 为统一接口，以 Message 为中心，扩展接口为 Channel, Transporter, Client, Server, Codec。

serialize 数据序列化层：可复用的一些工具，扩展接口为 Serialization, ObjectInput, ObjectOutput, ThreadPool。dubbo的序列化也支持多种协议。


# 2.总结
# 3.参考
https://segmentfault.com/a/1190000016741532