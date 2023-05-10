ubuntu 云镜像默认密码重置

# 背景

ubuntu的云镜像，默认是没有提供用户名密码的，当我们直接启动时，就无法登陆进去，真的是难受坏了。

# 解决办法

通过重置root密码来设置一个初始化密码即可

# 例子

以ubuntu22.04版本为例

## ①下载地址：

## https://cloud-images.ubuntu.com/jammy/current/

![image-20230510151524944](https://img2023.cnblogs.com/blog/1768648/202305/1768648-20230510152905020-1853020109.png)

## ②创建虚拟机

我这里的宿主机是centos7的，然后使用virt-manager来创建，步骤参考下面的截图，就下一步下一步就创建成功了

![image-20230510151906835](https://img2023.cnblogs.com/blog/1768648/202305/1768648-20230510152905519-1361168473.png)

![image-20230510152036513](https://img2023.cnblogs.com/blog/1768648/202305/1768648-20230510152905870-1165640157.png)

这样就起来了

![image-20230510152146612](https://img2023.cnblogs.com/blog/1768648/202305/1768648-20230510152906185-158205957.png)

但是不知道密码，无法登陆进去

## ③重置密码

如①所示，我这里下载的是jammy-server-cloudimg-amd64.img，为了保留源文件，我拷贝了一份

```
cp jammy-server-cloudimg-amd64.img jammy-server-cloudimg-amd64.my.img
yum install -y libguestfs-tools
virt-customize -a jammy-server-cloudimg-amd64.my.img --root-password password:123456
```

然后重置jammy-server-cloudimg-amd64.my.img密码为123456

然后重复第②步再次安装，使用123456就登陆进去了

![image-20230510152634984](https://img2023.cnblogs.com/blog/1768648/202305/1768648-20230510152906497-1645392853.png)

# 参考链接

https://www.cnblogs.com/zhangyy3/p/14793565.html