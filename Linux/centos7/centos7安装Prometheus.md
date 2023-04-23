centos7安装Prometheus

#### 1、下载

地址：https://prometheus.io/download/#prometheus

![image-20220721102441465](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220721102441465.png)

选择一个，然后直接下载，例如 [2.37版本](https://github.com/prometheus/prometheus/releases/download/v2.37.0/prometheus-2.37.0.linux-amd64.tar.gz)，下载后拷贝到centos7机器中，也可以直接在centos7中使用wget命令直接下载

```shell
wget https://github.com/prometheus/prometheus/releases/download/v2.37.0/prometheus-2.37.0.linux-amd64.tar.gz
```

备注：[其他版本地址](https://github.com/prometheus/prometheus/releases)

#### 2、解压和运行

```shell
tar -zxvf prometheus-2.37.0.linux-amd64.tar.gz
cd prometheus-2.37.0.linux-amd64
./prometheus
```

#### 3、浏览器查看

地址：http://172.16.131.28:9090/graph （需要替换自己的IP）

如果无法访问，先在主机中看一下是否可以访问：

```
curl http://172.16.131.28:9090/graph
```

如果主机可以，则可能是防火墙的问题，关闭防火墙或者添加防火墙端口9090，然后再访问浏览器看看

```
systemctl stop firewalld
```

如果主机不可以，那重来一遍吧

#### 4、配置服务

```shell
vi /etc/systemd/system/prometheus.service
```

然后输入

```ini
[Unit]
Description=Prometheus Service

[Service]
ExecStart=/home/prometheus-2.37.0.linux-amd64/prometheus --config.file=/home/prometheus-2.37.0.linux-amd64/prometheus.yml
Restart=always

[Install]
WantedBy=multi-user.target
```

注意ExecStart后面的命令，需要是你真正的prometheus可执行文件的地址

然后启动服务，配置开机自启动

```shell
systemctl daemon-reload && systemctl start prometheus && systemctl enable prometheus
```

