centos7安装alertmanager

### 1、下载

地址：https://prometheus.io/download/

![image-20220728102900309](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220728102900309.png)

例如 [0.24.0版本](https://github.com/prometheus/alertmanager/releases/download/v0.24.0/alertmanager-0.24.0.linux-amd64.tar.gz)(直接wget下载或者Windows下载好后拷贝进去Linux)

```shell
wget https://github.com/prometheus/alertmanager/releases/download/v0.24.0/alertmanager-0.24.0.linux-amd64.tar.gz
```

#### 2、启动

```shell
tar -zxvf alertmanager-0.24.0.linux-amd64.tar.gz
cd alertmanager-0.24.0.linux-amd64
./alertmanager
```

#### 3、验证启动

启动后，通过http://172.16.131.28:9093/访问（注意替换自己的IP）

如果无法访问，先在主机中看一下是否可以访问：

```shell
curl http://127.0.0.1:9093
```

如果主机可以，则可能是防火墙的问题，关闭防火墙或者添加防火墙端口9093，然后再访问浏览器看看

```shell
systemctl stop firewalld
```

如果主机不可以，那重来一遍吧，正常访问9093端口，应该能看到下面的网页

![image-20220728105749984](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220728105749984.png)

Alert菜单下可以查看Alertmanager接收到的告警内容。

Silences菜单下则可以通过UI创建静默规则

Status菜单，可以看到当前系统的运行状态以及配置信息。

#### 4、配置服务

```shell
vi /etc/systemd/system/alertmanager.service
```

然后输入

```ini
[Unit]
Description=AlertManager Service

[Service]
Restart=always
ExecStart=/home/alertmanager-0.24.0.linux-amd64/alertmanager --config.file=/home/alertmanager-0.24.0.linux-amd64/alertmanager.yml

[Install]
WantedBy=multi-user.target
```

注意ExecStart后面的命令，需要是你真正的alertmanager可执行文件的地址

然后启动服务，配置开机自启动

```shell
systemctl daemon-reload && systemctl start alertmanager && systemctl enable alertmanager
```

#### 5、与Prometheus集成

修改Prometheus的配置文件，配置alertmanager的地址进去

```shell
vi /home/prometheus-2.37.0.linux-amd64/prometheus.yml
```

![image-20220728111503274](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220728111503274.png)

主要就是这么一段

# Alertmanager configuration
```shell
alerting:
  alertmanagers:
    - static_configs:
        - targets: ['localhost:9093']
          # - alertmanager:9093
```

