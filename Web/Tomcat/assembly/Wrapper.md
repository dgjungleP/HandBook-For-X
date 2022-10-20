## 构成情况
- 一个`ServletPool` 组件 
- 一个`Servlet`组件
- 一个`Pipeline`组件

## 过滤器链
> 过滤器链是在Wapper的管道中，当到达基础阀门的时候，会创建出来，会在Servlet处理之前进行过滤


## Servlet

### 类型

- 普通Servlet
- JspServlet
- DefaultServlet


## Comet模式
> 一种服务端推技术，提供一种能让服务器端往客户端发送数据的方式

注：
1. 一般需要和`NIO`进行配合起来使用
2. 在Tomcat中需要实现`CometProcessor`来使用这么模式
3. 和WebSocket实现的差不多的行为

## [WebSocket](../../extend/agreement/WebSocket.md)协议的支持

注：
1. 需要使用WebSocket时，需要继承`WebSocketServlet`

## 异步处理的Servlet

注：
1. 需要将`任务、Request和Response`提交给用户自定义的线程池，而不是Tomcat的公共线程池