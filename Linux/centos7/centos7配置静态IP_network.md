Centos7 配置静态IP

## 1. 查看网卡的名称

使用`ip a`命令查看即可

![image-20220615165934465](https://img2023.cnblogs.com/blog/1768648/202304/1768648-20230404151325724-1295849511.png)

可以看出网卡名称是ens3，则可以推断出网卡配置文件为cat /etc/sysconfig/network-scripts/ifcfg-ens3

## 2. 查看默认网关

如果未修改前已经通过dhcp获取到IP了，此时可以上网，则可以看看网关，使用`ip route show`查看默认网关

![image-20230404142606911](https://img2023.cnblogs.com/blog/1768648/202304/1768648-20230404151326147-1298628062.png)

后续的网关可以配置为这个

## 3. 查看子网掩码

可以通过`ifconfig`查看子网掩码

![image-20230404143131116](https://img2023.cnblogs.com/blog/1768648/202304/1768648-20230404151326564-989649246.png)

## 4. 备份原始的网卡配置文件

备份

```
cp /etc/sysconfig/network-scripts/ifcfg-ens3 /etc/sysconfig/network-scripts/ifcfg-ens3.bak
```

看一下原始的配置文件，原始的配置文件我这里是dhcp获取IP的方式，所以是这样的

```
cat /etc/sysconfig/network-scripts/ifcfg-ens3.bak
```

![image-20220615170157263](https://img2023.cnblogs.com/blog/1768648/202304/1768648-20230404151326966-364001814.png)

原始的文件内容是

TYPE="Ethernet"
PROXY_METHOD="none"
BROWSER_ONLY="no"
BOOTPROTO="dhcp"
DEFROUTE="yes"
IPV4_FAILURE_FATAL="no"
IPV6INIT="yes"
IPV6_AUTOCONF="yes"
IPV6_DEFROUTE="yes"
IPV6_FAILURE_FATAL="no"
IPV6_ADDR_GEN_MODE="stable-privacy"
NAME="ens3"
UUID="216863d0-d514-45fa-aca7-85b8de268123"
DEVICE="ens3"
ONBOOT="yes"

## 5. 修改网卡配置文件

```
vi /etc/sysconfig/network-scripts/ifcfg-ens3
```

#### 修改`BOOTPROTO="dhcp"`为`BOOTPROTO="static"`

添加

#### `IPADDR=172.16.90.52` 这是静态IP

#### `NETMASK=255.255.255.0` 这是子网掩码

#### `GATEWAY=172.16.90.1` 这是网关

#### `DNS1=8.8.8.8` 这是DNS1

#### `DNS2=114.114.114.114` 这是DNS2

最终文件

![image-20220615170521559](https://img2023.cnblogs.com/blog/1768648/202304/1768648-20230404151327365-448654147.png)

内容
TYPE="Ethernet"
PROXY_METHOD="none"
BROWSER_ONLY="no"
BOOTPROTO="static"
DEFROUTE="yes"
IPV4_FAILURE_FATAL="no"
IPV6INIT="yes"
IPV6_AUTOCONF="yes"
IPV6_DEFROUTE="yes"
IPV6_FAILURE_FATAL="no"
IPV6_ADDR_GEN_MODE="stable-privacy"
NAME="ens3"
UUID="216863d0-d514-45fa-aca7-85b8de268123"
DEVICE="ens3"
ONBOOT="yes"
IPADDR=172.16.90.52
NETMASK=255.255.255.0
GATEWAY=172.16.90.1
DNS1=8.8.8.8
DNS2=114.114.114.114

子网掩码、网关可以询问网络管理员，网关也许第2步就获得了

## 6. 重启网络

```
systemctl restart network
```

## 7. 验证


## ![image-20220615170752439](https://img2023.cnblogs.com/blog/1768648/202304/1768648-20230404151327713-1534878512.png) 

参考 ：

[centos7设置静态IP地址](https://www.cnblogs.com/congcongdi/p/10149925.html)

[CentOS下的网络配置文件说明](https://www.cnblogs.com/kuliuheng/p/3208941.html)

