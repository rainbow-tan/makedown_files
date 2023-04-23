go sftp上传文件

## 1、上传单个文件

使用"golang.org/x/crypto/ssh"连接到Linux环境

使用"github.com/pkg/sftp"创建sftp客户端

然后传输文件

```go
package main

import (
   "fmt"
   "github.com/pkg/sftp"
   "golang.org/x/crypto/ssh"
   "io/ioutil"
   "net"
   "os"
   "path"
   "path/filepath"
   "time"
)

func hostKeyCallback(hostname string, remote net.Addr, key ssh.PublicKey) error {
   return nil

}

//获取一个STFP客户端连接
func connect(user, password, host string, port int) (*sftp.Client, error) {

   auth := []ssh.AuthMethod{ssh.Password(password)}

   config := &ssh.ClientConfig{
      User:            user,
      Auth:            auth,
      Timeout:         30 * time.Second,
      HostKeyCallback: hostKeyCallback,
   }

   // connect to ssh
   addr := fmt.Sprintf("%s:%d", host, port)
   client, err := ssh.Dial("tcp", addr, config)
   if err != nil {
      return nil, err
   }

   // create sftp client
   sftpClient, err := sftp.NewClient(client)
   if err != nil {
      return nil, err
   }
   fmt.Printf("连接%s@%s:%d成功！\n", user, host, port)
   return sftpClient, nil
}

//上传文件
func uploadFile(sftpClient *sftp.Client, localFilename string, remotePath string) (int, error) {

   file, err := os.Open(localFilename)
   if err != nil {
      return 0, err

   }
   defer func(srcFile *os.File) {
      err := srcFile.Close()
      if err != nil {
      }
   }(file)

   var remoteFileName = path.Join(remotePath, filepath.Base(localFilename))
   err = sftpClient.MkdirAll(remotePath)
   if err != nil {
      return 0, err
   }

   dstFile, err := sftpClient.Create(remoteFileName)
   if err != nil {
      return 0, err

   }
   defer func(dstFile *sftp.File) {
      err := dstFile.Close()
      if err != nil {

      }
   }(dstFile)

   ff, err := ioutil.ReadAll(file)
   if err != nil {
      return 0, err
   }
   n, err := dstFile.Write(ff)
   if err != nil {
      return n, err
   }
   fmt.Printf("上传文件成功, 保存到:%s\n", remoteFileName)
   return n, nil
}
func main() {
   user := "root"
   password := "******"
   host := "192.168.93.129"
   port := 22
   filename := "a.txt"
   remoteFilePath := "/home/tmp"

   client, err := connect(user, password, host, port)
   if err != nil {
      fmt.Println(fmt.Sprintf("连接%s:%d失败, err:%s", host, port, err))
      return
   }
   _, err = uploadFile(client, filename, remoteFilePath)
   if err != nil {
      fmt.Printf("上传失败, local file:%s, remote path:%s err:%s\n", filename, remoteFilePath, err)
      return
   }

}
```

## 2、上传当前目录下的所有文件，仅限当前目录下

```go
package main

import (
   "fmt"
   "github.com/pkg/sftp"
   "golang.org/x/crypto/ssh"
   "io/ioutil"
   "net"
   "os"
   "path"
   "path/filepath"
   "time"
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

func hostKeyCallback(hostname string, remote net.Addr, key ssh.PublicKey) error {
   return nil

}

//获取一个STFP客户端连接
func connect(user, password, host string, port int) (*sftp.Client, error) {

   auth := []ssh.AuthMethod{ssh.Password(password)}

   config := &ssh.ClientConfig{
      User:            user,
      Auth:            auth,
      Timeout:         30 * time.Second,
      HostKeyCallback: hostKeyCallback,
   }

   // connect to ssh
   addr := fmt.Sprintf("%s:%d", host, port)
   client, err := ssh.Dial("tcp", addr, config)
   if err != nil {
      return nil, err
   }

   // create sftp client
   sftpClient, err := sftp.NewClient(client)
   if err != nil {
      return nil, err
   }
   fmt.Printf("连接%s@%s:%d成功！\n", user, host, port)
   return sftpClient, nil
}

//上传文件
func uploadFile(sftpClient *sftp.Client, localFilename string, remotePath string, ch chan string) {

   file, err := os.Open(localFilename)
   if err != nil {
      ch <- fmt.Sprintf("上传失败, path:%s, err:%s", filepath.Base(localFilename), err)

   }
   defer func(srcFile *os.File) {
      err := srcFile.Close()
      if err != nil {
      }
   }(file)

   var remoteFileName = path.Join(remotePath, filepath.Base(localFilename))
   err = sftpClient.MkdirAll(remotePath)
   if err != nil {
      ch <- fmt.Sprintf("上传失败, path:%s, err:%s", filepath.Base(localFilename), err)
      return
   }

   dstFile, err := sftpClient.Create(remoteFileName)
   if err != nil {
      ch <- fmt.Sprintf("上传失败, path:%s, err:%s", filepath.Base(localFilename), err)
      return

   }
   defer func(dstFile *sftp.File) {
      err := dstFile.Close()
      if err != nil {

      }
   }(dstFile)

   ff, err := ioutil.ReadAll(file)
   if err != nil {
      ch <- fmt.Sprintf("上传失败, path:%s, err:%s", filepath.Base(localFilename), err)
      return
   }
   _, err = dstFile.Write(ff)
   if err != nil {
      ch <- fmt.Sprintf("上传失败, path:%s, err:%s", filepath.Base(localFilename), err)
      return
   }

   ch <- fmt.Sprintf("上传文件成功, 保存到:%s\n", remoteFileName)
}
func main() {
   user := "root"
   password := "abc123"
   host := "172.16.90.18"
   port := 22
   remoteFilePath := "/home/tmp"

   info, err := getCurrentDir(".")
   if err != nil {
      return
   }
   currentFiles := info.currentFiles

   client, err := connect(user, password, host, port)
   if err != nil {
      fmt.Println(fmt.Sprintf("连接%s:%d失败, err:%s", host, port, err))
      return
   }

   var chs = make(chan string, len(currentFiles))
   for _, file := range currentFiles {
      go uploadFile(client, file, remoteFilePath, chs)
   }
   count := 0
   for c := range chs {
      count++
      fmt.Printf("上传情况:%s\n", c)
      if count == len(currentFiles) {
         return
      }
   }

}
```

参考https://blog.csdn.net/fu_qin/article/details/78741854