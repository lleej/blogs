[TOC]

## super概念
通俗的讲：super()是将方法调用委托给实例的祖先树中的某个类的业务

在类的继承中，如果子类重定义了祖先树的某个方法，在子类中就会覆盖祖先树的同名方法。有时候，希望子类的方法能够扩展祖先树(而不是覆盖)，这就需要在子类的方法中使用super()函数调用祖先树的同名方法。（类似于DELPHI的inherit）
```Python
class Animal(object):
    def __init__(self, name):
        self.name = name
    def greet(self):
        print('Hello, I am', self.name)

class Dog(Animal):
    def greet(self):
        super(Dog, self).greet()
        print('WangWang...')

dog = Dog('Dog')
dog.greet()
>>> Hello, I am Dog
>>> WangWang...
```
## super使用
1. Python2.x版本，使用super(cls, inst)的格式，参数显式
2. Python3.x版本，使用super()的格式，参数隐式
## super优点
1. 采用计算的间接调用(indirect)方式，而不是硬连接(hardwired)的方式调用
    - 隔离变化。不使用委托(delegate)类的名字。即使修改基类，也不用改代码
    - 实现变化。计算是在运行时(runtime)进行，对于动态语言，可以在运行时方便的对计算结果进行处理，使结果指向其他类
2. 不用修改已有类型，利用现有类型就可以扩展出多种组合
    - 使用多重继承模式
    `class LoggingOD(loggingDict, collection.OrderdDict)`不用修改已有类型，也不用写任何代码，通过继承就可以创建一个具备记录支持&顺序排列的字典新类
## super深入
由于Python支持多重继承，在一个类的祖先树上，会出现一些独特的现象：没有直接继承关系的两个类，出现在同一个祖先树上。这块比较不好理解，接着往下看。
在开始介绍之前，先介绍一个概念MRO(Method Resolution Order)方法解析顺序列表，它代表了类继承的顺序。
通过调用一个类（不是实例）的`__MRO__`属性可以查看该类的继承顺序列表。

下面通过一个例子进行这部分概念的解释。
```Python
class Base(object):
    def __init__(self):
        print('Enter Base')
        print('Leave Base')
class A(Base):
    def __init__(self):
        print('Enter A')
        super(A, self).__init__()
        print('Leave A')
class B(Base):
    def __init__(self):
        print('Enter B')
        super(B, self).__init__()
        print('Leave B')
class C(A, B):
    def __init__(self):
        print('Enter C')
        super(C, self).__init__()
        print('Leave C')

c = C()
>>> Enter C
>>> Enter A
>>> Enter B
>>> Enter Base
>>> Leave Base
>>> Leave B
>>> Leave A
>>> Leave C
```
结果并不是我们预期的那样，怎么在类的继承链上B是A的父类了，它们两个可是没有任何的继承关系呢？
通过MRO进一步验证了上例中的输出结果
```Python
print(C.mro())
>>> [__main__.C, __main__.A, __main__.B, __main__.Base, object]
```
在Python中类的继承关系是通过MRO记录的，而super()函数则利用MRO查找类的继承关系并返回其上一级类型。
**也就是说：通常意义上的父类和super()函数的返回值没有半毛钱关系**
下面就是super()函数的实现原理了，看仔细了
```Python
# super()函数有两个参数
# cls 表示类
# inst 表示当前实例
# 输出: 在inst的类的MRO列表中，查找cls类的下一个类
def super(cls, inst):
    # 得到inst实例类的mro
    mro = inst.__class__.mro()
    # 从中查找cls的下一个类
    return mro[mro.index(cls) + 1]
```
## 注意事项
要使用super()实现对祖先树中各类方法调用的重新排列，就必须将类设计为合作类(cooperate)。合作类有几点注意事项。
#### 使用super()调用的方法必须存在
1. 确保祖先树/MRO上每个类都有该方法。比如说:`__init__`方法是每个类都存在的
2. 绝大多数类是从`object`继承的，对于`object`不具备的方法，应该如何处理呢？
3. 我们需要写一个在`object`之前的根类(Root)，其主要职责就是中断super()调用
```Python
class Root:
    def draw(self):
        # 将委托链(delegation)中断
        # 利用防御性编程(defensive programming)的assert确保中断调用链条
        assert not hasattr(super(), 'draw')
class Shape(Root):
    def __init__(self, shapename, **kw):
        self.shapename = shapename
        super().__init__(**kw)
    def draw(self):
        print('Drawing. Setting shape to:', self.shapename)
        super().draw()
class ColoredShape(Shape):
    def __init__(self, color, **kw):
        self.color = color
        super().__init__(**kw)
    def draw(self):
        print('Drawing. Setting color to', self.color)
        super().draw()

cs = ColoredShape(color='blue', shapename='square')
cs.draw()
print(ColoredShape.__mro__)
>>> Drawing. Setting color to blue
>>> Drawing. Setting shape to: square
>>> (<class '__main__.ColoredShape'>, <class '__main__.Shape'>, <class '__main__.Root'>, <class 'object'>)
```
4. 如果子类想要将其它类注入(inject)到MRO中，那这些类**必须从Root继承**，这样才能在调用`draw()`时，不会`super()`路由到`object`对象从而出现错误。这些规则(从Root继承)必须在类的声明中明确的记录，以便合作类(cooperater)的设计者知道。
    > 例如：在Python中，所有的新异常必须从BaseException继承
#### 调用者和被调用者必须有一个匹配的参数签名
1. 对于传统的预先知道的方法调用，这确实是一个挑战。因为，被调用类(callee)在编写时并不知道super()会调用谁？有可能是一个后写的注入的新类
2. 第一种方法：使用位置参数的固定签名。例如:`__setitem__(key, value)`。但这种方法不够灵活
3. 第二种方法: 使用关键字和关键字字典参数。每个方法使用自己需要的关键字参数，然后将其它剩余参数使用`**kw`传递给下一个方法。每一级玻璃它所需要的关键字参数，因此最终的空字典发送给根本不需要参数的方法。例如：`object.__init__`
```Python
class Shape:
    # 参数shapename的值为circle
    # 参数**kw的值为{}
    def __init__(self, shapename, **kw):
        self.shapename = shapename
        super().__init__(**kw)
class ColoredShape(Shape):
    # 参数color的值为blue
    # 参数**kw的值为{shapename='circle'}
    def __init__(self, color, **kw):
        self.color = color
        super().__init__(**kw)
cs = ColoredShape(color='blue', shapename='circle')
```
#### 方法每次出现都必须使用super()调用
在确保了调用的方法一定存在，以及调用参数签名一致的情况下，我们要做的就是：确保在类的每个方法中使用super()，以确保super()链不中断
#### 注入一个非协作(None-cooperative)类
1. 一个子类想要利用多重继承机制，纳入一个不具备协作能力的第三方类。不具备协作能力是指：不使用`super()`或者不是从`Root`继承或者`__init__`参数签名不一致等等。
```Python
# 不是从Root继承
# __init__参数签名不一致
class Moveable:
    def __init__(self, x, y):
        self.x = x
        self.y = y
    def draw(self):
        print('Drawing at position:', self.x, self.y)
    
```
2. 最简单的方法就是创建一个适配器类`Adapter class`
```Python
# 定义一个Adapter类，从Root继承
class MoveableAdapter(Root):
    # 定义**kw的签名
    def __init__(self, x, y, **kw):
        # 创建第三方类的对象实例
        self.moveable = Moveable(x, y)
        # 使用super()
        super().__init__(**kw)
    def draw(self):
        self.moveable.draw()
        super().draw()
```
3. 然后就可以创建注入了第三方非协作类的新类型
```Python
class MoveableColoredShape(ColoredShape, MoveableAdapter):
    pass

mcs = MoveableColoredShape(color='red', shapename='triangle',
        x=10, y=20)
mcs.draw()

>>> Drawing. Setting color to red
>>> Drawing. Setting shape to: triangle
>>> Drawing at position: 10 20
>>> (<class '__main__.MoveableColoredShape'>, <class '__main__.ColoredShape'>, <class '__main__.Shape'>, <class '__main__.MoveableAdapter'>, <class '__main__.Root'>, <class 'object'>)
```
如果调整基类的顺序后，输出结果就大相径庭
```Python
class MoveableColoredShape(MoveableAdapter, ColoredShape):
    pass

mcs = MoveableColoredShape(color='red', shapename='triangle',
        x=10, y=20)
mcs.draw()

>>> Drawing at position: 10 20
>>> Drawing. Setting color to red
>>> Drawing. Setting shape to: triangle
>>> (<class '__main__.MoveableColoredShape'>, <class '__main__.MoveableAdapter'>, <class '__main__.ColoredShape'>, <class '__main__.Shape'>, <class '__main__.Root'>, <class 'object'>)
```
