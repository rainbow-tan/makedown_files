Go语言sync.Map的简单使用（map是线程不安全的）

## 1、map是线程不安全的

Go语言中的 map 在并发情况下，只读是线程安全的，同时读写是线程不安全的。

可以这么复现，会报错： `fatal error: concurrent map read and map write`

```go
package main

import (
   "fmt"
   "time"
)

var count = 100000

func WriteNonCMap(nonCMap map[int]struct{}) {
   for index := 0; index < count; index++ {
      time.Sleep(2 * time.Nanosecond)
      nonCMap[index] = struct{}{}
   }
   fmt.Println("完成赋值")
}

func ReadNonCMap(nonCMap map[int]struct{}) {
   for index := 0; index < count; index++ {
      time.Sleep(3 * time.Nanosecond)
      fmt.Printf("%d==>%v\n", index, nonCMap[index])
   }
}
func main() {
   var a = make(map[int]struct{})
   go WriteNonCMap(a)
   go ReadNonCMap(a)
   select {}
}
```

运行

![image-20220914101922373](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220914101922373.png)

## 2、引入sync.Map

需要并发读写时，一般的做法是加锁，但这样性能并不高，Go语言在 1.9 版本中提供了一种效率较高的并发安全的 sync.Map，sync.Map 和 map 不同，不是以语言原生形态提供，而是在 sync 包下的特殊结构。

sync.Map 有以下特性：

- 无须初始化，直接声明即可。
- sync.Map 不能使用 map 的方式进行取值和设置等操作，而是使用 sync.Map 的方法进行调用，Store 表示存储，Load 表示获取，Delete 表示删除。
- 使用 Range 配合一个回调函数进行遍历操作，通过回调函数返回内部遍历出来的值，Range 参数中回调函数的返回值在需要继续迭代遍历时，返回 true，终止迭代遍历时，返回 false。

```go
package main

import (
   "fmt"
   "sync"
)

func traversal(k, v interface{}) bool {
   if k == "aaa" {
      fmt.Println("遍历到aaa, 结束遍历")
      return false
   } else {
      fmt.Printf("遍历到:%v, 继续遍历\n", k)
      return true
   }

}
func main() {
   var scene sync.Map
   // 将键值对保存到sync.Map
   scene.Store("greece", 97)
   scene.Store("london", 100)
   scene.Store("egypt", 200)
   // 从sync.Map中根据键取值
   fmt.Println(scene.Load("london"))

   v, ok := scene.Load("no key")
   if !ok {
      fmt.Println("不存在键\"no key\"")
   } else {
      fmt.Printf("获取的值是:%v\n", v)
   }
   fmt.Println("===============================")
   // 根据键删除对应的键值对
   scene.Delete("london")
   // 遍历所有sync.Map中的键值对
   scene.Range(func(k, v interface{}) bool {
      fmt.Println("iterate:", k, v)
      return true
   })
   fmt.Println("===============================")
   scene.Store("aaa", 1)
   scene.Store("aaa", 2)
   scene.Store("bb", 3)

   // 遍历所有sync.Map中的键值对
   scene.Range(func(k, v interface{}) bool {
      fmt.Println("iterate2:", k, v)
      return true
   })
   fmt.Println("===============================")
   // 遍历所有sync.Map中的键值对
   scene.Range(traversal)
}
```

参考：http://c.biancheng.net/view/34.html

https://www.jianshu.com/p/2ff54ea62e60

https://blog.csdn.net/eddycjy/article/details/117004395