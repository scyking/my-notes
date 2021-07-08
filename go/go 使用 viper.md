# go 使用 viper

标签（空格分隔）： go

---

[toc]

> 详见 [viper](https://github.com/spf13/viper)

## 概述

> `viper` 是一个配置解决方案库。

### 安装

```go
go get github.com/spf13/viper
```

### 支持功能

1. 可以设置默认值
1. 可以加载多种格式的配置文件，如JSON，TOML，YAML，HCL和Java属性配置文件
1. 应用程序运行过程中，保持监听和重新读取配置文件
1. 可以从环境变量读取配置
1. 可以从远程配置系统读取配置
1. 可以读取命令行标志作为配置
1. 可以从缓冲区中读取
1. 设置显式的值

### 配置方式优先级（由高到低）

1. 设置显示调用(`explicit call to Set`)
2. 命令行标志(`flag`)
3. 环境变量(`env`)
4. 配置文件(`config`)
5. 远程键/值存储(`key/value store`)
6. 默认值(`default`)

## 设置配置

> 将配置信息加载到`viper`中。

### 设置显示调用

#### 设置覆盖值

> 设置已有的变量，会对其进行覆盖。

```go
viper.Set("Verbose", true)
viper.Set("LogFile", LogFile)
```

#### 注册及使用别名

```go
viper.RegisterAlias("loud", "Verbose")

viper.Set("verbose", true)  // same result as next line
viper.Set("loud", true)     // same result as prior line

viper.GetBool("loud")       // true
viper.GetBool("verbose")    // true
```

### 命令行标志

```go
serverCmd.Flags().Int("port", 1138, "Port to run Application server on")
viper.BindPFlag("port", serverCmd.Flags().Lookup("port"))
```

### 环境变量

```go
SetEnvPrefix("spf")         // will be uppercased automatically
BindEnv("id")

os.Setenv("SPF_ID", "13")   // typically done outside of the app

id := Get("id")             // 13
```

### 配置文件

> 支持JSON，TOML，YAML，HCL和Java属性配置文件。

#### 读取配置文件

```go
// 配置文件名称
viper.SetConfigName("config")
// 配置文件后缀名（如配置文件名称无后缀，必须设置）
viper.SetConfigType("yaml") // REQUIRED if the config file does not have the extension in the name
// 配置文件所在路径，可设置多个
viper.AddConfigPath("/etc/appname/")   
viper.AddConfigPath("$HOME/.appname")
// "."表示工作目录
viper.AddConfigPath(".")

// 读取配置文件
err := viper.ReadInConfig()
if err != nil {
 panic(fmt.Errorf("Fatal error config file: %w \n", err))
}
```

#### 写入配置文件

```go
// 写入当前配置（会覆盖）
viper.WriteConfig()
// （不会覆盖）
viper.SafeWriteConfig()

// （会覆盖）
viper.WriteConfigAs("/path/to/my/.config")

// 如果已经被写入，会报错（不会覆盖）
viper.SafeWriteConfigAs("/path/to/my/.config")
viper.SafeWriteConfigAs("/path/to/my/.other_config")
```

#### 监听与重读配置文件

> 支持热加载配置，应用程序运行过程中，保持监听和重新读取配置文件。

```go
viper.WatchConfig()
viper.OnConfigChange(func(e fsnotify.Event) {
 fmt.Println("Config file changed:", e.Name)
})
```

#### 从`io.Reader`中读取配置

> `io.Reader`来源可以是文件、程序中生产的字符串以及网络连接中读取的字节流。

```go
viper.SetConfigType("yaml") 

var yamlExample = []byte(`
Hacker: true
name: steve
hobbies:
- skateboarding
- snowboarding
- go
clothing:
  jacket: leather
  trousers: denim
age: 35
eyes : brown
beard: true
`)

viper.ReadConfig(bytes.NewBuffer(yamlExample))

viper.Get("name") 
```

### 默认值

```go
viper.SetDefault("ContentDir", "content")
viper.SetDefault("LayoutDir", "layouts")
viper.SetDefault("Taxonomies", map[string]string{"tag": "tags", "category": "categories"})
```

## 获取配置

> 从`viper`中获取配置信息。

- `Get(key string) : interface{}`
- `GetBool(key string) : bool`
- `GetFloat64(key string) : float64`
- `GetInt(key string) : int`
- `GetIntSlice(key string) : []int`
- `GetString(key string) : string`
- `GetStringMap(key string) : map[string]interface{}`
- `GetStringMapString(key string) : map[string]string`
- `GetStringSlice(key string) : []string`
- `GetTime(key string) : time.Time`
- `GetDuration(key string) : time.Duration`
- `IsSet(key string) : bool`
- `AllSettings() : map[string]interface{}`
