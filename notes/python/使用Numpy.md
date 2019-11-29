# Numpy是什么

1. 使用`Python`进行科学计算的**基础包**
   - 一个强大的N维数组对象（齐次多维数组）
   - 复杂的（广播）功能
   - 用于集成C / C ++和Fortran代码的工具
   - 有用的线性代数，傅里叶变换和随机数功能
2. 提供大量函数和操作，**快速和高性能**进行数值计算
   - 机器学习模型。如对矩阵的各种数值计算，存储训练数据和机器学习模型的参数
   - 图像处理和计算机图形学。如对多维数据的计算和处理，镜像、旋转图像等
   - 数学任务。如数值积分、微分、内插、外推等

# 数组

> NumPy提供的最重要的数据结构

## 概念

- 齐次多维数组

## 特性

### 固定元素类型

1. 能存储同一类型的数据，已测试：int、float、str
2. 初始化时，固定存储数据类型。在生命周期内不可改变
3. 使用`Python`数组创建时，会根据数组中的数据类型(兼容性)确定
4. 给数组赋值时，会自动进行数据类型转换，失败则报错

```python
import numpy as np

my = np.array([1, 2, 3, 4, 5]) # 都是同一类型 int，则数组类型为 int
print(my)
print(my.dtype)
# 输出
[1 2 3 4 5]
int64 #整形

my = np.array([1, 2, 3, 4, 5.0]) # 有 int 和 flaot , float兼容int，则为float
print(my)
print(my.dtype)
# 输出
[1. 2. 3. 4. 5.]
float64 #浮点型

my = np.array([1, 2, 3, 4, '5']) # 有 int 和 str, str兼容int， 则为str
print(my)
print(my.dtype)
# 输出
['1' '2' '3' '4' '5']
<U1 #字符串长度1

my = np.array([1, 2, 3, 4.0, 'hello']) # 有 int float 和 str, str兼容其他类型， 则为str
print(my)
print(my.dtype)
# 输出
['1' '2' '3' '4.0' 'hello']
<U5 #字符串长度5

my = np.array([1, 2, 3, 4, 5])
my[0] = 'hello' # 固定类型的数组，对元素的类型进行自动转换，失败报错
Traceback (most recent call last):
  File "/Users/lijie/Work/oil-app/untitled/pc.py", line 76, in <module>
    my[0] = 'hello'
ValueError: could not convert string to float: 'hello'
```

### 不固定元素类型

1. 当存储以下类型数据时，数组没有固定类型。如：list、tuple、object等
2. 不固定元素类型的数组，可以存储任意类型元素，且可以任意修改

```python
my = np.array([1, 2, 3.0, '4', (5,)]) #tuple类型，则不强制进行类型一致
print(my)
print(my.dtype)
# 输出
[1 2 3.0 '4' (5,)]
object # 对象类型

my[0] = 'hello' # 可以给任意元素赋值任意类型的数据
print(my)
# 输出
['hello' 2 3.0 '4' (5,)]
```

