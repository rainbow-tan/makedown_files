go net包的基本使用之记录一些函数(包含一个简单的UDP客户端和服务端示例)

## 1、net.ResolveIPAddr

根据域名返回IP

## 2、net.ParseIP	

监测IP是否有效 有效返回IP, 无效返回nil

## 3、net.Interfaces	

获取系统的网卡信息

### 3.1、inter.Flags&net.FlagLoopback != 0

 判断是不是回环地址

**例子：**

```go
package main

import (
   "fmt"
   "net"
)

func main() {
   //根据域名返回IP
   address := "www.baidu.com"
   addr, err := net.ResolveIPAddr("ip", address)
   if err != nil {
      fmt.Printf("根据域名获取IP失败, e:%s\n", err)
   }
   fmt.Printf("域名 %s 的IP是 %s\n", address, addr)

   //检查IP地址格式是否有效 - 有效返回IP 无效返回nil
   s := "172.17.130.40"
   ip := net.ParseIP(s)
   if ip != nil {
      fmt.Printf("这是一个合法的IP:%s\n", s)
   } else {
      fmt.Printf("这不是一个合法的IP:%s\n", s)
   }

   //获取系统的网卡信息
   interfaces, err := net.Interfaces()
   if err != nil {
      fmt.Printf("获取系统的网卡信息失败, err:%s\n", err)
   }
   for _, inter := range interfaces {
      fmt.Printf("网卡信息是:%+v\n", inter)
      if inter.Flags&net.FlagLoopback != 0 {
         fmt.Printf("This is loopback lo:%+v\n", inter)
      }
   }


}
```

运行

![image-20220916142001283](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220916142001283.png)

## 4、net.ResolveUDPAddr

转换地址为UDP地址，可用于监听UDP

## 5、net.ListenUDP

监听UDP

## 6、.ReadFromUDP

从UDP中读取数据

**例子：**

4,5,6可用于构建一个简单的UDP监听服务端（service.go）

```go
package main

import (
   "fmt"
   "net"
)

func main() {

   //把地址转换为UDP的地址
   address := "127.0.0.1:8080"
   udpAddr, err := net.ResolveUDPAddr("udp", address)
   if err != nil {
      fmt.Println(fmt.Sprintf("转换UDP地址失败, err:%s", err))
      return
   }
   fmt.Println(fmt.Sprintf("转换UDP地址成功, address:%s", address))

   //监听UDP
   udp, err := net.ListenUDP("udp", udpAddr)
   if err != nil {
      fmt.Println(fmt.Sprintf("监听UDP失败, err:%s", err))
      return
   }
   fmt.Println(fmt.Sprintf("监听UDP成功, address:%s", address))

   //从UDP中读取数据
   b := make([]byte, 1024)
   n, addr, err := udp.ReadFromUDP(b)
   if err != nil {
      fmt.Println(fmt.Sprintf("接收客户端的数据失败, err:%s", err))
      return
   }
   fmt.Println(fmt.Sprintf("接收客户端的数据成功, 接收到的字节数:%d, data:%s", n, b))
   fmt.Println(fmt.Sprintf("客户端的信息:%+v", addr.String()))

}
```

## 7、net.Dial

连接给定的地址，以给定的形式

## 8、.Write

写数据到UDP,TCP中

**例子：**

7,8可构成一个简单的客户端（client.go）

```go
package main

import (
   "fmt"
   "net"
   "time"
)

func main() {

   //连接给定的network地址
   address := "127.0.0.1:8080"
   conn, err := net.Dial("udp", address)
   if err != nil {
      fmt.Println(fmt.Sprintf("连接失败, err:%s", err))
      return
   }
   fmt.Println(fmt.Sprintf("连接成功, address:%s", address))

   defer func(conn net.Conn) {
      err := conn.Close()
      if err != nil {
         fmt.Println(fmt.Sprintf("关闭连接失败, err:%s", err))
      }
      fmt.Println(fmt.Sprintf("关闭连接成功"))
   }(conn)

   //写数据到UDP
   msg := fmt.Sprintf("我是来自于client的信息,现在的时间是:%s", time.Now().Format("2006-01-02 15:04:05"))
   msgByte := []byte(msg)
   n, err := conn.Write(msgByte)
   if err != nil {
      fmt.Println(fmt.Sprintf("client发送失败, err:%s", err))
      return
   }
   fmt.Println(fmt.Sprintf("client发送成功%d个字节", n))

}
```

运行

![image-20220916144114981](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220916144114981.png)

![image-20220916144127572](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220916144127572.png)

## 9、net.JoinHostPort

拼接Host和Port

## 10、net.DialTimeout

连接给定的地址，以给定的形式 类似于net.Dial 但多个一个超时timeout

例子：检查端口是否被占用

```go
package main

import (
   "fmt"
   "net"
   "strconv"
   "time"
)

// ScanPort 检查端口是否被占用
func ScanPort(network string, hostname string, port int) bool {
   p := strconv.Itoa(port)
   addr := net.JoinHostPort(hostname, p)

   //连接该端口 能连接到就是已经被占用了 连接不到就是没被占用
   conn, err := net.DialTimeout(network, addr, 3*time.Second)
   if err != nil {
      fmt.Println(fmt.Sprintf("该端口未被占用, %s:%d, err:%s", hostname, port, err))
      return false
   }
   defer func(conn net.Conn) {
      err := conn.Close()
      if err != nil {
         fmt.Println(fmt.Sprintf("关闭连接失败:%s", err))
      }
      fmt.Println("关闭连接成功")
   }(conn)
   fmt.Println(fmt.Sprintf("该端口已被占用, %s:%d", hostname, port))
   return true
}
func main() {
   used := ScanPort("tcp", "127.0.0.1", 8000)
   fmt.Println(fmt.Sprintf("是否在使用:%v", used))
}
```

运行

![image-20220916144419017](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220916144419017.png)