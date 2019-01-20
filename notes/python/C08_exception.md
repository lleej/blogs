# 异常
## 异常处理
1. 避免程序的处理进入不可控状态
2. 语法: 
    ```
    try: 
        do ...
    except xxxError as e: 
        do ...
    except yyyError as e:
        do ...
    else:
        do ...
    finally
        do ...
    ```
3. 说明:
    - 异常捕获按顺序执行. 如果当前异常与`except`异常类匹配，则不再执行后面的异常处理代码
    - 异常类会把其子类的异常也“一网打尽”. 所以不要设置同一继承关系的异常类处理
    - 如果没有异常则会执行`else:`部分的代码块
    - 不管有没有异常，最终都会执行`finally`部分的代码块

## 调试
1. 使用print()输出变量值
2. 使用`assert xx == yy, '提示信息'`. 如果断言失败，则抛出`AssertionError` 
3. 使用`logging`类记录日志.
4. 使用pdb调试器
5. IDE环境

## 单元测试
1. 引用Python有内置的模块`unittest`
2. 创建测试类`class TestXXX(unittest.TestCase): ...`
    - 必须从`unittest.TestCase`继承
    - 类名建议以`Test`开头，但不是强制
3. 定义测试用例`def test_xxx(self): ...`
    - 定义类成员函数`(self)`
    - 函数名称必须遵守`test_xxx`规范. 注意`test_`的书写格式
4. 在测试用例中,使用各种断言进行判断
    - assertEqual(a, b). 判断 a == b
    - assertTrue(a == b). 判断 a == b 是真的
    - assertFalse(a == b). 判断 a == b 是假的
    - assertRaises(ErrObj). 判断是否会触发异常. ErrObj是异常类型.

