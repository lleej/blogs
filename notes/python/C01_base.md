# 基础
## 基本概念
0. 安装位置
   Macos
   系统自带2.7版本
   Python3.x版本 /usr/local/Frameworks/Python.framework/Versions/3.6/lib/python3.6
   第三方库文件 /usr/local/lib/python3.6/site-packages/mysql
1. 文件的后缀为.py
2. 文件的第一行`# -*- coding: utf-8 -*-`能解决中文的错误提示
3. 每行代码段不用添加`;`等结尾字符
4. 代码块跟在`:`后边
5. 代码编写采用缩进方式，缩进使用4个空格，不要混用`Tab`和空格
6. 行长建议不超过80个字符，注释的行长不超过72个字符
7. **大小写敏感**
8. 注释以`#`开头
9. `pass`用于定义空函数或空代码块，防止编译出错

## 数据类型和变量
1. 整数和浮点数，没有大小限制。也就是说不区分32位和64位
2. Python是动态语言，声明变量时不需要表明数据类型(如：int)
3. 没有机制保证常量不被修改，使用全部大写的名称来标明是一个常量

## 字符串和编码
1. 编码标准很多。ASCII(英文)、GB2312(中文)、Shift_JIS(日文)、Euc-kr(韩文)等等
2. 为了避免混用时出现乱码，`Unicode`应运而生
3. Unicode使用**两个字节**表示**一个字符**。但是对于纯英文(如：代码)来说，存储空间浪费了一倍
4. 可变长编码`UTF-8`，支持1-6个字节编码。ASCII字符使用1个字节、汉字使用3个字节。
5. ASCII、Unicode、UTF-8之间的关系

    |字符|ASCII|Unicode|UTF-8|
    |---|------|-------|-----|
    |A |01000001|00000000 01000001|01000001|
    |中|X|01001110 00101101|11100100 10111000 10101101|

6. 计算机内存使用Unicode编码，存储和传输时转换为UTF-8编码
7. 字符串和字节数组(字符串对象的encode()函数编码为字节数组；字节数组对象的decode()函数解码为字符串)
```Python
    >>> 'ABC'.encode('ascii')
    b'ABC'
    >>> '中文'.encode('utf-8')
    b'\xe4\xb8\xad\xe6\x96\x87'
    >>> '中文'.encode('ascii')
    Tranceback (most recent call last):
      File "<stdin>", line1, in <module>
    UnicodeEncodeError: 'ascii' codec can't encode characters in position 0-1: ordinal not in range(128)

    >>> b'ABC'.decode('ascii')
    'ABC'
    >>> b'\xe4\xb8\xad\xe6\x96\x87'.decode('utf-8')
    '中文'
```
8. 格式化
    方法一：使用%实现(更简单直观)
    ```
    >>> 'Hello, %s' % 'world'
    'Hello, world'
    >>> 'Hi, %s, you have $%d.' % ('Michael', 10000)
    'Hi, Michael, you have $10000.'
    ```
    方法二：使用format()函数实现
    ```
    >>> 'Hello, {0}, 成绩提升了 {1:.1f}%'.format('小明', 17.125)
    'Hello, 小明, 成绩提升了 17.1%'
    ```

## 列表和元组
1. 是Python的内置数据类型
2. list是用[]定义，tuple使用()定义
3. list可以对元素进行维护(添加、删除、修改)，而tuple不能对元素进行维护
4. 都可以使用下标进行寻址，第一下标是0，最后一个下标是-1，长度用len()函数取得
5. 同一个list/tuple支持不同数据类型的元素，支持list/tuple的嵌套
6. 定义一个元素的tuple，必须在元素后面加上`,`。因为()还有别的含义

### 列表

1. 列表是有序集合

2. 定义列表

   ```python
   # 直接赋值
   values = [1, 2, 3, 4]
   # 使用range()创建数值型列表
   values = list(range(1, 5))
   ```

   

3. 使用下标访问列表中的元素。从前向后遍历时：第一个元素下标`0`，依次加`1`，直到`元素个数 - 1`为止；从后向前遍历时：第一个元素下标为`-1`，依次 `-1`，直到`-元素个数 + 1`为止

   ```python
   numbers = [1, 2, 3, 4]
   # 第一个元素
   numbers[0]
   numbers[-3]
   # 最后一个元素
   numbers[3]
   numbers[-1]
   ```

4. 增加、修改、删除元素。

   ```python
   ### 增加元素 
   # 在最后列表尾部增加元素 append(item)
   numbers.append(5) 
   # 在列表中插入元素 insert(index, item)
   numbers.insert(0, 0)
   
   ### 修改元素
   numbers[0] = 100
   
   ### 删除元素
   # 删除元素 del 命令
   del numbers[0]
   # 删除并返回元素 item = pop([index])
   num = numbers.pop() # 删除最后一个，num为最后一个元素5
   numbers.pop(0) # 删除第一个
   # 按元素值删除 remove(item)
   # 如果有多个item值，则删除下标最小的那个
   numbers.remove(4) # 删除值为4的元素
   ```

4. 组织列表

   ```python
   # 永久性排序 sort() 改变列表顺序 字符串按首字母排序
   numbers.sort() # 从小到大
   numbers.sort(reverse=True) # 从大到小
   
   # 临时性排序 new = sorted(items) 返回一个排序后的副本
   new = sorted(numbers) # 从小到大
   new = sorted(numbers, reverse=True) # 从大到小
   
   # 对于排序操作而言，如果列表中的元素类型不相同，则不能进行以上操作
   # 例如：news = [0, 'zhangshan', 1, 'lisi']
   # 执行sort() 或 sorted() 时，会引发错误
   
   # 翻转列表元素顺序 reverse() 按下标翻转，而不是元素
   # 永久性改变
   numbers.reverse()
   
   # 列表长度 len()
   len(numbers)
   ```

5. 遍历列表

   ```python
   # 使用 for 语句遍历
   names = ['zhangsan', 'lisi', 'wangwu', 'zhaoliu']
   for name in names: # 依次从names列表中取出一个元素并赋值给name变量
     print(name)
   ```

   

6. 数值型列表

   ```python
   # range(start, stop, [step]) 包含start，但不包含stop。step表示步长，可选参数
   for value in range(1, 5):
     print(value) # 输出1, 2, 3, 4
     
   # 使用列表解析 简化生成代码
   values = [value for value in range(1, 11)]
   
   # 统计计算
   digits = [1, 2, 3, 4, 5]
   # min() 求列表中的最小值
   min(digits)
   # max() 求列表中的最大值
   max(digits)
   # sum() 求列表中值的总和
   sum(digits)
   ```

7. 切片

   ```python
   # 从列表中剪切部分元素 [start:stop:[step]] 
   # start -- 起始下标 不写默认从0开始
   # stop -- 结束下标 不写默认到最后一个(含) 写了则不含该下标
   # step -- 步长
   values = [value value in range(1, 11)]
   values[1:3] # 输出2 3
   values[:2] # 输出1 2 3
   values[-2:] # 输出9 10
   values[:] # 复制列表
   ```

### 元组

1. 元组是一种不可变的列表

2. 元组使用`()`而列表使用`[]`

3. 元组的创建、元素访问的方法与列表一致

   ```python
   # 创建元组
   digits = (1, 2)
   ```

   

## 条件判断
1. 根据Python的缩进规则，来执行条件判断的代码块 **切记!!!**

2. if elif else

   ```python
   userName = 'zhangshan'
   if userName == 'lisi':
     pass
   elif: userName == 'wangwu':
     pass
   else:
     pass
   ```

3. 判断表达式

   ```python
   ### 条件判断：相等 不等 大于 小于 大于等于 小于等于 在其中 不在其中
   ### 返回值：True False
   # 等于 ==
   userName == 'zhangsan'
   # 不等于 !=
   userName != 'zhangsan'
   # 大于 >
   num > 10
   # 小于 <
   num < 10
   # 大于等于
   num >= 10
   # 小于等于
   num <= 10
   # 在其中 常用于 列表 元组 集合 等类型
   1 in [1, 2, 3]
   # 不在其中
   1 not in [2, 3, 4]
   ```

## 循环

1. for循环`for item in items:`

2. for循环需要先生成items

3. while循环`while x < n:`

4. while循环是根据条件判断是否继续

5. continue、break的用法与别的语言一致

   ```python
   message = input("Please input a word I'll check it. for input 'quit' terminated.")
   while True:
     if message.lower() == 'quit':
       break
     print(message)
   
   current = 0
   while current < 10:
     current += 1
     if current % 2 == 0:
       continue
     print(current)
   ```

   

## 字典和集合
1. 字典dict和集合set是内置类型，使用{}定义
2. 字典dict存储键值对，而集合set只存储键值
3. 字典dict和集合set都是无序和无重复元素的集合
3. 字典dict和集合set的键值通过hash算法寻址，因此必须是 **不可变对象(字符串、整数、tuple)**
4. 特点(空间换时间)
    - 查找和插入的速度极快，不会随着key的增加而变慢
    - 需要占用大量的内存，内存浪费多
5. list特点
    - 查找和插入的时间随着元素的增加而增加
    - 占用空间小，浪费内存很少
6. 字典和集合对象，由于其自身的特点，在Python中大量应用
7. 集合对象可以使用`&` `|` 操作符进行交集和并集处理

### 字典

1. 定义字典

   ```python
   person = {'name': 'zhangsan', 'age': 20}
   person = dict({'name': 'zhangsan', 'age': 20})
   ```

2. 访问字典

   ```python
   # 使用键值作为下标,返回对应的值
   person['name'] # zhangsan
   ```

3. 修改、添加、删除

   ```python
   # 直接赋值的方式，如果存在该键值则修改，否则添加
   person['sex'] = 'man'
   
   # 使用del语句删除键值对
   del person['sex']
   ```

4. 遍历字典

   ```python
   # 使用dict.items() 列表 包含(key, value)元组的列表
   for k, v in dict.items():
     ...
   
   # 使用dict.keys() 列表 包含key的列表
   for k in dict.keys():
     ...
   
   # 使用dict.values() 列表 包含value的列表
   for v in dict.values():
     ...
   
   
   ```

5. 字典嵌套

   ```python
   # 字典中键-值对中的值是 字典类型 或者 列表类型
   cities = {
       'beijing': {
           'country': 'China',
           'population': '2000W',
           'fact': '首都',
       },
       'tianjin': {
           'country': 'China',
           'population': '1000W',
           'fact': '滨海城市',
       },
       'shanghai': {
           'country': 'China',
           'population': '1300W',
           'fact': '金融中心',
       },
   }
   # 列表中的数据项是 字典类型
   ```

   