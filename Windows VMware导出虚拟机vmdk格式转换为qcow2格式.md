Windows VMware导出虚拟机vmdk格式转换为qcow2格式

### 1、导出虚拟机vmdk格式

1. 右键要导出的虚拟机
2. 点击`管理`
3. 点击`克隆`
4. 选择`虚拟机中的当前状态`
5. 选择`创建完整克隆`



![image-20220802094047724](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220802094047724.png)

![image-20220802094213611](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220802094213611.png)

![image-20220802094238870](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220802094238870.png)

导出后是这样子的，会有很多文件

![image-20220802094521124](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220802094521124.png)

### 2、合成所有导出的文件为一个大文件

在VMware的安装目录，找到`vmware-vdiskmanager.exe`文件，通过该可执行文件，整合所有的零散文件为一个大文件

![image-20220802094710568](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220802094710568.png)

```
vmware-vdiskmanager.exe -r "C:\Users\dell\Desktop\新建文件夹\Ubuntu 64 位-cl1.vmdk" -t 0 "C:\Users\dell\Desktop\新建文件夹 (2)\all.vmdk"
```

执行命令分解

-r 参数后面是导出的(.*)cl1.vmdk文件路径

最后是要保存的位置和文件名

![image-20220802094953342](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220802094953342.png)

合并后是这样的，只有一个比较大的文件

![image-20220802095147195](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220802095147195.png)

### 3、安装qemu

去[官网](https://qemu.weilnetz.de/w64/)下载qemu（根据自己的操作系统下载即可）

![image-20220802095353641](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220802095353641.png)

安装根据提示即可

### 4、合并为qcow2文件

```shell
qemu-img.exe convert -p -f vmdk -O qcow2 "C:\Users\dell\Desktop\新建文件夹 (2)\all.vmdk" "C:\Users\dell\Desktop\新建文件夹 (2)\ubuntu22.04.qcow2"
```

![image-20220802104314950](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220802104314950.png)

参考：https://blog.csdn.net/mofeimo110/article/details/121914841