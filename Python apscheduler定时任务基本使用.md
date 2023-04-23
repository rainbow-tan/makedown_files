Python apscheduler定时任务基本使用

## 1、下载

```python
python -m pip install apscheduler
```

## 2、四个基本对象：

#### 2.1、触发器（triggers）：

触发器就是根据你指定的触发方式，比如是按照时间间隔，还是按照 crontab触发定时任务

- **date触发器** 指定日期触发一次（比如2022-5-6 14:10:00触发）

参数

run_date 指定时间

timezone 时区

- **interval触发器** 固定间隔时间触发一次（比如10秒触发一次，一周触发一次）

参数

1. weeks：周。整形。
2. days：一个月中的第几天。整形。
3. hours：小时。整形。
4. minutes：分钟。整形。
5. seconds：秒。整形。
6. start_date：间隔触发的起始时间。
7. end_date：间隔触发的结束时间。
8. jitter：触发的时间误差。

- **crontab触发器** 在某个确切的时间周期性的触发事件（每周一下午四点触发，每个月28号下午3点触发）

参数：

1. year：4位数字的年份。
2. month：1-12月份。
3. day：1-31日。
4. week：1-53周。
5. day_of_week：一个礼拜中的第几天（ 0-6或者 mon、 tue、 wed、 thu、 fri、 sat、 sun）。
6. hour： 0-23小时。
7. minute： 0-59分钟。
8. second： 0-59秒。
9. start_date： datetime类型或者字符串类型，起始时间。
10. end_date： datetime类型或者字符串类型，结束时间。
11. timezone：时区。
12. jitter：任务触发的误差时间。

也可使用表达式

![image-20220517143515361](https://img2022.cnblogs.com/blog/1768648/202205/1768648-20220517162617009-986088232.png)

#### 2.2、任务存储器（job stores）：

任务存储器是可以存储任务的地方，默认情况下任务保存在内存，也可将任务保存在各种数据库中。任务存储进去后，会进行序列化，然后也可以反序列化提取出来，继续执行。

#### 2.3、执行器（executors）：

执行器的目的是安排任务到线程池或者进程池中运行的。

#### 2.4、调度器（schedulers）：

任务调度器是属于整个调度的总指挥官。他会合理安排作业存储器、执行器、触发器进行工作，并进行添加和删除任务等。调度器通常是只有一个的。开发人员很少直接操作触发器、存储器、执行器等。因为这些都由调度器自动来实现了。

- [`BlockingScheduler`](https://apscheduler.readthedocs.io/en/3.x/modules/schedulers/blocking.html#apscheduler.schedulers.blocking.BlockingScheduler)：当调度程序是您的进程中唯一运行的东西时使用
- [`BackgroundScheduler`](https://apscheduler.readthedocs.io/en/3.x/modules/schedulers/background.html#apscheduler.schedulers.background.BackgroundScheduler)：当您不使用以下任何框架并希望调度程序在应用程序的后台运行时使用
- [`AsyncIOScheduler`](https://apscheduler.readthedocs.io/en/3.x/modules/schedulers/asyncio.html#apscheduler.schedulers.asyncio.AsyncIOScheduler)：如果您的应用程序使用 asyncio 模块，请使用
- [`GeventScheduler`](https://apscheduler.readthedocs.io/en/3.x/modules/schedulers/gevent.html#apscheduler.schedulers.gevent.GeventScheduler): 如果您的应用程序使用 gevent，请使用
- [`TornadoScheduler`](https://apscheduler.readthedocs.io/en/3.x/modules/schedulers/tornado.html#apscheduler.schedulers.tornado.TornadoScheduler)：如果您正在构建 Tornado 应用程序，请使用
- [`TwistedScheduler`](https://apscheduler.readthedocs.io/en/3.x/modules/schedulers/twisted.html#apscheduler.schedulers.twisted.TwistedScheduler)：如果您正在构建 Twisted 应用程序，请使用
- `QtScheduler`：如果您正在构建 Qt 应用程序，请使用

## 3、使用

### date触发器

https://apscheduler.readthedocs.io/en/stable/modules/triggers/date.html#module-apscheduler.triggers.date

```python
import datetime

from apscheduler.schedulers.blocking import BlockingScheduler


def my_job():
    print(f"现在时间:{datetime.datetime.now().strftime('%Y-%m-%d %H:%M:%S')}")


def demo():
    s = BlockingScheduler()
    s.add_job(my_job, 'date', timezone='Asia/Shanghai')  # 不给run_date参数默认为当前时间
    s.add_job(my_job, 'date', run_date=datetime.datetime(2022, 5, 17, 14, 49, 00), timezone='Asia/Shanghai')
    s.add_job(my_job, 'date', run_date="2022-5-17 14:50:00", timezone='Asia/Shanghai')
    s.start()


if __name__ == '__main__':
    demo()
```

运行

![image-20220517145147140](https://img2022.cnblogs.com/blog/1768648/202205/1768648-20220517162617362-1891648080.png)

###  interval触发器

https://apscheduler.readthedocs.io/en/stable/modules/triggers/interval.html#module-apscheduler.triggers.interval

```python
import datetime

from apscheduler.schedulers.blocking import BlockingScheduler


def my_job():
    print(f"现在时间:{datetime.datetime.now().strftime('%Y-%m-%d %H:%M:%S')}")


def demo():
    s = BlockingScheduler()
    s.add_job(my_job, 'interval', seconds=10, timezone='Asia/Shanghai')  # 10秒运行一次
    s.start()


if __name__ == '__main__':
    demo()
```

运行

![image-20220517145615003](https://img2022.cnblogs.com/blog/1768648/202205/1768648-20220517162617711-2110466241.png)

您还可以通过`start_date`和`end_date` 参数分别指定计划的开始日期和结束日期

如果开始日期是过去的，则触发器不会追溯触发多次，而是根据过去的开始时间从当前时间计算下一次运行时间。

```python
import datetime

from apscheduler.schedulers.blocking import BlockingScheduler


def my_job():
    print(f"现在时间:{datetime.datetime.now().strftime('%Y-%m-%d %H:%M:%S')}")


def demo():
    s = BlockingScheduler()
    s.add_job(my_job, 'interval', seconds=10, timezone='Asia/Shanghai',
              start_date="2022-05-17 14:00:00", end_date="2022-05-17 15:01:00")  # 10秒运行一次
    s.start()


if __name__ == '__main__':
    demo()
```

### crontab触发器

https://apscheduler.readthedocs.io/en/stable/modules/triggers/cron.html#module-apscheduler.triggers.cron

```python
import datetime

from apscheduler.schedulers.blocking import BlockingScheduler


def my_job():
    print(f"现在时间:{datetime.datetime.now().strftime('%Y-%m-%d %H:%M:%S')}")


def my_job1():
    print(f"my_job1:{datetime.datetime.now().strftime('%Y-%m-%d %H:%M:%S')}")


def demo():
    s = BlockingScheduler()
    s.add_job(my_job, 'cron', second='*/10', timezone='Asia/Shanghai', )  # 10秒运行一次
    s.add_job(my_job1, 'cron', second='3,6,9,15', timezone='Asia/Shanghai', )  # 每分钟的3,6,9,15秒时候运行
    try:
        s.start()
    except KeyboardInterrupt:
        print("定时定时任务")
        s.shutdown()
        print("停止完成")


if __name__ == '__main__':
    demo()
```

运行

![image-20220517150851911](https://img2022.cnblogs.com/blog/1768648/202205/1768648-20220517162617943-1855369979.png)

## 4、监听异常信息

```python
import datetime
import random

from apscheduler.events import EVENT_JOB_EXECUTED, EVENT_JOB_ERROR, EVENT_JOB_MISSED, JobExecutionEvent
from apscheduler.schedulers.blocking import BlockingScheduler


def my_job():
    print(f"现在时间:{datetime.datetime.now().strftime('%Y-%m-%d %H:%M:%S')}")
    if random.randint(0, 1):
        raise ValueError


def my_listener(event: JobExecutionEvent):
    if event.exception:
        print(f'异常:{event.exception}')
    else:
        print('正常工作')


def demo():
    s = BlockingScheduler()
    s.add_job(my_job, 'interval', seconds=3, timezone='Asia/Shanghai')
    s.add_listener(my_listener, EVENT_JOB_EXECUTED | EVENT_JOB_ERROR | EVENT_JOB_MISSED)
    s.start()


if __name__ == '__main__':
    demo()
```

![image-20220517154025953](https://img2022.cnblogs.com/blog/1768648/202205/1768648-20220517162618222-195307618.png)

## 5、add_job参数说明

### misfire_grace_time 

当运行时间错错了，可以有多少秒的容错率，比如运行时间是13:00，而服务重启了，启动时间是13:20，超过了20秒，是否要运行这个任务呢，如果设置misfire_grace_time大于20则运行，否则不运行，就是表示容错时间

```python
import datetime

from apscheduler.schedulers.blocking import BlockingScheduler


def my_job():
    print(f"现在时间:{datetime.datetime.now().strftime('%Y-%m-%d %H:%M:%S')}")


def demo():
    s = BlockingScheduler()
    s.add_job(my_job, 'date', run_date=datetime.datetime(2022, 5, 17, 15, 55, 00), timezone='Asia/Shanghai',
              misfire_grace_time=50)
    s.start()


if __name__ == '__main__':
    demo()
```

![image-20220517155704208](https://img2022.cnblogs.com/blog/1768648/202205/1768648-20220517162618483-165692450.png)

### max_instances

运行时最大存在的实例数量。假设运行时间是5秒，而任务间隔时间是3秒，当第一个任务在执行是，如果我设置max_instances=1，则不会开始执行第二个任务

```python
import datetime
import time

from apscheduler.schedulers.blocking import BlockingScheduler


def my_job():
    print(f"现在时间:{datetime.datetime.now().strftime('%Y-%m-%d %H:%M:%S')}")
    time.sleep(5)


def demo():
    s = BlockingScheduler()
    s.add_job(my_job, 'interval', seconds=3, timezone='Asia/Shanghai', max_instances=1)

    try:
        s.start()
    except(KeyboardInterrupt, SystemExit):
        print("定时定时任务")
        s.shutdown()
        print("停止完成")


if __name__ == '__main__':
    demo()
```

![image-20220517160212717](https://img2022.cnblogs.com/blog/1768648/202205/1768648-20220517162618729-2109157694.png)

正常应该是3秒以输出，但是设置了max_instances=1，则变成了6秒一输出，因为启动第二个任务时，第一个没跑完，因此跳过了第二个任务

### id=None, name=None

指定id和name，id需要是唯一的

### replace_existing

如果id存在，是否替换任务（如果替换，之前的记录还是会存在）

### next_run_time

下次运行时间

### coalesce

设置 coalesce为 False：设置这个目的是，比如由于某个原因导致某个任务积攒了很多次没有执行（比如有一个任务是1分钟跑一次，但是系统原因断了5分钟），如果 coalesce=True，那么下次恢复运行的时候，会只执行一次，而如果设置 coalesce=False，那么就不会合并，会5次全部执行。

### jitter

随机一个秒去运行，上下几秒，例如运行时间是15:10:10，jitter=3，就是15:10:7-15:10:13

## 6、需要参数的定时任务

通过args和kwargs传递即可

```python
import datetime

from apscheduler.schedulers.blocking import BlockingScheduler


def my_job(*args, **kwargs):
    print(f"现在时间:{datetime.datetime.now().strftime('%Y-%m-%d %H:%M:%S')}")
    print(f"args:{args}")
    print(f"kwargs:{kwargs}")


def demo():
    s = BlockingScheduler()
    s.add_job(my_job, 'interval', seconds=3, timezone='Asia/Shanghai', max_instances=1, args=("时间", "你好"),
              kwargs=dict(a=1, b=2))

    try:
        s.start()
    except(KeyboardInterrupt, SystemExit):
        print("定时定时任务")
        s.shutdown()
        print("停止完成")


if __name__ == '__main__':
    demo()
```

![image-20220517161621176](https://img2022.cnblogs.com/blog/1768648/202205/1768648-20220517162618948-1525899051.png)

## 7、使用装饰器添加定时任务

```python
import datetime

from apscheduler.schedulers.blocking import BlockingScheduler

s = BlockingScheduler()


@s.scheduled_job('interval', seconds=3, id="test", name="test")
def my_job(*args, **kwargs):
    print(f"现在时间:{datetime.datetime.now().strftime('%Y-%m-%d %H:%M:%S')}")
    print(f"args:{args}")
    print(f"kwargs:{kwargs}")


try:
    s.start()
except(KeyboardInterrupt, SystemExit):
    print("定时定时任务")
    s.shutdown()
    print("停止完成")
```

## 8、报错情况

如果遇到`cannot pickle '_io.BufferedReader' object`错误，可能是这个原因，你添加的函数是一个实例方法，把他变成静态方法就行了，例如

![image-20220914170423127](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220914170423127.png)

报错信息

![image-20220914170740088](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220914170740088.png)

变成静态方法就好了

![image-20220914170446883](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220914170446883.png)

学习链接：https://blog.csdn.net/weixin_42881588/article/details/111401799

官网：https://apscheduler.readthedocs.io/en/3.x/userguide.html#installing-apscheduler