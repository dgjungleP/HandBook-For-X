## HTTP请求/响应模型
![HTTP_requet_resonse.png](HTTP_requet_resonse.png)

1. 客户端和服务器先通过`3次握手`建立连接
2. 建立连接后，客户端发送一个请求到服务器（组装报文）
3. 服务端接收到请求报文后，对报文进行解析，组装成一定格式的响应报文，返回给客户端
4. 客户端浏览器接收到响应报文后，通过`浏览器内核`对其进行解析，按照一定的外观进行显示，然后与服务器断开连接

## 报文
### 请求报文
- 请求行
- 请求头部
- 请求体

### 响应报文
- 响应行
- 响应头部
- 响应体
