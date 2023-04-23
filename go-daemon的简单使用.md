# go-daemon的简单使用

没什么特殊的，就是记录一下笔记，和官网以及百度上的几乎一模一样

## 1、介绍

这是一个在 Go 中编写系统守护进程的库。

## 2、特性

- Go 协程安全的守护进程；
- 内置使用 pid 文件；
- 轻松处理系统信号；
- 守护进程的控制。

## 3、基本使用

1. 导入"github.com/sevlyar/go-daemon"
2. 实例化&daemon.Context
3. .Reborn()实现fork进程
4. 编写自己的服务
5. .Release()释放资源

```go
package main

import (
   "fmt"
   "html"
   "log"
   "net/http"
   "os"
   "github.com/sevlyar/go-daemon"
)

// 要终止守护进程，使用:
// kill `cat sample.pid`。
func main() {

   context := &daemon.Context{
      PidFileName: "sample.pid",
      PidFilePerm: 0644,
      LogFileName: "sample.log",
      LogFilePerm: 0640,
      WorkDir:     "./",                        //创建子进程时会切到该目录
      Umask:       027,                         //If Umask is non-zero, the daemon-process call Umask() func with given value.
      Args:        []string{"go-daemon守护进程服务"}, //传递给子进程的参数 是nil就用os.Args
   }

   d, err := context.Reborn() //拷贝上下文创建子进程 在父进程返回*os.Process 在子进程程返回的是nil 其他情况返回错误
   if err != nil {
      log.Fatal("创建守护进程失败, err:", err.Error())
   }
   if d != nil {
      log.Printf("这是在父进程的标志")
      return
   }

   defer func(context *daemon.Context) {
      err := context.Release()
      if err != nil {
         log.Printf("释放失败:%s", err.Error())
      }
      log.Printf("释放成功!!!")
   }(context)

   log.Print("守护进程启动!!!!!")
   serveHTTP()
   log.Print("子进程服务退出了~~~")
}

func serveHTTP() {
   log.Printf("参数:%v", os.Args)
   http.HandleFunc("/", httpHandler)
   err := http.ListenAndServe("127.0.0.1:8080", nil)
   if err != nil {
      fmt.Printf("启动服务失败, err:%s", err.Error())
      return
   }
}

func httpHandler(w http.ResponseWriter, r *http.Request) {
   log.Printf("request from %s: %s %q", r.RemoteAddr, r.Method, r.URL)
   _, err := fmt.Fprintf(w, "hello 你好啊!: %q", html.EscapeString(r.URL.Path))
   if err != nil {
      log.Printf("返回信息失败, err:%s", err)
      return
   }
}
```

## 4、处理信号

1. 导入"github.com/sevlyar/go-daemon"
2. .AddCommand()添加信号对应的处理函数
3. 发送信号给子进程
4. 实例化&daemon.Context
5. .Reborn()实现fork进程
6. 编写自己的服务
7. .Release()释放资源

```go
package main

import (
   "flag"
   "github.com/sevlyar/go-daemon"
   "log"
   "os"
   "syscall"
   "time"
)

var (
   signal = flag.String("s", "", `Send signal to the daemon:
  quit — graceful shutdown
  stop — fast shutdown
  reload — reloading the configuration file`)
)

func main() {
   flag.Parse()
   daemon.AddCommand(daemon.StringFlag(signal, "quit"), syscall.SIGQUIT, termHandler)
   daemon.AddCommand(daemon.StringFlag(signal, "stop"), syscall.SIGTERM, termHandler)
   daemon.AddCommand(daemon.StringFlag(signal, "reload"), syscall.SIGHUP, reloadHandler)

   context := &daemon.Context{
      PidFileName: "sample.pid",
      PidFilePerm: 0644,
      LogFileName: "sample.log",
      LogFilePerm: 0640,
      WorkDir:     "./",                        //创建子进程时会切到该目录
      Umask:       027,                         //If Umask is non-zero, the daemon-process call Umask() func with given value.
      Args:        []string{"go-daemon守护进程服务"}, //传递给子进程的参数 是nil就用os.Args
   }

   if len(daemon.ActiveFlags()) > 0 {
      log.Printf("活动的标志位, 数量:%d", len(daemon.ActiveFlags()))
      d, err := context.Search() //查询守护进程,成功返回*os.Process失败返回err 没有PID文件返回nil
      if err != nil {
         log.Fatalf("查询子进程失败: %s", err.Error())
      }
      log.Printf("存在的子进程是:%+v", d)
      err = daemon.SendCommands(d)
      if err != nil {
         log.Printf("发送信号给子进程失败, 进程:%+v, err:%s", d, err)
         return
      }
      return
   }

   d, err := context.Reborn() //拷贝上下文创建子进程 在父进程返回*os.Process 在子进程程返回的是nil 其他情况返回错误
   if err != nil {
      log.Fatal("创建守护进程失败, err:", err.Error())
   }
   if d != nil {
      log.Printf("这是在父进程的标志")
      return
   }

   defer func(context *daemon.Context) {
      err := context.Release()
      if err != nil {
         log.Printf("释放失败:%s", err.Error())
      }
      log.Printf("释放成功!!!")
   }(context)

   log.Print("守护进程启动!!!!!")

   go worker()

   err = daemon.ServeSignals() //调用信号对应的函数
   if err != nil {
      log.Printf("处理信号错误: %s", err.Error())
   }

   log.Print("子进程服务退出了~~~")
}

var (
   stop = make(chan struct{})
   done = make(chan struct{})
)

func worker() {
LOOP:
   for {
      time.Sleep(time.Second) // this is work to be done by worker.
      log.Printf("现在的时间是:%v", time.Now().String())
      select {
      case <-stop:
         log.Printf("收到结束信息")
         break LOOP
      default:
      }
   }
   done <- struct{}{}
}

func termHandler(sig os.Signal) error {
   log.Println("终止程序......")
   stop <- struct{}{}
   if sig == syscall.SIGQUIT {
      <-done
   }
   return daemon.ErrStop
}

func reloadHandler(sig os.Signal) error {
   log.Println("重新加载程序")
   return nil
}
```

发送信号可以这样

![image-20220909170851267](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220909170851267.png)

也可以直接`kill -s pid`

## 5、个人笔记：

context.Reborn() 返回 **err:daemon: Resource temporarily unavailable** 时，就是已经启动了。

context.Search() 返回的就是守护进程的信息

官网：https://github.com/sevlyar/go-daemon

参考：https://juejin.cn/post/6947271530023911461

