## 安装
```shell
go get -u -v gitlab.meitu.com/gocommons/accesslog
```
    
## 使用

### 初始化`AccessLogger`

```golang
acl, err := accesslog.NewLogger(
    accesslog.Output(os.Stdout),
    accesslog.Pattern(accesslog.DefaultPattern),
)

if err != nil {
    log.Panic(err)
}
```

上述代码中的`Output`和`Pattern`选项均使用的是默认值，与下面代码等效

```golang
acl, err := accesslog.NewLogger()

if err != nil {
    log.Panic(err)
}
```

### 自定义`AccessLogger`

#### 输出到文件，按时间滚动

1. 获取依赖: `go get -u -v gitlab.meitu.com/gocommons/cores/io` 
1. import: `import coresio "gitlab.meitu.com/gocommons/cores/io"`
1. 初始化`Writer`:
    + 按日滚动文件：
     
     ```golang
     rollingWriter, err := coresio.NewRollingFileWriter("./logs/access.log", coresio.NewDailyRollingManager())
    
     if err != nil {
        log.Panic(err)
     }
     ```
    + 按小时滚动或自定义时间格式: 
    
    ```golang
    rollingManager := coresio.NewTimePatternRollingManager("20060102-15")
    rollingWriter, err := coresio.NewRollingFileWriter("logs/access.log", rollingManager)
    
    if err != nil {
        log.Panic(err)
    }
    ```
1. 设置`Output`:

```golang
acl, err := accesslog.NewLogger(
    accesslog.Output(rollingWriter),
)

if err != nil {
    log.Panic(err)
}
```

#### 自定义日志输出格式

默认格式：`%{2006-01-02T15:04:05.999-0700}t %a - %{Host}i "%r" %s - %T "%{X-Real-IP}i" "%{X-Forwarded-For}i" %{Content-Length}i - %{Content-Length}o %b %{CDN}i`

Pattern说明：

```
%a - Remote IP address
%b - Bytes sent, excluding HTTP headers, or '-' if no bytes were sent
%B - Bytes sent, excluding HTTP headers
%H - Request protocol
%m - Request method
%q - Query string (prepended with a '?' if it exists, otherwise an empty string
%r - First line of the request
%s - HTTP status code of the response
%t - Time the request was received, in the format "18/Sep/2011:19:18:28 -0400".
%U - Requested URL path
%D - Time taken to process the request, in millis
%T - Time taken to process the request, in seconds
%{xxx}i - Incoming request headers
%{xxx}o - Outgoing response headers
%{xxx}t - Time the request was received, in the format of xxx
```

### 使用`AccessLogger`

#### 与 [gin](https://github.com/gin-gonic/gin) 框架一起使用

1. 获取依赖: `go get -u -v gitlab.meitu.com/gocommons/accesslog/gin` 
1. import: `import ginacl "gitlab.meitu.com/gocommons/accesslog/gin"`
1. 组合`gin.HandlerFunc`:

```golang
engine := gin.New()
engine.Use(ginacl.AccessLogFunc(acl))
```

#### 与 `net/http` 包一起使用

1. 获取依赖: `go get -u -v gitlab.meitu.com/gocommons/accesslog/nethttp` 
1. import: `import httpacl "gitlab.meitu.com/gocommons/accesslog/nethttp"`
1. Wrap `http.HandlerFunc`:

```golang
func handler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Hi there, I love %s!", r.URL.Path[1:])
}

func main() {
    http.HandleFunc("/", httpacl.NewHandlerFuncWithAccessLog(handler))
    http.ListenAndServe(":8080", nil)
}
```
