[TOC]

## 为什么使用pytest
1. 支持unittest和doctest
2. 支持对测试模块文件目录的嵌套搜索
3. 自动查找模块、类、函数中的测试用例.
   - 模块以`test_` 开头或 `_test`结尾. 如: test_xxx.py
   - 类以`Test`开头. 如: class TestCase(object):
   - 函数以`test_`开头. 如: def test_list():
4. 可以将测试结果以xml文件或者html文件形式输出
5. 可以进行复杂的功能测试
6. 可以和多种CI平台集成

## 安装pytest
1. 安装pytest. `$ pip install pytest`
2. 安装测试覆盖率. `$ pip install pytest-cov`
3. 安装测试报告. `$ pip install pytest-html`

## 使用pytest
### 定义测试模块
测试模块就是一个独立的.py文件. 为了使pytest测试框架能够自动发现, 文件命名必须遵守**以`test_`开头或`_test`结尾**的规范

### 单元测试类和用例函数
```Python
# 模块级的测试用例
# 名称以test_开头
def test_nuber_1():
    assert 3 * 4 == 12, '测试3*4=12'

# 测试类
# 类名以Test开头
class TestCase(object):
    # 运行类中的测试用例前调用1次.
    # 用来进行测试前的初始化工作
    def setup(self):
        pass
    
    # 运行类中的测试用例后调用1次
    # 用来进行测试后的清理工作
    def teardown(self):
        pass
    
    # 测试用例
    # 以test_开头
    # 普通断言(Python内置assert函数)
    def test_number_2(self):
        assert 5*6 == 30, '测试5*6==30'
    
    # 异常断言(pytest的raise函数)
    def test_error_1(self):
        with pytest.raise(RuntimeError) as e:
            1 / 0
```
### 高级特性
`@pytest.fixture`装饰器
作用类似于unittest的setup()和teardown()函数
1. 可以命名，通过函数、类、模块、项目来激活
2. 放到不同的模块中，可以触发其他fixture
3. 支持简单的单元测试和复杂的功能测试，可以配置其参数，将fixture作为参数传递. 而且可以跨模块、类来共享使用
```Python
import pytest

@pytest.fixture
def smtp():
    import smtplib
    return smtplib.SMTP("smtp.gmail.com")

def test_ehlo(smtp):
    response, msg = smtp.ehlo()
    assert response == 250
    assert 0 # for demo purposes
```
以上测试代码的讲解:
1. pytest通过`test_`前缀找到了`test_ehlo`测试用例
2. pytest发现这个测试用例需要一个`smtp`的函数入参
3. pytest通过`@pytest.fixture`装饰器找到了有同样名称`smtp`的`fixture`函数
4. 调用`smtp()`函数，返回一个实例对象
5. 调用`test_ehlo(<smtp instance>)，执行测试断言

如果fixture函数名出现拼写错误或者函数入参不存在：
```Python
_________________________ ERROR at setup of test_ehlo _________________________
file d:\Works\Python\pythonstudy\my_test.py, line 24
  def test_ehlo(smtp):
E       fixture 'smtp' not found
>       available fixtures: cache, capfd, capfdbinary, caplog, capsys, capsysbinary, cov, doctest_namespace, metadata, monkeypatch, pytestconfig, record_xml_property, recwarn, smtp1, tmpdir, tmpdir_factory
>       use 'pytest --fixtures [testpath]' for help on them.
```  

### 运行pytest
pytest测试框架会自动查找并运行所有测试用例. 
```Python
py.test               # 当前目录下所有测试  
py.test test_mod.py   # 指定模块测试  
py.test somepath      # 指定目录下所有测试  
py.test -k stringexpr # 测试用例名称匹配"string expression"
                      # 例如: stringexpr为"MyClass and not method"  
                      # TestMyClass.test_something可以  
                      # TestMyClass.test_method_simple不可用  
py.test test_mod.py::test_func # 指定测试用例
                      # 例如"test_mod.py::test_func"仅仅测试test_func in test_mod.py 
```
如果指定--html参数，则生成HTML格式的测试报告，并存储到outputfile中
```Python
$ py.test --html=outputfile
```

