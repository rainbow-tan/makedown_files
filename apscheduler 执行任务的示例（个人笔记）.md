apscheduler 执行任务的示例（个人笔记）

#### Miss测试情况展示

| 测试代码 | 线程数量 | 任务个数 | 任务运行时间 | misfire_grace_time        | Miss数量 | 说明                                     |
| -------- | -------- | -------- | ------------ | ------------------------- | -------- | ---------------------------------------- |
| 示例1    | 3        | 20       | 180秒        | 不填（不填就是undefined） | 17       |                                          |
| 示例2    | 3        | 20       | 180秒        | 120秒                     | 17       | 测试misfire_grace_time小于任务运行的时间 |
| 示例3    | 3        | 20       | 180秒        | 240秒                     | 14       | 测试misfire_grace_time大于任务运行的时间 |

#### 示例1

```python
import datetime
import threading
import time

from apscheduler.executors.pool import ThreadPoolExecutor, ProcessPoolExecutor
from apscheduler.jobstores.sqlalchemy import SQLAlchemyJobStore
from apscheduler.schedulers.blocking import BlockingScheduler


def log(msg):
    thread = threading.current_thread()
    name = thread.name
    print(f"[{thread.ident}][{name}]{msg}")


user = 'root'  # 用户名
password = 'gip123GIP'  # 密码
host = '127.0.0.1'  # 主机IP
port = '3306'  # 主机端口
dbname = 'gip'  # 数据库名称
url = f"mysql+pymysql://{user}:{password}@{host}:{port}/{dbname}?charset=utf8"  # 使用pymysql连接数据库,字符集为UTF8

job_stores = {
    'default': SQLAlchemyJobStore(url=url, tablename='my_tasks')  # 定时任务表名为my_tasks
}
executors = {
    'default': ThreadPoolExecutor(3),
    'processpool': ProcessPoolExecutor(1)
}
job_defaults = {
    'coalesce': True,  # 堆积后只执行最后一个
    'max_instances': 1,  # 最大的实例只能存在一个

}
scheduler = BlockingScheduler(executors=executors, job_defaults=job_defaults, jobstores=job_stores,
                              timezone='Asia/Shanghai')


def my_job():
    wait = 60 * 3
    log(f"my_job现在时间:{datetime.datetime.now()}, 将要睡眠{wait}秒")
    time.sleep(wait)
    log(f"my_job现在时间:{datetime.datetime.now()}, 睡眠{wait}秒完成")


minutes = list(range(5, 60, 5))
minutes = list(map(lambda x: str(x), minutes))
log(f"主线程的minutes:{minutes}")
for i in range(20):
    scheduler.add_job(my_job, 'cron',
                      id=f'my_job_{i + 1}',
                      name=f'我的job_{i + 1}',
                      minute=','.join(minutes),
                      timezone='Asia/Shanghai',
                      replace_existing=True,

                      )
scheduler.start()
"""
测试代码                 示例1
线程数量                 3
任务个数                 20
misfire_grace_time      不填 （不填默认是undefined）       
Miss数量                17
"""
"""
------------------控制台输出------------------
F:\Python3.9.12\python.exe E:/WorkSpace/TMP/TMPPYTHON/定时任务学习.py
[27892][MainThread]主线程的minutes:['5', '10', '15', '20', '25', '30', '35', '40', '45', '50', '55']
[20660][ThreadPoolExecutor-0_0]my_job现在时间:2022-12-29 11:05:00.008106, 将要睡眠180秒
[9776][ThreadPoolExecutor-0_1]my_job现在时间:2022-12-29 11:05:00.040455, 将要睡眠180秒
[21152][ThreadPoolExecutor-0_2]my_job现在时间:2022-12-29 11:05:00.072798, 将要睡眠180秒
[20660][ThreadPoolExecutor-0_0]my_job现在时间:2022-12-29 11:08:00.021440, 睡眠180秒完成
[base.py][20660][ThreadPoolExecutor-0_0]Run time of job "我的job_4 (trigger: cron[minute='5,10,15,20,25,30,35,40,45,50,55'], next run at: 2022-12-29 11:10:00 CST)" was missed by 0:03:00.021440
[base.py][20660][ThreadPoolExecutor-0_0]Run time of job "我的job_5 (trigger: cron[minute='5,10,15,20,25,30,35,40,45,50,55'], next run at: 2022-12-29 11:10:00 CST)" was missed by 0:03:00.021440
[base.py][20660][ThreadPoolExecutor-0_0]Run time of job "我的job_6 (trigger: cron[minute='5,10,15,20,25,30,35,40,45,50,55'], next run at: 2022-12-29 11:10:00 CST)" was missed by 0:03:00.021440
[base.py][20660][ThreadPoolExecutor-0_0]Run time of job "我的job_7 (trigger: cron[minute='5,10,15,20,25,30,35,40,45,50,55'], next run at: 2022-12-29 11:10:00 CST)" was missed by 0:03:00.021440
[base.py][20660][ThreadPoolExecutor-0_0]Run time of job "我的job_8 (trigger: cron[minute='5,10,15,20,25,30,35,40,45,50,55'], next run at: 2022-12-29 11:10:00 CST)" was missed by 0:03:00.021440
[base.py][20660][ThreadPoolExecutor-0_0]Run time of job "我的job_9 (trigger: cron[minute='5,10,15,20,25,30,35,40,45,50,55'], next run at: 2022-12-29 11:10:00 CST)" was missed by 0:03:00.022437
[base.py][20660][ThreadPoolExecutor-0_0]Run time of job "我的job_10 (trigger: cron[minute='5,10,15,20,25,30,35,40,45,50,55'], next run at: 2022-12-29 11:10:00 CST)" was missed by 0:03:00.022437
[base.py][20660][ThreadPoolExecutor-0_0]Run time of job "我的job_11 (trigger: cron[minute='5,10,15,20,25,30,35,40,45,50,55'], next run at: 2022-12-29 11:10:00 CST)" was missed by 0:03:00.022437
[base.py][20660][ThreadPoolExecutor-0_0]Run time of job "我的job_12 (trigger: cron[minute='5,10,15,20,25,30,35,40,45,50,55'], next run at: 2022-12-29 11:10:00 CST)" was missed by 0:03:00.022437
[base.py][20660][ThreadPoolExecutor-0_0]Run time of job "我的job_13 (trigger: cron[minute='5,10,15,20,25,30,35,40,45,50,55'], next run at: 2022-12-29 11:10:00 CST)" was missed by 0:03:00.022437
[base.py][20660][ThreadPoolExecutor-0_0]Run time of job "我的job_14 (trigger: cron[minute='5,10,15,20,25,30,35,40,45,50,55'], next run at: 2022-12-29 11:10:00 CST)" was missed by 0:03:00.022437
[base.py][20660][ThreadPoolExecutor-0_0]Run time of job "我的job_15 (trigger: cron[minute='5,10,15,20,25,30,35,40,45,50,55'], next run at: 2022-12-29 11:10:00 CST)" was missed by 0:03:00.022437
[base.py][20660][ThreadPoolExecutor-0_0]Run time of job "我的job_16 (trigger: cron[minute='5,10,15,20,25,30,35,40,45,50,55'], next run at: 2022-12-29 11:10:00 CST)" was missed by 0:03:00.022437
[base.py][20660][ThreadPoolExecutor-0_0]Run time of job "我的job_17 (trigger: cron[minute='5,10,15,20,25,30,35,40,45,50,55'], next run at: 2022-12-29 11:10:00 CST)" was missed by 0:03:00.022437
[base.py][20660][ThreadPoolExecutor-0_0]Run time of job "我的job_18 (trigger: cron[minute='5,10,15,20,25,30,35,40,45,50,55'], next run at: 2022-12-29 11:10:00 CST)" was missed by 0:03:00.022437
[base.py][20660][ThreadPoolExecutor-0_0]Run time of job "我的job_19 (trigger: cron[minute='5,10,15,20,25,30,35,40,45,50,55'], next run at: 2022-12-29 11:10:00 CST)" was missed by 0:03:00.022437
[base.py][20660][ThreadPoolExecutor-0_0]Run time of job "我的job_20 (trigger: cron[minute='5,10,15,20,25,30,35,40,45,50,55'], next run at: 2022-12-29 11:10:00 CST)" was missed by 0:03:00.022437
[9776][ThreadPoolExecutor-0_1]my_job现在时间:2022-12-29 11:08:00.051389, 睡眠180秒完成
[21152][ThreadPoolExecutor-0_2]my_job现在时间:2022-12-29 11:08:00.081279, 睡眠180秒完成


[20660][ThreadPoolExecutor-0_0]my_job现在时间:2022-12-29 11:10:00.554302, 将要睡眠180秒
[21152][ThreadPoolExecutor-0_2]my_job现在时间:2022-12-29 11:10:00.581813, 将要睡眠180秒
[9776][ThreadPoolExecutor-0_1]my_job现在时间:2022-12-29 11:10:00.596889, 将要睡眠180秒
[base.py][20660][ThreadPoolExecutor-0_0]Run time of job "我的job_4 (trigger: cron[minute='5,10,15,20,25,30,35,40,45,50,55'], next run at: 2022-12-29 11:15:00 CST)" was missed by 0:03:00.569315
[base.py][20660][ThreadPoolExecutor-0_0]Run time of job "我的job_5 (trigger: cron[minute='5,10,15,20,25,30,35,40,45,50,55'], next run at: 2022-12-29 11:15:00 CST)" was missed by 0:03:00.570163
[base.py][20660][ThreadPoolExecutor-0_0]Run time of job "我的job_6 (trigger: cron[minute='5,10,15,20,25,30,35,40,45,50,55'], next run at: 2022-12-29 11:15:00 CST)" was missed by 0:03:00.571160
[base.py][20660][ThreadPoolExecutor-0_0]Run time of job "我的job_7 (trigger: cron[minute='5,10,15,20,25,30,35,40,45,50,55'], next run at: 2022-12-29 11:15:00 CST)" was missed by 0:03:00.571160
[base.py][20660][ThreadPoolExecutor-0_0]Run time of job "我的job_8 (trigger: cron[minute='5,10,15,20,25,30,35,40,45,50,55'], next run at: 2022-12-29 11:15:00 CST)" was missed by 0:03:00.571160
[base.py][20660][ThreadPoolExecutor-0_0]Run time of job "我的job_9 (trigger: cron[minute='5,10,15,20,25,30,35,40,45,50,55'], next run at: 2022-12-29 11:15:00 CST)" was missed by 0:03:00.572120
[base.py][20660][ThreadPoolExecutor-0_0]Run time of job "我的job_10 (trigger: cron[minute='5,10,15,20,25,30,35,40,45,50,55'], next run at: 2022-12-29 11:15:00 CST)" was missed by 0:03:00.572120
[base.py][20660][ThreadPoolExecutor-0_0]Run time of job "我的job_11 (trigger: cron[minute='5,10,15,20,25,30,35,40,45,50,55'], next run at: 2022-12-29 11:15:00 CST)" was missed by 0:03:00.572120
[base.py][20660][ThreadPoolExecutor-0_0]Run time of job "我的job_12 (trigger: cron[minute='5,10,15,20,25,30,35,40,45,50,55'], next run at: 2022-12-29 11:15:00 CST)" was missed by 0:03:00.572120
[base.py][20660][ThreadPoolExecutor-0_0]Run time of job "我的job_13 (trigger: cron[minute='5,10,15,20,25,30,35,40,45,50,55'], next run at: 2022-12-29 11:15:00 CST)" was missed by 0:03:00.572120
[base.py][20660][ThreadPoolExecutor-0_0]Run time of job "我的job_14 (trigger: cron[minute='5,10,15,20,25,30,35,40,45,50,55'], next run at: 2022-12-29 11:15:00 CST)" was missed by 0:03:00.572120
[base.py][20660][ThreadPoolExecutor-0_0]Run time of job "我的job_15 (trigger: cron[minute='5,10,15,20,25,30,35,40,45,50,55'], next run at: 2022-12-29 11:15:00 CST)" was missed by 0:03:00.572120
[base.py][20660][ThreadPoolExecutor-0_0]Run time of job "我的job_16 (trigger: cron[minute='5,10,15,20,25,30,35,40,45,50,55'], next run at: 2022-12-29 11:15:00 CST)" was missed by 0:03:00.572120
[base.py][20660][ThreadPoolExecutor-0_0]Run time of job "我的job_17 (trigger: cron[minute='5,10,15,20,25,30,35,40,45,50,55'], next run at: 2022-12-29 11:15:00 CST)" was missed by 0:03:00.572120
[base.py][20660][ThreadPoolExecutor-0_0]Run time of job "我的job_18 (trigger: cron[minute='5,10,15,20,25,30,35,40,45,50,55'], next run at: 2022-12-29 11:15:00 CST)" was missed by 0:03:00.572120
[base.py][20660][ThreadPoolExecutor-0_0]Run time of job "我的job_19 (trigger: cron[minute='5,10,15,20,25,30,35,40,45,50,55'], next run at: 2022-12-29 11:15:00 CST)" was missed by 0:03:00.572120
[base.py][20660][ThreadPoolExecutor-0_0]Run time of job "我的job_20 (trigger: cron[minute='5,10,15,20,25,30,35,40,45,50,55'], next run at: 2022-12-29 11:15:00 CST)" was missed by 0:03:00.572120
[20660][ThreadPoolExecutor-0_0]my_job现在时间:2022-12-29 11:13:00.569315, 睡眠180秒完成
[21152][ThreadPoolExecutor-0_2]my_job现在时间:2022-12-29 11:13:00.584872, 睡眠180秒完成
[9776][ThreadPoolExecutor-0_1]my_job现在时间:2022-12-29 11:13:00.600453, 睡眠180秒完成

进程已结束,退出代码-1
"""
```

##### 说明：

最大可运行的线程数为3

当20个任务都在同一个时间点即5分钟时运行，则最多会运行3个任务，此时会运行①②③任务，

运行完成后，3个线程空闲，因为运行的任务等待3分钟，运行结束就是8分了，此时看到misfire_grace_time为undefined，则不再运行其他任务，查看到其他任务应该运行时间是5分时，这时候已经是8分了，则就输出没运行的任务是（4-20），每个任务错过自己运行时间3分钟（8-5）则miss数量就是17个

接着时间来到10分钟，该第二次运行任务了，运行情况和上面一样，都是运行①②③任务，miss（4-20任务）

#### 示例2

```python
import datetime
import threading
import time

from apscheduler.executors.pool import ThreadPoolExecutor, ProcessPoolExecutor
from apscheduler.jobstores.sqlalchemy import SQLAlchemyJobStore
from apscheduler.schedulers.blocking import BlockingScheduler


def log(msg):
    thread = threading.current_thread()
    name = thread.name
    print(f"[{thread.ident}][{name}]{msg}")


user = 'root'  # 用户名
password = 'gip123GIP'  # 密码
host = '127.0.0.1'  # 主机IP
port = '3306'  # 主机端口
dbname = 'gip'  # 数据库名称
url = f"mysql+pymysql://{user}:{password}@{host}:{port}/{dbname}?charset=utf8"  # 使用pymysql连接数据库,字符集为UTF8

job_stores = {
    'default': SQLAlchemyJobStore(url=url, tablename='my_tasks')  # 定时任务表名为my_tasks
}
executors = {
    'default': ThreadPoolExecutor(3),
    'processpool': ProcessPoolExecutor(1)
}
job_defaults = {
    'coalesce': True,  # 堆积后只执行最后一个
    'max_instances': 1,  # 最大的实例只能存在一个

}
scheduler = BlockingScheduler(executors=executors, job_defaults=job_defaults, jobstores=job_stores,
                              timezone='Asia/Shanghai')


def my_job():
    wait = 60 * 3
    log(f"my_job现在时间:{datetime.datetime.now()}, 将要睡眠{wait}秒")
    time.sleep(wait)
    log(f"my_job现在时间:{datetime.datetime.now()}, 睡眠{wait}秒完成")


minutes = list(range(19, 60, 5))
minutes = list(map(lambda x: str(x), minutes))
log(f"主线程的minutes:{minutes}")
for i in range(20):
    scheduler.add_job(my_job, 'cron',
                      id=f'my_job_{i + 1}',
                      name=f'我的job_{i + 1}',
                      minute=','.join(minutes),
                      timezone='Asia/Shanghai',
                      replace_existing=True,
                      misfire_grace_time=60 * 2
                      )
scheduler.start()
"""
测试代码                 示例2
线程数量                 3
任务个数                 20
misfire_grace_time      120秒(小于每个运行任务的时间，即miss时间肯定大于120秒) （预期情况应该和示例1一样）
Miss数量                17
"""
"""
------------------控制台输出------------------
F:\Python3.9.12\python.exe E:/WorkSpace/TMP/TMPPYTHON/定时任务学习-示例2.py
[28624][MainThread]主线程的minutes:['19', '24', '29', '34', '39', '44', '49', '54', '59']
[1708][ThreadPoolExecutor-0_0]my_job现在时间:2022-12-29 11:19:00.008462, 将要睡眠180秒
[16916][ThreadPoolExecutor-0_1]my_job现在时间:2022-12-29 11:19:00.067983, 将要睡眠180秒
[27060][ThreadPoolExecutor-0_2]my_job现在时间:2022-12-29 11:19:00.091367, 将要睡眠180秒
[base.py][1708][ThreadPoolExecutor-0_0]Run time of job "我的job_4 (trigger: cron[minute='19,24,29,34,39,44,49,54,59'], next run at: 2022-12-29 11:24:00 CST)" was missed by 0:03:00.014889
[base.py][1708][ThreadPoolExecutor-0_0]Run time of job "我的job_5 (trigger: cron[minute='19,24,29,34,39,44,49,54,59'], next run at: 2022-12-29 11:24:00 CST)" was missed by 0:03:00.015771
[base.py][1708][ThreadPoolExecutor-0_0]Run time of job "我的job_6 (trigger: cron[minute='19,24,29,34,39,44,49,54,59'], next run at: 2022-12-29 11:24:00 CST)" was missed by 0:03:00.016766
[base.py][1708][ThreadPoolExecutor-0_0]Run time of job "我的job_7 (trigger: cron[minute='19,24,29,34,39,44,49,54,59'], next run at: 2022-12-29 11:24:00 CST)" was missed by 0:03:00.016766
[base.py][1708][ThreadPoolExecutor-0_0]Run time of job "我的job_8 (trigger: cron[minute='19,24,29,34,39,44,49,54,59'], next run at: 2022-12-29 11:24:00 CST)" was missed by 0:03:00.017752
[base.py][1708][ThreadPoolExecutor-0_0]Run time of job "我的job_9 (trigger: cron[minute='19,24,29,34,39,44,49,54,59'], next run at: 2022-12-29 11:24:00 CST)" was missed by 0:03:00.017752
[base.py][1708][ThreadPoolExecutor-0_0]Run time of job "我的job_10 (trigger: cron[minute='19,24,29,34,39,44,49,54,59'], next run at: 2022-12-29 11:24:00 CST)" was missed by 0:03:00.017752
[base.py][1708][ThreadPoolExecutor-0_0]Run time of job "我的job_11 (trigger: cron[minute='19,24,29,34,39,44,49,54,59'], next run at: 2022-12-29 11:24:00 CST)" was missed by 0:03:00.017752
[base.py][1708][ThreadPoolExecutor-0_0]Run time of job "我的job_12 (trigger: cron[minute='19,24,29,34,39,44,49,54,59'], next run at: 2022-12-29 11:24:00 CST)" was missed by 0:03:00.017752
[base.py][1708][ThreadPoolExecutor-0_0]Run time of job "我的job_13 (trigger: cron[minute='19,24,29,34,39,44,49,54,59'], next run at: 2022-12-29 11:24:00 CST)" was missed by 0:03:00.017752
[base.py][1708][ThreadPoolExecutor-0_0]Run time of job "我的job_14 (trigger: cron[minute='19,24,29,34,39,44,49,54,59'], next run at: 2022-12-29 11:24:00 CST)" was missed by 0:03:00.017752
[base.py][1708][ThreadPoolExecutor-0_0]Run time of job "我的job_15 (trigger: cron[minute='19,24,29,34,39,44,49,54,59'], next run at: 2022-12-29 11:24:00 CST)" was missed by 0:03:00.017752
[base.py][1708][ThreadPoolExecutor-0_0]Run time of job "我的job_16 (trigger: cron[minute='19,24,29,34,39,44,49,54,59'], next run at: 2022-12-29 11:24:00 CST)" was missed by 0:03:00.017752
[base.py][1708][ThreadPoolExecutor-0_0]Run time of job "我的job_17 (trigger: cron[minute='19,24,29,34,39,44,49,54,59'], next run at: 2022-12-29 11:24:00 CST)" was missed by 0:03:00.017752
[base.py][1708][ThreadPoolExecutor-0_0]Run time of job "我的job_18 (trigger: cron[minute='19,24,29,34,39,44,49,54,59'], next run at: 2022-12-29 11:24:00 CST)" was missed by 0:03:00.017752
[base.py][1708][ThreadPoolExecutor-0_0]Run time of job "我的job_19 (trigger: cron[minute='19,24,29,34,39,44,49,54,59'], next run at: 2022-12-29 11:24:00 CST)" was missed by 0:03:00.017752
[base.py][1708][ThreadPoolExecutor-0_0]Run time of job "我的job_20 (trigger: cron[minute='19,24,29,34,39,44,49,54,59'], next run at: 2022-12-29 11:24:00 CST)" was missed by 0:03:00.018749
[1708][ThreadPoolExecutor-0_0]my_job现在时间:2022-12-29 11:22:00.014889, 睡眠180秒完成
[16916][ThreadPoolExecutor-0_1]my_job现在时间:2022-12-29 11:22:00.075266, 睡眠180秒完成
[27060][ThreadPoolExecutor-0_2]my_job现在时间:2022-12-29 11:22:00.105322, 睡眠180秒完成
[1708][ThreadPoolExecutor-0_0]my_job现在时间:2022-12-29 11:24:00.544940, 将要睡眠180秒
[27060][ThreadPoolExecutor-0_2]my_job现在时间:2022-12-29 11:24:00.605333, 将要睡眠180秒
[16916][ThreadPoolExecutor-0_1]my_job现在时间:2022-12-29 11:24:00.621918, 将要睡眠180秒

[1708][ThreadPoolExecutor-0_0]my_job现在时间:2022-12-29 11:27:00.557013, 睡眠180秒完成
[base.py][1708][ThreadPoolExecutor-0_0]Run time of job "我的job_4 (trigger: cron[minute='19,24,29,34,39,44,49,54,59'], next run at: 2022-12-29 11:29:00 CST)" was missed by 0:03:00.557013
[base.py][1708][ThreadPoolExecutor-0_0]Run time of job "我的job_5 (trigger: cron[minute='19,24,29,34,39,44,49,54,59'], next run at: 2022-12-29 11:29:00 CST)" was missed by 0:03:00.558023
[base.py][1708][ThreadPoolExecutor-0_0]Run time of job "我的job_6 (trigger: cron[minute='19,24,29,34,39,44,49,54,59'], next run at: 2022-12-29 11:29:00 CST)" was missed by 0:03:00.558023
[base.py][1708][ThreadPoolExecutor-0_0]Run time of job "我的job_7 (trigger: cron[minute='19,24,29,34,39,44,49,54,59'], next run at: 2022-12-29 11:29:00 CST)" was missed by 0:03:00.559021
[base.py][1708][ThreadPoolExecutor-0_0]Run time of job "我的job_8 (trigger: cron[minute='19,24,29,34,39,44,49,54,59'], next run at: 2022-12-29 11:29:00 CST)" was missed by 0:03:00.559021
[base.py][1708][ThreadPoolExecutor-0_0]Run time of job "我的job_9 (trigger: cron[minute='19,24,29,34,39,44,49,54,59'], next run at: 2022-12-29 11:29:00 CST)" was missed by 0:03:00.560019
[base.py][1708][ThreadPoolExecutor-0_0]Run time of job "我的job_10 (trigger: cron[minute='19,24,29,34,39,44,49,54,59'], next run at: 2022-12-29 11:29:00 CST)" was missed by 0:03:00.560019
[base.py][1708][ThreadPoolExecutor-0_0]Run time of job "我的job_11 (trigger: cron[minute='19,24,29,34,39,44,49,54,59'], next run at: 2022-12-29 11:29:00 CST)" was missed by 0:03:00.560019
[base.py][1708][ThreadPoolExecutor-0_0]Run time of job "我的job_12 (trigger: cron[minute='19,24,29,34,39,44,49,54,59'], next run at: 2022-12-29 11:29:00 CST)" was missed by 0:03:00.560019
[base.py][1708][ThreadPoolExecutor-0_0]Run time of job "我的job_13 (trigger: cron[minute='19,24,29,34,39,44,49,54,59'], next run at: 2022-12-29 11:29:00 CST)" was missed by 0:03:00.560019
[base.py][1708][ThreadPoolExecutor-0_0]Run time of job "我的job_14 (trigger: cron[minute='19,24,29,34,39,44,49,54,59'], next run at: 2022-12-29 11:29:00 CST)" was missed by 0:03:00.560019
[base.py][1708][ThreadPoolExecutor-0_0]Run time of job "我的job_15 (trigger: cron[minute='19,24,29,34,39,44,49,54,59'], next run at: 2022-12-29 11:29:00 CST)" was missed by 0:03:00.560019
[base.py][1708][ThreadPoolExecutor-0_0]Run time of job "我的job_16 (trigger: cron[minute='19,24,29,34,39,44,49,54,59'], next run at: 2022-12-29 11:29:00 CST)" was missed by 0:03:00.560019
[base.py][1708][ThreadPoolExecutor-0_0]Run time of job "我的job_17 (trigger: cron[minute='19,24,29,34,39,44,49,54,59'], next run at: 2022-12-29 11:29:00 CST)" was missed by 0:03:00.560019
[base.py][1708][ThreadPoolExecutor-0_0]Run time of job "我的job_18 (trigger: cron[minute='19,24,29,34,39,44,49,54,59'], next run at: 2022-12-29 11:29:00 CST)" was missed by 0:03:00.561007
[base.py][1708][ThreadPoolExecutor-0_0]Run time of job "我的job_19 (trigger: cron[minute='19,24,29,34,39,44,49,54,59'], next run at: 2022-12-29 11:29:00 CST)" was missed by 0:03:00.561007
[base.py][1708][ThreadPoolExecutor-0_0]Run time of job "我的job_20 (trigger: cron[minute='19,24,29,34,39,44,49,54,59'], next run at: 2022-12-29 11:29:00 CST)" was missed by 0:03:00.561007
[27060][ThreadPoolExecutor-0_2]my_job现在时间:2022-12-29 11:27:00.618123, 睡眠180秒完成
[16916][ThreadPoolExecutor-0_1]my_job现在时间:2022-12-29 11:27:00.633115, 睡眠180秒完成

进程已结束,退出代码-1

"""
```

##### 说明：

最大可运行的线程数为3

当20个任务都在同一个时间点即19分时运行，则最多会运行3个任务，此时会运行①②③任务，

运行完成后，3个线程空闲，因为运行的任务等待3分钟，运行结束就是22分钟了，此时看到misfire_grace_time为2分钟，每个任务错过自己运行时间3分钟（22-19），因为misfire_grace_time时间2分钟小于错过时间3分钟，则不再运行其他任务，则就输出没运行的任务是（4-20），则miss数量就是17个

接着时间来到24分钟，情况和上面一样，都是运行①②③任务，miss（4-20任务）

因为misfire_grace_time小于任务运行时间，则任务运行完，miss时间必定大于misfire_grace_time时间，任务肯定不运行

#### 示例3

```python
import datetime
import threading
import time

from apscheduler.executors.pool import ThreadPoolExecutor, ProcessPoolExecutor
from apscheduler.jobstores.sqlalchemy import SQLAlchemyJobStore
from apscheduler.schedulers.blocking import BlockingScheduler


def log(msg):
    thread = threading.current_thread()
    name = thread.name
    print(f"[{thread.ident}][{name}][{datetime.datetime.now()}]{msg}")


user = 'root'  # 用户名
password = 'gip123GIP'  # 密码
host = '127.0.0.1'  # 主机IP
port = '3306'  # 主机端口
dbname = 'gip'  # 数据库名称
url = f"mysql+pymysql://{user}:{password}@{host}:{port}/{dbname}?charset=utf8"  # 使用pymysql连接数据库,字符集为UTF8

job_stores = {
    'default': SQLAlchemyJobStore(url=url, tablename='my_tasks')  # 定时任务表名为my_tasks
}
executors = {
    'default': ThreadPoolExecutor(3),
    'processpool': ProcessPoolExecutor(1)
}
job_defaults = {
    'coalesce': True,  # 堆积后只执行最后一个
    'max_instances': 1,  # 最大的实例只能存在一个

}
scheduler = BlockingScheduler(executors=executors, job_defaults=job_defaults, jobstores=job_stores,
                              timezone='Asia/Shanghai')


def my_job(name):
    wait = 60 * 3  # 每个任务运行3分钟
    log(f"{name}现在时间:{datetime.datetime.now()}, 将要睡眠{wait}秒")
    time.sleep(wait)
    log(f"{name}现在时间:{datetime.datetime.now()}, 睡眠{wait}秒完成")


minutes = list(range(25, 60, 10))
minutes = list(map(lambda x: str(x), minutes))
log(f"主线程的minutes:{minutes}")
for i in range(20):
    scheduler.add_job(my_job, 'cron',
                      args=(f'我的job_{i + 1}',),
                      id=f'my_job_{i + 1}',
                      name=f'我的job_{i + 1}',
                      minute=','.join(minutes),
                      timezone='Asia/Shanghai',
                      replace_existing=True,
                      misfire_grace_time=60 * 4
                      )
scheduler.start()
"""
测试代码                 示例3
线程数量                 3
任务个数                 20
misfire_grace_time      240秒(大于每个运行任务的时间)
Miss数量                14
"""
"""
------------------控制台输出------------------
"""
"""
F:\Python3.9.12\python.exe E:/WorkSpace/TMP/TMPPYTHON/定时任务学习-示例3.py
[5212][MainThread][2022-12-29 14:23:56.856332]主线程的minutes:['25', '35', '45', '55']
[18096][ThreadPoolExecutor-0_0][2022-12-29 14:25:00.017374]我的job_1现在时间:2022-12-29 14:25:00.017374, 将要睡眠180秒
[23020][ThreadPoolExecutor-0_1][2022-12-29 14:25:00.048638]我的job_2现在时间:2022-12-29 14:25:00.048638, 将要睡眠180秒
[2408][ThreadPoolExecutor-0_2][2022-12-29 14:25:00.071523]我的job_3现在时间:2022-12-29 14:25:00.071523, 将要睡眠180秒
[18096][ThreadPoolExecutor-0_0][2022-12-29 14:28:00.032764]我的job_1现在时间:2022-12-29 14:28:00.032764, 睡眠180秒完成
[18096][ThreadPoolExecutor-0_0][2022-12-29 14:28:00.032764]我的job_4现在时间:2022-12-29 14:28:00.032764, 将要睡眠180秒
[23020][ThreadPoolExecutor-0_1][2022-12-29 14:28:00.063454]我的job_2现在时间:2022-12-29 14:28:00.063454, 睡眠180秒完成
[23020][ThreadPoolExecutor-0_1][2022-12-29 14:28:00.063454]我的job_5现在时间:2022-12-29 14:28:00.063454, 将要睡眠180秒
[2408][ThreadPoolExecutor-0_2][2022-12-29 14:28:00.079228]我的job_3现在时间:2022-12-29 14:28:00.079228, 睡眠180秒完成
[2408][ThreadPoolExecutor-0_2][2022-12-29 14:28:00.079228]我的job_6现在时间:2022-12-29 14:28:00.079228, 将要睡眠180秒

[18096][ThreadPoolExecutor-0_0][2022-12-29 14:31:00.047449]我的job_4现在时间:2022-12-29 14:31:00.047449, 睡眠180秒完成
[base.py][18096][ThreadPoolExecutor-0_0][2022-12-29 14:31:00.047449]Run time of job "我的job_7 (trigger: cron[minute='25,35,45,55'], next run at: 2022-12-29 14:35:00 CST)" was missed by 0:06:00.047449
[base.py][18096][ThreadPoolExecutor-0_0][2022-12-29 14:31:00.048412]Run time of job "我的job_8 (trigger: cron[minute='25,35,45,55'], next run at: 2022-12-29 14:35:00 CST)" was missed by 0:06:00.048412
[base.py][18096][ThreadPoolExecutor-0_0][2022-12-29 14:31:00.049405]Run time of job "我的job_9 (trigger: cron[minute='25,35,45,55'], next run at: 2022-12-29 14:35:00 CST)" was missed by 0:06:00.049405
[base.py][18096][ThreadPoolExecutor-0_0][2022-12-29 14:31:00.049405]Run time of job "我的job_10 (trigger: cron[minute='25,35,45,55'], next run at: 2022-12-29 14:35:00 CST)" was missed by 0:06:00.049405
[base.py][18096][ThreadPoolExecutor-0_0][2022-12-29 14:31:00.049405]Run time of job "我的job_11 (trigger: cron[minute='25,35,45,55'], next run at: 2022-12-29 14:35:00 CST)" was missed by 0:06:00.049405
[base.py][18096][ThreadPoolExecutor-0_0][2022-12-29 14:31:00.049405]Run time of job "我的job_12 (trigger: cron[minute='25,35,45,55'], next run at: 2022-12-29 14:35:00 CST)" was missed by 0:06:00.049405
[base.py][18096][ThreadPoolExecutor-0_0][2022-12-29 14:31:00.049405]Run time of job "我的job_13 (trigger: cron[minute='25,35,45,55'], next run at: 2022-12-29 14:35:00 CST)" was missed by 0:06:00.049405
[base.py][18096][ThreadPoolExecutor-0_0][2022-12-29 14:31:00.049405]Run time of job "我的job_14 (trigger: cron[minute='25,35,45,55'], next run at: 2022-12-29 14:35:00 CST)" was missed by 0:06:00.049405
[base.py][18096][ThreadPoolExecutor-0_0][2022-12-29 14:31:00.049405]Run time of job "我的job_15 (trigger: cron[minute='25,35,45,55'], next run at: 2022-12-29 14:35:00 CST)" was missed by 0:06:00.049405
[base.py][18096][ThreadPoolExecutor-0_0][2022-12-29 14:31:00.049405]Run time of job "我的job_16 (trigger: cron[minute='25,35,45,55'], next run at: 2022-12-29 14:35:00 CST)" was missed by 0:06:00.049405
[base.py][18096][ThreadPoolExecutor-0_0][2022-12-29 14:31:00.049405]Run time of job "我的job_17 (trigger: cron[minute='25,35,45,55'], next run at: 2022-12-29 14:35:00 CST)" was missed by 0:06:00.049405
[base.py][18096][ThreadPoolExecutor-0_0][2022-12-29 14:31:00.049405]Run time of job "我的job_18 (trigger: cron[minute='25,35,45,55'], next run at: 2022-12-29 14:35:00 CST)" was missed by 0:06:00.049405
[base.py][18096][ThreadPoolExecutor-0_0][2022-12-29 14:31:00.049405]Run time of job "我的job_19 (trigger: cron[minute='25,35,45,55'], next run at: 2022-12-29 14:35:00 CST)" was missed by 0:06:00.049405
[base.py][18096][ThreadPoolExecutor-0_0][2022-12-29 14:31:00.049405]Run time of job "我的job_20 (trigger: cron[minute='25,35,45,55'], next run at: 2022-12-29 14:35:00 CST)" was missed by 0:06:00.049405
[23020][ThreadPoolExecutor-0_1][2022-12-29 14:31:00.077757]我的job_5现在时间:2022-12-29 14:31:00.077757, 睡眠180秒完成
[2408][ThreadPoolExecutor-0_2][2022-12-29 14:31:00.093241]我的job_6现在时间:2022-12-29 14:31:00.093241, 睡眠180秒完成
[18096][ThreadPoolExecutor-0_0][2022-12-29 14:35:00.610859]我的job_1现在时间:2022-12-29 14:35:00.610859, 将要睡眠180秒
[2408][ThreadPoolExecutor-0_2][2022-12-29 14:35:00.656935]我的job_2现在时间:2022-12-29 14:35:00.656935, 将要睡眠180秒
[23020][ThreadPoolExecutor-0_1][2022-12-29 14:35:00.690417]我的job_3现在时间:2022-12-29 14:35:00.690417, 将要睡眠180秒


[18096][ThreadPoolExecutor-0_0][2022-12-29 14:38:00.614531]我的job_1现在时间:2022-12-29 14:38:00.614531, 睡眠180秒完成
[18096][ThreadPoolExecutor-0_0][2022-12-29 14:38:00.614531]我的job_4现在时间:2022-12-29 14:38:00.614531, 将要睡眠180秒
[2408][ThreadPoolExecutor-0_2][2022-12-29 14:38:00.660644]我的job_2现在时间:2022-12-29 14:38:00.660644, 睡眠180秒完成
[2408][ThreadPoolExecutor-0_2][2022-12-29 14:38:00.660644]我的job_5现在时间:2022-12-29 14:38:00.660644, 将要睡眠180秒
[23020][ThreadPoolExecutor-0_1][2022-12-29 14:38:00.690666]我的job_3现在时间:2022-12-29 14:38:00.690666, 睡眠180秒完成
[23020][ThreadPoolExecutor-0_1][2022-12-29 14:38:00.690666]我的job_6现在时间:2022-12-29 14:38:00.690666, 将要睡眠180秒

[18096][ThreadPoolExecutor-0_0][2022-12-29 14:41:00.618613]我的job_4现在时间:2022-12-29 14:41:00.618613, 睡眠180秒完成
[base.py][18096][ThreadPoolExecutor-0_0][2022-12-29 14:41:00.618613]Run time of job "我的job_7 (trigger: cron[minute='25,35,45,55'], next run at: 2022-12-29 14:45:00 CST)" was missed by 0:06:00.618613
[base.py][18096][ThreadPoolExecutor-0_0][2022-12-29 14:41:00.618613]Run time of job "我的job_8 (trigger: cron[minute='25,35,45,55'], next run at: 2022-12-29 14:45:00 CST)" was missed by 0:06:00.618613
[base.py][18096][ThreadPoolExecutor-0_0][2022-12-29 14:41:00.618613]Run time of job "我的job_9 (trigger: cron[minute='25,35,45,55'], next run at: 2022-12-29 14:45:00 CST)" was missed by 0:06:00.618613
[base.py][18096][ThreadPoolExecutor-0_0][2022-12-29 14:41:00.618613]Run time of job "我的job_10 (trigger: cron[minute='25,35,45,55'], next run at: 2022-12-29 14:45:00 CST)" was missed by 0:06:00.618613
[base.py][18096][ThreadPoolExecutor-0_0][2022-12-29 14:41:00.618613]Run time of job "我的job_11 (trigger: cron[minute='25,35,45,55'], next run at: 2022-12-29 14:45:00 CST)" was missed by 0:06:00.618613
[base.py][18096][ThreadPoolExecutor-0_0][2022-12-29 14:41:00.618613]Run time of job "我的job_12 (trigger: cron[minute='25,35,45,55'], next run at: 2022-12-29 14:45:00 CST)" was missed by 0:06:00.618613
[base.py][18096][ThreadPoolExecutor-0_0][2022-12-29 14:41:00.618613]Run time of job "我的job_13 (trigger: cron[minute='25,35,45,55'], next run at: 2022-12-29 14:45:00 CST)" was missed by 0:06:00.618613
[base.py][18096][ThreadPoolExecutor-0_0][2022-12-29 14:41:00.618613]Run time of job "我的job_14 (trigger: cron[minute='25,35,45,55'], next run at: 2022-12-29 14:45:00 CST)" was missed by 0:06:00.618613
[base.py][18096][ThreadPoolExecutor-0_0][2022-12-29 14:41:00.619614]Run time of job "我的job_15 (trigger: cron[minute='25,35,45,55'], next run at: 2022-12-29 14:45:00 CST)" was missed by 0:06:00.619614
[base.py][18096][ThreadPoolExecutor-0_0][2022-12-29 14:41:00.619614]Run time of job "我的job_16 (trigger: cron[minute='25,35,45,55'], next run at: 2022-12-29 14:45:00 CST)" was missed by 0:06:00.619614
[base.py][18096][ThreadPoolExecutor-0_0][2022-12-29 14:41:00.619614]Run time of job "我的job_17 (trigger: cron[minute='25,35,45,55'], next run at: 2022-12-29 14:45:00 CST)" was missed by 0:06:00.619614
[base.py][18096][ThreadPoolExecutor-0_0][2022-12-29 14:41:00.619614]Run time of job "我的job_18 (trigger: cron[minute='25,35,45,55'], next run at: 2022-12-29 14:45:00 CST)" was missed by 0:06:00.619614
[base.py][18096][ThreadPoolExecutor-0_0][2022-12-29 14:41:00.619614]Run time of job "我的job_19 (trigger: cron[minute='25,35,45,55'], next run at: 2022-12-29 14:45:00 CST)" was missed by 0:06:00.619614
[base.py][18096][ThreadPoolExecutor-0_0][2022-12-29 14:41:00.619614]Run time of job "我的job_20 (trigger: cron[minute='25,35,45,55'], next run at: 2022-12-29 14:45:00 CST)" was missed by 0:06:00.619614
[2408][ThreadPoolExecutor-0_2][2022-12-29 14:41:00.664414]我的job_5现在时间:2022-12-29 14:41:00.664414, 睡眠180秒完成
[23020][ThreadPoolExecutor-0_1][2022-12-29 14:41:00.694828]我的job_6现在时间:2022-12-29 14:41:00.694828, 睡眠180秒完成

"""
```

##### 说明：

最大可运行的线程数为3

当20个任务都在同一个时间点即25分钟时运行，则最多会运行3个任务，此时会运行①②③任务，

运行完后，释放3个线程，此时时间是28分，由于misfire_grace_time是4分钟，目前任务miss时间为3分钟（本该25分运行，但现在是28分了），由于miss时间3分钟小于misfire_grace_time4分钟，则空闲的3个线程，继续运行④⑤⑥任务

运行④⑤⑥任务结束后，释放3个线程，此时时间是31分，这时候miss时间变成了6分（本该25分运行，但显示是31分了）因为misfire_grace_time时间4分钟小于错过时间6分钟，则不再运行其他任务，则就输出没运行的任务是（7-20），则miss数量就是14个

接着时间来到35分钟，该第二次运行任务了，运行情况和上面一样，都是运行①②③④⑤⑥任务，miss（7-20任务）

#### Maximum number of running instances测试情况展示

| 测试代码 | 线程数量 | 任务个数 | 任务运行时间 | max_instances |
| -------- | -------- | -------- | ------------ | ------------- |
| 示例4    | 10       | 4        | 300秒        | 1             |
| 示例5    | 10       | 4        | 300秒        | 2             |
|          |          |          |              |               |

#### 示例4

```python
import datetime
import threading
import time

from apscheduler.executors.pool import ThreadPoolExecutor
from apscheduler.jobstores.sqlalchemy import SQLAlchemyJobStore
from apscheduler.schedulers.blocking import BlockingScheduler


def log(msg):
    thread = threading.current_thread()
    name = thread.name
    print(f"[{thread.ident}][{name}][{datetime.datetime.now()}]{msg}")


user = 'root'  # 用户名
password = 'gip123GIP'  # 密码
host = '127.0.0.1'  # 主机IP
port = '3306'  # 主机端口
dbname = 'gip'  # 数据库名称
url = f"mysql+pymysql://{user}:{password}@{host}:{port}/{dbname}?charset=utf8"  # 使用pymysql连接数据库,字符集为UTF8

job_stores = {
    'default': SQLAlchemyJobStore(url=url, tablename='my_tasks')  # 定时任务表名为my_tasks
}
executors = {
    'default': ThreadPoolExecutor(10),
}
job_defaults = {
    'coalesce': True,  # 堆积后只执行最后一个
    'max_instances': 1,  # 最大的实例只能存在一个

}
scheduler = BlockingScheduler(executors=executors, job_defaults=job_defaults, jobstores=job_stores,
                              timezone='Asia/Shanghai')


def my_job(name):
    wait = 60 * 5  # 每个任务运行5分钟
    log(f"{name}现在时间:{datetime.datetime.now()}, 将要睡眠{wait}秒")
    time.sleep(wait)
    log(f"{name}现在时间:{datetime.datetime.now()}, 睡眠{wait}秒完成")


minutes = list(range(18, 60, 4))
minutes = list(map(lambda x: str(x), minutes))
log(f"主线程的minutes:{minutes}")
for i in range(4):
    scheduler.add_job(my_job, 'cron',
                      args=(f'我的job_{i + 1}',),
                      id=f'my_job_{i + 1}',
                      name=f'我的job_{i + 1}',
                      minute=','.join(minutes),
                      timezone='Asia/Shanghai',
                      replace_existing=True,
                      max_instances=1,

                      )
scheduler.start()
"""
测试代码                 示例4
线程数量                 10
任务个数                 4
misfire_grace_time      不填 （不填默认是undefined）  
max_instances           1
"""
"""
------------------控制台输出------------------
"""
"""
F:\Python3.9.12\python.exe E:/WorkSpace/TMP/TMPPYTHON/定时任务学习-示例4.py
[1380][MainThread][2022-12-29 17:17:26.687603]主线程的minutes:['18', '22', '26', '30', '34', '38', '42', '46', '50', '54', '58']
[19680][ThreadPoolExecutor-0_0][2022-12-29 17:18:00.017169]我的job_1现在时间:2022-12-29 17:18:00.017169, 将要睡眠300秒
[23432][ThreadPoolExecutor-0_1][2022-12-29 17:18:00.043474]我的job_2现在时间:2022-12-29 17:18:00.043474, 将要睡眠300秒
[7436][ThreadPoolExecutor-0_2][2022-12-29 17:18:00.077417]我的job_3现在时间:2022-12-29 17:18:00.077417, 将要睡眠300秒
[18316][ThreadPoolExecutor-0_3][2022-12-29 17:18:00.134879]我的job_4现在时间:2022-12-29 17:18:00.134879, 将要睡眠300秒

[scheduler/base.py][1380][MainThread][2022-12-29 17:22:00.141691]Execution of job "我的job_1 (trigger: cron[minute='18,22,26,30,34,38,42,46,50,54,58'], next run at: 2022-12-29 17:22:00 CST)" skipped: maximum number of running instances reached (1)
[scheduler/base.py][1380][MainThread][2022-12-29 17:22:00.166242]Execution of job "我的job_2 (trigger: cron[minute='18,22,26,30,34,38,42,46,50,54,58'], next run at: 2022-12-29 17:22:00 CST)" skipped: maximum number of running instances reached (1)
[scheduler/base.py][1380][MainThread][2022-12-29 17:22:00.224094]Execution of job "我的job_3 (trigger: cron[minute='18,22,26,30,34,38,42,46,50,54,58'], next run at: 2022-12-29 17:22:00 CST)" skipped: maximum number of running instances reached (1)
[scheduler/base.py][1380][MainThread][2022-12-29 17:22:00.273737]Execution of job "我的job_4 (trigger: cron[minute='18,22,26,30,34,38,42,46,50,54,58'], next run at: 2022-12-29 17:22:00 CST)" skipped: maximum number of running instances reached (1)
[19680][ThreadPoolExecutor-0_0][2022-12-29 17:23:00.027593]我的job_1现在时间:2022-12-29 17:23:00.027593, 睡眠300秒完成
[23432][ThreadPoolExecutor-0_1][2022-12-29 17:23:00.057570]我的job_2现在时间:2022-12-29 17:23:00.057570, 睡眠300秒完成
[7436][ThreadPoolExecutor-0_2][2022-12-29 17:23:00.088082]我的job_3现在时间:2022-12-29 17:23:00.088082, 睡眠300秒完成
[18316][ThreadPoolExecutor-0_3][2022-12-29 17:23:00.149652]我的job_4现在时间:2022-12-29 17:23:00.149652, 睡眠300秒完成

[19680][ThreadPoolExecutor-0_0][2022-12-29 17:26:00.155200]我的job_1现在时间:2022-12-29 17:26:00.155200, 将要睡眠300秒
[7436][ThreadPoolExecutor-0_2][2022-12-29 17:26:00.201077]我的job_2现在时间:2022-12-29 17:26:00.201077, 将要睡眠300秒
[23432][ThreadPoolExecutor-0_1][2022-12-29 17:26:00.242965]我的job_3现在时间:2022-12-29 17:26:00.242965, 将要睡眠300秒
[18316][ThreadPoolExecutor-0_3][2022-12-29 17:26:00.283856]我的job_4现在时间:2022-12-29 17:26:00.283856, 将要睡眠300秒
[scheduler/base.py][1380][MainThread][2022-12-29 17:30:00.175536]Execution of job "我的job_1 (trigger: cron[minute='18,22,26,30,34,38,42,46,50,54,58'], next run at: 2022-12-29 17:30:00 CST)" skipped: maximum number of running instances reached (1)
[scheduler/base.py][1380][MainThread][2022-12-29 17:30:00.223952]Execution of job "我的job_2 (trigger: cron[minute='18,22,26,30,34,38,42,46,50,54,58'], next run at: 2022-12-29 17:30:00 CST)" skipped: maximum number of running instances reached (1)
[scheduler/base.py][1380][MainThread][2022-12-29 17:30:00.264844]Execution of job "我的job_3 (trigger: cron[minute='18,22,26,30,34,38,42,46,50,54,58'], next run at: 2022-12-29 17:30:00 CST)" skipped: maximum number of running instances reached (1)
[scheduler/base.py][1380][MainThread][2022-12-29 17:30:00.338418]Execution of job "我的job_4 (trigger: cron[minute='18,22,26,30,34,38,42,46,50,54,58'], next run at: 2022-12-29 17:30:00 CST)" skipped: maximum number of running instances reached (1)
[19680][ThreadPoolExecutor-0_0][2022-12-29 17:31:00.163958]我的job_1现在时间:2022-12-29 17:31:00.163958, 睡眠300秒完成
[7436][ThreadPoolExecutor-0_2][2022-12-29 17:31:00.208547]我的job_2现在时间:2022-12-29 17:31:00.208547, 睡眠300秒完成
[23432][ThreadPoolExecutor-0_1][2022-12-29 17:31:00.253158]我的job_3现在时间:2022-12-29 17:31:00.253158, 睡眠300秒完成
[18316][ThreadPoolExecutor-0_3][2022-12-29 17:31:00.298470]我的job_4现在时间:2022-12-29 17:31:00.298470, 睡眠300秒完成
[19680][ThreadPoolExecutor-0_0][2022-12-29 17:34:00.193731]我的job_1现在时间:2022-12-29 17:34:00.193731, 将要睡眠300秒
[23432][ThreadPoolExecutor-0_1][2022-12-29 17:34:00.228506]我的job_2现在时间:2022-12-29 17:34:00.228506, 将要睡眠300秒
[7436][ThreadPoolExecutor-0_2][2022-12-29 17:34:00.270395]我的job_3现在时间:2022-12-29 17:34:00.270395, 将要睡眠300秒
[18316][ThreadPoolExecutor-0_3][2022-12-29 17:34:00.336678]我的job_4现在时间:2022-12-29 17:34:00.336678, 将要睡眠300秒
[scheduler/base.py][1380][MainThread][2022-12-29 17:38:00.163817]Execution of job "我的job_1 (trigger: cron[minute='18,22,26,30,34,38,42,46,50,54,58'], next run at: 2022-12-29 17:38:00 CST)" skipped: maximum number of running instances reached (1)
[scheduler/base.py][1380][MainThread][2022-12-29 17:38:00.209770]Execution of job "我的job_2 (trigger: cron[minute='18,22,26,30,34,38,42,46,50,54,58'], next run at: 2022-12-29 17:38:00 CST)" skipped: maximum number of running instances reached (1)
[scheduler/base.py][1380][MainThread][2022-12-29 17:38:00.233737]Execution of job "我的job_3 (trigger: cron[minute='18,22,26,30,34,38,42,46,50,54,58'], next run at: 2022-12-29 17:38:00 CST)" skipped: maximum number of running instances reached (1)
[scheduler/base.py][1380][MainThread][2022-12-29 17:38:00.292550]Execution of job "我的job_4 (trigger: cron[minute='18,22,26,30,34,38,42,46,50,54,58'], next run at: 2022-12-29 17:38:00 CST)" skipped: maximum number of running instances reached (1)
[19680][ThreadPoolExecutor-0_0][2022-12-29 17:39:00.197506]我的job_1现在时间:2022-12-29 17:39:00.197506, 睡眠300秒完成
[23432][ThreadPoolExecutor-0_1][2022-12-29 17:39:00.243092]我的job_2现在时间:2022-12-29 17:39:00.243092, 睡眠300秒完成
[7436][ThreadPoolExecutor-0_2][2022-12-29 17:39:00.273014]我的job_3现在时间:2022-12-29 17:39:00.273014, 睡眠300秒完成
[18316][ThreadPoolExecutor-0_3][2022-12-29 17:39:00.348157]我的job_4现在时间:2022-12-29 17:39:00.348157, 睡眠300秒完成

"""
```

##### 说明：

使用10个线程运行

每个任务睡眠5分钟，共4个任务，第一次运行为18分时，10个线程运行，但任务只有4个，完全足够，第二次运行时间为22分，第二次执行时间间隔为4分钟（22-18），当第一轮任务还没跑完呢，第二轮任务开始了，由于max_instances=1，则最大可运行任务为一个，因此抛出异常maximum number of running instances reached (1)，不再运行

但第三轮任务26分，开始时，没有任务在运行，则开始运行①②③④，同理，到30分时，继续抛出异常maximum number of running instances reached (1)，

所有默认就是每隔两轮任务跑一次①②③④任务

#### 示例5

```python
import datetime
import threading
import time

from apscheduler.executors.pool import ThreadPoolExecutor
from apscheduler.jobstores.sqlalchemy import SQLAlchemyJobStore
from apscheduler.schedulers.blocking import BlockingScheduler


def log(msg):
    thread = threading.current_thread()
    name = thread.name
    print(f"[{thread.ident}][{name}][{datetime.datetime.now()}]{msg}")
    info = '[{:<10s}][{:<25s}][{:<25s}]{}'.format(thread.ident,name,datetime.datetime.now(),msg)
    print(info)


user = 'root'  # 用户名
password = 'gip123GIP'  # 密码
host = '127.0.0.1'  # 主机IP
port = '3306'  # 主机端口
dbname = 'gip'  # 数据库名称
url = f"mysql+pymysql://{user}:{password}@{host}:{port}/{dbname}?charset=utf8"  # 使用pymysql连接数据库,字符集为UTF8

job_stores = {
    'default': SQLAlchemyJobStore(url=url, tablename='my_tasks')  # 定时任务表名为my_tasks
}
executors = {
    'default': ThreadPoolExecutor(10),
}
job_defaults = {
    'coalesce': True,  # 堆积后只执行最后一个
    'max_instances': 1,  # 最大的实例只能存在一个

}
scheduler = BlockingScheduler(executors=executors, job_defaults=job_defaults, jobstores=job_stores,
                              timezone='Asia/Shanghai')


def my_job(name):
    wait = 60 * 5  # 每个任务运行5分钟
    log(f"{name}现在时间:{datetime.datetime.now()}, 将要睡眠{wait}秒")
    time.sleep(wait)
    log(f"{name}现在时间:{datetime.datetime.now()}, 睡眠{wait}秒完成")


minutes = list(range(24, 60, 4))
minutes = list(map(lambda x: str(x), minutes))
log(f"主线程的minutes:{minutes}")
for i in range(4):
    scheduler.add_job(my_job, 'cron',
                      args=(f'我的job_{i + 1}',),
                      id=f'my_job_{i + 1}',
                      name=f'我的job_{i + 1}',
                      minute=','.join(minutes),
                      timezone='Asia/Shanghai',
                      replace_existing=True,
                      max_instances=2,

                      )
scheduler.start()
"""
测试代码                 示例5
线程数量                 10
任务个数                 4
misfire_grace_time      不填 （不填默认是undefined）  
max_instances           2
"""
"""
------------------控制台输出------------------
"""
"""
F:\Python3.9.12\python.exe E:/WorkSpace/TMP/TMPPYTHON/定时任务学习-示例5.py
[21104][MainThread][2022-12-30 10:23:12.634029]主线程的minutes:['24', '28', '32', '36', '40', '44', '48', '52', '56']
[284][ThreadPoolExecutor-0_0][2022-12-30 10:24:00.007074]我的job_1现在时间:2022-12-30 10:24:00.007074, 将要睡眠300秒
[27032][ThreadPoolExecutor-0_1][2022-12-30 10:24:00.033059]我的job_2现在时间:2022-12-30 10:24:00.033059, 将要睡眠300秒
[15468][ThreadPoolExecutor-0_2][2022-12-30 10:24:00.064011]我的job_3现在时间:2022-12-30 10:24:00.064011, 将要睡眠300秒
[6812][ThreadPoolExecutor-0_3][2022-12-30 10:24:00.106820]我的job_4现在时间:2022-12-30 10:24:00.106820, 将要睡眠300秒
[16024][ThreadPoolExecutor-0_4][2022-12-30 10:28:00.139789]我的job_1现在时间:2022-12-30 10:28:00.139789, 将要睡眠300秒
[15472][ThreadPoolExecutor-0_5][2022-12-30 10:28:00.174180]我的job_2现在时间:2022-12-30 10:28:00.174180, 将要睡眠300秒
[17628][ThreadPoolExecutor-0_6][2022-12-30 10:28:00.224052]我的job_3现在时间:2022-12-30 10:28:00.224052, 将要睡眠300秒
[22412][ThreadPoolExecutor-0_7][2022-12-30 10:28:00.291115]我的job_4现在时间:2022-12-30 10:28:00.291115, 将要睡眠300秒
[284][ThreadPoolExecutor-0_0][2022-12-30 10:29:00.015293]我的job_1现在时间:2022-12-30 10:29:00.015293, 睡眠300秒完成
[27032][ThreadPoolExecutor-0_1][2022-12-30 10:29:00.045368]我的job_2现在时间:2022-12-30 10:29:00.045368, 睡眠300秒完成
[15468][ThreadPoolExecutor-0_2][2022-12-30 10:29:00.076542]我的job_3现在时间:2022-12-30 10:29:00.076542, 睡眠300秒完成
[6812][ThreadPoolExecutor-0_3][2022-12-30 10:29:00.107325]我的job_4现在时间:2022-12-30 10:29:00.107325, 睡眠300秒完成

[284][ThreadPoolExecutor-0_0][2022-12-30 10:32:00.176150]我的job_1现在时间:2022-12-30 10:32:00.176150, 将要睡眠300秒
[15468][ThreadPoolExecutor-0_2][2022-12-30 10:32:00.206857]我的job_2现在时间:2022-12-30 10:32:00.206857, 将要睡眠300秒
[27032][ThreadPoolExecutor-0_1][2022-12-30 10:32:00.256790]我的job_3现在时间:2022-12-30 10:32:00.256790, 将要睡眠300秒
[6812][ThreadPoolExecutor-0_3][2022-12-30 10:32:00.315379]我的job_4现在时间:2022-12-30 10:32:00.315379, 将要睡眠300秒
[16024][ThreadPoolExecutor-0_4][2022-12-30 10:33:00.152695]我的job_1现在时间:2022-12-30 10:33:00.152695, 睡眠300秒完成
[15472][ThreadPoolExecutor-0_5][2022-12-30 10:33:00.183775]我的job_2现在时间:2022-12-30 10:33:00.183775, 睡眠300秒完成
[17628][ThreadPoolExecutor-0_6][2022-12-30 10:33:00.230206]我的job_3现在时间:2022-12-30 10:33:00.230206, 睡眠300秒完成
[22412][ThreadPoolExecutor-0_7][2022-12-30 10:33:00.305963]我的job_4现在时间:2022-12-30 10:33:00.305963, 睡眠300秒完成
[16024][ThreadPoolExecutor-0_4][2022-12-30 10:36:00.172009]我的job_1现在时间:2022-12-30 10:36:00.172009, 将要睡眠300秒
[17628][ThreadPoolExecutor-0_6][2022-12-30 10:36:00.210402]我的job_2现在时间:2022-12-30 10:36:00.210402, 将要睡眠300秒
[15472][ThreadPoolExecutor-0_5][2022-12-30 10:36:00.269251]我的job_3现在时间:2022-12-30 10:36:00.269251, 将要睡眠300秒
[22412][ThreadPoolExecutor-0_7][2022-12-30 10:36:00.318878]我的job_4现在时间:2022-12-30 10:36:00.318878, 将要睡眠300秒

"""
```

##### 说明：

使用10个线程运行

运行时间['24', '28', '32', '36', '40', '44', '48', '52', '56']

24分开始运行①②③④任务，需要运行到29分

28分开始第二轮，还有剩余线程可用，且max_instances=2，则再次运行①②③④任务，运行到33分

29分运行结束第一轮任务，此时任务都运行结束了，都不miss，等待下一轮调度。

32分开始第三轮任务，至此与24分运行一样，以此类推

#### 个人总结

先看线程个数，然后每个线程一个任务，默认执行任务的顺序应该是先按任务执行时间排序，同一时间，按添加顺序排序，执行完后，释放出线程，线程去看此时是否还有需要执行的任务，如果有，则看misfire_grace_time参数，如果miss时间小于misfire_grace_time参数，或者misfire_grace_time是None，则线程执行对应任务，否则抛出was missed by异常，且任务执行时间变成下一次任务时间

当线程空闲，且需要执行任务，会先看max_instances参数，决定是否执行，max_instances表示最大执行的数量，即一个任务在运行中，又遇到要执行该任务了，看最大可运行该任务的次数，次数超过max_instances则抛出异常，否则同时运行

如果总是遇到was missed by 异常，则可以考虑添加多个线程执行，多个进程执行，默认是10个线程执行，官网提到。