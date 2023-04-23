ubuntu 22.04 开机自启动脚本

## 1、完善rc-local.service服务

```
vi /lib/systemd/system/rc-local.service 
```

添加红色框框部分

```
[Install]
WantedBy=multi-user.target  
Alias=rc-local.service
```

![image-20230423140543196](https://img2023.cnblogs.com/blog/1768648/202304/1768648-20230423141753418-85745472.png)

## 2、添加/etc/rc.local文件

1. 创建文件`touch /etc/rc.local`

2. 在/etc/rc.local文件里面输入要运行的shell命令

3. 添加可执行权限 `chmod +x /etc/rc.local`

   例如

![image-20230423141111733](https://img2023.cnblogs.com/blog/1768648/202304/1768648-20230423141753918-1739098010.png)

## 3、设置开启启动rc-local服务

执行`systemctl enable rc-local.service`即可

参考链接：https://blog.csdn.net/pazzn/article/details/126102646