## 模型

### Schdule.newThread
> 每次都启动一个新线程，适用于任务数量少，任务耗时长

### Schdule.io
> 类似于一个无线线程池，适用于I/O密集且需要较少的CPU资源的任务

### Schdule.computation
> 根据核心数量来控制大小，适用于CPU密集型的任务

### Schdule.from
> 将一个JDK的`Excuto`转化

### Schdule.immediate
> 以阻塞的方式调度,会立即执行给定的任务

### Schdule.trampoline
> 阻塞式的方式调度，会等待已经调度任务玩不执行完成之后才开始

### Schdule.test
> 仅用来测试