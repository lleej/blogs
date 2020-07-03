title: 63.Go反射-Type类型
date: 2020-05-24
tags: Go
categories: Go语言
layout: post

------

摘要：本节介绍`Go`语言内置的反射机制中一个重要的数据类型`reflect.Type`

<!-- more -->

## 是什么

`reflect.Type`是一个接口类型，其类型声明如下：

```go
type Type interface {
	// 支持所有类型
	Align() int
	FieldAlign() int
	Method(int) Method
	MethodByName(string) (Method, bool)
	NumMethod() int
	Name() string
	PkgPath() string
	Size() uintptr
	String() string
	Kind() Kind
	Implements(u Type) bool
	AssignableTo(u Type) bool
	ConvertibleTo(u Type) bool
	Comparable() bool

	// 部分类型安全
	//	Int*, Uint*, Float*, Complex*: Bits
	//	Array: Elem, Len
	//	Chan: ChanDir, Elem
	//	Func: In, NumIn, Out, NumOut, IsVariadic.
	//	Map: Key, Elem
	//	Ptr: Elem
	//	Slice: Elem
	//	Struct: Field, FieldByIndex, FieldByName, FieldByNameFunc, NumField
	Bits() int
	ChanDir() ChanDir
	IsVariadic() bool
	Elem() Type
	Field(i int) StructField
	FieldByIndex(index []int) StructField
	FieldByName(name string) (StructField, bool)
	FieldByNameFunc(match func(string) bool) (StructField, bool)
	In(i int) Type
	Key() Type
	Len() int
	NumField() int
	NumIn() int
	NumOut() int
	Out(i int) Type
	common() *rtype
	uncommon() *uncommonType
}

```

同时`reflect`包中声明了`rtype`类型，是`Type`接口类型的具体实现类型

```go
type rtype struct {
	size       uintptr
	ptrdata    uintptr // number of bytes in the type that can contain pointers
	hash       uint32  // hash of type; avoids computation in hash tables
	tflag      tflag   // extra type information flags
	align      uint8   // alignment of variable with this type
	fieldAlign uint8   // alignment of struct field with this type
	kind       uint8   // enumeration for C
	// function for comparing objects of this type
	// (ptr to object A, ptr to object B) -> ==?
	equal     func(unsafe.Pointer, unsafe.Pointer) bool
	gcdata    *byte   // garbage collection data
	str       nameOff // string form
	ptrToThis typeOff // type for pointer to this type, may be zero
}
```

## 结构体

在`Value`类型一节，我们知道可以通过`Field(i)`方法，获得结构体的成员信息，是`structField`类型的变量

```go
type StructField struct {
	Name string
	PkgPath string

	Type      Type      // field type
	Tag       StructTag // field tag string
	Offset    uintptr   // offset within struct, in bytes
	Index     []int     // index sequence for Type.FieldByIndex
	Anonymous bool      // is an embedded field
}

// A StructTag is the tag string in a struct field.
type StructTag string
func (tag StructTag) Get(key string) string {}
func (tag StructTag) Lookup(key string) (value string, ok bool) {}
```

其中，`Tag`是结构体成员的字段标识（我们在结构体一节中介绍过）。`Tag`标签可以在序列化`JSON`串时，通过设置`JSON`对应的名字。其中`json`成员标签让我们可以选择成员的名字和抑制零值成员的输出

我们可以通过反射机制获取标识信息，实现自动化框架中`http`请求查询参数自动匹配的功能

### 实现思路

要实现查询参数的自动解析，就要实现查询条件中参数名称与接收参数的结构体间的映射，并将参数值写入结构体中

#### 定义匿名结构体

用于存放请求中传递的参数，并使用成员标识将成员注解为参数名

```go
var data struct {
        Labels     []string `http:"l"`
        MaxResults int      `http:"max"`
        Exact      bool     `http:"x"`
    }
```

### 将结构体成员分解到Map中

- 遍历匿名声明的结构体变量，将成员放到`Map`中
- 如果成员有`http`的标识，用该标识中的名字作为`key`
- 如果成员没有`http`的标识，用该成员名作为`key`

```Go
func Unpack(req *http.Request, ptr interface{}) error {
    if err := req.ParseForm(); err != nil {
        return err
    }

    // Build map of fields keyed by effective name.
    fields := make(map[string]reflect.Value)
    v := reflect.ValueOf(ptr).Elem() // the struct variable
    for i := 0; i < v.NumField(); i++ {
        fieldInfo := v.Type().Field(i) // a reflect.StructField
        tag := fieldInfo.Tag           // a reflect.StructTag
        name := tag.Get("http")
        if name == "" {
            name = strings.ToLower(fieldInfo.Name)
        }
        fields[name] = v.Field(i)
    }
```

### 将参数匹配到Map中

- 遍历`http`请求中的所有参数
- 根据参数名定位`Map`中的成员`Value`，取出`Value`变量
- 如果`Value`变量是切片类型，则将参数值添加到变量中
- 如果`Value`变量是其他类型，则将参数值替换变量中

```go
// Update struct field for each parameter in the request.
    for name, values := range req.Form {
        f := fields[name]
        if !f.IsValid() {
            continue // ignore unrecognized HTTP parameters
        }
        for _, value := range values {
            if f.Kind() == reflect.Slice {
                elem := reflect.New(f.Type().Elem()).Elem()
                if err := populate(elem, value); err != nil {
                    return fmt.Errorf("%s: %v", name, err)
                }
                f.Set(reflect.Append(f, elem))
            } else {
                if err := populate(f, value); err != nil {
                    return fmt.Errorf("%s: %v", name, err)
                }
            }
        }
```

这样就实现了自动解析并匹配`http`请求中参数的作用

```bash
$ ./fetch 'http://localhost:12345/search?x=true&l=golang&l=programming&max=100'
Search: {Labels:[golang programming] MaxResults:100 Exact:true}
```

### 分析

- 结构体中成员数量要与可选参数数量一致
- 结构体中成员的名称要能够表述出参数的含义，这样便于后期处理
- 使用`http`标识，用于将成员名称与参数名称进行自动匹配
- 分解结构体中的成员，从结构体转换为`map[string]reflect.Value`类型
- 虽然能够自动进行名称的匹配，但还是依赖于静态的参数信息，不能动态调整参数数量