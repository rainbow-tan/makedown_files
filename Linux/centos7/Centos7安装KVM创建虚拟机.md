Centos7安装KVM创建虚拟机

一、安装前准备工作

1.安装KVM

\# yum -y install qemu-kvm libvirt virt-install bridge-utils

2.验证模块是否加载成功，如下图表示加载成功：

\# lsmod | grep kvm

![img](https://pic4.zhimg.com/80/v2-82a9062605dbb202d0f9849edd26ea3f_720w.png)

3.启动虚拟化和使能开机项

\# systemctl start libvirtd

\# systemctl enable libvirtd

\# systemctl list-unit-files |grep libvirtd.service

![img](https://pic3.zhimg.com/80/v2-a606c29322b22e7bf561dce5d44ee846_720w.png)

4.kvm配置网桥

方法一：

\# brctl addbr br0

\# brctl addif br0 enp1s0f0

\# ifconfig enp1s0f0 0.0.0.0

\# ifconfig br0 10.10.10.4/24 up

验证测试：

![img](https://pic2.zhimg.com/80/v2-0c2114a2df980f3c86bcad7f7bc0d7e1_720w.jpg)

方法二：

方式一只是暂时地设置，重启后配置失效，所以需要使用方法二进行配置文件永久配置。

首先需要在/etc/sysconfig/network-scripts/目录下对网口enp1s0f0的ifcfg-enp1s0f0配置文件进行备份。

创建ifcfg-br0文件，内容如下：

![img](https://pic2.zhimg.com/80/v2-6fc9dc4b0ba6ec1d5b710aec5fd30c09_720w.jpg)

修改原来的ifcfg-enp1s0文件，内容如下：

![img](https://pic3.zhimg.com/80/v2-8f273b7a8bf9ce2e3e3ddc5cc0cad7ba_720w.jpg)

重启网络服务

\# systemctl restart network

验证测试：

![img](https://pic1.zhimg.com/80/v2-d49a55a261c64499ff69197cf4bc800c_720w.jpg)

二、安装KVM虚拟机

1.上传准备好的系统镜像安装文件，本文中使用的是Centos7 Minimal。

![img](https://pic1.zhimg.com/80/v2-daa67117c3e2b269ce017dd4d13042b4_720w.png)

2.创建虚拟机存放目录

\# mkdir /root/kvm

3.运行virt-install创建虚拟机

virt-install --name=kvm --ram=2048 --vcpus=2 --disk path=/root/kvm/centos01.img,size=20,bus=virtio --accelerate --cdrom /root/iso/CentOS-7-x86_64-Minimal-1810.iso --vnc --vncport=5910 --vnclisten=0.0.0.0 --network bridge=br0,model=virtio --noautoconsole

![img](https://pic1.zhimg.com/80/v2-aa4ee73ffe4d7c54ceb502f868350130_720w.png)

virt-install选项，下面列出一些常用的，具体可以通过--help查看

1. --name #虚拟机名称
2. --ram #分配给虚拟机的内存，单位MB
3. --vcpus #分配给虚拟机的cpu个数
4. --cdrom #指定CentOS镜像ISO文件路径
5. --disk #指定虚拟机raw文件路径
6. --size #虚拟机文件大小，单位GB
7. --bus #虚拟机磁盘使用的总线类型，为了使虚拟机达到好的性能，这里使用virtio
8. --cache #虚拟机磁盘的cache类型
9. --network bridge #指定桥接网卡
10. --model #网卡模式，这里也是使用性能更好的virtio

4.用VNC连接进行安装，这里连接密码默认是“123456”

![img](https://pic3.zhimg.com/80/v2-40f48bc1852e65eea309a819e4f79fae_720w.jpg)

5.连接后进入安装界面，直接按照正常安装流程安装即可

![img](https://pic1.zhimg.com/80/v2-d11a6c72b44f44bf221f3241afebfc44_720w.jpg)

![img](https://pic4.zhimg.com/80/v2-1d761edde0def68f3c947fa875acec83_720w.jpg)

![img](https://pic1.zhimg.com/80/v2-bd79a8aa170137249ac262b8c439d058_720w.jpg)

6.宿机上列出虚拟机

\# virsh list --all

![img](https://pic4.zhimg.com/80/v2-39d941ce24267d757d78d95befcc4f97_720w.png)

7.启动虚拟机并查看状态

\# virsh start kvm

![img](https://pic3.zhimg.com/80/v2-a956074efe988fdd40cf8dbb3b3c669e_720w.jpg)

8.设置自启动

\# virsh autostart kvm

![img](https://pic4.zhimg.com/80/v2-580ba4c5116b012a098710eb334bc173_720w.png)

9.VNC重新连接虚拟机，账号密码为安装过程中配置的账号密码

![img](https://pic1.zhimg.com/80/v2-c3b58b34f56af7d5246677ae178298cc_720w.jpg)

10.手动配置IP进行连通性测试

![img](https://pic2.zhimg.com/80/v2-7f3dd02381cd6f4278bf5bae54373399_720w.jpg)

11.安装测试完毕

12.其它常用的virsh命令

关机：virsh shutdown kvm

虚拟机名称修改：virsh domrename kvm kvm1

虚拟机修改磁盘文件名称：mv /root/kvm/centos01.img /root/kvm/centos001.img

修改配置文件：virsh edit kvm

开机：virsh start kvm

强制关闭虚拟机：virsh destroy kvm

删除定义虚拟机：virsh undefine kvm



学习：https://zhuanlan.zhihu.com/p/321968250