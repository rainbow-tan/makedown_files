Go语言的安装

## 1、下载地址

https://studygolang.com/dl

![image-20220608195906159](https://img2022.cnblogs.com/blog/1768648/202206/1768648-20220611191031188-548710968.png)

Windows系统下载msi文件（例如https://golang.google.cn/dl/go1.18.3.windows-386.msi），然后安装就直接下一步，下一步就行

Linux系统下载tar.gz文件（例如 https://studygolang.com/dl/golang/go1.16.13.linux-amd64.tar.gz），然后这样安装

- 拷贝下载好的tar.gz文件到/home/tmp目录，或者下载

```
cd /home/tmp
wget https://studygolang.com/dl/golang/go1.16.13.linux-amd64.tar.gz
```

- 解压到/usr/local文件夹

```
tar -C /usr/local/ -zxf go1.16.13.linux-amd64.tar.gz
```

- 配置环境变量

```
echo "export PATH=$PATH:/usr/local/go/bin" >> /etc/profile
```

- 使文件生效

```
source /etc/profile
```

- 验证是否安装成功

```
cd /root
go env
```

![image-20220614152823005](https://img2022.cnblogs.com/blog/1768648/202206/1768648-20220614153104337-1807547173.png)

## 2、配置国内Go 模块代理。

代理地址是 https://goproxy.cn/

![image-20220608200417457](https://img2022.cnblogs.com/blog/1768648/202206/1768648-20220611191031892-1420467874.png)

Go 1.13 版本及以上（推荐）

打开你的终端并执行
```
go env -w GO111MODULE=on
go env -w GOPROXY=https://goproxy.cn,direct
```
版本太低的参考官网 https://goproxy.cn/
