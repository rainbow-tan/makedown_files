centos7安装mysql5.7

## 环境搭建、清理旧安装包

进入到目录 /usr/local/ 中

```
cd /usr/local/
```

创建目录 /usr/local/tools，如果有则忽略

```
mkdir -p tools
```

创建 /usr/local/mysql 目录，如果已存在则忽略

```
mkdir -p mysql
```

进入到目录 /usr/local/tools 中

```
cd tools/
```

查看系统中是否已安装 MySQL 服务

```
rpm -qa | grep mysql
或
yum list installed | grep mysql
```

如果已安装则删除 MySQL 及其依赖的包

```
yum -y remove mysql-libs.x86_64
```

## 安装yum源

```
wget http://repo.mysql.com/mysql57-community-release-el7-8.noarch.rpm
```

```
rpm -ivh mysql57-community-release-el7-8.noarch.rpm
```

```
rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2022
```

## 安装mysql-server

```
yum install mysql-server -y
```

## 启动mysqld

```
systemctl start mysqld && systemctl enable mysqld
```

## 获取mysql密码

```
grep 'temporary password' /var/log/mysqld.log
```

![image-20221223121244475](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20221223121244475.png)

## 登陆mysql修改密码

`mysql -u root -p`输入密码后登陆

然后修改密码

```
SET PASSWORD = PASSWORD('your new password');
```

设置密码永不过期

```
ALTER USER 'root'@'localhost' PASSWORD EXPIRE NEVER; 
```

刷新

```
 flush privileges;
```

## 设置root可远程连接

`mysql -u root -p`输入密码后登陆

```
use mysql;
select host, user from user;
update user set host = '%' where user = 'root';
select host, user from user;
flush privileges;
```

## 测试连接

![image-20221223122130146](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20221223122130146.png)

## 学习链接

https://blog.csdn.net/ic_xcc/article/details/121425926

https://my.oschina.net/scottCoder/blog/857378# 

https://blog.csdn.net/iiiiiilikangshuai/article/details/100905996