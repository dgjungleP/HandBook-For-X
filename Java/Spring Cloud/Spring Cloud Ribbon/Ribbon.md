> 客户端负载均衡器
> 

说明：
1. 提供`轮询、随机、权重`等多种方式实现负载均衡
2. 基本不单独存在，而是在每个微服务提供者中都存在的基础设施模块
3. RPC调用以及API网关代理请求的RPC请求实际都是使用的它

## 原理
>  1. 从Eureka Clinet实例获取Provider服务列表清单
>  2. 定期通过`IPing`实例判断清单中的Provider的可用性
>  3. 每次请求来的使用根据`IRule`策略类计算出最终的Provider


## 负载均衡策略
> 可以在配置文件中对`ribbon.NFLoadBalanceRuleClassName`进行配置


### RandomRule
> 随机从清单中选取

### RoundRobinRule
> 使用线性轮询，每次都取下一个节点

### WeightedResponseTimeRule
> 根据请求响应时间来计算权重，根据权重选取，每30秒更新一次权重值

### BestAvaliableRule
> 选取清单中连接数最少且可用的

### RetryRule
> 会在一定时间内进行循环重试，对Provider进行判活和选举

### AvailabilityFilteringRule
> 对线性轮询进行扩展，对列表内的服务进行可用性（`超时，连接数过多`）过滤，然后再进行线性轮询

### ZoneAvoidanceRule
> 对线性轮询进行扩展，除了超时和连接数过多的节点，还会对不符合`Zone`区域的节点进行过滤


## 配置类型
### 配置文件
- 手动配置Provider清单（一般不会使用）
- RPC请求超时配置
- 重试机制配置
### 代码配置
1. 实现`RequstInterceptor`接口并使用`@Configuration`注解
2. `IRule`可以通过`@Bean`注入
3. 全局配置在`@EnableFeignClients`上的`defaultConfiguration`上声明
4. 局部配置在`@FeignClient`上的`configuration`上声明

