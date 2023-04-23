go控制台输出乱码 | go执行命令输出乱码 | 编码bytes为gbk字符串

## 1、问题描述

使用go运行ping命令时，输出乱码，现象如下

代码

```go
package main

import (
   "fmt"
   "os/exec"
)

func ping(ip string) {
   cmd := exec.Command("ping", ip)
   out, err := cmd.CombinedOutput()
   if err != nil {
      fmt.Printf("ping %s 失败, err:%s\n", ip, err)
      return
   }
   output := string(out)
   fmt.Printf("ping %s 成功, 返回信息:%s\n", ip, output)
}
func main() {
   ping("172.16.90.19")
}
```

现象：无论是cmd控制台，还是goland控制台，都输出乱码

![image-20221009102116158](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20221009102116158.png)

## 2、原因

谷歌后，大概原因是go默认只支持utf-8编码，而控制台是gbk，所以，当想使用gbk的控制台显示utf-8编码的字符时，就无法显示了。

## 3、解决

知道了原因，则使用gbk编码bytes就行

`"golang.org/x/text/encoding/simplifiedchinese"`

`output, err := simplifiedchinese.GBK.NewDecoder().Bytes(out)`

代码

```go
package main

import (
   "fmt"
   "golang.org/x/text/encoding/simplifiedchinese"
   "os/exec"
)

func ping(ip string) {
   cmd := exec.Command("ping", ip)
   out, err := cmd.CombinedOutput()
   if err != nil {
      fmt.Printf("ping %s 失败, err:%s\n", ip, err)
      return
   }
   output, err := simplifiedchinese.GBK.NewDecoder().Bytes(out)
   if err != nil {
      fmt.Printf("编码 %+v 失败, err:%s\n", out, err)
      return
   }
   fmt.Printf("ping %s 成功, 返回信息:%s\n", ip, output)
}
func main() {
   ping("172.16.90.19")
}
```

![image-20221009102754514](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20221009102754514.png)

参考：https://blog.csdn.net/rznice/article/details/88122923