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
6. **大小写敏感**
7. 注释以`#`开头
8. `pass`用于定义空函数或空代码块，防止编译出错

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
6. 定义一个元素的tuple，必须在元素后面加上,。因为()还有别的含义

## 条件判断
1. 根据Python的缩进规则，来执行条件判断的代码块 **切记!!!**
2. if elif else

## 循环
1. for循环`for item in items:`
2. for循环需要先生成items
3. while循环`while x < n:`
4. while循环是根据条件判断是否继续
5. continue、break的用法与别的语言一致

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

