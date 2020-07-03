title: 80.Go常用包-gorm
date: 2020-05-28
tags: Go包
categories: Go语言
layout: post

------

摘要：本节介绍`Go ORM`最热门的第三方包`gorm`的用法...

<!-- more -->

> 地址：http://github.com/jinzhu/gorm
>
> 文档：https://gorm.io/zh_CN/

## 安装

```go
go get -u github.com/jinzhu/gorm
```

## 模型

`ORM`是对象关系模型的缩写，通俗的讲就是将数据库的操作，转换为结构体`struct`的操作。结构体`struct`映射为数据表的实体

### Model

`gorm`包中定义了基本的`Model`结构体类型，用于嵌入用户自定义的`Model`中

类型包含四个成员：

```go
type Model struct {
	ID        uint `gorm:"primary_key"` // ID 在orm中命名为ID的字段默认就是 primary_key
	CreatedAt time.Time // 创建时间
	UpdatedAt time.Time // 更新时间
	DeletedAt *time.Time `sql:"index"` // 删除时间
}
```

### 自定义模型

嵌入还是不嵌入`gorm.Model`都是允许的，完全可以根据团队的风格定义表结构

```go
type User struct {
  gorm.Model	// 嵌入了Model类型的四个成员
  Name         		string
  Age          		sql.NullInt64
  Birthday     		*time.Time
  Email        		string  `gorm:"type:varchar(100);unique_index"`
  Role         		string  `gorm:"size:255"` // 设置字段大小为255
  MemberNumber 		*string `gorm:"unique;not null"` // 设置会员号（member number）唯一并且不为空
  Num          		int     `gorm:"AUTO_INCREMENT"` // 设置 num 为自增类型
  Address      		string  `gorm:"index:addr"` // 给address字段创建名为addr的索引
	CreditCard   		CreditCard  // One-To-One relationship (has one - use CreditCard's UserID as foreign key)
  Emails       		[]Email // One-To-Many relationship (has many - use Email's UserID as foreign key)
  BillingAddress 	Address // One-To-One relationship (belongs to - use BillingAddressID as foreign key)
  ShippingAddress Address // One-To-One relationship (belongs to - use ShippingAddressID as foreign key)
  IgnoreMe        int `gorm:"-"`   // Ignore this field
  Languages       []Language `gorm:"many2many:user_languages;"` // Many-To-Many relationship, 'user_languages' is join table
}
```

### 结构体标记

我们知道，结构体标记可以在序列化和反序列化时，起到重要的作用。当然也可以利用这个特性，用于定义成员（即表中的字段）的属性

结构体标记的`tag`为`gorm`，标记值要用`""`包含。可以有多个标记项，项间用`;`分隔

| 结构体标记（Tag） | 描述                                                         |
| :---------------- | :----------------------------------------------------------- |
| `Column`          | 指定列名，如：`column:myid`                                  |
| `Type`            | 指定列数据类型，如：`type:varchar(200)`                      |
| `Size`            | 指定列大小，默认值`255`，如：`size: 200`                     |
| `PRIMARY_KEY`     | 将列指定为主键，如：`primary_key`                            |
| `UNIQUE`          | 将列指定为唯一，如：`unique`                                 |
| `DEFAULT`         | 指定列默认值，如：`default:10`                               |
| `PRECISION`       | 指定列精度，如：`precision:2`                                |
| `NOT NULL`        | 将列指定为非 NULL，如：`not null`                            |
| `AUTO_INCREMENT`  | 指定列是否为自增类型，如：`auto_increment`                   |
| `INDEX`           | 创建具有或不带名称的索引，如果多个索引同名则创建复合索引，如：`index:addr` |
| `UNIQUE_INDEX`    | 和 `INDEX` 类似，只不过创建的是唯一索引，如：`unique_index`  |
| `EMBEDDED`        | 将结构设置为嵌入                                             |
| `EMBEDDED_PREFIX` | 设置嵌入结构的前缀                                           |
| `-`               | 忽略此字段                                                   |

### 使用约定

在`gorm`中一些使用约定

#### 表名

`gorm`将结构体`struct`类型名称的复数形式，默认作为数据表的名称使用

```go
type User struct { // 默认表名为 users
  ...
}
```

如果要将表名默认值修改为`user`，即去掉复数形式

```go
// 禁用默认表名的复数形式，如果置为 true，则 `User` 的默认表名是 `user`
db.SingularTable(true)
```

如果要将表名默认值修改为`jkpt_users`，即在复数形式前添加前缀

```go
db.SingularTable(false)
// 定义 DefaultTableNameHandler 来设置
gorm.DefaultTableNameHandler = func (db *gorm.DB, defaultTableName string) string  {
  return "jkpt_" + defaultTableName;
}
```

如果要修改某个表的名称，即对于个别表名进行修改（一劳永逸）

```go
// 给结构体添加方法：TableName
func (User) TableName() string {
  return "profiles"
}
// 还可以根据条件设置不同的值
func (u User) TableName() string {
  if u.Role == "admin" {
    return "admin_users"
  } else {
    return "users"
  }
}
```

或者，每次使用修改表名后的`Table`对象进行访问（更加灵活）

```go
// 使用User结构体创建名为`deleted_users`的表
db.Table("deleted_users").CreateTable(&User{})

var deleted_users []User
db.Table("deleted_users").Find(&deleted_users)
//// SELECT * FROM deleted_users;

db.Table("deleted_users").Where("name = ?", "jinzhu").Delete()
```

#### 字段名

结构体`struct`中成员的名称的小写形式，是默认的字段名称

```go
type User struct {
  ID        uint      // column name is `id`
  Name      string    // column name is `name`
}
```

如果字段名使用驼峰命名方式，如：`CreateAt`，则两个单词间默认使用`_`连接

```go
type User struct {
  CreatedAt time.Time // column name is `created_at`
}
```

可以使用`tag`设置字段名称

```go
type Animal struct {
    AnimalId    int64     `gorm:"column:beast_id"`         // set column name to `beast_id`
    Birthday    time.Time `gorm:"column:day_of_the_beast"` // set column name to `day_of_the_beast`
    Age         int64     `gorm:"column:age_of_the_beast"` // set column name to `age_of_the_beast`
}
```

使用`CreateAt`字段存储记录的创建时间，由`gorm`框架实现

```go
db.Create(&user) // will set `CreatedAt` to current time

// To change its value, you could use `Update`
db.Model(&user).Update("CreatedAt", time.Now())
```

使用`UpdateAt`字段存储记录的更新时间，由`gorm`框架实现

```go
// Whenever one or more `user` fields are edited using Save() or Update(), `UpdatedAt` will be set to current time
db.Save(&user) // will set `UpdatedAt` to current time
db.Model(&user).Update("name", "jinzhu") // will set `UpdatedAt` to current time
```

使用`DeleteAt`字段存储记录的删除时间（逻辑删除），由`gorm`框架实现

只要当前表中有名为`DeleteAt`的字段，执行删除操作时会设置该字段的值为当前时间，但不会物理删除该记录。而且，在执行查询操作时，这些记录不会被找到

#### 索引

如果一个字段名称为`ID`，则`gorm`默认将这个字段设置为`primary_key`，即主键

```go
type User struct {
  ID   string // 名为`ID`的字段会默认作为表的主键
  Name string
}
```

手动设置一个字段为`primary_key`

```go
type Animal struct {
  AnimalId int64 `gorm:"primary_key"` // set AnimalId to be primary key
  Name     string
  Age      int64
}
```

#### 字段类型

可以使用`Go`中的基础类型，`gorm`自动将这些基础类型转换为对应数据库的类型

```go
type User struct {
  ID        uint      // 类型 int
  Name      string    // 类型 varchar(255) 
}
```

数据类型的对应关系是：



还可以使用`sql.NullInt64`



类型可以是`*xxx`指针类型



## 连接数据库

`gorm`支持很多种数据库，前提是导入对应的驱动程序

连接数据库成功后，返回`gorn.DB`类型的变量，后续的数据库操作都依赖于它

### MySQL

常用的数据库驱动有以下两种，都采用匿名导入的方式

```go
import _ "github.com/go-sql-driver/mysql"
import _ "github.com/jinzhu/gorm/dialects/mysql"
```

在连接数据库时，必须在连接串中添加一些参数设置，使数据库操作行为正确

```go
import (
  "github.com/jinzhu/gorm"
  _ "github.com/jinzhu/gorm/dialects/mysql"
)

func main() {
  db, err := gorm.Open("mysql", "user:password@(127.0.0.1)/dbname?charset=utf8&parseTime=True&loc=Local")
  defer db.Close()
}
```

- 指定主机：`(localhost)`
- 指定字符集：`charset=utf8`
- 使用`time.Time`类型：`parseTime=True`

### PostgreSQL

```go
import (
  "github.com/jinzhu/gorm"
  _ "github.com/jinzhu/gorm/dialects/postgres"
)

func main() {
  db, err := gorm.Open("postgres", "host=myhost port=myport user=gorm dbname=gorm password=mypassword")
  defer db.Close()
}
```

### Sqlite3

```go
import (
  "github.com/jinzhu/gorm"
  _ "github.com/jinzhu/gorm/dialects/sqlite"
)

func main() {
  db, err := gorm.Open("sqlite3", "/tmp/gorm.db")
  defer db.Close()
}
```

使用 `:memory:` 替换文件路径，使用内存作为临时数据，性能非常好。**对`GORM`应用进行测试时使用**

### SQL Server

```go
import (
  "github.com/jinzhu/gorm"
  _ "github.com/jinzhu/gorm/dialects/mssql"
)

func main() {
  db, err := gorm.Open("mssql", "sqlserver://username:password@localhost:1433?database=dbname")
  defer db.Close()
}
```

## 操作

使用上节中连接数据库的代码，成功连接数据库后，就可以对数据库表进行操作，包括对表数据的`CRUD`操作以及表结构的`CRUD`操作

所有这些操作，都是由`gorm.DB`类型提供的方法

### 模型操作

#### 数据迁移

自动迁移你的模型，使之保持最新状态。迁移**只会**创建表、缺失的列、缺失的索引，**不会**更改现有列的类型或删除未使用的列，以此来保护您的数据。在生产环境中执行迁移操作是安全的，不会破坏生产数据

需要传入需要迁移的对象，书写格式如下：（能否提供类似于`django`中的迁移命令，如果有几十个表就很难维护了）

```go
db.AutoMigrate(&User{}, &Product{}, &Order{})
```

#### 创建表

```go
// 为模型 `User` 创建表
db.CreateTable(&User{})
```

#### 删除表

```go
// 删除模型 `User` 的表
db.DropTable(&User{})

// 删除表 `users`
db.DropTable("users")

// 删除模型 `User` 的表和表 `products`
db.DropTableIfExists(&User{}, "products")

```

#### 修改表

修改表结构，使用的是`Model()`方法，该方法同样返回一个`gorm.DB`类型

```go
func (s *DB) Model(value interface{}) *DB { ... }
```

修改表结构包括：列、索引和外键的操作

```go
// 修改模型 `User` 的 description 列的类型为 `text` 
db.Model(&User{}).ModifyColumn("description", "text")
// 删除模型 `User` 的 description 列
db.Model(&User{}).DropColumn("description")
// 为 `name` 列添加名为 `idx_user_name` 的普通索引
db.Model(&User{}).AddIndex("idx_user_name", "name")
// 为多列添加唯一索引
db.Model(&User{}).AddUniqueIndex("idx_user_name_age", "name", "age")
// 删除索引
db.Model(&User{}).RemoveIndex("idx_user_name")
// 添加外键
// 第一个参数： 外键字段
// 第二个参数：目标表名(字段)
// 第三个参数：删除时
// 第四个参数： 更新时
db.Model(&User{}).AddForeignKey("city_id", "cities(id)", "RESTRICT", "RESTRICT")
// 移除外键
db.Model(&User{}).RemoveForeignKey("city_id", "cities(id)")
```

#### 检查表

```go
// 检查模型 User 的表是否存在
db.HasTable(&User{})

// 检查表 users 是否存在
db.HasTable("users")
```

### 数据操作

在`gorm`中，对数据的`CRUD`操作，在后台都要经过一系列的处理（并不是`Insert/Update/Delete`的简单语句）



- `Create`

  源代码文件`callback_create.go`中，对`Create`操作的回调`Hook`进行了说明（默认是`9`个）

  注意：这些回调函数是按照顺序执行的，因此其注册的顺序很重要

  ```go
  // Define callbacks for creating
  func init() {
  	DefaultCallback.Create().Register("gorm:begin_transaction", beginTransactionCallback)
  	DefaultCallback.Create().Register("gorm:before_create", beforeCreateCallback)
  	DefaultCallback.Create().Register("gorm:save_before_associations", saveBeforeAssociationsCallback)
  	DefaultCallback.Create().Register("gorm:update_time_stamp", updateTimeStampForCreateCallback)
  	DefaultCallback.Create().Register("gorm:create", createCallback)
  	DefaultCallback.Create().Register("gorm:force_reload_after_create", forceReloadAfterCreateCallback)
  	DefaultCallback.Create().Register("gorm:save_after_associations", saveAfterAssociationsCallback)
  	DefaultCallback.Create().Register("gorm:after_create", afterCreateCallback)
  	DefaultCallback.Create().Register("gorm:commit_or_rollback_transaction", commitOrRollbackTransactionCallback)
  }
  ```

- `Delete`

  源代码文件`callback_delete.go`中，对`Delete`操作的回调`Hook`进行了说明（默认是`5`个）

  ```go
  // Define callbacks for deleting
  func init() {
  	DefaultCallback.Delete().Register("gorm:begin_transaction", beginTransactionCallback)
  	DefaultCallback.Delete().Register("gorm:before_delete", beforeDeleteCallback)
  	DefaultCallback.Delete().Register("gorm:delete", deleteCallback)
  	DefaultCallback.Delete().Register("gorm:after_delete", afterDeleteCallback)
  	DefaultCallback.Delete().Register("gorm:commit_or_rollback_transaction", commitOrRollbackTransactionCallback)
  }
  ```

- `Update`

  源代码文件`callback_update.go`中，对`Upate`操作的回调`Hook`进行了说明（默认是`9`个）

  ```go
  // Define callbacks for updating
  func init() {
  	DefaultCallback.Update().Register("gorm:assign_updating_attributes", assignUpdatingAttributesCallback)
  	DefaultCallback.Update().Register("gorm:begin_transaction", beginTransactionCallback)
  	DefaultCallback.Update().Register("gorm:before_update", beforeUpdateCallback)
  	DefaultCallback.Update().Register("gorm:save_before_associations", saveBeforeAssociationsCallback)
  	DefaultCallback.Update().Register("gorm:update_time_stamp", updateTimeStampForUpdateCallback)
  	DefaultCallback.Update().Register("gorm:update", updateCallback)
  	DefaultCallback.Update().Register("gorm:save_after_associations", saveAfterAssociationsCallback)
  	DefaultCallback.Update().Register("gorm:after_update", afterUpdateCallback)
  	DefaultCallback.Update().Register("gorm:commit_or_rollback_transaction", commitOrRollbackTransactionCallback)
  }
  ```

- `Query`

  源代码文件`callback_query.go`中，对`Query`操作的回调`Hook`进行了说明（默认是`3`个）

  ```go
  // Define callbacks for querying
  func init() {
  	DefaultCallback.Query().Register("gorm:query", queryCallback)
  	DefaultCallback.Query().Register("gorm:preload", preloadCallback)
  	DefaultCallback.Query().Register("gorm:after_query", afterQueryCallback)
  }
  
  ```

#### 插入数据

实例化一个数据对象，本例为：`User`类型的实例`user`，使用`Create()`方法将该实例添加到数据库相应的表中

```go
user := User{Name: "Jinzhu", Age: 18, Birthday: time.Now()}
db.Create(&user)
```

注意：插入操作会自动屏蔽零值的字段（`SQL`语句中不会包含该字段）。意味着：使用默认值初始化实例成员时，这些成员初始化为零值，在插入数据时不会出现在`SQL`语句中

#### 修改数据



#### 删除数据

#### 查询数据

#### Hook中操作

用户可以对数据表的`Hook`操作进行定义，在`Hook`中可以对数据进行验证或修改。如果返回`error`则对应的`CRUD`操作被终止

举例：想在`BeforeCreate` 时修改字段的值，使用`scope.SetColumn`方法

```go
func (user *User) BeforeCreate(scope *gorm.Scope) error {
  scope.SetColumn("ID", uuid.New())
  return nil
}
```

