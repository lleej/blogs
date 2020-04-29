title: 22.Go函数之Deferred函数
date: 2020-04-27
tags: Go
categories: Go语言
layout: post

------

摘要：本节介绍`Go`语言的函数的Deferred函数，即延迟函数

<!-- more -->

## 延迟函数

在代码中，有些时候为了保证创建的`资源`被有效的释放，需要在多个分支中添加相同的释放代码。一方面，可读性和可维护性差；另一方面，如果不小心少些了一个，会导致非预期的错误，而且很难发现

在`Go`语言中引入了延迟函数(`deferred`)的概念，使用关键字`defer`的函数调用，将会延迟执行，在包含该`defer`语句的函数执行完毕后（正常`return`或异常`panic`），才会执行。

如果一个函数中存在多个`defer`语句，则执行顺序与声明顺序相反

```go
package ioutil
func ReadFile(filename string) ([]byte, error) {
    f, err := os.Open(filename)
    if err != nil {
        return nil, err
    }
    defer f.Close()
    return ReadAll(f)
}
```

可以利用延迟函数，实现在调试时，记录函数进入和退出的时间

```go
func bigSlowOperation() {
    defer trace("bigSlowOperation")() // don't forget the
    extra parentheses
    // ...lots of work…
    time.Sleep(10 * time.Second) // simulate slow
    operation by sleeping
}
func trace(msg string) func() {
    start := time.Now()
    log.Printf("enter %s", msg)
    return func() { 
        log.Printf("exit %s (%s)", msg,time.Since(start)) // 闭包
    }
}
```

- `defer`语句中不要忘了写最后的`()`，这是一个返回函数值的函数。`trace`函数会立即执行，而返回函数则延迟执行
- `trace`函数中使用了闭包，返回的内部匿名函数访问其`start`变量

再看一个例子，对于理解函数返回的执行顺序有帮助，前面提到过

```go
func double(x int) (result int) {
    defer func() { fmt.Printf("double(%d) = %d\n", x,result) }()
    return x + x
}
_ = double(4)
// Output:
// "double(4) = 8"
```

对于有名返回值`(result int)`来说，`return x + x`分两步执行

- 第一步：`result = x + x`，将计算结果赋值给返回值变量
- 第二步：`return`返回（返回值是保存在堆中的）

而`defer`语句是在`return`前调用的，这时`result = x + x`已经执行完毕了，因此可以使用该变量获得执行结果

利用这些特性，还可以在`defer`语句中修改返回值

```go
func triple(x int) (result int) {
    defer func() { result += x }()
    return double(x)
}
fmt.Println(triple(4)) // "12"
```

### 注意事项

不要在循环体中直接使用`defer`，有可能导致资源被耗尽

```go
for _, filename := range filenames {
    f, err := os.Open(filename)
    if err != nil {
        return err
    }
    defer f.Close() // NOTE: risky; could run out of file
    descriptors
    // ...process f…
}
```

对于需要循环处理的`资源`，可以将其代码封装到一个独立的函数中

```go
for _, filename := range filenames {
    if err := doFile(filename); err != nil {
        return err
    }
}
func doFile(filename string) error {
    f, err := os.Open(filename)
    if err != nil {
        return err
    }
    defer f.Close()
    // ...process f…
}
```

