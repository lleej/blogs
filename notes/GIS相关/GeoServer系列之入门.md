title: GeoServer系列之入门
date: 2020-06-02
tags: GIS
categories: GIS
layout: traft

------

摘要：分多个章节介绍`GIS`的一些基本概念、术语，开源`GIS`平台的安装、部署，如何使用`Google`/`Baidu`的地图瓦片，地图常用功能的开发说明以及`Openlayer`的使用...

<!-- more -->

## 术语

### OGC

开放地理信息联盟（`Open Geospatial Consortium`）

### GML

地理标记语言（Geography Markup Language），用于描述地理数据的`XML`

### 地图服务规范

#### WFS

`Web`要素服务（`Web Feature Service`）

定义了：对`GIS`系统中的空间数据，采用`GML`编码的方式进行`CRUD`等事务操作的**规范**，定义了五种操作：

- `GetCapabilites`：返回`Web`要素服务性能描述文档（用`XML`描述）
- `DescribeFeatureType`：返回描述可以提供服务的任何要素结构的`XML`文档
- `GetFeature`：一个获取要素实例的请求提供服务
- `Transaction`：为事务请求提供服务
- `LockFeature`：处理在一个事务期间对一个或多个要素类型实例上锁的请求

通俗的讲，我们通过客户端（如：浏览器）选中某个元素（如：超市）时获取到的信息（如：名称、营业时间等）

#### WMS

`Web`地图服务（`Web Map Service`）

定义了：使用具有地理空间位置信息的数据制作地图，将地图定义为地理数据可视的表现的规范。定义了三种操作：

- `GetCapabitities`：返回服务级元数据，它是对服务信息内容和要求参数的一种描述
- `GetMap`：返回一个地图影像，其地理空间参考和大小参数是明确定义了的
- `GetFeatureInfo`：返回显示在地图上的某些特殊要素的信息

通俗的讲，我们通过客户端（如：浏览器）看到的地图，就是使用`WMS`服务获得的

#### WCS

`Web`覆盖服务（`Web Coverage Service`）

定义了：面向空间影像数据，将包含地理位置值的地理空间数据作为**覆盖**（`Coverage`）在网上相互交换的规范。定义了三种操作：

组成：GetCapabilities，GetCoverage和DescribeCoverageType：

- `GetCapabilities`：操作返回描述服务和数据集的`XML`文档
- `GetCoverage`：操作是在`GetCapabilities`确定什么样的查询可以执行、什么样的数据能够获取之后执行的，它使用通用的覆盖格式返回地理位置的值或属性
- `DescribeCoverageType`：操作允许客户端请求由具体的`WCS`服务器提供的任一覆盖层的完全描述



## PostGis

### 介绍

`Postgis`是`Postgres`数据库的空间数据库引擎，扩展了`Postgres`数据库存储空间数据类型数据的功能

使用空间数据前，用户创建的数据库需要执行扩展命令，安装`Postgis`扩展

```sql
CREATE EXTENSION postgis;
```

判断一个用户创建的数据库是否支持`Postgis`，可以使用如下指令

```go
SELECT postgis_full_version();
```



## Docker

需要下载`Postgis`和`geoserver`的镜像

- `Postgis`：`docker pull kartoza/postgis`
- `geoserver`：```docker pull kartoza/geoserver```

配置`docker-compose.yml`，启动文件

```yaml
version: '3.1'
services:
  geoserver:
    restart: always
    image: kartoza/geoserver # 镜像的名称
    container_name: geoserver # 容器的名称
    ports:
      - 8080:8080 # 默认使用8080端口，格式：本机端口：容器端口
    volumes:
      - ./geoserver-data:/opt/geoserver/data_dir # 目录映射，格式：本地目录：容器Volumn
    links:
      - postgis:postgis # 链接Postgis数据库，没有配置则不能连接到数据库，注意：数据库连接时的主机名为:postgis
  postgis:
    restart: always
    image: kartoza/postgis
    container_name: postgis
    ports:
      - 5432:5432 # postgis 默认使用5432端口
    environment:
      POSTGRES_USER: postgres # 设置数据库访问的用户名
      POSTGRES_PASS: postgres # 设置数据库访问的密码
      ALLOW_IP_RANGE: 0.0.0.0/0 # 取消客户端访问限制
    volumes:
      - ./postgis-data:/var/lib/postgresql # 目录映射，格式：本地目录：容器Volumn
```

**注意：看`docker`镜像的文档，各种`Volumn`和`environment`的名称是不一样的**

## 工作区

工作区：组织图层的容器，将相似的图层组织在一起

图层使用**工作区名:图层名**的形式进行访问，如：`sf:states`

### 基本信息

工作区名称：最多`10`个字符，不能含空格

命名空间`URI`：需要是唯一的标识符，不需要标识实际位置。通常使用**项目URL+工作区名**的格式，如：`http://www.openplans.org/topp`

**注意：隔离工作区的命名空间可以与正常工作区一样**

## 数据存储

存储栅格 / 矢量数据，数据源可以是文件/文件组、数据库、栅格文件或目录等

### 存储类型

尽管数据源有许多潜在的格式，但是只有四种存储：

| 类型                                          | 描述                          |
| :-------------------------------------------- | :---------------------------- |
| ![文件栅格](./assets/data_stores_type1.png)   | 文件中的栅格数据              |
| ![文件矢量](./assets/data_stores_type3.png)   | 文件中的矢量数据              |
| ![数据库矢量](./assets/data_stores_type2.png) | 数据库中的矢量数据            |
| ![服务器矢量](./assets/data_stores_type5.png) | 矢量服务器（`Web`功能服务器） |

### 数据源

添加存储，需要指定数据源，`GeoServer`默认包含多种数据源，每种数据源需要配置不同的连接属性

但不管使用哪种数据源，都需要配置：所属工作区和数据源名称

## 图层

定义：表示地理要素集合的栅格或矢量数据集。向量层类似于`featureTypes`，栅格层类似于`coverage`

所有层都对应一个数据源（数据存储），层与定义存储的工作区相关联

**一个数据源中可以包含多个图层数据**

### 图层类型

图层可分为两种类型的数据：栅格和矢量。这两种格式在存储空间信息的方式上有所不同。

- 向量类型。将有关要素类型的信息存储为数学路径
  - 点：单个`x,y`坐标
  - 线：系列`x,y`坐标
  - 多边形：系列`x,y`坐标在同一位置开始和结束
- 栅格类型。是地球表面要素的基于单元的表示形式
  - 每个单元格都有一个不同的值
  - 所有具有相同值的单元格都代表一个特定的功能

| 类型                                 | 描述         |
| :----------------------------------- | :----------- |
| ![栅格](./assets/raster_icon.png)    | 栅格（网格） |
| ![多边形](./assets/polygon_icon.png) | 多边形       |
| ![线](./assets/line_string_icon.png) | 线           |
| ![点](./assets/point_icon.png)       | 点           |

### 基本信息

- 图层名称：用于在`WMS`请求中引用图层的标识符
- 标题：用于向客户端简要标识该层的可读描述
- 关键字：与图层关联的短词汇列表，以帮助目录搜索
- 元数据链接：允许链接到描述数据层的外部文档

### 坐标系统

> `SRC`：空间参考系统

- 本地`SRS`：指定存储图层的坐标系
- 声明`SRS`：`GeoServer`发布给客户端的坐标系
- `SRS`处理：确定当两个`SRS`不同时`GeoServer`处理投影的方式
  - 强制使用声明（默认）：将声明`SRS`作用到数据上并覆盖数据
  - 重新投影到声明：当本地数据集的`CRS`与任何官方`EPSG`不匹配时，使用此设置
  - 保持本地：这是在极少数情况下应使用的设置

### 边界框

边界框：确定图层中数据的范围

- 本地边界框。本地`SRS`中指定的数据边界
  - 单击**从数据计算**按钮或**从SRS边界计算**按钮生成
  - 所使用的`SRS`取决于所选的**SRS处理**：选择*强制声明*或*重新投影为声明*时选择声明的`SRS`，否则使用本地`SRS`
  - 如果`SRS`没有定义边界，则不会生成任何边界
- 纬度/经度边界框。以地理坐标指定的边界
  - 单击**从本机边界计算**按钮来计算这些边界

## 图层组

一组图层的容器，可以作为一个图层被客户端访问

图层组中的图层，**只能属于一个工作区**



## 数据源

### shapefile

`shapefile`是一种常用的地理空间**矢量**数据格式，由`4`个文件组成：`.shp`、`.shx`、`.dbf`、`.prj`，这些文件必须在一个目录中

### 栅格数据

栅格数据`raster`

- GIFF

  `georeferenced TIFF（Tagged Image File Format）`

- GTOPO30

  数字高程模型（Digital Elevation Model）数据集，水平网格间隔为`30`弧秒（`arc seconds`）

- WorldImage

  是一个纯文本文件（`jgw`、`tfw`），对栅格图像文件（`jpg`、`tif`）进行地理信息配置，一般与图像文件一同使用



