apscheduler mysql 持久化任务

## 1、下载第三方包

这里使用pymysql连接mysql

```
pip install apscheduler

pip install pymysql

pip install sqlalchemy
```

## 2、直接参考代码

```python
import datetime

from apscheduler.executors.pool import ThreadPoolExecutor, ProcessPoolExecutor
from apscheduler.jobstores.sqlalchemy import SQLAlchemyJobStore
from apscheduler.schedulers.blocking import BlockingScheduler

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
    'default': ThreadPoolExecutor(20),
    'processpool': ProcessPoolExecutor(5)
}
job_defaults = {
    'coalesce': True,  # 堆积后只执行最后一个
    'max_instances': 1,  # 最大的实例只能存在一个

}
scheduler = BlockingScheduler(executors=executors, job_defaults=job_defaults, jobstores=job_stores,
                              timezone='Asia/Shanghai')


def my_job():
    print(f"现在时间:{datetime.datetime.now()}")


scheduler.add_job(my_job, 'cron', id='my_job', second='*/10', timezone='Asia/Shanghai',
                  replace_existing=True)  # 10秒运行一次
scheduler.start()

```

数据库

![image-20221226141038846](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20221226141038846.png)

## 3、一些个人测试情况供参考



## 参考链接：

https://apscheduler.readthedocs.io/en/3.x/userguide.html#code-examples

https://www.jianshu.com/p/e36236c1df08

https://blog.csdn.net/wyw875960418/article/details/107977717

https://www.moban555.com/article/1270071.html

https://blog.csdn.net/hqzxsc2006/article/details/89950695