centos虚拟机无法被宿主机访问web服务

### 1、现象

在虚拟机中可以访问自己的web服务，IP是192.168.40.129

![1653988482724](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\1653988482724.png)

![1653988570837](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\1653988570837.png)

在宿主机却不可以

![1653988616376](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\1653988616376.png)

### 2、解决办法

#### 2.1、关闭防火墙即可

```
sudo systemctl stop firewalld
```

![1653988717492](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\1653988717492.png)

关于防火墙开关可参考 https://www.cnblogs.com/rainbow-tan/p/16300235.html

#### 2.2、如果不想关闭防火墙，则可以通过添加端口号来进行暴露

 **查看防火墙规则** 

```
firewall-cmd --list-all
```

![1653989412540](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\1653989412540.png)

**添加端口**

```
firewall-cmd --zone=public --add-port=8010/tcp --permanent  
```

permanent永久生效，没有此参数重启后失效

**添加端口后需要重新加载防火墙**

```
 firewall-cmd --reload 
```

**查看端口是否开启**（yes是开启，no是未开启）

```
firewall-cmd --zone=public --query-port=8010/tcp
```

![1653989477084](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\1653989477084.png)

**删除端口**

```
firewall-cmd --zone=public --remove-port=8010/tcp --permanent
```

**查看开启了哪些端口**

```
firewall-cmd --list-ports
```

因此，我们只要添加8010端口，然后重启防火墙即可

学习链接：https://blog.csdn.net/Honnyee/article/details/81535464