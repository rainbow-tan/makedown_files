go查看某段ip是否通的脚本

#### 例1：

```go
package main

import (
   "fmt"
   "golang.org/x/text/encoding/simplifiedchinese"
   "os/exec"
   "strings"
   "sync"
)

func ping(ip string, pong chan string, wg *sync.WaitGroup) {
   defer wg.Done()
   cmd := exec.Command("ping", ip)
   out, err := cmd.CombinedOutput()
   if err != nil {
      fmt.Printf("ping %s 失败, err:%s\n", ip, err)
      pong <- fmt.Sprintf("%s  ====>  %v", ip, false)
      return
   }
   output, err := simplifiedchinese.GBK.NewDecoder().Bytes(out)
   if err != nil {
      fmt.Printf("编码 %+v 失败, err:%s\n", out, err)
      pong <- fmt.Sprintf("%s  ====>  %v", ip, false)
      return
   }
   info := string(output)
   fmt.Printf("ping %s 成功, 返回信息:%s\n", ip, info)
   exist := strings.Contains(info, "TTL=")
   if !exist {
      pong <- fmt.Sprintf("%s  ====>  %v", ip, false)
      return
   }
   pong <- fmt.Sprintf("%s  ====>  %v", ip, true)
}

func main() {
   prex := "172.16.90."
   start := 10
   end := 30
   result := make(chan string, end-start)
   wg := sync.WaitGroup{}
   wg.Add(end - start)
   for i := start; i < end; i++ {
      go ping(fmt.Sprintf("%s%d", prex, i), result, &wg)
   }
   wg.Wait()
   close(result)
   for v := range result {
      fmt.Printf("%s\n", v)
   }
}
```

运行

![image-20221009172321300](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20221009172321300.png)

#### 例2

```go
package main

import (
   "fmt"
   "golang.org/x/text/encoding/simplifiedchinese"
   "os/exec"
   "strings"
)

type pinger struct {
   ip   string
   tong chan bool
}

func (p *pinger) ping() {
   fmt.Printf("ip:%s\n", p.ip)

   cmd := exec.Command("ping", p.ip)
   out, err := cmd.CombinedOutput()
   if err != nil {
      fmt.Printf("ping %s 失败, err:%s\n", p.ip, err)
      p.tong <- false
      return
   }
   output, err := simplifiedchinese.GBK.NewDecoder().Bytes(out)
   if err != nil {
      fmt.Printf("编码 %+v 失败, err:%s\n", out, err)
      p.tong <- false
      return
   }
   info := string(output)
   fmt.Printf("ping %s 成功, 返回信息:%s\n", p.ip, info)
   exist := strings.Contains(info, "TTL=")
   if !exist {
      p.tong <- false
      return
   }
   p.tong <- true
}

func main() {
   prex := "172.16.90."
   start := 10
   end := 30
   var pingers []pinger
   for i := start; i <= end; i++ {
      pingers=append(pingers,pinger{ip: fmt.Sprintf("%s%d",prex,i),tong: make(chan bool,1)})
   }
   for _,v:=range pingers{
      fmt.Printf("%s ===> %v\n",v.ip,v.tong)
   }
   for _,v:=range pingers{
      p:=v
      go p.ping()
   }
   for _,v:=range pingers{
      fmt.Printf("%s ===> %v\n",v.ip,<-v.tong)
   }
   fmt.Println("结束")
}
```

运行

![image-20221010140945500](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20221010140945500.png)