>  协调多个`Subscriber`,并共享一个底层的订阅
>  会始终保证底层最多只有一个`Subscriber`，但实际上它又允许多个`Subscriber`共享相同的底层资源


## 使用方式

### 使用publish().refCount()实现单次订阅（share()）

### 强行订阅publish().connect()


## 应用场景
1. 不管何时订阅，都需要确保它们接收到相同的事件序列

