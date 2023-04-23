centos安装python39(其他版本也是一样的流程)

### 1、下载python39

可以去官网下载，也可以去华为云下载

华为云： https://mirrors.huaweicloud.com/python/3.9.0/

![1653980125528](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\1653980125528.png)

我下载的是python3.9.0 https://mirrors.huaweicloud.com/python/3.9.0/Python-3.9.0.tgz

### 2、拷贝到Linux环境（当然也可以直接在Linux环境使用wget直接下载）

一般先安装一下依赖，不然编译会有问题

```shell
sudo yum -y install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gdbm-devel db4-devel libpcap-devel xz-devel libffi-devel
```

### 3、安装

解压

```shell
tar -zxvf Python-3.9.0.tgz
```

配置安装路径

```shell
cd Python-3.9.0/

./configure prefix=/usr/local/python39
```

编译

```shell
make && make install
```

添加软链接

```shell
ln -s /usr/local/python39/bin/python3 /usr/bin/python3

ln -s /usr/local/python39/bin/pip3 /usr/bin/pip3
```

### 4、安装验证

```shell
cd /home

python3
```

成功截图

![1653980572005](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\1653980572005.png)




