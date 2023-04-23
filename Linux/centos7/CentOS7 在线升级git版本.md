CentOS7 在线升级git版本

## CentOS7上的Git版本默认是1.8，感觉有点低，需要升级一下

```shell
#git --version
git version 1.8.3.1
```

## 执行以下命令升级

```
yum remove git

yum install -y https://packages.endpointdev.com/rhel/7/os/x86_64/endpoint-repo.x86_64.rpm

yum -y install git
```

## 验证

```
#git --version
git version 2.36.0
```

补充：安装脚本

自己写了一个sh安装脚本，大家可以试一试

```sh
yum remove -y git &&
mkdir -p /home/git-2.38-package &&
cd /home/git-2.38-package &&
yum install -y wget &&
wget https://mirrors.edge.kernel.org/pub/software/scm/git/git-2.38.0.tar.gz --no-check-certificate &&
yum install -y curl-devel expat-devel openssl-devel gcc-c++ &&
tar -zxvf git-2.38.0.tar.gz &&
cd git-2.38.0 &&
./configure --prefix=/usr/local/git &&
make -j8 &&
make install -j8 &&
echo "export PATH=$PATH:/usr/local/git/bin" >> /etc/profile &&
source /etc/profile &&
git --version
```
