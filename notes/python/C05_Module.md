# 模块
## 模块
1. 模块是一组Python代码的集合, 一个独立的.py文件就是一个模块，模块的名称就是文件名
2. 模块命名不能重复，否则会发生冲突(当然也不能与系统模块名冲突，否则将无法使用系统模块)
3. 为了避免模块命名冲突，在Python中也有包(Package)的概念，包是以目录的形式存在
4. 包中(目录下)，必须有一个`__init__.py`文件，否则就是一个普通目录
5. `__init__.py`文件也是一个模块，其模块名是包名(即目录的名称)
6. 举例：bocoits\abc.py
    - 包名: bocoits
    - 模块名: bocoits.abc
7. 模块便于维护(代码有限)，也便于重用(写好的模块可以直接引用)

#### 标准模板
1. 模板头部
```
#!/usr/bin/env python3
# -*- coding: utf-8 -*-

'这是一个模块的标准模板'

__author__ = 'jacklee'
```
2. 模板头部说明
    - 注释1: 能够让.py文件可以直接在Unit/Linux/Mac上运行
    - 注释2: 表示.py文件本身使用标准UTF-8编码
    - 字符串: 模板的文档注释，任何模块代码的第一个字符串为视为模块的文档注释
    - 变量: `__author__`把作者写进去

#### 模块命名
1. 不能包含`.`. 如：`module.test`是错误的
2. 不能以数字开头. 如：`05module`是错误的
    
## 如何使用模块
要使用模块，必须先导入模块
#### 导入模块
1. 使用`import`语句
    - 语法: `import 模块名`
    - 一次可以导入多个模块. 如: `import module1[, module2[,...moduleN]`
    - 包名以`.`分隔, 如：`import module.test`，包名module，模块名test
    - 不可以导入函数、变量、类
2. 使用`from...import`语句
    - 导入函数[变量]: `from collections import iterable`
    - 导入模块: `from mycompay import utils`
    - 导入相对路径: `from . import moduleA`, 导入模块当前目录下的moduleA模块
3. 导入模块寻址
    - Python解释器默认使用`sys.path`中的目录查找导入的模块

4. 符号表(导入后的可见名称)
    - 使用`import`导入的模块，模块名称可见
    - 使用`from...import`导入的函数[变量]，只有函数[变量]可见
    - 使用`from...import *`导入所有函数[变量](除`_`开头的名字)可见
    - 使用内置函数`dir(模块名)`可以查看当前模块所有变量[函数]

#### 使用模块
1. 使用`import`语句导入的，使用模块内部的变量[函数]时，必须带上完整的模块名称(含包名)
如：`import sys  args = sys.argv`
3. 使用`from...import`语句导入的，使用`import`后的模块名[函数名[变量名]]即可

#### 模块内变量[函数]定义
1. 开放给其它模块使用的变量[函数]用标准的驼峰命名方式
2. 模块内私有的变量[函数]加入`_`前缀(Python不限制). 如：`_print()`就是私有函数

#### 其他
1. __init__.py文件. 包的初始化文件
2. __all__变量. 管理所有通过`from ... import *`语句导入的模块名，如: __all__ = ['a', 'b', 'c']
3. __main__变量. 主模块的名称. 可以通过判断模块的`__main__`变量是否为主模块
4. 

## 引入第三方模块
1. 使用pip工具. Linux/Mac下已经安装了.
    - 安装pip工具
    - 在Python官网/[pypi](https://pypi.python.org)网站查找库名
    - 运行`pip3 install 库名`安装
2. 使用`Anaconda`工具
    - 安装[Anaconda](https://www.anaconda.com/download)
3. 模块搜索
    - 默认搜索当前目录、已安装内置模块、第三方模块. 存于`sys.path`变量中
    - 在程序中动态添加搜索目录. `sys.path.append('目录')
    - 设置环境变量`PYTHONPATH`
