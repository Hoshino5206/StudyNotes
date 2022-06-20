## Ribbon 负载均衡

### 简介

主要提供客户端的软件负载均衡和服务调用。Ribbon 客户端组件提供一系列完善的配置项如连接超时，重试等。简单的说，就是在配置文件中列出 Load Balencer 后面所有的机器，Ribbon 会自动的帮助你基于某种规则（如简单轮询，随机连接等）去连接这些机器。我们很容易使用 Ribbon 实现自定义的负载均衡算法。

**Ribbon 目前也进入维护,基本上不准备更新了**

**进程内 LB(本地负载均衡)**

将 LB 逻辑集成到消费方，消费方从服务注册中心获知有哪些地址可用，然后自己再从这些地址中选择一个合适的服务器。

集中式（LB）（服务器负载均衡）

集中式 LB，即在服务的消费方和提供方之间使用独立的 LB 设施（可以是硬件 F5 软件 nginx 等）

**区别**

Ribbon 本地负载均衡客户端 VS Nginx 服务端负载均衡：

Nginx 是服务器负载均衡，客户端所有请求都会交给 Nginx，然后由 Ngixn 实现转发。

Ribbon 本地父子负载均衡，在调用微服务接口时候，会在服务注册中心上获取信息表之后缓存到 JV

**Ribbon 就是负载均衡+RestTemplate**

Ribbon 其实就是一个软负载均衡的客户端组件，它可以和其他所需要请求的客户端结合使用，和 Eureka 结合只是其中一个实例。

![](.\media\Ribbon的7.png)

Ribbon 在工作时分成两步：

第一步选择 EurekaServer，它优先选择在同一区域内负载较少的 Server

第二步在根据用户指定的策略，在从 server 取到的服务注册列表中选择一个地址。

其中 Ribbon 提供了多种策略：轮询，随机，和响应时间加权。

### 原理

ILoadBalance 负载均衡器：ribbon 是一个为客户端提供负载均衡功能的服务，它内部提供了一个叫做 ILoadBalance 的接口代表负载均衡器的操作，比如有添加服务器操作、选择服务器操作、获取所有的服务器列表、获取可用的服务器列表等等。

流程：

LoadBalancerClient（RibbonLoadBalancerClient 是实现类）在初始化的时候（execute 方法），会通过 ILoadBalance（BaseLoadBalancer 是实现类）向 Eureka 注册中心获取服务注册列表，并且每 10s 一次向 EurekaClient 发送“ping”，来判断服务的可用性，如果服务的可用性发生了改变或者服务数量和之前的不一致，则从注册中心更新或者重新拉取。LoadBalancerClient 有了这些服务注册列表，就 根据具体的 IRule（路由）来进行负载均衡。

IRule 接口代表负载均衡策略，choose 方法时具体的选择服务器方法，其中 RandomRule 表示随机策略、RoundRobinRule 表示轮询策略、WeightedResponseTimeRule 表示加权策略、BestAvailableRule 表示请求数最少策略等等。

#### 负载策略

随机策略：

```java
int index = rand.nextInt(serverCount); // 使用jdk内部的Random类随机获取索引值index
server = upList.get(index); // 得到服务器实例
```

轮询策略：

最大权重：

有一个默认每 30 秒更新一次权重列表的定时任务，该定时任务会根据实例的响应时间来更新权重列表，choose 方法做的事情就是，用一个(0,1)的随机 double 数乘以最大的权重得到 randomWeight，然后遍历权重列表，找出第一个比 randomWeight 大的实例下标，然后返回该实例

最少并发请求：

```java
for (Server server: serverList) { // 遍历每个服务器
        ServerStats serverStats = loadBalancerStats.getSingleServerStat(server); // 获取各个服务器的状态
        if (!serverStats.isCircuitBreakerTripped(currentTime)) { // 没有触发断路器的话继续执行
            int concurrentConnections = serverStats.getActiveRequestsCount(currentTime); // 获取当前服务器的请求个数
            if (concurrentConnections < minimalConcurrentConnections) { // 比较各个服务器之间的请求数，然后选取请求数最少的服务器并放到chosen变量中
                minimalConcurrentConnections = concurrentConnections;
                chosen = server;
            }
        }
    }
```

HashCode

### 使用 Ribbon:

1,默认我们使用 eureka 的新版本时,它默认集成了 ribbon

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-neflix-eureka-server</artifactId>
</dependency>
```

**==这个 starter 中集成了 ribbon 了==**

2,我们也可以手动引入 ribbon

**放到 order 模块中,因为只有 order 访问 pay 时需要负载均衡**

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-neflix-ribbon</artifactId>
</dependency>
```

3,RestTemplate 类:

```java
// 返回对象为响应体中数据转化成的对象，基本上可以理解为Json
@GetMapping(" / consumer/ payment/get/{id}")
public CommonResult<Payment> getPayment(@Pathvariable("id") Long id){
	return restTemplate.getForobject(PAYMENT_SRV+"/payment/get/"+id,CommonResult.class);
}
//返回对象为ResponseEntity对象，包含了响应中的一些重要信息，比如响应头、响应状态码、响应体等
@GetMapping( "/ consumer/ payment/getForEntity/{id}")
public commonResult<Payment> getPayment2(@Pathvariable("id") Long id){
	ResponseEntity<commonResult> entity = restTemplate.getForEntit(PAYMENT_SRV+" /payment/get/"+id，commonResul);
    if(entity.getstatuscode().is2xxSuccessful()){
        return entity.getBody();
    }e1se {
        return new CommonResult( code: 444，message: "操作失败");
    }
}
@GetMapping("/consumer/payment/getForEntity/{id}")
public commonResult<Payment> getPayment2(@Pathvariable("id") Long id){
	ResponseEntitycCommonResult> entity = restTemplate.getForEntity( url:PAYINENT_URL+"/payment/get/"+id,CommonResul.class);
	if(entity.getstatuscode().is2xxsuccessful()){
        return entity.getBody();//这个responseEntity中有判断,这里是判断,状态码是不是2xx,
    }else{
        return new commonResult<>( code: 444,message:"操作失败");
    }
}
```

```java
RestTemplate的:
		xxxForObject()方法,返回的是响应体中的数据
    xxxForEntity()方法.返回的是entity对象,这个对象不仅仅包含响应体数据,还包含响应体信息(状态码等)
```

#### 自定义负载均衡算法:

1,ribbon 的轮询算法原理

rest 接口第几次请求数%服务器集群总数量 = 实际调用服务器位置下标，每次服务重启后 rest 接口计数从 1 开始

![](.\media\Ribbon的21.png)

2,自定义负载均衡算法:

- **给**pay 模块(8001,8002),的 controller 方法添加一个方法,返回当前节点端口

```java
@value( "${server.port}")
private string serverPort;

@GetMapping(value = "/ payment/lb")
public string getPaymentLB()
{
    return serverPort;
}

```

- **修改 order 模块**

去掉@LoadBalanced

```java
@configuration
public class Applicationcontextconfig{
	@Bean
	//@loadBalanceda
	public RestTemplate getRestTemplate() {
        return new RestTemplate();
    }
}
```

3,自定义接口

![](.\media\Ribbon的29.png)

==具体的算法在实现类中实现==

4,接口实现类

![](.\media\Ribbon的25.png)

![](.\media\Ribbon的26.png)

5,修改 controller:

![](.\media\Ribbon的27.png)

![](.\media\Ribbon的28.png)

6,启动服务,测试即可