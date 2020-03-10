## 用途

- 为非阻塞套接字提供I/O事件循环
- 基于`asyncio`事件循环
- 应用可以使用`IOLoop`或`asyncio`接口
- `IOLoop`以线程为边界，即每个线程使用独立的对象，当然不同进程也不同（即使是`fork`）
- 可以使用`tornado`中的类`TCPClient`等，也可以作为通用的事件循环使用，通过回调接口添加处理函数

## 用法

- 使用`IOLoop.current()`类方法可以得到实例
- 将套接字的读写事件纳入事件循环，需要调用`add_handler()`方法
- 内置`TCPClient`、`TCPServer`等类，在类内部已经完成了上述调用
- `IOLoop.start()`方法启动事件循环，直到处理函数中手动调用`IOLoop.stop()`方法。必须在处理函数中调用是因为，当前进程在该方法执行后，就进入无限循环状态。后续的代码只能待调用`stop()`方法后才能继续执行
- 支持的事件类型
  - 使用`epoll`
  - `READ`：可以读，包括对方关闭套接字
  - `WRITE`：可以写
  - `ERROR`：发生错误

## 接口

### 启动/停止

- `IOLoop.start()`。启动事件循环方法，启动后当前线程进入事件循环，程序被阻塞。通常情况下，放在执行代码的最后
- `IOLoop.stop()`。停止事件循环方法，需要在事件处理函数中调用。
  - 调用时，如果当前事件循环已经启动，则循环终端在处理完队列中的事件后才会退出
  - 调用时，如果当前事件循环未启动，则在下次启动时立即退出

### 实例化

- `IOLoop.current()`

  - 不要直接实例化`IOLoop`，提供了`IOLoop.current()`类方法
  - 唯一的参数`instance = True`
    - 默认值为`True`，即如果当前线程没有`IOLoop`实例，则创建；已有则返回该实例
    - 第一次执行时，使用`False`参数，则返回的是`None`实例
  - `IOLoop.instance()`已经被废弃

  ```python
  from tornado import ioloop
  
  l1 = ioloop.IOLoop.current(False)
  print(l1, id(l1))  
  # None 4305012752
  
  l2 = ioloop.IOLoop.current()
  print(l2, id(l2))  
  # <tornado.platform.asyncio.AsyncIOMainLoop object at 0x104f21278> 4377940600
  
  l3 = ioloop.IOLoop.current(False)
  print(l3, id(l3))
  # <tornado.platform.asyncio.AsyncIOMainLoop object at 0x104f21278> 4377940600
  
  ```

- `IOLoop.clear_curent()`

  - 类方法
  - 手动清除当前线程的`IOLoop`，但是并没有释放该实例
  - 通常应用于`unittest`测试框架中

  ```python
  from tornado import ioloop
  
  l1 = ioloop.IOLoop.current()
  print(l1, id(l1))
  # <tornado.platform.asyncio.AsyncIOMainLoop object at 0x104d30048> 4375904328
  
  ioloop.IOLoop.clear_current()
  print(l1, id(l1))
  # <tornado.platform.asyncio.AsyncIOMainLoop object at 0x104d30048> 4375904328
  
  l1 = ioloop.IOLoop.current()
  print(l1, id(l1))
  # <tornado.platform.asyncio.AsyncIOMainLoop object at 0x104d30048> 4375904328
  ```

### 其他

- `IOLoop.close()`
  - 关闭事件循环，释放资源
  - 如果给定参数`True`，则关闭`IOLoop`自身创建以及注册到`IOLoop`中的文件描述符`fd`
  - 必须在`IOLoop`彻底停止后才能执行，因此最好在`IOLoop.start()`后面调用，不要放到`IOLoop.stop()`后面
  - 一般情况下，一个进程/线程只有一个事件循环实例，没有必要进行清理
- `IOLoop.run_sync(func: Callable,timeout: float = None) → Any)`
  - 启动`IOLoop`、执行`func`，停止`IOLoop`
  - 可以理解为只为执行`func`而启动一次事件循环
  - `func`必须返回`awaitable`或者`None`，如果返回`awaitable`则事件循环会等到返回值处理完后才返回。可以理解为：提供了在程序的主函数中执行异步函数的方法（只有在`async`函数中才能使用`await`）

### 事件接口

- `IOLoop.add_handler`
  - 将`fd`的指定事件的处理函数注册到事件循环中
  - `IOLoop.add_handler(fd: Union[int, tornado.ioloop._Selectable], handler: Callable[[...], None], events: int) → None)`
    - `fd`：即文件描述符，可以是文件或套接字
    - `handler`：即处理函数，当`fd`上有事件后触发，执行函数`handler(fd, events)`
    - `events`：监控的事件类型，如：`IOLoop.READ`、`IOLoop.WRITE`、`IOLoop.ERROR`
- `IOLoop.update_handler`
  - 更新`fd`监控的事件
  - `IOLoop.update_handler(fd: Union[int, tornado.ioloop._Selectable], events: int) → None`

- `IOLoop.remove_handler`
  - 停止监控`fd`的事件
  - `IOLoop.remove_handler(fd: Union[int, tornado.ioloop._Selectable]) → None`

### 回调接口

如果要在事件循环中执行某个函数，可以使用一系列的回调接口。这些接口各有功用

- `add_callback`
  - `add_callback(callback: Callable, *args, **kwargs) → None`
  - 参数的数量一定与`callback`函数声明的一致，否则报错
  - `callback`函数只会执行一次
  - 不能取消
  - 与`spawn_callback`相同
- `add_timeout`
  - `add_timeout(deadline: Union[float, datetime.timedelta], callback: Callable[[...], None], *args, **kwargs)→ object`
  - 在`deadline`时间点执行`callback`函数
  - 只执行一次
  - 可以取消。调用`remove_timeout`，传入返回的对象句柄
- `call_at`
  - `call_at(when: float, callback: Callable[[...], None], *args, **kwargs)→ object`
  - 在`when`时点执行
  - 可以取消
- `call_later`
  - `call_later(delay: float, callback: Callable[[...], None], *args, **kwargs)→ object`
  - 在`delay`延时后执行，单位：秒
  - 可以取消