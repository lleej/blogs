## 用途

- 用于非阻塞文件和套接字的读写操作

## 用法

- 包含了四个类
  - `BaseIOStream`
  - `IOStream`
  - `SSLIOStream`
  - `PipeIOStream`

## 基类

`BaseIOStream`可以看做是一个抽象基类。它封装了对`fd`的监听、事件处理以及`IOStream`的读写操作

继承的子类必须完成以下函数的重载，这些函数在基类中并没有实现

- `fileno()`
- `close_fd()`
- `write_to_fd()`
- `read_from_fd()`
- `get_fd_error()`
- `set_nodelay()`
- `_handle_connect()`

### 重要接口

- `_handle_events(self, fd: Union[int, ioloop._Selectable], events: int) -> None`
  - 内部函数，避免外部调用
  - 事件循环的处理函数
  - 根据事件类型执行对`fd`的读写操作
- `_add_io_state(self, state: int) -> None`
  - 内部函数，避免外部调用
  - 调用`IOLoop.add_handler()`函数，将`fd`的处理函数`_handle_events`注册到事件循环中

### 读写操作

提供了一个写操作方法，以及5个读操作方法

- `read_bytes(self, num_bytes: int, partial: bool = False) -> Awaitable[bytes]`
  - 读取指定长度的数据
  - 如果`partial`为`True`，则只要读到数据就会返回
  - 返回读到的数据
- `read_into(self, buf: bytearray, partial: bool = False) -> Awaitable[int]`
  - 读取数据并写入`buf`参数中
  - 如果`partial`为`True`，则只要读到数据就会返回；否则，`buf`被填满后才返回
  - 返回读取到的字节数量
- `read_until(self, delimiter: bytes, max_bytes: int = None) -> Awaitable[bytes]`
  - 读取数据直到数据中有`delimiter`数据存在，可以用于以`delimiter`作为分隔符的情况
  - 如果指定了`max_bytes`，当读取的数据长度等于该值时没有找到`delimiter`则关闭连接
  - 返回读取到的数据，包含`delimiter`
- `read_until_regex(self, regex: bytes, max_bytes: int = None) -> Awaitable[bytes]`
  - 读取数据直到数据匹配`regex`
  - 如果指定了`max_bytes`，当读取的数据长度等于该值时没有找到匹配的`regex`则关闭连接
  - 返回读取到的数据，包含匹配`regex`前的数据
- `read_until_close(self) -> Awaitable[bytes]`
  - 读取数据直到连接关闭
  - 读取的数据长度不超过`max_buffer_size`
- `write(self, data: Union[bytes, memoryview]) -> "Future[None]"`
  - 异步写操作
  - 返回`Future`对象
  - 数据可以是`bytes`或者`memoryview`类型

读取操作一般常用的是`read_bytes`和`read_into`，两者的区别是：

- `read_bytes`将读取的数据作为返回值
- `read_into`将读取的数据作为入口参数返回

后三个读取操作可能导致连接关闭，一般情况下要谨慎使用

## IOStream类

针对套接字的读写类

- `_init_`
  - 传入`socket`实例
- `connect(self: _IOStreamType, address: tuple, server_hostname: str = None) -> "Future[_IOStreamType]"`
  - 使用`address`参数传入`IP`和`port`，一般不使用`server_hostname`参数
  - 当创建实例时，传入的`socket`没有调用`connect`时调用