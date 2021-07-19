# go 使用 fasthttp

---

[toc]

> 详见 [fasthttp](https://github.com/valyala/fasthttp)
> [fasthttp 参考手册](https://github.com/DavidCai1993/my-blog/issues/35)

## 安装

```bash
go get -u github.com/valyala/fasthttp
```

## 示例

### GET 请求

```go
func main() {
    url := `http://httpbin.org/get`

    status, resp, err := fasthttp.Get(nil, url)
    if err != nil {
        fmt.Println("请求失败:", err.Error())
        return
    }

    if status != fasthttp.StatusOK {
        fmt.Println("请求没有成功:", status)
        return
    }

    fmt.Println(string(resp))
}
```

### POST 请求

```go
func main() {
    url := `http://httpbin.org/post?key=123`
    
    // 填充表单，类似于 net/url
    args := &fasthttp.Args{}
    args.Add("name", "test")
    args.Add("age", "18")

    status, resp, err := fasthttp.Post(nil, url, args)
    if err != nil {
        fmt.Println("请求失败:", err.Error())
        return
    }

    if status != fasthttp.StatusOK {
        fmt.Println("请求没有成功:", status)
        return
    }

    fmt.Println(string(resp))
}
```
