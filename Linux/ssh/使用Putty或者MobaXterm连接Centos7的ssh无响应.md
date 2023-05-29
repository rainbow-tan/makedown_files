使用Putty或者MobaXterm连接Centos7的ssh无响应

## 现象：

使用putty或者MobaXterm连接ssh时，长时间未响应，如图

![image-20220620095015432](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220620095015432.png)

## 解决思路：

以连接`192.168.10.22`为例

### 1、先ping一下,ping通才可以连接`ping 192.168.10.22`

![image-20220620095449342](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220620095449342.png)

### 2、关闭防火墙，再次连接试一试，可能是防火墙没有添加该服务，或者没有暴露端口

查看防火墙状态`systemctl status firewalld`

关闭防火墙`systemctl stop firewalld`

### 3、如果防火墙关闭了，也连接不上，则可以查看是不是ssh的问题

找一个同网段的机器，例如找`192.168.10.23`，然后使用ssh命令连接`192.168.10.22`,看一看具体卡在哪

```
ssh -v root@192.168.10.22
```

①**如果出现The authenticity of host xxx can't be established则修改配置文件/etc/ssh/ssh_config**

修改/etc/ssh/ssh_config文件中的配置

添加StrictHostKeyChecking no
![image-20220620104250612](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220620104250612.png)

②**如果是卡在SSH2_MSG_SERVICE_ACCEPT received**

![image-20220620100329560](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220620100329560.png)

则修改`192.168.10.22`ssh配置文件/etc/ssh/sshd_config

```
vi /etc/ssh/sshd_config
```

修改GSSAPIAuthentication  no

修改UseDNS  no

![image-20220620102153085](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220620102153085.png)

![image-20220620102239966](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220620102239966.png)

然后重启服务`systemctl restart sshd`

