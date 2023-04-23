# Navicat使用HTTP通道连接远程服务器的SQLite文件（包含服务端搭建http服务）

## 基本原理

数据库端口没开放外网访问的时候，Navicat在外网无法访问数据库。

可以通过在服务器上运行PHP服务，配合官方的ntunnel_sqlite.php脚本进行连接数据库。

ntunnel_sqlite.php脚本可以连接SQLite数据库并执行SQL语句，因为它们都在内网和PHP支持SQLite。

虽然Navicat无法连接上SQLite，但是Navicat对数据库所有的查询可以让PHP代为查询，然后把结果返回给Navicat。

所以把一个php脚本放到服务器上，就可以让Navicat间接连接数据库，对数据库进行操作了。

## 搭建环境

### 一、PHP环境

#### 1、安装php5.x环境

```shell
yum install -y php
```

```shell
yum install -y php-mysql php-fpm php-cli php-dba php-embedded php-gd php-common php-bcmatch php-enchant php-devel
```

#### 2、启动php-fpm服务

```shell
systemctl start php-fpm
systemctl enablephp-fpm
```

#### 3、验证php服务已经安装成功并启动了php-fpm服务

输入`php -v` 如果安装成功，会显示版本信息

输入 `netstat -nulpt | grep php-fpm`如果启动php-fpm服务成功，会显示监听在9000端口

![image-20220823143130860](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220823143130860.png)

### 二、Nginx环境

#### 1、安装nginx环境

我的机器中已经有了nginx环境，因此这里不介绍nginx环境的安装，应该`yum install nginx`就行

#### 2、拷贝ntunnel_sqlite.php文件到Linux机器中

ntunnel_sqlite.php文件位置：

在Navicat安装位置中可以找到ntunnel_sqlite.php文件，其他两个类似文件是支持MySQL和pgsql的

![image-20220823102309787](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220823102309787.png)

这里以拷贝到`/nano`文件夹为例，拷贝进去以后我直接给了个777权限

![image-20220823144256832](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220823144256832.png)

#### 3、配置nginx服务

vim /etc/nginx/conf.d/shar.conf

```nginx
server {
  listen 8011 default_server ;
  client_max_body_size 10240M;
  server_name _;
  server_tokens off;
  add_header X-Frame-Options SAMEORIGIN;


  location / {
	root /nano ;
  }

  location ~ \.php$ {		
  	root           /nano;		
  	fastcgi_pass   127.0.0.1:9000;		
  	fastcgi_index  index.php;		
  	fastcgi_param  SCRIPT_FILENAME $document_root$fastcgi_script_name;		
  	include        fastcgi_params;	
  }

}
```

**配置文件说明：**监听8011端口，访问以.php结束的文件会走fastcgi服务，这个就是解析php，代理到之前启动的php-fpm服务，端口就是9000，不太懂php,简单理解就是这样子了

然后重启nginx服务即可，如果是新安装的nginx服务，直接启动就可以了

重启nginx服务 `nginx -s reload`或者启动nginx服务  `systemctl startnginx`

nginx的配置比较简单，就粗略写了一下

### 三、验证http服务已经搭好

#### 1、验证搭建完成

完成了一和二就搭建玩了http服务，现在去验证一下，直接访问 http://172.17.130.40:8011/ntunnel_sqlite.php （换自己的IP）

出现下面就代表搭建成功了

![image-20220823145145325](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220823145145325.png)

#### 2、验证可以连接sqlite

在Database File框中输入服务器上sqlite的地址，注意是相对于ntunnel_sqlite.php文件所在位置的路径，点击Test Connection测试

![image-20220823145538099](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220823145538099.png)

出现Connection Success!就表示连接成功了，接下来就可以去Navicat里连接了，失败情况我也没遇到，就没有错误列举了。

### 四、Navicat使用http方式远程连接sqlite

#### 1、创建sqlite连接面板

![image-20220823145906290](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220823145906290.png)

#### 2、填写连接名和第三步一样的相对路径

![image-20220823145821881](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220823145821881.png)

#### 3、添加http路径，也是第三部的路径

![image-20220823145731552](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220823145731552.png)

#### 4、测试连接

![image-20220823150105160](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220823150105160.png)

参考：

[Navicat使用HTTP通道连接远程服务器的SQLite文件（下一朵云）](https://www.orcy.net.cn/1944.html)

[Navicat使用HTTP通道远程连接SQLite](https://ya2.top/articles/navicat%E4%BD%BF%E7%94%A8http%E9%80%9A%E9%81%93%E8%BF%9C%E7%A8%8B%E8%BF%9E%E6%8E%A5sqlite/)

[CentOS7系统中php安装配置](https://zhuanlan.zhihu.com/p/142586726)