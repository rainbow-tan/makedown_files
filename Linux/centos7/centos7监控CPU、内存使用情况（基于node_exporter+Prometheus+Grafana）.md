centos7监控CPU、内存使用情况（基于node_exporter+Prometheus+Grafana）

#### 1、下载、安装、启动

参考以下链接，下载，安装并启动各服务

[centos7安装Grafana ](https://www.cnblogs.com/rainbow-tan/p/16501388.html)

[centos7安装Prometheus](https://www.cnblogs.com/rainbow-tan/p/16501304.html)

[centos7安装node exporter](https://www.cnblogs.com/rainbow-tan/p/16499529.html)

#### 2、Prometheus配置node_exporter

修改Prometheus的配置文件，添加node exporter节点

```shell
vi /home/prometheus-2.37.0.linux-amd64/prometheus.yml
```

添加

```shell
job_name: "node_exporter_localhost"     
  static_configs:    
    - targets: ["localhost:9100"]
```

![image-20220721140542185](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220721140542185.png)

重启Prometheus服务

```shell
systemctl restart prometheus
```

验证是否添加成功

打开：http://172.16.131.28:9090/graph （注意替换自己的IP）

输入up，回车，如果看到之前配置的node_exporter_localhost服务，则表示配置完成

![image-20220721185739374](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220721185739374.png)

这样配置后，Prometheus就可以收集到node exporter的数据了

#### 3、配置grafana

##### grafana配置数据源，把Prometheus作为grafana的数据源

地址：http://172.16.131.28:3000/ 登陆（注意替换自己的IP）（默认用户名密码admin/admin）

选择data sources

![image-20220721185952350](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220721185952350.png)

选择Prometheus

![image-20220721190010663](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220721190010663.png)

配置Prometheus的地址 http://172.16.131.28:9090/  （注意替换自己的IP）然后保存

![image-20220721190106206](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220721190106206.png)

配置仪表盘模板

我是小白，我直接去官网选择一个觉得可以的[仪表板模板](https://grafana.com/grafana/dashboards/?pg=hp&plcmt=lt-box-dashboards&dataSource=prometheus&collector=nodeexporter) 例如Node Exporter Full

![image-20220722103708404](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220722103708404.png)

点进[Node Exporter Full](https://grafana.com/grafana/dashboards/1860)进去，可以看到ID是1860

![image-20220722103805091](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220722103805091.png)

导入1860Node Exporter Full 

点击import

![image-20220722104202285](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220722104202285.png)

输入1860，点击load

![image-20220722104244021](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220722104244021.png)

选择数据源，之前我们导入的Prometheus数据源，输入名称，选择文件夹，文件夹之前没创建，所有默认只有General,ID可以自己点击change uid改，也可以不改，然后点击import

![image-20220722104335828](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220722104335828.png)

然后就出来了

![image-20220722104627389](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220722104627389.png)

以后可以在这找到

![image-20220722104754532](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220722104754532.png)

