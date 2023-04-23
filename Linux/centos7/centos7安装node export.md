centos7安装node export

#### 1、下载

地址:https://prometheus.io/download/#node_exporter

![image-20220720185445593](https://img2022.cnblogs.com/blog/1768648/202208/1768648-20220825163828429-899812697.png)

选择一个，然后直接下载，例如 [1.3.1版本](https://github.com/prometheus/node_exporter/releases/download/v1.3.1/node_exporter-1.3.1.linux-amd64.tar.gz)，下载后拷贝到centos7机器中，也可以直接在centos7中使用wget命令直接下载

```shell
wget https://github.com/prometheus/node_exporter/releases/download/v1.3.1/node_exporter-1.3.1.linux-amd64.tar.gz
```

#### 2、解压，运行

```shell
tar -zxf node_exporter-1.3.1.linux-amd64.tar.gz
cd node_exporter-1.3.1.linux-amd64
./node_exporter
```

#### 3、浏览器查看

地址:http://172.16.131.28:9100/（注意替换自己的IP）

![image-20220720190722730](https://img2022.cnblogs.com/blog/1768648/202208/1768648-20220825163828755-1692925870.png)

如果无法访问，先在主机中看一下是否可以访问：

```
curl http://127.0.0.1:9100/metrics 
```

如果主机可以，则可能是防火墙的问题，关闭防火墙或者添加防火墙端口9100，然后再访问浏览器看看

```
systemctl stop firewalld
```

如果主机不可以，那重来一遍吧

#### 4、配置服务

如果想配置为服务，则可以

```shell
vi /etc/systemd/system/node_exporter.service
```

然后输入

```ini
[Unit]
Description=Node Exporter
Wants=network.target
After=network.target

[Service]
ExecStart=/home/node_exporter-1.3.1.linux-amd64/node_exporter
Restart=always

[Install]
WantedBy=multi-user.target
```

注意ExecStart后面的命令，需要是你真正的node_exporter可执行文件的地址

然后启动服务，配置开机自启动

```shell
systemctl daemon-reload && systemctl start node_exporter && systemctl enable node_exporter
```

#### 5、与Prometheus集成

vim /home/prometheus-2.37.0.linux-amd64/prometheus.yml

![image-20220825161224435](https://img2022.cnblogs.com/blog/1768648/202208/1768648-20220825163829093-1900816612.png)

```ini
  - job_name: "node exporter"
    metrics_path: '/metrics'
    scheme: 'http'
    static_configs:
      - targets: ["localhost:9100"]
```

Prometheus页面验证：

![image-20220825161253452](https://img2022.cnblogs.com/blog/1768648/202208/1768648-20220825163829514-1582015466.png)

#### 6、优化：脚本安装

上面的可以不care直接运行该脚本即可，但未与Prometheus集成

```shell
mkdir -p /etc/node-exporter &&
cd /etc/node-exporter &&
yum install -y wget tar &&
wget https://github.com/prometheus/node_exporter/releases/download/v1.4.0/node_exporter-1.4.0.linux-amd64.tar.gz &&
tar -zxvf node_exporter-1.4.0.linux-amd64.tar.gz &&
cat > /etc/systemd/system/node_exporter.service << EOF &&
[Unit]
Description=node_exporter
After=network.target
[Service]
Type=simple
User=root
ExecStart=/etc/node-exporter/node_exporter-1.4.0.linux-amd64/node_exporter
Restart=always
[Install]
WantedBy=multi-user.target
EOF
systemctl daemon-reload &&
systemctl start node_exporter &&
systemctl enable node_exporter &&
systemctl start firewalld &&
systemctl enable firewalld &&
firewall-cmd --permanent --add-port=9100/tcp &&
firewall-cmd --reload &&
systemctl stop firewalld &&
systemctl disable firewalld
```
