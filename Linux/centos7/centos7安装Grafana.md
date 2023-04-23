centos7安装Grafana

#### 1、下载

下载指引 ：https://grafana.com/grafana/download?pg=get&plcmt=selfmanaged-box1-cta1

![image-20220721112223236](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220721112223236.png)

```shell
wget https://dl.grafana.com/enterprise/release/grafana-enterprise-9.2.3-1.x86_64.rpm && yum install grafana-enterprise-9.2.3-1.x86_64.rpm
```

#### 2、启动

```shell
systemctl daemon-reload && systemctl start grafana-server && systemctl enable grafana-server
```

#### 3、验证

打开浏览器，输入 http://172.16.131.28:3000/login (端口为3000)，打开Grafana控制面板， 初始默认账号和密码均为 admin，初次登录会提示修改密码，也可以不修改密码。

如果无法访问，先在主机中看一下是否可以访问：

```shell
curl http://127.0.0.1:3000/login
```

如果主机可以，则可能是防火墙的问题，关闭防火墙或者添加防火墙端口3000，然后再访问浏览器看看

```shell
systemctl stop firewalld
```

如果主机不可以，那重来一遍吧

![image-20220721113557932](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220721113557932.png)

#### 4、简单使用

##### 4.1、添加Prometheus数据源

（1）使用默认密码admin/admin登陆 http://172.17.140.17:3000/login (注意替换自己的IP) 

（2）点击![img](C:\Users\dell\AppData\Local\Temp\SGPicFaceTpBq\17796\03D174F1.png)图标，找到data sources，然后点击data sources

![image-20220826100526294](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220826100526294.png)

(3)点击Prometheus，添加Prometheus数据源

![image-20220826100709840](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220826100709840.png)

（4）配置Prometheus网址和保存数据源

![image-20220826101726948](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220826101726948.png)

（5）查看所有添加的数据源

![image-20220826102315298](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220826102315298.png)

##### 4.2、导入模板，可视化数据

##### 当然用熟悉后，可以自己编写，自定义自己的可视化界面

（1）配置仪表盘模板

我是小白，我直接去官网选择一个觉得可以的[仪表板模板](https://grafana.com/grafana/dashboards/?pg=hp&plcmt=lt-box-dashboards&dataSource=prometheus&collector=nodeexporter) 例如Node Exporter Full

![image-20220722103708404](https://img2022.cnblogs.com/blog/1768648/202207/1768648-20220722105116275-269255811.png)

点进[Node Exporter Full](https://grafana.com/grafana/dashboards/1860)进去，可以看到ID是1860

![image-20220722103805091](https://img2022.cnblogs.com/blog/1768648/202207/1768648-20220722105116796-433383520.png)

（2）导入1860Node Exporter Full

点击import

![image-20220722104202285](https://img2022.cnblogs.com/blog/1768648/202207/1768648-20220722105117202-1187405458.png)

输入1860，点击load

![image-20220722104244021](https://img2022.cnblogs.com/blog/1768648/202207/1768648-20220722105117512-500726343.png)

选择数据源，之前我们导入的Prometheus数据源，输入名称，选择文件夹，文件夹之前没创建，所有默认只有General,ID可以自己点击change uid改，也可以不改，然后点击import

![image-20220722104335828](https://img2022.cnblogs.com/blog/1768648/202207/1768648-20220722105117849-979474415.png)

然后就出来了

![image-20220722104627389](https://img2022.cnblogs.com/blog/1768648/202207/1768648-20220722105118200-602515396.png)

以后可以在这找到

![image-20220722104754532](https://img2022.cnblogs.com/blog/1768648/202207/1768648-20220722105118603-898731655.png)