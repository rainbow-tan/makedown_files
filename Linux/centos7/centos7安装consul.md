centos7安装consul

## 1、在线下载consul

官网地址：https://www.consul.io/downloads

![image-20220722154449971](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220722154449971.png)

在线安装consul

```shell
sudo yum install -y yum-utils && sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/RHEL/hashicorp.repo && sudo yum -y install consul
```

## 2、修改consul配置文件

vi /etc/consul.d/server.json	（新建server.json配置文件）

```json
{
    "server": true,
    "bootstrap": true,
    "advertise_addr": "{{ GetInterfaceIP \"eth0\" }}",
    "ui_config": {
        "enabled": true
    },
    "data_dir": "/etc/consul",
    "log_level": "INFO",
    "addresses": {
        "http": "0.0.0.0"
    },
    "ports": {
        "http": 8500
    }
}
```



## 3、启动consul

```shell
systemctl start consul && systemctl enable consul
```

![image-20221107101636110](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20221107101636110.png)

打开网址查看：http://172.16.131.28:8500/ui/ (需要替换自己的IP)（访问不了时，关闭防火墙或者防火墙添加端口）

![image-20220722155450691](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220722155450691.png)

#### 3、服务注册与反注册

注册服务与反注册：https://www.cnblogs.com/wangguishe/p/15605006.html

#### 4、与Prometheus集成

vim /home/prometheus-2.37.0.linux-amd64/prometheus.yml

![image-20220825155909051](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220825155909051.png)

```yaml
  - job_name: "nano"
    consul_sd_configs:
      - server: "127.0.0.1:8500"
        services: [ "nano-vm-exporter" ]
    relabel_configs:
      - regex: __meta_consul_service_metadata_(.+)
        action: labelmap
    honor_labels: true
```

说明：job name就叫nano，从127.0.0.1:8500抓取数据，127.0.0.1:8500就是consul的地址，发现服务名称叫nano-vm-exporter下的服务，替换标签信息，保留原始标签