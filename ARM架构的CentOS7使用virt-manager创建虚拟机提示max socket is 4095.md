ARM架构的CentOS7使用virt-manager创建虚拟机提示max socket is 4095

## 问题描述

ARM架构的CentOS7安装libvirt、qemu-kvm和virt-manager后，启动virt-manager后报错：Error polling connection 'qemu:///system': internal error: Socket 8442 can't be handled (max socket is 4095)

具体报错

Error polling connection 'qemu:///system': internal error: Socket 8442 can't be handled (max socket is 4095)

Traceback (most recent call last):
  File "/usr/share/virt-manager/virtManager/engine.py", line 389, in _handle_tick_queue
    conn.tick_from_engine(**kwargs)
  File "/usr/share/virt-manager/virtManager/connection.py", line 1473, in tick_from_engine
    raise e  # pylint: disable=raising-bad-type
libvirtError: internal error: Socket 8442 can't be handled (max socket is 4095)

截图

![image-20230131152821554](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20230131152821554.png)

## 系统信息

[root@localhost ~]# cat /etc/os-release
NAME="CentOS Linux"
VERSION="7 (AltArch)"
ID="centos"
ID_LIKE="rhel fedora"
VERSION_ID="7"
PRETTY_NAME="CentOS Linux 7 (AltArch)"
ANSI_COLOR="0;31"
CPE_NAME="cpe:/o:centos:centos:7"
HOME_URL="https://www.centos.org/"
BUG_REPORT_URL="https://bugs.centos.org/"

CENTOS_MANTISBT_PROJECT="CentOS-7"
CENTOS_MANTISBT_PROJECT_VERSION="7"
REDHAT_SUPPORT_PRODUCT="centos"
REDHAT_SUPPORT_PRODUCT_VERSION="7"

[root@localhost ~]#

图示

![image-20230201101931596](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20230201101931596.png)

可以看到是**ARM架构的CentOS7系统**

## 情况复现

①新安装的ARM架构的CentOS7新机器，下载安装libvirt qemu-kvm virt-manager组件

```
yum install -y libvirt qemu-kvm virt-manager
```

②启动libvirtd

```
systemctl start libvirtd && systemctl enable libvirtd
```

③查看libvirtd的版本

```
libvirtd --version
```

![image-20230201102723506](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20230201102723506.png)

可以看到是4.5.0版本

④启动virt-manager

```
virt-manager
```

这样就看到了之前提到的问题

## 原因

可能是libvirtd太旧了，需要升级一下

## 解决

```
yum install -y wget 
```

```
wget https://libvirt.org/sources/libvirt-4.7.0.tar.xz --no-check-certificate
```

```
yum install -y gcc libnl-devel libxml2-devel yajl-devel device-mapper-devel libpciaccess-devel libnl3-devel netcf-devel gnutls gnutls-devel yajl-devel perl-ExtUtils-Embed AAVMF-20180508-6.gitee3198e672e2.el7.noarch
```

```
xz -d libvirt-4.7.0.tar.xz 
```

```
tar -xvf libvirt-4.7.0.tar
```

```
cd libvirt-4.7.0
```

```
./configure --prefix=/usr --localstatedir=/var --sysconfdir=/etc
```

```
make -j
```

```
make install -j
```

```
ldconfig
```

```
libvirtd --version
```

![image-20230201121345552](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20230201121345552.png)

可以看到右4.5.0升级到了4.7.0

重新启动libvirtd

```
systemctl daemon-reload 
systemctl restart libvirtd
```

启动virt-manager查看，问题已经解决

```
virt-manager
```

![image-20230201121555861](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20230201121555861.png)

## 总结

ARM架构的CentOS7使用virt-manager创建虚拟机提示max socket is 4095，解决该错误只需要升级libvirt到4.7.0即可，升级方式为手动编译

## 参考链接

https://bbs.huaweicloud.com/forum/thread-40762-1-1.html

https://blog.csdn.net/sukysun125/article/details/89402782