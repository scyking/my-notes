# go 错误处理

标签（空格分隔）： go

---

[toc]

## go 标准包支持

### `error`是个`interface`

```go
type error interface {
    Error() string
}
```

### 创建`error`

```go
// example 1
func Sqrt(f float64) (float64, error) {
    if f < 0 {
        return 0, errors.New("math: square root of negative number")
    }
    // implementation
}

// example 2
if f < 0 {
    return 0, fmt.Errorf("math: square root of negative number %g", f)
}
```

### 自定义`error`

```go
// errorString is a trivial implementation of error.
type errorString struct {
    s string
}

func (e *errorString) Error() string {
    return e.s
}
```

## 如何处理错误

### 问题

- 函数该如何返回错误，是用值，还是用特殊的错误类型
- 如何检查被调用函数返回的错误，是判断错误值，还是用类型断言
- 程序中每层代码在碰到错误的时候，是每层都处理，还是只用在最上层处理，如何做到优雅
- 日志中的异常信息不够完整、缺少`stack strace`，不方便定位错误原因

### 错误处理策略

1. 返回和检查错误值
    > 通过特定值表示成功和不同的错误，上层代码检查错误的值，来判断被调用`func`的执行状态。
    > 这种策略是最不灵活的错误处理策略，上层代码需要判断返回错误值是否等于特定值。如果想修改返回的错误值，则会破坏上层调用代码的逻辑。

    ```go
    buf := make([]byte, 100)
    n, err := r.Read(buf)   // 如果修改 r.Read，在读到文件结尾时，返回另外一个 error，比如 io.END，而不是 io.EOF，则所有调用 r.Read 的代码都必须修改
    buf = buf[:n]
    if err == io.EOF {
        log.Fatal("read failed:", err)
    }
    ```

    > 永远不要通过判断Error方法返回的字符串是否包含特定字符串，来决定错误处理的方式。
    > 返回特定错误值的方式，增加了公共库和调用代码的耦合性。让模块之间产生了依赖。

1. 自定义错误类型
    > 通过自定义的错误类型来表示特定的错误，上层代码通过类型断言判断错误的类型

    ```go
    // 定义错误类型
    type MyError struct {
        Msg string
        File string
        Line int
    }
    
    func (e *MyError) Error() string {
        return fmt.Sprintf("%s:%d: %s", e.File, e.Line, e.Msg)
    }
    
    // 被调用函数中
    func doSomething() {
        // do something
        return &MyError{"Something happened", "server.go", 42}
    }
    
    // 调用代码中
    err := doSomething()
    if err, ok := err.(SomeType); ok {    // 使用 类型断言 获得错误详细信息
        //...
    }
    ```

    > 相比于“返回和检查错误值”，很大一个优点在于可以将 **底层错误** 包起来一起返回给上层，这样可以提供更多的上下文信息。
    > 然而，这种方式依然会增加模块之间的依赖。

1. 隐藏内部细节的错误处理
    > 假设上层代码不知道被调用函数返回的错误任何细节，直接再向上返回错误。
    > 这种策略之所以叫“隐藏内部细节的错误处理”，是因为当上层代码碰到错误发生的时候，不知道错误的内部细节。
    > 作为上层代码，你需要知道的就是被调用函数是否正常工作。 如果你接受这个原则，将极大降低模块之间的耦合性。

    ```go
    import “github.com/quux/bar”
    
    func fn() error {
        x, err := bar.Foo()
        if err != nil {
            return err
        }
        // use x
    }
    ```

    > 上面的例子中，`Foo`这个方法不承诺返回的错误的具体内容。
    > 这样，`Foo`函数的开发者可以不断调整返回错误的内容来提供更多的错误信息，而不会破坏`Foo`提供的协议。这就是“隐藏内部细节”的内涵。
    > 缺点是获取到的错误信息不完整。

### 最合适的错误处理策略

1. 输出详细错误信息
    > 使用`github.com/pkg/errors`包。该包提供了如下API：

    ```go
    // Wrap annotates cause with a message.
    func Wrap(cause error, message string) error
    
    // Cause unwraps an annotated error.
    func Cause(err error) error
    ```

    > 示例：

    ```go
    func ReadFile(path string) ([]byte, error) {
     f, err := os.Open(path)
     if err != nil {
      return nil, errors.Wrap(err, "open failed")
     }
     defer f.Close()
     buf, err := ioutil.ReadAll(f)
     if err != nil {
      return nil, errors.Wrap(err, "read failed")
     }
     return buf, nil
    }
    ```

1. 为了行为断言错误，而非为了类型
    > 在有些场景下，仅仅知道是否出错是不够的。比如，和进程外其它服务通信，需要了解错误的属性，以决定是否需要重试操作。

    ```go
    type temporary interface {
        Temporary() bool    // IsTemporary returns true if err is temporary.
    }
    
    func IsTemporary(err error) bool {
        te, ok := err.(temporary)
        return ok && te.Temporary()
    }
    ```

1. 不要忽略错误，也不要重复处理错误
    > 遇到错误，而不去处理，导致信息缺失，会增加后期的运维成本。

    ```go
    func Write(w io.Writer, buf []byte) {
        w.Write(buf)    // Write(p []byte) (n int, err error)，Write方法的定义见 https://golang.org/pkg/io/#Writer
    }
    ```

    > 重复处理，添加了不必要的处理逻辑，导致信息冗余，也会增加后期的运维成本。

    ```go
    func Write(w io.Writer, buf []byte) error {
        _, err := w.Write(buf)
        if err != nil {
            log.Println("unable to write:", err)    // 第1次错误处理
    
            return err
        }
        return nil
    }
    
    func main() {
        // create writer and read data into buf
    
        err := Write(w, buf)
        if err != nil {
            log.Println("Write error:", err)        // 第2次错误处理
            os.Exit(1)
        }
    
        os.Exit(0)
    }
    ```
