# 面向对象
## 类和实例
1. 一切皆为对象
2. 类名的首字母大写
3. 类的构建函数为`__init__`
4. 类的方法第一个参数必须是`self`，在调用时不用传入

```python
class Student(object):
	def __init__(self, name, score):
		self.name = name
		self.score = score

	def print_score(self):
		print('%s: %s' % (self.name, self.score))

tom = Student('Tom', 90)
Lisa = Student('Lisa', 80)
tom.print_score()
Lisa.print_score()
print('%s: %s' % (tom.name, tom.score))
```



## 访问限制

1. 类中的所有变量都可以直接访问
2. 实例可以动态绑定属性，因此同一个类的不同实例可能不相同
3. 为了保护内部变量不被访问，在命名上添加`__`两个下划线前缀。这时，解释器自动给该变量添加了前缀`_类名`，但不建议直接操作
4. 添加get_属性/set_属性方法，对属性进行读写操作
5. 因为实例可以动态绑定属性，因此通过属性名直接访问的可能不是真正的

```python
## 动态绑定属性
tom.age = 20
print("%s: %d" % (tom.name, tome.age))

# 加入访问限制
class Student(object):
	def __init__(self, name, score):
		self.__name = name
		self.__score = score

	def print_score(self):
		print('%s: %s' % (self.__name, self.__score))

tom = Student('Tom', 90)
Lisa = Student('Lisa', 80)
tom.print_score()
Lisa.print_score()

# 下面这句会报错,因为没有name属性了
#print('%s: %s' % (tom.name, tom.score))
# 下面这句会报错,也没有__name属性了
#print('%s: %s' % (tom.__name, tom.__score))
# 下面这句可以直接访问,但强烈不建议
print('%s: %s' % (tom._Student__name, tom._Student__score))

## 使用get/set方法，保护变量
## 把Student对象中的gender对外隐藏
class Student(object):
	def __init__(self, name, gender):
		self.__name = name
		self.__gender = gender
	def get_gender(self):
		return self.__gender
	def set_gender(self, gender):
		self.__gender = gender

bart = Student('Bart', 'male')
if bart.get_gender() != 'male':
    print('测试失败!')
else:
    bart.set_gender('female')
    if bart.get_gender() != 'female':
        print('测试失败!')
    else:
        print('测试成功!')
```

## 继承和多态
1. 继承。子类继承父类的方法和属性；子类具有父类类型`isinstance()`
2. 多态。继承自同一父类的不同子类，在执行时体现不同的状态
3. 鸭子类型。只要目标对象与参照对象的接口一致，就可以直接使用。不像静态语言必须存在继承关系

```python
### 继承
# 基类
class Animal():
  def __init__(self, name):
    self.name = name
  def run(self):
    print("Animal", self.name, "Running.")

# 继承类
class Dog(Animal):
  def __init__(self, name):
    super().__init__(name)
  
  def run(self):
    print("Dog", self.name, "Running.")

class Cat(Animal):
  def __init__(self, name):
    super().__init__(name)
  
  def run(self):
    print("Cat", self.name, "Running.")

# 实例化
animal = Animal("A:xiao")
dog = Dog("D:WangWang")
cat = Cat("C:MiaoMiao")
animal.run()
dog.run()
cat.run()
```

```python
## 继承关系
## 通过isinstance查看
print('dog is a Dog.', isinstance(dog, Dog))
print('dog is a Animal.', isinstance(dog, Animal))

print('animal is a Animal.', isinstance(animal, Animal))
print('animal is a Dog.', isinstance(animal, Dog))
```

```python
## 多态
def run_twice(animal):
	animal.run()
	animal.run()

## 传入对象不同，执行结果不同
run_twice(animal)
run_twice(dog)
run_twice(cat)
```

```python
## 鸭子类型
class Car(object):
	def run(self):
		print('Car is running...')

car = Car()
run_twice(car)
```

## 获取对象信息
全部是Python内置函数
1. 使用`type(obj, bases, dict)`函数. 返回对象类型. 不考虑继承关系
    - obj. 实例对象
    - bases. 基类的元组
    - dict. 字典. 类内定义的命名空间变量
2. 使用`isinstance(obj, classinfo)`函数. 判断对象类型，考虑继承关系
    - obj. 实例对象
    - classinfo. 直接或间接类型、基本类型或它们组成的元组. 元组时存在一种就为真
3. 使用`dir([obj])`函数. 返回参数的变量、方法和定义的类型列表
    - obj. 可选参数. 对象、变量、类型
4. 使用`hasattr(obj, prop)`函数. 判断对象是否包含对应的属性
    - obj. 实例对象
    - prop. 属性名字符串
5. 使用`getattr(obj, prop[, default])`函数. 返回对象的属性值
    - obj. 实例对象
    - prop. 属性名字符串
    - default. 默认返回值, 如果不提供该参数, 在没有对应属性时触发AttributeError
6. 使用`setattr(obj, prop, value)`函数. 设置对象的属性
    - obj. 实例对象
    - prop. 属性名字符串
    - value. 属性值

```python
# 导入模块
import types

class T(object):
	def __len__(self):
		return 100
def fn():
	pass
t = T()
# type(obj, bases, dict)函数
# 返回对象的类型
print(type(12)) # <class 'int'>
print(type('12')) # <class 'str'>
print(type(None)) # <class 'NoneType'>
print(type(types)) # <class 'module'>
print(type(abs)) # <class 'builtin_function_or_method'>
print(type(T)) # <class 'type'>
print(type(t)) # <class '__main__.T'>
print(type(fn)) # <class 'function'>

print(type(t) == T) # True
print(type(12) == int) # True

# 类型
# types.FunctionType, types.BuiltinFunctionType, types.LambdaType, types.GeneratorType
print('fn is FunctionType? ', type(fn) == types.FunctionType) # True
print('abs is BuiltinFunctionType? ', type(abs) == types.BuiltinFunctionType) # True
print('lambda x: x is LambdaType? ', type(lambda x: x) == types.LambdaType) # True
print('(x for x in range(10)) is GeneratorType? ', type((x for x in range(10))) == types.GeneratorType) # True


# isinstance(obj, classinfo)函数
## classinfo是元组时，只要有obj的类型就为真, 顺序没有影响
print('isinstance(12, (int, object))', isinstance(12, (int, str))) # True
print('isinstance(abs, (int, types.BuiltinFunctionType))', isinstance(abs, (int, types.BuiltinFunctionType))) # True

# dir([obj])
#返回
#['T', '__annotations__', '__author__', '__builtins__', '__cached__', '__doc__', 
#'__file__', '__loader__', '__name__', '__package__', '__spec__', 'fn', 't', 'types']
print(dir())

print(dir(t))
print('t的长度: ', len(t))

# hasattr(obj, prop)
print('对象t有x属性吗? ', hasattr(t, 'x')) # False
# setattr(obj, prop, value)
setattr(t, 'x', 10)
# getattr(obj, prop[, default])
print('t对象的x属性值: ', getattr(t, 'x'))
# getattr, 没有属性值时，返回404，不报错
print('t对象的y属性值: ', getattr(t, 'y', 404))

print('现在, 对象t有x属性吗? ', hasattr(t, 'x'))

# 最好的方式
if hasattr(t, 'x'):
	print(getattr(t, 'x'))
```



## 实例属性和类属性
1. 实例属性. 实例可以任意绑定属性，通过self变量或者实例变量
    - self.name = '111'
    - s.score = 90
2. 类属性. 类也可以绑定属性，需要在class中定义
    - name = '111'
3. 当类属性与实例属性名称相同时，通过访问实例属性时，类属性将被屏蔽

```python
# ctype 是 类变量 
class Car():
    ctype = 'human'
    def __init__(self, name):
        self.name = name
        
car1 = Car("1")
car2 = Car("2")
print("car1:", car1.name)
# 可以使用实例 或者 类 来访问类变量
print("car2:", car2.ctype)
print("Car:", Car.ctype)
```



#### 总结
1. 实例属性属于各个实例所有，各自可以不同，互不干扰
2. 类属性属于类所有，所有实例共享一个属性
3. 类属性和实例属性不要重名！！！

