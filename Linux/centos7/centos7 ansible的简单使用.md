# 0、介绍

ansible是新出现的自动化运维工具，基于Python开发，集合了众多运维工具（puppet、cfengine、chef、func、fabric）的优点，实现了批量系统配置、批量程序部署、批量运行命令等功能。
ansible是基于模块工作的，本身没有批量部署的能力。真正具有批量部署的是ansible所运行的模块，ansible只是提供一种框架。
主要包括：
    (1)、连接插件connection plugins：负责和被监控端实现通信；
    (2)、host inventory：指定操作的主机，是一个配置文件里面定义监控的主机；
    (3)、各种模块核心模块、command模块、自定义模块；
    (4)、借助于插件完成记录日志邮件等功能；
    (5)、playbook：剧本执行多个任务时，非必需可以让节点一次性运行多个任务。

# 1、安装

```shell
yum install epel-release
yum install ansible
```
# 2、修改ansible配置文件

（1）配置文件

/etc/ansible/ansible.cfg  主配置文件,配置ansible工作特性(一般无需修改)
/etc/ansible/hosts     主机清单(将被管理的主机放到此文件)
/etc/ansible/roles/    存放角色的目录

（2）修改几个地方方便使用

vi /etc/ansible/ansible.cfg

```shell
host_key_checking = False               # 检查对应服务器的host_key，建议取消注释
log_path=/var/log/ansible.log           # 日志文件,建议取消注释
module_name   = command                 # 默认模块,建议取消注释
```

# 3、配置要操作的机器

**ansible需要远程进去要操作的机器，因此需要提前把要控制的机器都配好ssh，使用`ssh root@xxx.xx.xx.xx`可以连接进去**

vim /etc/ansible/hosts

在里面添加要远程控制的机器的IP以及用户名密码

![image-20221012172806417](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20221012172806417.png)

配置完后，先看一看能不能都ping通

执行命令`ansible all -m ping`

![image-20221012172946822](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20221012172946822.png)

# 4、常用命令及模块

## 命令

- 列出所有主机 `ansible all --list`

- ping所有主机 `ansible all -m ping`

## 模块

### copy 模块

功能：将本机的文件拷贝到远程主机中

参数：

- **src**参数 ：用于指定需要copy的文件或目录。

- **dest**参数 ：用于指定文件将被拷贝到远程主机的哪个目录中，dest为必须参数。
- **content**参数 ：当不使用src指定拷贝的文件时，可以使用content直接指定文件内容，src与content两个参数必有其一，否则会报错。
- **force**参数 : 当远程主机的目标路径中已经存在同名文件，并且与ansible主机中的文件内容不同时，是否强制覆盖，可选值有yes和no，默认值为yes，表示覆盖，如果设置为no，则不会执行覆盖拷贝操作，远程主机中的文件保持不变。
- **backup**参数 : 当远程主机的目标路径中已经存在同名文件，并且与ansible主机中的文件内容不同时，是否对远程主机的文件进行备份，可选值有yes和no，当设置为yes时，会先备份远程主机中的文件，然后再将ansible主机中的文件拷贝到远程主机。
- **owner**参数 : 指定文件拷贝到远程主机后的属主，但是远程主机上必须有对应的用户，否则会报错。
- **group**参数 : 指定文件拷贝到远程主机后的属组，但是远程主机上必须有对应的组，否则会报错。
- **mode**参数 : 指定文件拷贝到远程主机后的权限，如果你想将权限设置为”rw-r--r--“，则可以使用mode=0644表示，如果你想要在user对应的权限位上添加执行权限，则可以使用mode=u+x表示。

示例

- 拷贝单个文件|拷贝多个文件|拷贝目录

```yaml
---
- hosts: all               # 目标主机组或名
  remote_user: root
  gather_facts: false       # 不收集对应主机的信息
  tasks:
    - name: 拷贝单个文件
      copy:
        src: "aaa.txt"
        dest: "/home/tmp/ansible/copy_single_file/"
        owner: root
        group: root
        mode: 0755
        backup: yes

    - name: 拷贝多个文件
      copy:
        src: "{{ item.src }}"
        dest: "{{ item.dest }}"
        owner: root
        group: root
        mode: 0755
        backup: yes
      with_items:
          - { src: "bbb.sh", dest: "/home/tmp/ansible/copy_single_file/" }
          - { src: "ccc.txt", dest: "/home/tmp/ansible/copy_single_file/" }

    - name: 拷贝目录
      copy:
        src: "eee/"
        dest: "/home/tmp/ansible/copy_single_file/eee/"
        owner: root
        group: root
        mode: 0755
        backup: yes
```

https://blog.csdn.net/qq_39677803/article/details/123040746

### service模块

功能：调整远程服务器的服务（启动|停止|重启|开机自启|开机非自启）

参数：

| 参数名  | 必填 | 默认值 | 选项                             | 备注                                                     |
| ------- | ---- | ------ | -------------------------------- | -------------------------------------------------------- |
| name    | yes  |        |                                  | 需要进行操作的service名字                                |
| state   | no   |        | stared/stoped/restarted/reloaded | service最终操作后的状态。                                |
| enabled | no   |        | yes/no                           | 是否开机自启动<br />*enabled*和*state*至少要有一个被定义 |

示例

```yaml
---
- hosts: all
  tasks:
    - name: 启动服务
      service:
        name: "{{ item }}"       # 服务名
        enabled: yes      # 是否开机自启
        state: started    # 启动
      loop: [firewalld,nginx]    # 要循环的列表

    - name: 停止服务
      service:
        name: "{{ item }}"            # 服务名
        enabled: no        # 是否开机自启
        state: stopped        # 启动
      loop: [ firewalld ]    # 要循环的列表

    - name: 重启服务
      service:
        name: "{{ item }}"            # 服务名
        enabled: yes        # 是否开机自启
        state: restarted        # 启动
      loop: [ nginx ]    # 要循环的列表
```

https://blog.csdn.net/omaidb/article/details/120919481

参考链接：

https://blog.csdn.net/A_art_xiang/article/details/120524817

