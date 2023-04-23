Windows上编译可运行在Linux上的go二进制文件

## 1、前言

默认Windows上编译的go二进制为exe,只能运行在Windows上，而想要在Linux上运行，则需要到Linux的平台编译。有没有一种办法可以直接在Windows上编译，然后直接拿到Linux就可以运行呢？这几天就看到一个文章，就是做的这个事情，也比较简单，但很有用，记录一下。

实现功能：Windows上编译go，部署到Linux运行

参考链接：https://blog.csdn.net/wykqh/article/details/121861036

## 2、步骤

打开cmd，运行以下命令（设置为Linux平台，应该是这么理解，然后是设置为amd64架构）

```bash
SET GOOS=linux
SET GOARCH=amd64
```

然后正常编译即可

```bash
go build -o xxx xxx.go
```

## 3、例子

go代码

```go
package main

import (
   "fmt"
   "golang.org/x/crypto/ssh"
   "net"
   "os"
   "os/exec"
   "time"
)

func hostKeyCallback(hostname string, remote net.Addr, key ssh.PublicKey)error  {
   fmt.Println(fmt.Sprintf("hostname:%s",hostname))
   fmt.Println(fmt.Sprintf("remote:%+v",remote))
   fmt.Println(fmt.Sprintf("key:%+v",key))
   return nil

}

func main(){
   const DefaultPathPerm       = 0744
   path:="/etc/node-exporter"
   err := os.MkdirAll(path, DefaultPathPerm)
   if err != nil {
      fmt.Println(fmt.Sprintf("创建目录失败, 目录:%s, err:%s",path,err.Error()))
      return
   }

   cmsStr :="yum install -y wget tar"
   cmd:=exec.Command("yum","install","-y","wget","tar")
   combinedOutput, err := cmd.CombinedOutput()
   if err != nil {
      fmt.Println(fmt.Sprintf("执行命令失败, 命令:'%s', err:%s", cmsStr,err.Error()))
      return
   }
   fmt.Println(fmt.Sprintf("执行命令成功,命令:'%s', 输出:%s", cmsStr,combinedOutput))

   user:="root"
   passwd:="123456"
   addr:="192.168.108.216:22"


   auth := make([]ssh.AuthMethod, 0)
   auth = append(auth, ssh.Password(passwd))
   clientConfig := ssh.ClientConfig{
      Config:            ssh.Config{},
      User:              user,
      Auth:              auth,
      HostKeyCallback:   hostKeyCallback,
      BannerCallback:    nil,
      ClientVersion:     "",
      HostKeyAlgorithms: nil,
      Timeout:           30*time.Second,
   }

   client, err := ssh.Dial("tcp",addr,&clientConfig)
   if err != nil {
      fmt.Println(fmt.Sprintf("SSH连接Linux失败, addr:%s, err:%s",addr,err))
      return
   }
   fmt.Println(fmt.Sprintf("SSH连接Linux成功, addr:%s",addr))
   defer func(client *ssh.Client) {
      err := client.Close()
      if err != nil {
         fmt.Println(fmt.Sprintf("关闭SSH失败:%s, err:%s",addr,err))
      }
      fmt.Println(fmt.Sprintf("关闭SSH成功:%s, err:%s",addr,err))
   }(client)


}
```

编译

![image-20220930150500201](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220930150500201.png)



拷贝到Linux运行

![image-20220930150552592](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220930150552592.png)

