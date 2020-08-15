### 一、eureka注册中心

##### 服务注册

eureka client在第一次心跳时向eureka server注册

注册时提供自身元数据：主机名、端口号、监控指标、URL

默认30秒发送一次心跳，90秒内未收到续约，进行服务删除

client退出时发送cancel命令，server收到cancel删除该节点

client会缓存server获取的注册表信息，并定期更新注册表信息，默认30s

client会处理注册表的合并等内容

##### 面试点

**CAP理论**

CP ：类似MySQL CA：数据库和分布式数据库 AP：redis

**多注册中心比较：**

Zookeper和eureka

eureka主要保证AP特性，zookeeper优先CP特性

zookeeper获取数据后对所有节点做广播，广播结束之前，所有节点不可见数据

**eureka注册慢**

根本原因在于eureka的AP特性，

client延迟注册，默认30s

server响应缓存，默认30s

server缓存刷新，默认30s

**eureka自我保护**

server自动更新续约更新阈值

server续约更新频率低于阈值进入保护模式

自我保护模式下server不会剔除任何注册信息

### 二、Ribbon

##### ribbon架构图

![image-20200804004941518](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200804004941518.png)

ribbon是客户端负载均衡器

ribbon的核心功能：服务发现、服务选择规则、服务监听

ribbon和eureka整合

通过@LoadBalanced提供负载均衡支持

通过ribbon.eureka.enabled=false禁用eureka

**ribbon核心之IRule**

![image-20200805002750354](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200805002750354.png)

继承AbstractLoadBalancerRule自定义rule

应用灰度发布10%的流量在新的功能，90%流量在旧的功能，已经访问过新功能的流量不能再访问旧的功能

**IPing**

探测服务存活状态，保证服务可用。

![image-20200805003800992](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200805003800992.png)

**参数配置**

默认参数配置 DefaultClientConfigImpl

Ribbon Key定义：CommonClientConfigKey

Ribbon参数分为全局配置和指定客户端配置

指定客户端：<client>.ribbon.<key>=<value>

全局配置：ribbon.<client>.<key>=<value>

配置生效原则为就近生效

**ribbon之ServerList**

ServerList是ribbon存储的可用服务列表，ribbon不是直接从eureka直接获取可用服务列表的，而是从serverlist获取的，serverlist可能是从eureka获取的，serverlist可以手动设置，常用从eureka自动获取

### 三、Hystrix

**Hystrix架构图**

![image-20200805010743638](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200805010743638.png)

用于处理延迟和容错的开源库

主要用于避免级联故障，提高系统弹性

解决由于扇出导致的雪崩效应

核心是隔离术和熔断机制

**Hystrix作用**

服务隔离和服务熔断

服务降级、限流和快速失败

请求合并与请求缓存

自带单体和集群监控

**Hystrix命令模式**

HystrixCommand:

默认线程隔离

execute():单次处理，同步执行

queue():单次执行，异步执行

observe():分为阻塞式调用和非阻塞式调用,多次处理，Hot处理，执行Command的Run方法，加载、注册Subscribe对象，将run方法结果注入到Subscribe对象的onNext方法

toObserve():多次处理，Cold处理，每次订阅需要新的Command对象，加载、注册Subscribe对象，执行Command的Run方法，将run方法结果注入到Subscribe对象的onNext方法

HystrixObservableCommand:

默认信号量隔离

**差异**

HystrixCommand会以隔离的形式完成run方法调用

HystrixObservableCommand:使用当前线程进行调用

**Hystrix配置之GroupKey**

必须设置

作为分组监控和报警的作用

作为线程池的默认名称

**Hystrix配置之CommandKey**

非必须设置

默认通过反射类名命名CommandKey

在Setting中加入andCommandKey进行命名

**Hystrix请求特性**

Hystrix支持将请求结果进行本地缓存

通过实现getCacheKey方法来判断是否使用缓存

请求缓存要求请求必须在同一个上下文

可以通过RequestCacheEnable开启

**Hystrix请求合并**

将多个请求合并成一次请求

要求两次请求足够近（默认500ms）

分为局部合并和全局合并

Collapser可以设置相关参数

但是很少有机会对同一个服务进行多次http调用，并且间隔足够近

**Hystrix作用**

**Hystrix隔离术ThreadPoolKey**

默**认使用GroupKey命名线程池

在Setting中加入andThreadPoolKey进行命名

Hystrix提供了信号量和线程两种隔离手段

线程隔离会在单独的线程中执行业务逻辑

信号量隔离在调用线程上执行

官方推荐优先线程隔离

![image-20200807004618749](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200807004618749.png)

应用自身完全受保护，不会受其他依赖影响

有效降低接入新服务的风险

依赖服务出现问题，应用自身可以快速反应问题

可以通过实施刷新动态属性减少依赖问题影响

**信号量隔离**

是轻量级的隔离术，在当前线程执行，而不是创建新线程执行

无网络开销的情况情况下推荐使用信号量隔离

信号量通过计数器与请求线程比对进行限流

**Hystrix降级处理**

command降级需要实现fallback方法

observablecommand降级实现resumewithfallback方法

fallback与run方法是一对一关系，run方法出错，fallback就会执行

降级触发：

HystrixBadRequestException以外的异常

运行超时或熔断器处于开启状态

线程池或信号量已满

快速失败：

当出现异常的时候，执行fallback，处理异常

当不实现fallback的时候直接抛出异常

**Hystrix熔断机制**

熔断器是一个开关，用来控制流量是否执行业务逻辑，打开状态下，直接执行fallback，半开启状态，间歇性让请求触发触发run方法，成功则退出开启状态（5秒），关闭状态下，支持处理业务请求。默认开启5秒后进入半开启状态

核心指标：快照时间窗、请求总数阈值、错误百分比阈值（请求综述是否达到预设阈值，失败率是否达到预设阈值）

![image-20200808015528795](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200808015528795.png)

**线程池设置**

和QPS相关，

### 四、Feign组件

feign是一个http客户端，很大程度上简化了http调用方式，弥补了springcloud的http调用缺陷（大量重复工作量）

feign包含了多种http的调用形式（springmvc@RequestMapping @RequestParam @PathVariable @RequestHeader@Requestbody），可以整合ribbon和hystrix，提供了多种http底层支持

实现了可插拔注解支持，包括feign和jax-rs注解

支持可插拔的http编码解码器

支持http请求和响应的压缩

**参数解释**

primary：指定feignclient接口实现类的优先级，feign只有接口，却可以使用，其实是自动生成实现类。

configuration：针对单个接口提供不同的配置，自定义配置要在spring作用于范围之外，自定义内容为：

![image-20200809005625231](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200809005625231.png)

fallback和fallbackfactory

**feign多组件集成**

集成ribbon实现负载均衡（默认集成）

集成hystrix实现命令封装（feign.hystrix.enable=true）

集成hystrix实现业务降级

fallback：实现接口并且完成降级业务逻辑

fallbackfactory 创建工厂类，实现fallbackfactory接口，在create方法中完成fallback子类实现，添加fallbackfactory参数，指定工厂类

**feign之http性能优化**

feign默认使用jdk自带的http方式，可以替换成Apache httpclient（添加Apache httpclient依赖，feign.httpclient.enable=true）

**feign之http解压缩**

feign支持gzip的请求压缩

feign.compress.request.enable=true

.mime-type

.min-request-size

feign.compress.response.enable=true

**feign继承**

微服务的目标是大量复用，feign会导致重复工作量

feign提供继承特性解决该问题，接口复用最多只能有一层，切忌多继承

### 五、zuul网关

**网关**

请求转发与请求过滤

zuul还可以鉴权与容错，衔接ribbon和hystrix

**请求路由表达**

？：匹配任意单字符

*：匹配任意数量字符

**：匹配任意数量字符，支持多级目录

**zuul高级架构**

![image-20200810001110938](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200810001110938.png)

requestcontext是线程安全的

![image-20200810001449659](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200810001449659.png)

**自定义filter**

集成zuulfilter并实现相对应方法

设置filter类型，级别及是否启用

![image-20200810001823244](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200810001823244.png)

![image-20200810002018206](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200810002018206.png)

![image-20200810002043133](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200810002043133.png)

**zuul和zuul2**

zuul使用阻塞式线程完成业务调用

zuul2使用异步线程完成业务调用

**zuul和hystrix整合**

**cookie和头信息处理**

zuul过滤了一些非安全信息，通过设置sensitive-heads 选择过滤哪些

### 六、微服务安全

**JWT**

json web token，用于身份认证及信息加密

![image-20200810003709605](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200810003709605.png)

**CORS**

跨域资源共享



**SpringSecurity**