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



