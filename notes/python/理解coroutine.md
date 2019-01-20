[TOC]


## 什么是协程
协程：又称微线程、纤程，英文Coroutine。 

> 维基百科：“协程是为非抢占式多任务产生子程序的计算机程序组件，协程允许不同入口点在不同位置暂停或开始执行程序”。

协程是为实现异步编程/并发编程而生的。
- 异步编程：简单来说就是代码执行的顺序在程序运行前是未知的（异步而非同步）
- 并发编程：是代码的执行不依赖于其他部分，即便是全都在同一个线程内执行（并发不是并行）

技术角度看：“协程就是你可以暂停执行的函数”，可以把它理解成“像生成器一样”。

#### 总结
1. 是一种子程序（函数）
2. 不同于传统的子程序间的层级调用（通过栈实现），协程在程序内部可以中断，然后转而执行别的程序，并在适当的时候再返回执行
3. 类似于CPU的中断/多线程切换
4. 比多线程的执行效率高：没有线程切换的开销（一秒钟切换个上百万次系统都抗的住）、不需要多线程的锁机制
5. 在多核CPU环境中，建议使用多进程+协程的模式 

## 知识点
#### 同步vs异步 阻塞vs非阻塞
> 老张爱喝茶，废话不说，煮开水
出场人物：老张，水壶两把（普通水壶，简称水壶；会响的水壶，简称响水壶）
    1. 老张把水壶放到火上，立等水开（同步阻塞）
    2. 老张把水壶放到火上，去客厅看电视，时不时去厨房看看水开没有（同步非阻塞）
    3. 老张把响水壶放到火上，立等水开（异步阻塞）
    4. 老张把响水壶放到火上，去客厅看电视，水壶响之前不再去看它了（异步非阻塞）

同步vs异步：对于水壶而言（普通水壶：同步；响水壶：异步）
响水壶可以在水开后提示老张，这是普通水壶所不能及的。

阻塞vs非阻塞：对于老张而言（立等的老张：阻塞；看电视的老张：非阻塞）
不管用的是不是响水壶（异步），完全由老张来决定是立等还是干别的
**技术角度看**
同步vs异步：是功能（代码段/函数）的实现方式。具备异步的特征就是异步，是确定的
阻塞vs非阻塞：是指调度功能的方式。不管功能是同步还是异步，调度都可以选择阻塞/非阻塞。是可以选择的。
当然了，最优的选择是异步+非阻塞方式.
#### 协程vs回调
协程和回调都是异步模式。协程相对于回调具有更容易编写、可读性更高的特点。
回调：代码被“分片”了（一部分用来调用、一部分用来回调）。而是调用和回调之间还需要进行沟通。代码的可读性很低，也不符合人类的习惯。
协程：通过对异步操作的封装既保留了性能也保证了代码的容易编写和可读性。

#### 协程vs异步编程
异步编程：异步+非阻塞
构成要件：协程（异步函数）、事件调度器、任务管理器
1. 创建协程
2. 将协程放入任务管理其中
3. 将任务管理器放入事件调度器中执行

## Python协程演进
#### Python2.2 生成器
[PEP 255][1]
那时也把它称为迭代器，因为它实现了迭代器协议。生成器允许创建一个在计算下一个值时不会浪费内存空间的迭代器。
1. 生成器是迭代器，在函数中使用了`yield value`就是一个生成器
2. 生成器不能直接调用，只能通过调用next(g)来获得值（`for ... in g`的实现方式一致）
3. 生成器在运行到`yield value`语句时会中断，等待下一次调用next(g)
```Python
def lazy_range(up_to):
    """
    生成器返回一个从0到up_to值的整数序列，但不会占用up_to + 1的内存空间
    """
    index = 0
    while index < up_to:
        yield index
        index += 1
```
#### Python2.5 增强生成器
[PEP 342][2]
为生成器引入了`send()`方法，不仅可以暂停生成器，而且能够传递值到生成器暂停的地方。（Python突然就有了协程的概念）
```
receive             =  yield         value
传入参数                              返回值
send(param)传入                      re = g.send()获得返回值
receive = param                     re = value
```
这行代码执行分为三个步骤
1. 向函数外抛出(返回)value
2. 暂停，等待next()或send()恢复
3. 赋值receive = MockGetValue()。MockGetValue()用来接收send()发送进来的值

也就是说，生成器先返回一个值并暂停，当再次被唤醒时才接收send()发送进来的值。因此，使用send()函数激活一个生成器时使用`g.send(None)`传入一个`None`值
```Python
def gen():
    value = 0
    while True:
        receive = yield value
        if receive == 'e':
            break
        value = 'got: %s' % receive
# 获得生成器
g = gen()
# 发送send(None)启动
print(g.send(None))
>>> 0
# 消费生成器
print(g.send('hello'))
>>> got: hello
# 发送退出消息
print(g.send('e'))
>>> StopIteration
# 关闭生成器
g.close()
```
执行步骤解析
1. 通过`g.send(None)`或者`next(g)`启动生成器函数。执行到第一个`yield`语句结束的位置。只执行了`yield`语句的前两个步骤，并没有给`receive`赋值
2. 通过`g.send('hello')`传入`hello`，从上次暂停的位置继续执行，执行`yield`的第三步，将`receive`赋值为`hello`。继续执行计算出`value = 'got: hello'`。回到`while True`循环。执行`yield`前两步，将`value`值返回

再看一个例子，通过send()函数调整迭代器的输出结果
```Python
def jumping_range(up_to):
    """
    生成器返回一个从0到up_to值的整数序列
    向生成器send()一个值改变序列的值
    """
    index = 0
    while index < up_to:
        jump = yield index
        if jump is None:
            jump = 1
        index += jump

if __name__ == '__main__':
    iterator = jumping_range(5)
    print(next(iterator))  
    >>> 0
    print(iterator.send(2))  
    >>> 2
    print(next(iterator)) 
    >>> 3
    print(iterator.send(-1))
    >>> 2
    for x in iterator:
        print(x)  
    >>> 3 4
```
#### Python3.3 yield from
[PEP 380][3]
严格来说，这一特性让你能够从迭代器（生成器刚好也是迭代器）中返回任何值，从而可以干净利索的方式重构生成器。
```Python
def lazy_range(up_to):
    """
    生成器返回一个从0到up_to值的整数序列
    """
    index = 0
    def gratuitous_refactor():
        while index < up_to:
            yield index
            index += 1
    yield from gratuitous_refactor()
```
yield from 通过让重构变得简单，也让你能够将生成器串联起来，使返回值可以在调用栈中上下浮动，而不需对编码进行过多改动。
```Python
def bottom():
    # Returning the yield lets the value that goes up the call stack to come right back
    # down.
    return (yield 42)

def middle():
    return (yield from bottom())

def top():
    return (yield from middle())

# Get the generator.
gen = top()
value = next(gen)
print(value)  # Prints '42'.
try:
    value = gen.send(value * 2)
except StopIteration as exc:
    value = exc.value
print(value)  # Prints '84'.
```

[1]: https://www.python.org/dev/peps/pep-0255/ "Simple Generators"
[2]: https://www.python.org/dev/peps/pep-0342/ "Coroutines via Enhanced Generators"
[3]: https://www.python.org/dev/peps/pep-0380/ "Syntax for Delegating to a Subgenerator"