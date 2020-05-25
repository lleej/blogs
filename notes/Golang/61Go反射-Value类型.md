title: 61.Go反射-Value类型
date: 2020-05-21
tags: Go
categories: Go语言
layout: post

------

摘要：本节介绍`Go`语言内置的反射机制中一个重要的数据类型`reflect.Value`

<!-- more -->

`Value`类型，不仅可以容纳任意类型的值，而且其成员包含`rtype`属性（即`Type`）

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

`Value`类型提供了很多的方法，但只有少数方法能对任意值都安全调用

## 任意值安全

可以对任意值都安全调用的方法。**不会导致`panic`异常**

### Type()

使用`Value`的`Type()`函数，获得其对应的`Type`

```go
t := v.Type()           // a reflect.Type
fmt.Println(t.String()) // "int"
```

### Interface()

使用`Value`的`Interface()`函数，获得其接口值

```go
v := reflect.ValueOf(3) // a reflect.Value
x := v.Interface()      // an interface{}
i := x.(int)            // an int
fmt.Printf("%d\n", i)   // "3"
```

### Kind()

使用`Value`的`Kind()`函数，获得**有限的**数据类型

返回的数据类型，可以用于类型判断，根据不同的具体类型进行相应的处理

```go
switch v.Kind()
```

有限的数据类型都是底层类型，包括：

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

## 部分类型安全

### 整数

包括`int`、`int8`、`int16`、`int32`、`int64`等有符号整数类型

#### Int()

返回`Value`包含的有符号整数值

包括：`uint`、`uint8`、`uint16`、`uint32`、`uint64`、`uintptr`等无符号整数类型

#### Uint()

返回`Value`包含的无符号整数值

### 字符串

#### String()

返回`Value`包含的字符串值

### 数组、切片、字符串

这些类型都有长度`len`和下标`index`属性，相应的`Value`类型也具有响应的方法

#### Index(i)

根据索引值`i`，返回对应的元素值，返回值是`Value`类型

```go
v.Index(i)
```

#### Len()

返回`Value`中元素的个数，等同于`Go`语言内置的`Len()`函数功能

```go
for i := 0; i < v.Len(); i++ {
	...
```

### 结构`struct`

可以获得结构体的成员数以及每个成员的信息

#### NumField()

返回结构体`struct`中的成员数量

#### Field(i) 

根据索引值`i`，返回对应的成员。索引从`0`开始

`Type`和`Value`类型均提供了该方法，但两者有着本质的区别：

- `Type`类型的方法，返回的是成员属性的结构体（成员名称、成员类型等）
- `Value`类型的方法，返回的是成员的值

```go
 for i := 0; i < v.NumField(); i++ {
 	fieldPath := fmt.Sprintf("%s.%s", path, v.Type().Field(i).Name)
 		display(fieldPath, v.Field(i))
```

### Map

#### MapKeys()

返回`Map`中所有`Key`值的切片`slice`，`slice`中每个元素对应`map`的一个`key`

#### MapIndex(key)

根据`key`值，返回`map`中对应的`value`

### 接口和指针

根据接口和指针，获得对应变量的值

#### Elem()

返回指针指向的变量，返回值为`Value`类型

#### IsNil()

显式地测试`Value`是否是空指针。如果是空指针，操作也是安全的，其`Kind()`返回的类型是`Invalid`

```go
if v.IsNil() {
	...
}
```



## 类型方法集

`reflect`提供了获取任意类型方法集的用法，并可以在枚举后，直接调用这些方法

### NumMethod()

`Value`类型的方法，返回`Value`包含的具体类型的方法数量

### Method(i)

`Type`和`Value`两种类型，均提供了`Method(i)`方法

- `Type`类型中的方法，返回的是方法属性的结构体（方法名称、方法类型）
- `Value`类型中的方法，返回的是方法的值（我们知道方法也是值）

### Call()

`Value`类型中的方法，调用一个`Func`类型的`Value`

```go
// Print prints the method set of the value x
func Print(x interface{}) {
    v := reflect.ValueOf(x)
    t := v.Type()
    fmt.Printf("type %s\n", t)

    for i := 0; i < v.NumMethod(); i++ {
        methType := v.Method(i).Type()
        fmt.Printf("func (%s) %s%s\n", t, t.Method(i).Name,
            strings.TrimPrefix(methType.String(), "func"))
    }
}
```



## Value和Interface{}

### 相同点

- 均能装载任意类型的值
- 常用于参数和变量使用

### 区别

- **空接口**隐藏了值的内部细节，只有知道具体类型时，才能使用类型断言访问其属性和方法

- `Value`自身就包含了其类型的信息，并提供了很多方法来检查其内容

