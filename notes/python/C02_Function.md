# 函数
## 基本概念
1. 重复的代码段可以封装到函数中，便于重用和维护
2. 函数可以理解为一种最基本的代码抽象方式

## 内置函数
1. [内置函数](https://docs.python.org/3/library/functions.html)
2. 终端中使用命令行交互`help(函数名)`

## 定义函数
1. 使用`def 函数名(参数):`语句来定义函数
2. 函数返回值用`return`语句返回，一旦执行到`return`时，函数就执行完毕
3. 没有`return`语句，函数执行完毕后返回`None`
5. 函数可以返回多个值，实际是一个tuple类型的值，多个变量可以同时接收一个tuple，写起来更方便
5. 使用三个引号对，来指定函数描述

```python
# 定义函数
def my_abs(x):
  '''
  在这里对函数进行说明
  '''
	if x >= 0:
		return x
	else:
		return -x
	print('end.') #永远也不会执行(因为前面有 return )

print(my_abs(-10))

# 空函数，必须有函数体，因此使用pass
# 返回 None
def nop():
	pass

def my_opt_abs(x):
  # 对参数类型进行检查
	if not isinstance(x, (int, float)):
		raise TypeError('bad operand type for my_opt_abs(): %s' % type(x))
	if x >= 0:
		return x
	else:
		return -x
print(my_opt_abs((1, 2, 3)))

# 返回多个值
def get_user_info(index):
  ...
  return users[index]['name'], users[index]['password']
```

## 调用函数

1. 参数数量和类型不对，会提示`TypeError`错误
2. 可以给函数设置一个别名
3. 调用第三方的函数，需要`import`函数定义文件

## 函数的参数
对于任意函数，都可以通过类似`func(*args, **kw)`的形式调用，不论参数如何定义
### 位置参数

在定义形参时，通常使用位置参数(或者成为必选参数)

1. 定义：`def power(x):` x是一个位置参数
2. 调用：必须传入参数x
3. 说明：必须放在其他类型变量的前面

在调用函数传入实参时：

1. 必须按照形参位置传入参数
2. 使用**关键字实参**，不依赖于位置

```python
def display_pet(type, name):
  print("宠物类型: %s. 宠物名: %s." % (type, name))

# 使用默认方式传参
display_pet('hamster', 'harry') # 宠物类型: hamster. 宠物名: harry
# 如果参数顺序调换
display_pet('harry', 'hamster') # 宠物类型: harry. 宠物名: hamster
# 使用关键字实参传参
display_pet(name='harry', type='hamster') # 宠物类型: hamster. 宠物名: harry
```

### 默认值参数

在定义形参时，给形参设置默认值

1. 定义：`def power(x, n=2):` n=2是一个默认值参数
2. 调用：`power(10, 3)`传入3；`power(10)`默认2；`power(10, n=3)`多个默认值参数需要指定参数名
3. 用途：简化函数调用；扩展参数时向下兼容；
4. 说明：支持多个默认参数；默认参数应放在最后； **默认参数指向不变对象**

```python
# 定义一个函数，4个位置参数，其中两个有默认值
def sum(a, b, c=3, d=4):
  return a + b + c + d

# 调用方式1 使用默认参数
sum(1, 2) # a=1, b=2, c=3, d=4
# 调用方式2 关键字实参
sum(a=1, b=2) # a=1, b=2, c=3, d=4
# 调用方式3 不使用默认值
sum(1, 2, 5, 6) # a=1, b=2, c=5, d=6
# 调用方式4 使用一个默认值
sum(1, 2, c=5) # a=1, b=2 c=5, d=4

# 默认值参数必须是: 不可变对象
# 否则就会像本例一样，调用的结果不随人愿
def add_end(L=[]):
	L.append('END')
	return L
print(add_end()) # ['END']
print(add_end()) # ['END', 'END'] 每次调用的结果不一致

# 修正如下，将默认值由[]列表改为不可变对象None
def add_end(L=None):
	if L is None:
		L = []
	L.append('END')
	return L
print(add_end()) # ['END']
print(add_end()) # ['END']
```

### 可变参数

当传入的参数不确定时，使用可变参数。例如：对多个数值进行求和(参数数量根据需求是可变的)

1. 定义：`def power(*args):` 在参数前面加上`*`
2. 调用：`power(1, 2, 3, 4)`直接传入；`power(*(1, 2, 3, 4))`传入tuple，记得在参数前面加`*`
3. 原理：使用加`*`的参数`*args`，Python将创建一个名为`args`的空元组，将收到的所有值添加到元组中
4. 说明：**传入tuple**；可以传入任意个参数或者0个参数

```python
# 替代方案 是传入一个 列表 / 元组
def calc(numbers):
    sum = 0
    for n in numbers:
        sum = sum + n * n
    return sum
nums = (1, 2, 3)
print(calc(nums)) # 14

# 如果使用可变参数，可以传入多个参数，或者*() *[] *'字符串'...
def calc(*args):
    sum = 0
    for n in args:
        sum = sum + n * n
    return sum
# 多个参数实参
print(calc(1, 2, 3)) # 14
# 使用*[]传入多个实参
print(calc(*[1, 2, 3]))
# 使用*()传入多个实参
nums = (1, 2, 3)
print(calc(*nums))
```

### 命名关键字参数

1. 定义1：`def power(x, n, *, city, job)` 在参数前面加入一个`*,`分隔，`*`参数不接受任何实参。在`*`后的参数传入实参时，必须使用命名关键字方式(即键值对)
2. 定义2：`def power(x, n, *args, city, job)` 在`可变参数*args`后的参数传入实参时，必须使用命名关键字方式(即键值对)
3. 调用：`power(1, 2, city='Beijing', job='Engineer')`
4. 说明：对于关键字参数，如果需要指定参数名称，就需要定义命名关键字参数

```python
# 没有可变参数的情况定义
def person(name, age, *, city, job):
    print('name:', name, 'age:', age, 'city:', city, 'job:', job)
person('Jack', 24, city='Beijing', job='Engineer')

# 有可变参数的情况定义
def person(name, age, *args, city, job):
    print('name:', name, 'age:', age, 'args:',
          args, 'city:', city, 'job:', job)
person('Jack', 24, city='Beijing', job='Engineer')
```

### 可变关键字参数

需要接受任意数量的实参，但预先不知道传递给函数的会是什么类型的参数。使用键值对方式传入

1. 定义：`def power(**dw)` 在参数前面加上`**`
2. 调用：`power(x=1, n=2)`直接传入；`power(**{x:1, n:2})`传入`dict`
3. 说明：**传入dict**；可以传入任意个参数或者0个参数
4. 原理：使用加`**`的参数`**kw`，Python将创建一个名为`kw`的空字典，将收到的所有键值对参数添加到字典中
5. 用途：扩展函数功能，预留出可扩展的参数，例如：用户信息

```python
def person(name, age, **kw):
    print(type(kw))
    print('name:', name, 'age:', age, 'other:', kw)
# 常用的调用方法
person('Michael', 30) # name='Michael' age=30 kw={} 
person('Michael', 30, city='Beijing', job='Engineer') # name='Michael' age=30 kw={'city'='Beijing', 'job'='Engineer'}
extra = {'city': 'Beijing', 'job': 'Engineer'}
person('Michael', 30, **extra)
```

### 参数组合

1. 可以使用以上5种参数组合
2. 组合顺序为：位置参数、默认值参数、可变参数、命名关键字参数/关键字参数
3. 组合不要使用太多，会导致函数接口的可理解性很差

```python
# 参数组合
# 参数顺序：位置参数、可选参数、可变参数、命名关键字参数、关键字参数
def func1(a, b, c=0, *args, **kw):
    print('a=', a, 'b=', b, 'c=', c, 'args=', args, 'kw=', kw)
def func2(a, b, c=0, *, d=0, **kw):
    print('a=', a, 'b=', b, 'c=', c, 'd=', d, 'kw=', kw)
func1(1, 2) # a= 1 b= 2 c= 0 args= () kw= {}
func1(1, 2, 3) # a= 1 b= 2 c= 3 args= () kw= {}
func1(1, 2, 3, 4) # a= 1 b= 2 c= 3 args= (4,) kw= {}
func1(1, 2, 3, 4, 5) # a= 1 b= 2 c= 3 args= (4, 5) kw= {}
func1(1, 2, 3, 4, 5, x=99) # a= 1 b= 2 c= 3 args= (4, 5) kw= {'x': 99}
func1(1, 2, 3, ext=None) # a= 1 b= 2 c= 3 args= () kw= {'ext': None}
func2(1, 2) # a= 1 b= 2 c= 0 d= 0 kw= {}
func2(1, 2, d=99, ext='Beijing') # a= 1 b= 2 c= 0 d= 99 kw= {'ext': 'Beijing'}
```

## 递归函数

1. 在函数内部调用自身，就是递归函数
2. 特点: 定义简单、逻辑清晰(对比循环逻辑)
3. 注意: 防止栈(stack)溢出，嵌套的层级不能过多。当然可以采用 **尾递归**优化来解决。
4. 尾递归: 在函数返回时，调用自身本身 并且 `return`语句不能包含表达式。符合这两个条件时，编译器就会对 **尾递归**做优化，不论调用多少起，都只占用一个栈帧。(Python解释器不支持)

