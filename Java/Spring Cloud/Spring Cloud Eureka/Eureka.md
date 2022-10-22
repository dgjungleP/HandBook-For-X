
## 概览

- Eureka所治理的每一个微服务实例都被称为`Provider Instance`
- 每一个`Provider Instance`都包含一个`Eureka Client`



## 功能说明

### Eureka Client
- 向`Eureka Sever`完成对`Provider Instance`的管理
- 向`Eureka Server`获取`Provider Instance`的清单，并`缓存在本地`


### Eureka Server

注：
- 在配置是注意`Zone`和`Region`
- 定义了多个与`Provider Instance`相关的事件，可以监听（使用`@EventListener`）
- 保护模式建议在网络环境好、通信延迟低的场景中关闭（它是一种应对网络异常的安全保护措施）