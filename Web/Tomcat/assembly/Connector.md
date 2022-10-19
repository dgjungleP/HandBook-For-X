## 构成情况
### 典型组成情况
- `Protocal`组件
- `Mapper`组件
- `CoyoteAdaptor`组件

## 作用

- 负责接收客户端连接和客户端请求加工
	- 接受请求生成Request
	- 响应求情生成Response


## 组件作用

### Protocal
> 对不同的地通信协议进行封装

#### Endpoint
> 对接收端进行抽象，根据不同的`I/O模式`有不同的类型

##### Acceptor
> 专门用于接受客户端连接的接收器

##### Excutor
> 处理客户端请求的线程池

###### BIO构成说明
- LimitLacth
>控制流量
- Acceptor，
>监听是否有套接字连接，并连接套接字，做尽可能少的事情，并将任务交给Excutor
- ServerSocketFactory
> 用以封装套接字，选择使用`安全的`还是`不安全的`连接方式
- Excutor
> 去触发执行提交过来的任务
- SocketProcessor
> 定义套接字的任务，确定需要执行的逻辑，任务主要有三种：
> - 处理套接字并响应客户端
> - 连接数计数器减一
> - 关闭套接字

###### NIO构成说明
- LimitLacth
- Acceptor
  > 同样监听是否有套接字，提交任务对象也变为了`SocketChannel`
- NioChannrel和SecureNioChannel
>  非阻塞通道，负责读取/写入数据到缓冲区
- SocketProcessor
- Poller
> 对连接进行管理，不断便利事件列表，对相应的连接做出处理（如果有线程池，则将任务封装为SocketProcessor,提交给线程池处理，如果没有线程池，则Poller自己处理）
- Poller池

#### Processor
> 处理客户端请求的处理器，根据不同的`协议`和不同的`I/O模型`，有不同的类型

##### BIO构成说明
- InternalInputBuffer
> 套接字缓冲输入装置，主要责任为：
> - 提供一种缓冲模式
> - 提供填充缓冲区的方法
> - 提供解析协议请求行的方法
> - 提供解析协议请求头的方法
> - 组装请求对象
- InternalOutputBuffer
>套接字输出缓冲装置
- Request
> 使用了`门面模式`来处理数据安全的问题
- Response
>  使用了`门面模式`来处理敏感数据隔离的问题
- ActionHook

NIO构成说明
- InternalNioInputBuffer
> 不再使用流进行读取，而是采用通道进行读取
- InternalNioOutputBuffer
- Request
- Response
- ActionHook

### Mapper
> 路由器，对客户端请求`URL`进行映射，将请求转发到对应的`Host`,`Context`,`wapper`组件来处理响应客户端（寻找`Servlet`）

### CototeAdaptor
> 适配器，负责将Connector和Engine容器进行适配，包接收到的请求报文解析生成的`请求对象`和`响应对象`，传递给Engine容器进行处理

## 其他说明
- Connector可以使用Service共享的线程池，也可以使用自己私有的线程池
- 在实际工作中Tomcat保持着到`用的时候再转换`的思想，所以协议信息被解析到Request对象中的时候还是保持的ASCLL码的状态（利用内部实现的`MessageBytes`灵活的处理）
- `ByteChunk`就是`MessageBytes`最核心的部分，他封装了字节缓冲器以及对字节缓冲区的操作
- 可以使用`过滤器`实现较好的扩展性和逻辑解耦
- Tomcat中主要的过滤器
	- IdentityInputFilter
	- VoidInputFilter
	- BufferedInputFilter
	- ChunkedInputFilter
- Tomcat中`门面模式`的巧用
- Tomcat中`钩子机制`的巧用
- 对于频繁生成和消除对象的场景下，我们可以将对象缓存起来，然后重置里面的属性，下一次继续使用（前提是对象需要支持这种操作）
- 除了HTTP的Connector，Tomcat还支持AJP的Connector可以了解一下

