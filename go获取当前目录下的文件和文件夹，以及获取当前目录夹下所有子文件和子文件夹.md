go获取当前目录下的文件和文件夹，以及获取当前目录夹下所有子文件和子文件夹

## 1、获取当前目录下的文件夹和文件

思路：使用ioutil.ReadDir(path)来遍历当前文件夹，使用IsDir()判断是文件夹还是文件，然后收集信息

```go
package main

import (
   "fmt"
   "io/ioutil"
   "path/filepath"
)

type currentDirInfo struct {
   currentDirs  []string
   currentFiles []string
}

func getCurrentDir(path string) (currentDirInfo, error) {
   infos, err := ioutil.ReadDir(path)
   if err != nil {
      return currentDirInfo{}, nil
   }
   var currentDirs []string
   var currentFiles []string
   for _, info := range infos {
      fmt.Printf("Name:%-30s    Size:%-10d字节    Modtime:%s    IsDir:%v\n", info.Name(), info.Size(), info.ModTime().Format("2006-01-02 03:04:05"), info.IsDir())
      abs, err := filepath.Abs(info.Name())
      if err != nil {
         return currentDirInfo{}, nil
      }
      if info.IsDir() {
         currentDirs = append(currentDirs, abs)
      } else {
         currentFiles = append(currentFiles, abs)
      }
   }
   fmt.Printf("=======================当前文件夹==============================\n")
   for _, v := range currentDirs {
      fmt.Printf("当前文件夹:%s\n", v)
   }
   fmt.Printf("========================当前文件=============================\n")
   for _, v := range currentFiles {
      fmt.Printf("当前文件:%s\n", v)
   }

   return currentDirInfo{currentDirs: currentDirs, currentFiles: currentFiles}, nil
}
func main() {
   _, err := getCurrentDir(".")
   if err != nil {
      return
   }
}
```

运行

![image-20221010144034652](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20221010144034652.png)

## 2、获取当前目录下的所有文件和所有文件夹（包括子目录）

```go
package main

import (
   "fmt"
   "io/fs"
   "path/filepath"
)

func walk(path string, info fs.FileInfo, err error) error {
   fmt.Printf("%s %+v %v\n", path, info.Name(), err)
   return err
}

type currentDirInfo struct {
   currentDirs  []string
   currentFiles []string
}

func getDirInfo(currentDir string) (currentDirInfo, error) {
   var currentDirs []string
   var currentFiles []string
   abs, err := filepath.Abs(currentDir)
   if err != nil {
      return currentDirInfo{}, nil
   }
   err = filepath.Walk(currentDir, func(path string, info fs.FileInfo, err error) error {
      absPath := filepath.Join(abs, path)
      if info.IsDir() {
         currentDirs = append(currentDirs, absPath)
      } else {
         currentFiles = append(currentFiles, absPath)
      }
      return err
   })
   if err != nil {
      return currentDirInfo{}, nil
   }
   fmt.Printf("=======================当前文件夹==============================\n")
   for _, v := range currentDirs {
      fmt.Printf("当前文件夹:%s\n", v)
   }
   fmt.Printf("========================当前文件=============================\n")
   for _, v := range currentFiles {
      fmt.Printf("当前文件:%s\n", v)
   }

   return currentDirInfo{currentDirs: currentDirs, currentFiles: currentFiles}, nil
}
func main() {
   _, err := getDirInfo(".")
   if err != nil {
      return
   }
}
```

运行

![image-20221010145807811](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20221010145807811.png)

参考：

https://www.jianshu.com/p/8b181edc9989