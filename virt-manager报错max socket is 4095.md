virt-manager报错max socket is 4095

## 写在前面

本次实验的系统是**ARM**架构的**centos7**

[root@localhost txh]# cat /etc/os-release
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

[root@localhost txh]

图示

![image-20230131163626999](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20230131163626999.png)

## 问题描述

启动virt-manager后报错：Error polling connection 'qemu:///system': internal error: Socket 8442 can't be handled (max socket is 4095)

截图

![image-20230131152821554](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20230131152821554.png)

## 解决办法：



升级libvirt版本，步骤

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
./configure --prefix=/usr --localstatedir=/var --sysconfdir=/etc
make -j
make install -j
ldconfig
libvirtd --version
```

另外说明

如果是ARM架构的机器，出现

Warning: Failed to setup UEFI for AArch64: Did not findany UEFI binary path for arch aarch64'Install options are limited.

图示

![image-20230131163348591](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20230131163348591.png)

运行命令即可

```
yum install AAVMF-20180508-6.gitee3198e672e2.el7.noarch
```

## 参考链接

https://bbs.huaweicloud.com/forum/thread-40762-1-1.html

https://blog.csdn.net/sukysun125/article/details/89402782