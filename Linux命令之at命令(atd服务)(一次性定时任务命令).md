Linux命令之at命令(atd服务)(一次性定时任务命令)

## 前言

当我们想循环执行任务时（如每天中午12点运行某个文件），会选择Linux下的crontab命令，但当我们只想执行一次定时任务时（如今天中午12点运行某个文件，以后都不再需要运行，运行一次就够了），可以选择at命令

## 安装与启动

如果想运行at命令，则需要安装atd服务，并配置为自启动。

以centos7为例

下载atd服务：`yum -y install at`

启动atd服务：`systemctl start atd`

配置自启动：`systemctl enable atd`

![image-20221222114017637](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20221222114017637.png)

## at命令的访问控制

at 命令的访问控制是依靠 /etc/at.allow（白名单）和 /etc/at.deny（黑名单）这两个文件来实现的，具体规则如下：

- 如果系统中有 /etc/at.allow 文件，那么只有写入 /etc/at.allow 文件（白名单）中的用户可以使用 at 命令，其他用户不能使用 at 命令（注意，/etc/at.allow 文件的优先级更高，也就是说，如果同一个用户既写入 /etc/at.allow 文件，又写入 /etc/at.deny 文件，那么这个用户是可以使用 at 命令的）。
- 如果系统中没有 /etc/at.allow 文件，只有 /etc/at.deny 文件，那么写入 /etc/at.deny 文件（黑名单）中的用户不能使用 at 命令，其他用户可以使用 at 命令。不过这个文件对 root 用户不生效。
- 如果系统中这两个文件都不存在，那么只有 root 用户可以使用 at 命令。
- 系统中默认只有 /etc/at.deny 文件，而且这个文件是空的，因此，系统中所有的用户都可以使用 at 命令。不过，如果我们打算控制用户的 at 命令权限，那么只需把用户写入 /etc/at.deny 文件即可。

## at命令的参数

| 选项          | 含义                                                         |
| ------------- | ------------------------------------------------------------ |
| -m            | 当 at 工作完成后，无论命令是否输出，都用 E-mail 通知执行 at 命令的用户。 |
| -c 工作标识号 | 显示该 at 工作的实际内容。                                   |
| -t 时间       | 在指定时间提交工作并执行，时间格式为 [[CC]YY]MMDDhhmm。      |
| -d            | 删除某个工作，需要提供相应的工作标识号（ID），同 atrm 命令的作用相同。 |
| -l            | 列出当前所有等待运行的工作，和 atq 命令具有相同的额作用。    |
| -f 脚本文件   | 指定所要提交的脚本文件。                                     |

对应可以使用的时间格式

| 格式                       | 用法                                                         |
| -------------------------- | ------------------------------------------------------------ |
| HH:MM                      | 比如 04:00 AM。如果时间已过，则它会在第二天的同一时间执行。  |
| Midnight（midnight）       | 代表 12:00 AM（也就是 00:00）。                              |
| Noon（noon）               | 代表 12:00 PM（相当于 12:00）。                              |
| Teatime（teatime）         | 代表 4:00 PM（相当于 16:00）。                               |
| 英文月名 日期 年份         | 比如 January 15 2018 表示 2018 年 1 月 15 号，年份可有可无。 |
| MMDDYY、MM/DD/YY、MM.DD.YY | 比如 011518 表示 2018 年 1 月 15 号。                        |
| now+时间                   | 以 minutes、hours、days 或 weeks 为单位，例如 now+5 days 表示命令在 5 天之后的此时此刻执行。 |

## at命令的例子

### 【例 1】两分钟后运行`python3 demo.py`

demo.py内容

```python
import datetime
import os.path

now_d = datetime.datetime.now()
now = now_d.strftime("%Y-%m-%d %H:%M:%S")
print(f"现在的时间是:{now}")
folder_name = './debug/'
folder_name = os.path.abspath(folder_name)
if not os.path.exists(folder_name):
    os.makedirs(folder_name)
with open(os.path.join(folder_name, now_d.strftime("%Y-%m-%d_%H-%M-%S") + '.txt'), 'w', encoding='utf-8') as f:
    f.write(f"现在的时间是:{now}")
```

步骤：

1. 输入 at now + 2 minutes 进入到at命令界面
2. 输入python3 demo.py
3. 按下Ctrl+D保存at命令

![image-20221222133228240](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20221222133228240.png)

两分钟后查看当前目录下的debug目录下有没有文件生成即可。

### 【例 2】2022年12月22日13:46分运行python3 demo.py

![image-20221222134503377](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20221222134503377.png)

## at记录存放位置

**当使用Ctrl+D保存信息后，记录被存在了 /var/spool/at/ 目录**

添加了三个任务，则对应的 /var/spool/at/ 目录多了三个文件

直接cat文件，可以看到和使用at -c job_id输出信息一致

![image-20221222135257882](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20221222135257882.png)

## 参考链接：

http://c.biancheng.net/view/1090.html

https://www.yiibai.com/linux/at.html

https://linux.cn/article-13710-1.html