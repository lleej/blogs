title: Django-REST框架
date: 2019-06-25
tags: django
categories: python
layout: draft

------

摘要：学习REST框架的官方文档，整理出使用框架的步骤和最佳实践。

问题：

1. 在`models.py`中创建一个实体类后，在`admin管理界面`中的权限列表中就可以看到该实体类的四个权限：`add`、`modify`、`delete`、`view`
2. 如何向系统用户表中添加自定义的字段？还是需要自己维护单独的用户表？

<!-- more -->

# 环境

1. Python高于3.5版本
2. Django高于2.0版本

可选安装包

1. coreapi (模式生成支持)
2. markdown (对可浏览API的支持)
3. pygments (为markdown处理添加语法突出显示)
4. django-filter (过滤支持)
5. django-guardian (对象级权限支持)

# 创建项目

创建新的`Django`项目，然后启动一个新的应用程序

## 项目设置

创建项目专用目录

```shell
# 创建项目目录
mkdir tutorial
cd tutorial
```

## 创建虚拟环境

为了保证项目中使用的包不污染全局环境，针对项目创建虚拟环境

```shell
# 创建虚拟环境
python3 -m venv env
# 激活虚拟环境
source env/bin/activate
# 退出虚拟环境
deactivate
```

## 安装依赖包

```shell
# 安装Django和REST框架
pip install django
pip install djangorestframework
# 选装项目使用的包
pip install markdown # 支持markdown语法
pip install Pygments # 支持高亮显示
pip install django-filter # 过滤
pip install coreapi # 自动生成API文档
pip install pyyaml # 文档YAML的OpenAPI格式
pip install django-cors-headers # 跨域支持 CORS
```

## 创建项目

在项目专用目录中创建项目`tutorial`

```shell
# 别忘了最后的 .
django-admin startproject tutorial .
cd tutorial
# 创建应用程序
django-admin startapp snippets
cd ..
```

完成项目初始化后，项目的目录结构为

```shell
./manage.py
./tutorial
./tutorial/__init__.py
./tutorial/snippets
./tutorial/snippets/__init__.py
./tutorial/snippets/admin.py
./tutorial/snippets/apps.py
./tutorial/snippets/migrations
./tutorial/snippets/migrations/__init__.py
./tutorial/snippets/models.py
./tutorial/snippets/tests.py
./tutorial/snippets/views.py
./tutorial/settings.py
./tutorial/urls.py
./tutorial/wsgi.py
```

## 创建超级用户

```shell
# 超级用户用户管理Django后台
python manage.py createsuperuser
# 按提示输入用户名 邮箱 密码
```

## 设置

在`settings.py`中设置`INSTALLED_APPS`

```python
INSTALLED_APPS = (
	...
  'rest_framework'
)
```

# 创建应用程序

我们假设创建一个`snippets`的应用程序

一个应用程序只有一个`models.py`文件，也就是这个应用的实体类(ORM)模块。

`Django`通过这种方式，进行模块的划分

## 创建应用

```shell
# 1.使用django-admin创建
django-admin startapp snippets
# 2.使用manage.py创建
python manage.py startapp snippets
```

## 配置应用

将应用程序添加到`settings.py`中

```python
INSTALLED_APPS = (
	...
  'snippets.apps.SnippetConfig'
)
```

# 创建一个REST

## 创建实体类

通过`Django`脚手架自动初始化应用程序的`models.py`模块，在该模块中声明实体类

```python
# 引入models模块，使用其中的Model类
from django.db import models

# Create your models here.

# 从Model类继承，Model类封装了对表进行SQL操作的方法
# 实体类的名称即为表名
class Snippet(models.Model):
    # 定义字段
    created = models.DateTimeField(auto_now_add=True)
    title = models.CharField(max_length=100, blank=True, default='')
    code = models.TextField()
    linenos = models.BooleanField(default=False)
    language = models.CharField(
        choices=LANGUAGE_CHOICES, default='python', max_length=100)
    style = models.CharField(choices=STYLE_CHOICES,
                             default='friendly', max_length=100)
    owner = models.ForeignKey(
        'auth.User', related_name='snippets', on_delete=models.CASCADE)
    highlighted = models.TextField()

    class Meta:
        ordering = ('created',)

    def save(self, *args, **kwargs):
        lexer = get_lexer_by_name(self.language)
        linenos = 'table' if self.linenos else False
        options = {'title': self.title} if self.title else {}
        formatter = HtmlFormatter(
            style=self.style, linenos=linenos, full=True, **options)
        self.highlighted = highlight(self.code, lexer, formatter)
        super().save(*args, **kwargs)
```

## 迁移数据

基于`models.py`模块中定义的实体类，自动创建相应的表。

### makemigrations

`makemigrations`指令，根据`models.py`中的模型定义生成迁移文件

```bash
# 语法
# 不提供应用程序，默认将所有应用的Models.py中的实体类进行ORM操作
python manage.py makemigrations [应用名称]
```

执行`makemigrations`指令后，在应用程序的`migrations`目录下，创建相应的初始化脚本`*.py`

对应用程序的`Models.py`中的实体进行修改后，该指令会生成新的脚本文件

### migrate

`migrate`指令，负责执行或取消执行迁移，即具体更新数据库操作

```bash
# 语法
# 不提供应用程序，默认对所有应用的脚本进行数据库操作
python manage.py migrate [应用名称]
```

框架如何知道哪个脚本已经执行，哪些脚本没有执行？

执行后，将脚本文件与模块的对应关系保存在`django_migrations`表中。

如果新的脚本没有执行，或者出现异常需要重新进行，可以检查或手动修改表中的数据

### 重新初始化

在开发过程中，难免对`Models.py`进行修改。每次修改都会生成新的脚本文件，脚本文件增多难免出现问题。

因此，到应用程序开发的后期`Models.py`相对稳定时，可以对脚本进行重构。

- 删除应用下`migrations`目录下的所有脚本文件
- 删除`django_migrations`表中的所有记录
- 重新执行`makemigrations`和`migrate`指令

### 其他指令

`sqlmigrate`指令，显示迁移文件中的`SQL`语句

`showmigrations`指令，列出项目额迁移及其状态

```shell
python manage.py makemigrations snippets
python manage.py migrate
```

## 创建序列化类

将实体类对象与`JSON`串进行双向转换。脚手架默认没有创建该模块，需要手动添加，命名为`serializers.py`

```python
# 引入serializers模块
from rest_framework import serializers
from django.contrib.auth.models import User
from snippets.models import Snippet

"""
# 使用 Serializer 类进行序列化
class SnippetSerializer(serializers.Serializer):
    id = serializers.IntegerField(read_only=True)
    title = serializers.CharField(
        required=True, allow_blank=True, max_length=100)
    code = serializers.CharField(style={'base_template', 'textarea.html'})
    linenos = serializers.BooleanField(required=False)
    language = serializers.ChoiceField(
        choices=LANGUAGE_CHOICES, default='python')
    style = serializers.ChoiceField(choices=STYLE_CHOICES, default='friendly')

    def create(self, validated_data):
        '''
        创建并返回一个新的'Snippet'实例，给定有效数据
        '''
        return Snippet.objects.create(**validated_data)

    def update(self, instance, validated_data):
        instance.title = validated_data.get('title', instance.title)
        instance.code = validated_data.get('code', instance.code)
        instance.linenos = validated_data.get('linenos', instance.linenos)
        instance.language = validated_data.get('language', instance.language)
        instance.style = validated_data.get('style', instance.style)
        instance.save()
        return instance
"""

# 使用 ModelSerializer类序列化，可以简化代码
class SnippetSerializer(serializers.ModelSerializer):
    class Meta:
        model = Snippet
        fields = ('id', 'title', 'code', 'linenos', 'language', 'style')

# 用户表中没有snippet的数据，需要手动创建，并在`Meta`类中添加该字段
class UserSerializer(serializers.ModelSerializer):
    snippets = serializers.PrimaryKeyRelatedField(
        many=True, queryset=Snippet.objects.all())

    class Meta:
        model = User
        fields = ('id', 'username', 'snippets')
```

## 定义视图

### 常规模式

视图是连接Web请求和对象处理的桥梁，脚手架自动创建`views.py`

使用`rest`框架的`generics`模块，可以使用基于类的通用视图来简化代码

有三种方式实现视图：

1. 基于函数的视图`@api_view`
2. 基于类的视图，继承自`APIView`
3. 基于类的通用视图，继承自`generics.XXXAPIView`

```python
from rest_framework import generics
from django.contrib.auth.models import User
from snippets.models import Snippet
from snippets.serializers import SnippetSerializer, UserSerializer

# 基于类的通用视图
class SnippetList(generics.ListCreateAPIView):
    # 设置数据集
    queryset = Snippet.objects.all()
    # 设置序列化类
    serializer_class = SnippetSerializer

class SnippetDetail(generics.RetrieveUpdateDestroyAPIView):
    queryset = Snippet.objects.all()
    serializer_class = SnippetSerializer

class UserList(generics.ListAPIView):
    queryset = User.objects.all()
    serializer_class = UserSerializer

class UserDetail(generics.RetrieveAPIView):
    queryset = User.objects.all()
    serializer_class = UserSerializer

'''
from rest_framework import status
from rest_framework.decorators import APIView
from rest_framework.response import Response
from django.http import Http404
from snippets.models import Snippet
from snippets.serializers import SnippetSerializer

# 基于类的Views
class SnippetList(APIView):
    def get(self, request, format=None):
        snippets = Snippet.objects.all()
        serializer = SnippetSerializer(snippets, many=True)
        return Response(serializer.data)

    def post(self, request, format=None):
        serializer = SnippetSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

class SnippetDetail(APIView):
    def get_object(self, pk):
        try:
            return Snippet.objects.get(pk=pk)
        except Snippet.DoesNotExist:
            raise Http404

    def get(self, request, pk, format=None):
        snippet = self.get_object(pk)
        serializer = SnippetSerializer(snippet)
        return Response(serializer.data)

    def put(self, request, pk, format=None):
        snippet = self.get_object(pk)
        serializer = SnippetSerializer(snippet, data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

    def delete(self, request, pk, format=None):
        snippet = self.get_object(pk)
        snippet.delete()
        return Response(status=status.HTTP_204_NO_CONTENT)
'''

'''
from rest_framework import status
from rest_framework.decorators import api_view
from rest_framework.response import Response
from snippets.models import Snippet
from snippets.serializers import SnippetSerializer

# 基于函数的View
@api_view(['GET', 'POST'])
def snippet_list(request):
    if request.method == 'GET':
        snippets = Snippet.objects.all()
        serializer = SnippetSerializer(snippets, many=True)
        return Response(serializer.data)

    elif request.method == 'POST':
        serializer = SnippetSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)


@api_view(['GET', 'PUT', 'DELETE'])
def snippet_detail(request, pk):
    try:
        snippet = Snippet.objects.get(pk=pk)
    except Snippet.DoesNotExist:
        return Response(status=status.HTTP_404_NOT_FOUND)

    if request.method == 'GET':
        serializer = SnippetSerializer(snippet)
        return Response(serializer.data)
    elif request.method == 'PUT':
        serializer = SnippetSerializer(snippet, data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
    elif request.method == 'DELETE':
        snippet.delete()
        return Response(status=status.HTTP_204_NO_CONTENT)
'''

```

### 使用ViewSet模式

```python
from rest_framework import viewsets
from rest_framework.decorators import action
from rest_framework.response import Response
from rest_framework import permissions, renderers
from django.contrib.auth.models import User
from snippets.models import Snippet
from snippets.serializers import SnippetSerializer, UserSerializer
from snippets.permissions import IsOwnerOrReadOnly

# 基于viewset

# 将同一模型中的多个视图合并成一个
# userlist 和 userdetail 合并成一个

class UserViewSet(viewsets.ReadOnlyModelViewSet):
    '''
    viewset 自动提供 list 和 detail 动作
    '''
    queryset = User.objects.all()
    serializer_class = UserSerializer

class SnippetViewSet(viewsets.ModelViewSet):
    '''
    viewset 自动提供 list create retrieve update 和 destroy 动作

    另外，我们可以在其中扩展一个 highlight 动作
    '''
    queryset = Snippet.objects.all()
    serializer_class = SnippetSerializer
    permission_classes = (
        permissions.IsAuthenticatedOrReadOnly, IsOwnerOrReadOnly,)
		
    # 额外的方法使用@action装饰器
    @action(detail=True, renderer_classes=[renderers.StaticHTMLRenderer])
    def highlight(self, request, *args, **kwargs):
        snippet = self.get_object()
        return Response(snippet.highlighted)

    def perform_create(self, serializer):
        serializer.save(owner=self.request.user)
```



## 定义路由

建立请求地址与处理函数的关系。涉及到两个路由文件

### 应用程序的路由文件

文件位置：`[应用程序目录]\urls.py`

```python
# 项目的路由文件
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('snippets.urls')),
]
```

### 项目的路由文件

文件位置：`[项目根目录]\urls.py`

#### 常规视图模式

在常规的视图模式下`（4.4.1）`，需要自定义路由地址和视图之间的关系

```python
# 应用程序的路由文件
from django.urls import path
from snippets import views

urlpatterns = [
    path('snippets/', views.SnippetList.as_view()),
    path('snippets/<int:pk>/', views.SnippetDetail.as_view()),
    path('users/', views.UserList.as_view()),
    path('users/<int:pk>/', views.UserDetail.as_view()),
]
```

#### 路由模式下

在路由模式(ViewSet)下，`router`模块定义了标准的`url`地址，避免了`url`地址格式的不统一

```python
from django.urls import path, include
from snippets import views
from rest_framework.routers import DefaultRouter

# 使用路由router注册views
router = DefaultRouter()
router.register('snippts', views.SnippetViewSet)
router.register('users', views.UserViewSet)

urlpatterns = [
    path('', include(router.urls)),
]
```

## 定义权限

自定义对象访问权限，创建`permissions.py`模块

```python
from rest_framework import permissions

class IsOwnerOrReadOnly(permissions.BasePermission):
    '''
    自定义权限，仅允许对象的所有者编辑。其他人都可以只读查看
    '''
    def has_object_permission(self, request, view, obj):
        if request.method in permissions.SAFE_METHODS:
            return True
        return obj.owner == request.user
```

修改对象的视图类`views.py`，加入权限控制

```python
class SnippetList(generics.ListCreateAPIView):
    queryset = Snippet.objects.all()
    serializer_class = SnippetSerializer
    # 设置权限类，注意`permission_classes`是一个复数
    # 第一个权限表示授权用户才能修改，否则只读
    # 第二个自定义权限表示只有所有者才能修改
    permission_classes = (
        permissions.IsAuthenticatedOrReadOnly, IsOwnerOrReadOnly)

    def perform_create(self, serializer):
        serializer.save(owner=self.request.user)
        return super().perform_create(serializer)


class SnippetDetail(generics.RetrieveUpdateDestroyAPIView):
    queryset = Snippet.objects.all()
    serializer_class = SnippetSerializer
    permission_classes = (
        permissions.IsAuthenticatedOrReadOnly, IsOwnerOrReadOnly)
```

修改项目级的路由文件`urls.py`，加入权限认证

```python
urlpatterns = [
    ...
    path('api-auth/', include('rest_framework.urls')),
]
```

## 自动化文档

### 依赖项

1. Coreapi
2. pyyaml

### 代码实现

在应用程序的`urls.py`文件中，加入如下代码

```python
from rest_framework.schemas import get_schema_view

schema_view = get_schema_view(title='Pastebin API')

urlpatterns = [
    path('schema/', schema_view),
    ...
]
```



# 模型

## 继承

模型也支持继承关系，在实际使用时，针对不同场景，有不同的解决方案。很巧妙

## 抽象继承

**不会创建抽象模型对应的数据表**

场景：对于多个模型都使用的共有属性，可以使用抽象模型

使用：在`Meta`类中加入`abstract = True`即可

```python
from django.db import models

class CommonInfo(models.Model):
    name = models.CharField(max_length=100)
    age = models.PositiveIntegerField()

    class Meta:
        abstract = True

class Student(CommonInfo):
    home_group = models.CharField(max_length=5)
```

`Student`模型会继承`CommonInfo`中所有的字段`name, age`

不会创建`CommonInfo`对应的数据表

## 多表继承

**每个模型均会创建对应的数据表，两个表之间建立`OneToOne`的关联关系**

场景：监控项目中的设备信息表的设计。所有设备都有相同的属性，通过设备表可以查询到所有的设备，而不用检索不同的表。但是，不同类型的设备有一些特有的字段。这样就可以采用这种方式

```python
from django.db import models

class Place(models.Model):
    name = models.CharField(max_length=50)
    address = models.CharField(max_length=80)

class Restaurant(Place):
    serves_hot_dogs = models.BooleanField(default=False)
    serves_pizza = models.BooleanField(default=False)
```

而且，对于`Place`的实例，可以通过`place.restaurant`返回`Restaurant`实例

多表继承实际上是`django`框架在后台自动添加了一个`OneToOneField`

注意：`parent_linke=True`指向其父类模型表的关系

```python
place_ptr = models.OneToOneField(
    Place, on_delete=models.CASCADE,
    parent_link=True,
)
```

如果希望自己指定字段名称，可以自己手动创建关联字段。

注意：

1. 基类必须手动指定主键`primary_key`，并设置默认值
2. 子类的`OneToOneField`参数中加入`parent_link=True`

```python
class Place(models.Model):
    id = models.AutoField(primary_key=True, default=1)
    name = models.CharField(max_length=50)
    address = models.CharField(max_length=80)

    def __str__(self):
        return self.name


class Restaurant(Place):
    place = models.OneToOneField(Place, on_delete=models.CASCADE, parent_link=True)
    serves_hot_dogs = models.BooleanField(default=False)
    serves_pizza = models.BooleanField(default=False)

    def __str__(self):
        return self.name
```



## 代理模型

**如果只想通过子类扩展基类的Python功能，而不增加新的数据表**

在模型的 `Meta` 类中添加 `proxy = True`

```python
from django.db import models

class Person(models.Model):
    first_name = models.CharField(max_length=30)
    last_name = models.CharField(max_length=30)

class MyPerson(Person):
    class Meta:
        proxy = True

    def do_something(self):
        # ...
        pass
```



## 未理解的部分

### 关联关系

1. `ManyToMany`、`ForienKey`、`OneToOne`等的应用场景
2. `related_name`、`related_query_name`的使用场景



# 竞争条件

我们的 `vote()` 视图代码有一个小问题。代码首先从数据库中获取了 `selected_choice` 对象，接着计算 `vote` 的新值，最后把值存回数据库。如果网站有两个方可同时投票在 *同一时间* ，可能会导致问题。同样的值，42，会被 `votes` 返回。然后，对于两个用户，新值43计算完毕，并被保存，但是期望值是44。

这个问题被称为 *竞争条件* 。如果你对此有兴趣，你可以阅读 [Avoiding race conditions using F()](https://docs.djangoproject.com/zh-hans/2.2/ref/models/expressions/#avoiding-race-conditions-using-f) 来学习如何解决这个问题。

```python
reporter = Reporters.objects.get(name='Tintin')
reporter.stories_filed += 1
reporter.save()
```

使用`connection.queries`查看到的`sql`指令为

```sql
UPDATE `polls_choice` 
	SET `question_id` = 1, `choice_text` = 'Not much', `votes` = 5 
	WHERE `polls_choice`.`id` = 1
```

替换为F()后

```python
from django.db.models import F

reporter = Reporters.objects.get(name='Tintin')
reporter.stories_filed = F('stories_filed') + 1
reporter.save()
```

使用`connection.queries`查看到的`sql`指令为

```sql
UPDATE `polls_choice` 
	SET `question_id` = 1, `choice_text` = 'Not much', `votes` = (`polls_choice`.`votes` + 1) 
	WHERE `polls_choice`.`id` = 1
```

