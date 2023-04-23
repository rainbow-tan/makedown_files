Centos7 firewalld防火墙基本使用

firewalld主要是在centos7以后才作为默认的防火墙管理工具的

firewalld有一个zone(域)的概念，每一个zone可以有自己规则，使用不同的zone就可以过滤不同的网络请求。

### 1、获取所有的域zones

```
firewall-cmd --get-zones
```

![image-20220615190918228](https://img2023.cnblogs.com/blog/1768648/202304/1768648-20230413140045233-1285733713.png)

输出是 block dmz drop external home internal public trusted work

- ##### 每个zone说明：

阻塞区域（block）：任何传入的网络数据包都将被阻止。
工作区域（work）：相信网络上的其他计算机，不会损害你的计算机。
家庭区域（home）：相信网络上的其他计算机，不会损害你的计算机。
公共区域（public）：不相信网络上的任何计算机，只有选择接受传入的网络连接。
隔离区域（DMZ）：隔离区域也称为非军事区域，内外网络之间增加的一层网络，起到缓冲作用。对于隔离区域，只有选择接受传入的网络连接。
信任区域（trusted）：所有的网络连接都可以接受。
丢弃区域（drop）：任何传入的网络连接都被拒绝。
内部区域（internal）：信任网络上的其他计算机，不会损害你的计算机。只有选择接受传入的网络连接。
外部区域（external）：不相信网络上的其他计算机，不会损害你的计算机。只有选择接受传入的网络连接。

- ##### 域文件就是XML文件

域文件保存在`/usr/lib/firewalld/zones/`目录下，以XML的形式保存

![image-20220616094251107](https://img2023.cnblogs.com/blog/1768648/202304/1768648-20230413140045651-1685608482.png)

### 2、查看默认域zone

```
firewall-cmd --get-default-zone
```

如果不进行改变，默认就是public，设置了别的，就是别的

![image-20220615191714514](https://img2023.cnblogs.com/blog/1768648/202304/1768648-20230413140046089-180004475.png)

### 3、查看所有活动的域zones

```
firewall-cmd --get-active-zones
```

![image-20220615192049966](https://img2023.cnblogs.com/blog/1768648/202304/1768648-20230413140046437-940915611.png)

### 4、更改默认的域zone

```
firewall-cmd --set-default-zone=trusted
```

![image-20220615192208831](https://img2023.cnblogs.com/blog/1768648/202304/1768648-20230413140046793-945550843.png)

### 5、创建新的域zone

```
firewall-cmd --permanent --new-zone=test_zone
```

创建完后需要加载防火墙才生效`firewall-cmd --reload`

![image-20220615192504486](https://img2023.cnblogs.com/blog/1768648/202304/1768648-20230413140047167-706414892.png)

### 6、开启端口让外部可以访问

```
firewall-cmd --permanent --add-port=8080/tcp
```

- --permanent 表示永久开启，如果不加，则重启服务器后失效

- --add-port=8080/tcp 表示外面可以使用8080，且协议是tcp才可以访问进来

- 开启完后需要加载防火墙才生效`firewall-cmd --reload`

- 开启的是默认域zone

可以指定给某个域开启某个端口

```
firewall-cmd --permanent --zone=public --add-port=8080/tcp
```

### 7、删除端口让外部无法访问

```
firewall-cmd --permanent --remove-port=8080/tcp
```

可以指定给某个域删除某个端口

```
firewall-cmd --permanent --zone=public --add-port=8080/tcp
```

删除完后需要加载防火墙才生效`firewall-cmd --reload`

### 8、添加一段端口

指定域

```
firewall-cmd --permanent --add-port=9001-9100/tcp --zone=public
```
不指定域默认当前域

```
firewall-cmd --permanent --add-port=9001-9100/tcp
```

开启完后需要加载防火墙才生效`firewall-cmd --reload`

### 9、开放ip以及ip段

新建永久规则，开放192.168.1.1单个源IP的访问

```
firewall-cmd --permanent --add-source=192.168.1.1
```

新建永久规则，开放192.168.1.0/24整个源IP段的访问

```
firewall-cmd --permanent --add-source=192.168.1.0/24
```

移除上述规则

```
firewall-cmd --permanent --remove-source=192.168.1.1
```

```
firewall-cmd --permanent --remove-source=192.168.1.0/24
```

需要加载防火墙才生效`firewall-cmd --reload`

当然也可以指定域，不过一般都是设置默认域

### 10、查询端口是否开放

指定域

```
firewall-cmd --query-port=22/tcp --zone=public
```

不指定域默认当前域

```
firewall-cmd --query-port=22/tcp
```

![image-20220616110445315](https://img2023.cnblogs.com/blog/1768648/202304/1768648-20230413140047491-943206312.png)

输出yes就是开放，no就是不开放

### 11、显示添加的IP和IP段

指定域

```
firewall-cmd --list-sources --zone=public
```
不指定域默认当前域
```
firewall-cmd --list-sources
```

![image-20220616111317254](https://img2023.cnblogs.com/blog/1768648/202304/1768648-20230413140047822-324127627.png)

### 12.显示添加的端口

指定域

```
firewall-cmd --zone=public  --list-port
```

不指定域默认当前域

```
firewall-cmd --list-port
```

![image-20220616105808771](https://img2023.cnblogs.com/blog/1768648/202304/1768648-20230413140048159-890908372.png)

### 13、添加和删除服务

指定域

```
firewall-cmd --permanent --zone=home --add-service=ssh
```

```
firewall-cmd --permanent --zone=home --remove-service=ssh
```

不指定域默认当前域

```
firewall-cmd --permanent --add-service=ssh
```

```
firewall-cmd --permanent --remove-service=ssh
```

### 14、显示添加的服务

指定域

```
 firewall-cmd --zone=public --list-services
```
不指定域默认当前域

```
 firewall-cmd --list-services
```

### 15、添加多个服务

指定域

```
firewall-cmd --permanent --zone=home --add-service={http,https}
```

不指定域默认当前域

```
firewall-cmd --permanent --add-service={http,https}
```

### 16、查看服务的规则

```
firewall-cmd --info-service=ssh
```

![image-20220616112059886](https://img2023.cnblogs.com/blog/1768648/202304/1768648-20230413140048565-268815269.png)

### 17、显示所有配置规则

指定域

```
firewall-cmd --zone=public --list-all
```

不指定域默认当前域

```
firewall-cmd --list-all
```

![image-20220616105920789](https://img2023.cnblogs.com/blog/1768648/202304/1768648-20230413140048958-974939994.png)

### 18、配置网卡属于哪个域

```
firewall-cmd --zone=home --change-interface=eth0
```

### 19、查看某个网卡属于哪个域

查看eth0属于那个域

```
firewall-cmd --get-zone-of-interface=eth0
```

![image-20220616094819673](https://img2023.cnblogs.com/blog/1768648/202304/1768648-20230413140049244-387502483.png)

例子：

原来的网卡br0在public上，想要切换到trusted域

原来的网卡br0属于public域

![image-20230413135624188](https://img2023.cnblogs.com/blog/1768648/202304/1768648-20230413140049620-2027893167.png)

从public域上移除网卡br0

![image-20230413135653785](https://img2023.cnblogs.com/blog/1768648/202304/1768648-20230413140049933-2090894691.png)

添加网卡br0到新的trusted域中

![image-20230413135821478](https://img2023.cnblogs.com/blog/1768648/202304/1768648-20230413140050239-987821215.png)

### 20、重新加载防火墙

- 重新加载防火墙不会中断用户连接

例如：如果用户连接了某个服务，比如ssh，则该用户还可以连接，直到他自己断开，或者无动作超时断开，新的规则把ssh禁用了，则下次断开就连接不了了，该次并不会断开

```
firewall-cmd --reload
```

### 21、添加富规则

例如：要允许来自地址 172.16.70.81/24 的连接访问 ssh 服务，请运行以下命令：

```
firewall-cmd --zone=public --add-rich-rule='rule family="ipv4" source address="172.16.70.81/24" service name="ssh" log prefix="ssh" level="info" accept' --permanent
```

添加完后需要加载防火墙才生效`firewall-cmd --reload`

![image-20220616104945811](https://img2023.cnblogs.com/blog/1768648/202304/1768648-20230413140050594-795869970.png)

### 22、删除富规则

```
firewall-cmd --zone=public --remove-rich-rule='rule family="ipv4" source address="172.16.70.81/24" service name="ssh" log prefix="ssh" level="info" accept' --permanent
```

删除完后需要加载防火墙才生效`firewall-cmd --reload`

![image-20220616105144703](https://img2023.cnblogs.com/blog/1768648/202304/1768648-20230413140050990-1954615474.png)

PS:一般都是先查看防火墙规则，然后直接复制粘贴，进行删除，这样比较方便

### 23、写在最后

任何修改操作，配置完成后，需要重新装载firewall。或者重新启动firewalld服务。

```
firewall-cmd --reload
service firewalld restart
```

### 24、学习链接

 [如何在 Linux 中配置 firewalld 规则](https://cloud.tencent.com/developer/article/1908296)

[CentOS7防火墙放行或限制指定IP和端口（firewall）](https://www.cnblogs.com/zmqcoding/p/14700374.html)
