### 服务消费者ribbon和feign实战

#### 常用的服务间调用方式讲解
1. RPC
- 远程过程调用，像调用本地服务一样调用服务器的服务。
- 支持同步、异步调用。
- 客户端和服务器之间建立TCP连接，可以一次建立一个，也可以多个调用复用一次连接。
- RPC数据包小

2. Rest(http)
- http请求，支持多种协议和功能。
- 开发方便成本低。
- http数据包大

#### 微服务调用方式之ribbon实战 订单调用商品服务
1. 创建order_service项目
- 依赖
```
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
        </dependency>
```
2. 开发伪下单接口
3. 使用ribbon（类似于HttpServlet，URLConnection）
```
启动类增加注解
    @Bean
    @LoadBalanced
    public RestTemplate restTemplate() {
        
        return new RestTemplate();
    }
```
4. 根据名称进行调用商品，获取商品详情

#### Ribbon负载均衡
__ribbon服务间调用负载均衡源码分析__
+ 完善下单接口

+ 分析@LoadBalanced
1. 首先从注册中心获取provider的列表。
2. 通过一定的策略选择其中一个节点
3. 再返回给RestTemplate

__LoadBalanced注解的作用是用LoadBalancedClient包装RestTemplate__
```
public interface LoadBalancerClient extends ServiceInstanceChooser {
    <T> T execute(String serviceId, LoadBalancerRequest<T> request) throws IOException;

    <T> T execute(String serviceId, ServiceInstance serviceInstance, LoadBalancerRequest<T> request) throws IOException;

    URI reconstructURI(ServiceInstance instance, URI original);
}
```
__LoadBalancedClient实现类__
```
public class RibbonLoadBalancerClient implements LoadBalancerClient {
```
__选择服务实例__
```
    public ServiceInstance choose(String serviceId, Object hint) {
        Server server = this.getServer(this.getLoadBalancer(serviceId), hint);
        return server == null ? null : new RibbonLoadBalancerClient.RibbonServer(serviceId, server, this.isSecure(server, serviceId), this.serverIntrospector(serviceId).getMetadata(server));
    }
```
```
   protected Server getServer(ILoadBalancer loadBalancer, Object hint) {
        return loadBalancer == null ? null : loadBalancer.chooseServer(hint != null ? hint : "default");
    }
```
__IloadBalance：负载均衡策略__
```
public interface ILoadBalancer {
    void addServers(List<Server> var1);

    Server chooseServer(Object var1);

    void markServerDown(Server var1);

    /** @deprecated */
    @Deprecated
    List<Server> getServerList(boolean var1);

    List<Server> getReachableServers();

    List<Server> getAllServers();
}
```
__ILoadBalance实现类BaseLoadBalance:获取服务列表__
```
负载均衡策略默认是：RoundRobinRule

private static final IRule DEFAULT_RULE = new RoundRobinRule();

 public List<Server> getAllServers() {
        return Collections.unmodifiableList(this.allServerList);
}
```
__负载均衡策略__
```
public interface IRule {
    Server choose(Object var1);

    void setLoadBalancer(ILoadBalancer var1);

    ILoadBalancer getLoadBalancer();
}
```

![负载均衡策略截图](/springcloud核心组件/images/负载均衡策略截图.png)

#### 服务间调用之负载均衡策略调整实战
__实战调整默认负载均衡策略实战__
+ 更改负载均衡策略 
```
官方文档
users:
  ribbon:
    NIWSServerListClassName: com.netflix.loadbalancer.ConfigurationBasedServerList
    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.WeightedResponseTimeRule

项目配置
#自定义负载均衡策略
product-service:
  ribbon:
     NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule 策略的全路径    
```
