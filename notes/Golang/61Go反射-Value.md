title: 61.Go反射-Value
date: 2020-05-21
tags: Go
categories: Go语言
layout: post

------

摘要：本节介绍`Go`语言内置的反射机制中一个重要的数据类型`reflect.Value`

<!-- more -->

`Value`类型，不仅可以容纳任意类型的值，而且其成员包含`rtype`属性（即`Type`）

`Value`类型提供了很多的方法，但只有少数方法能对任意值都安全调用

## 任意值

可以对任意值都安全调用的方法

### Type()

可以使用`Value`的`Type()`函数，获得其对应的`Type`。**任意值**

```go
t := v.Type()           // a reflect.Type
fmt.Println(t.String()) // "int"
```

### Interface()

可以使用`Value`的`Interface()`函数，获得其接口值。**任意值**

```go
v := reflect.ValueOf(3) // a reflect.Value
x := v.Interface()      // an interface{}
i := x.(int)            // an int
fmt.Printf("%d\n", i)   // "3"
```

### Kind()

可以使用`Value`的`Kind()`函数，获得有限的数据类型。**任意值**

```go
switch v.Kind()
```

支持的有限数据类型都有哪些？看看源码，都是底层类型

```go
type Kind uint
const (
	Invalid Kind = iota
	Bool
	Int
	Int8
	Int16
	Int32
	Int64
	Uint
	Uint8
	Uint16
	Uint32
	Uint64
	Uintptr
	Float32
	Float64
	Complex64
	Complex128
	Array
	Chan
	Func
	Interface
	Map
	Ptr
	Slice
	String
	Struct
	UnsafePointer
)
```

## 部分类型

### Index()

只支持`Slice`、数组或字符串类型的值调用，如果对其它类型调用则会导致`panic`异常

传入索引值，获得对应的元素值，返回的值也是一个`Value`类型

###NumField() & Field() 

只支持`struct`类型的值调用

`NumField()`：返回结构体`struct`中的成员数量

`Field(i)`：传入成员索引值，返回对应的成员，索引从`0`开始

### MapKeys() & MapIndex()

只支持`map`类型的值调用

`MapKeys()`：返回`Value`类型的`slice`，`slice`的每个元素对应`map`的一个`key`

`MapIndex(key)`：返回`map`中`key`对应的`value`

### Elem()

只支持`pointer`和`interface`类型的值调用

`Elem()`：返回指针指向的变量，`Value`类型

如果指针是`nil`，操作也是安全的，这时指针是`Invalid`类型。可以用`IsNil`方法来显式地测试一个空指针

## Value vs interface{}

`Value`和`interface{}`都能装载任意的值，可以作为参数和变量使用。但两者还是有着本质的不同：

- `interface{}`：空接口隐藏了值的内部细节，只有知道具体类型时才能使用类型断言访问其属性和方法
- `Value`：自身包含类型信息，提供了很多的方法检查其内容

```go
type Value struct {
	// typ holds the type of the value represented by a Value.
	typ *rtype

	// Pointer-valued data or, if flagIndir is set, pointer to data.
	// Valid when either flagIndir is set or typ.pointers() is true.
	ptr unsafe.Pointer

	// flag holds metadata about the value.
	// The lowest bits are flag bits:
	//	- flagStickyRO: obtained via unexported not embedded field, so read-only
	//	- flagEmbedRO: obtained via unexported embedded field, so read-only
	//	- flagIndir: val holds a pointer to the data
	//	- flagAddr: v.CanAddr is true (implies flagIndir)
	//	- flagMethod: v is a method value.
	// The next five bits give the Kind of the value.
	// This repeats typ.Kind() except for method values.
	// The remaining 23+ bits give a method number for method values.
	// If flag.kind() != Func, code can assume that flagMethod is unset.
	// If ifaceIndir(typ), code can assume that flagIndir is set.
	flag

	// A method value represents a curried method invocation
	// like r.Read for some receiver r. The typ+val+flag bits describe
	// the receiver r, but the flag's Kind bits say Func (methods are
	// functions), and the top bits of the flag give the method number
	// in r's type's method table.
}
```

## 修改值

通过反射机制可以修改变量的值，没错**可以修改变量的值**

要修改变量的值，需要两步操作：获得存储变量的内存地址，修改内存地址中的值

### 取地址

不是所有的`Value`都是**可取地址**的，看下面的例子

```go
x := 2                   // value   type    variable?
a := reflect.ValueOf(2)  // 2       int     no
b := reflect.ValueOf(x)  // 2       int     no
c := reflect.ValueOf(&x) // &x      *int    no
d := c.Elem()            // 2       int     yes (x)
```

如何正确取`Value`的地址呢？

```go
x := 2
a := reflect.ValueOf(&x).Elem()
```

每当我们通过指针间接地获取的`reflect.Value`都是可取地址的

怎么知道一个`Value`是否可以取地址呢？使用`CanAddr()`方法

```go
fmt.Println(a.CanAddr()) // "false"
fmt.Println(b.CanAddr()) // "false"
fmt.Println(c.CanAddr()) // "false"
fmt.Println(d.CanAddr()) // "true"
```

### 设置新值

先介绍一种比较麻烦的用法：

- 调用`Addr()`方法获得`Value`
- 调用`Interface()`方法获得其`interface{}`接口值
- 对于已知类型，使用类型断言机制转换成具体类型
- 变量赋值

```go
x := 2
d := reflect.ValueOf(&x).Elem()   // d refers to the variable x
px := d.Addr().Interface().(*int) // px := &x
*px = 3                           // x = 3
fmt.Println(x)                    // "3"
```

常用的做法是，对于可取地址的`Value`值，调用`Set*`函数，直接设置新值即可

```go
d.Set(reflect.ValueOf(4))
fmt.Println(x) // "4"
```

`Set`函数的参数是`Value`类型，当不确定`d`的具体类型时，可以用该方法

```go
d := reflect.ValueOf(&x).Elem()
d.SetInt(3)
fmt.Println(x) // "3"
```

还有很多`Set*`方法，用于设置具体类型的值。如：`SetInt`、`SetUint`、`SetString`和`SetFloat`等

在设置新值时一定要注意，变量的类型与值类型必须匹配

- 对于`interface{}`接口类型的变量，只能使用`Set`方法，而不能用对应具体类型的`Set*`方法
- 无论用哪种`Set`方法，变量类型和值类型必须匹配

```go
d.Set(reflect.ValueOf(int64(5))) // panic: int64 is not assignable to int

var y interface{}
ry := reflect.ValueOf(&y).Elem()
ry.SetInt(2)                     // panic: SetInt called on interface Value
```

是不是所有的可取地址的`Value`都可以设置值呢？答案是否定的

可以调用`CanSet()`方法，返回当前变量是否可以设置值

```go
stdout := reflect.ValueOf(os.Stdout).Elem() // *os.Stdout, an os.File var
fmt.Println(fd.CanAddr(), fd.CanSet()) // "true false"
```

我们可以从下例中看到，使用反射机制可以打破导出规则的限制读取结构体中未导出的成员，但不能修改这些成员

```go
fd := stdout.FieldByName("fd")
fmt.Println(fd.Int()) // "1"
fd.SetInt(2)          // panic: unexported field
```



