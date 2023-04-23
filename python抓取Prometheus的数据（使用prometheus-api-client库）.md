python抓取Prometheus的数据（使用prometheus-api-client库）

## 0、写在前面

我们要想抓取Prometheus的数据，一般想到的就是requests请求，爬虫的方式来抓取，这是可行的，当然，还有一个第三方库直接封装好了，直接用就行，代码也比较少，源码点进去就能看明白，这个库叫`prometheus-api-client`，[github地址](https://github.com/4n4nd/prometheus-api-client-python)和 [pypi地址](https://pypi.org/project/prometheus-api-client/)

## 1、下载

```sh
python -m pip install prometheus-api-client
```

## 2、使用

### 连接Prometheus

使用`PrometheusConnect`进行连接，使用check_prometheus_connection()检查连接状态

例如

```python
from prometheus_api_client import PrometheusConnect

prom = PrometheusConnect(url="http://172.17.140.17:9090", headers=None, disable_ssl=True)
ok = prom.check_prometheus_connection()  # 检查连接状态
print(f"连接Prometheus:{prom.url}, 状态:{'连接成功' if ok else '连接失败'}")
"""
url - (str) url for the prometheus host
headers – (dict) A dictionary of http headers to be used to communicate with the host. Example: {“Authorization”: “bearer my_oauth_token_to_the_host”}
disable_ssl – (bool) If set to True, will disable ssl certificate verification for the http requests made to the prometheus host

url Prometheus的连接url
如果Prometheus需要认证, 则需要在headers中添加认证 {“Authorization”: “bearer my_oauth_token_to_the_host”}
disable_ssl 是否禁止ssl认证 http就禁止 https 则需要验证
"""
```

运行

![image-20221021111459396](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20221021111459396.png)

### 获取所有的指标

使用`all_metrics(self, params: dict = None)`获取所有指标

底层是调用`get_label_values(self, label_name: str, params: dict = None)`传递的label_name为`__name__`

```python
all_metrics = prom.all_metrics()
print("------所有指标------")
for metrics in all_metrics:
    print(metrics)
print('------------------')
```

运行

![image-20221021111820817](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20221021111820817.png)

### 根据标签抓取值

使用`get_label_values(self, label_name: str, params: dict = None)`抓取,调用的接口是:/api/v1/label/{label_name}/values

可以传的值就是定义的标签，默认就有的是job,instance，然后自己可以定义

- 例如传入默认的标签

```python
label_name = "job"
params = {}
result = prom.get_label_values(label_name, params)
print(f"-------------------- 抓取的标签:{label_name} 参数:{params} --------------------")
for r in result:
    print(r)
"""
调用的接口是 /api/v1/label/{}/values
"""
```

运行

![image-20221021134146478](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20221021134146478.png)

对应的页面是

![image-20221021135424203](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20221021135424203.png)

- 例如传递自定义的标签

```python
label_name = "end_time"
params = None
result = prom.get_label_values(label_name, params)
print(f"-------------------- 抓取的标签:{label_name} 参数:{params} --------------------")
print(f"个数:{len(result)}")
for r in result:
    print(r)
"""
调用的接口是 /api/v1/label/end_time/values
"""
```

运行

![image-20221021135613873](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20221021135613873.png)

对应的页面

![image-20221021134710941](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20221021134710941.png)

还可以传递一个params参数，但不知道有什么用，我看[官网的测试案例](https://github.com/4n4nd/prometheus-api-client-python/blob/master/tests/test_prometheus_connect.py)也没举例，暂时忽略，源码参数描述是这样子的，传递一个时间？或者其他参数，带着一起访问，不过一般好像也用不到

`:param params: (dict) Optional dictionary containing GET parameters to be sentalong with the API request, such as "time"`

### 获取当前的指标数据

1. 使用`get_current_metric_value(self, metric_name, label_config, params)`获取，调用的接口是 /api/v1/query

例如

```python
query = "up"
label_config = {"user": "dev"}
values = prom.get_current_metric_value(query, label_config=label_config)
"""
调用的接口是 /api/v1/query
"""
for v in values:
    print(v)
```

运行

![image-20221021141527540](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20221021141527540.png)

对应的页面是

![image-20221021141550482](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20221021141550482.png)

2. 也可以使用`custom_query(self, query: str, params: dict = None)`获取，调用的接口是 /api/v1/query

例如：

```python
query = "up{user='dev'}"
values = prom.custom_query(query)
"""
调用的接口是 /api/v1/query
"""
for v in values:
    print(v)
```

运行和对应的页面同上，因为都是一个指标。

这两个函数的区别是：

`get_current_metric_value()`把`label_config`拆开拼接了一下，整合成一个完整的PQL。

而`custom_query(self, query: str, params: dict = None)`则直接传递一个PQL即可，后续不处理，复杂的PQL直接用这个就行

通过查看源码，一目了然

![image-20221021142906678](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20221021142906678.png)

获取指定时间间隔的数据

使用`get_metric_range_data()`获取，调用的接口是/api/v1/query_range

例如：抓取两天前的数据，以一天为单位，

```python
query = 'up{instance=~"172.16.90.22:9100"}'
start_time = parse_datetime("2d")
end_time = parse_datetime("now")
chunk_size = timedelta(days=1)
result = prom.get_metric_range_data(query, None, start_time, end_time, chunk_size)
"""
调用的接口是  /api/v1/query_range
"""
print(f'result:{result}')
with open('a.json', 'w') as f:
    json.dump(result, f)
```

运行

![image-20221021150650185](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20221021150650185.png)

对应的页面

![image-20221021150814146](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20221021150814146.png)

## 3、查看抓取的指标

要处理获取到的指标，一般都是直接遍历，然后处理，但官方提供了一个类叫**MetricsList**，它会初始化获取到的指标数据为指标对象**Metrics**，构成一个列表，可以方便的查看指标名称和标签。

```python
query = "up"
label_config = {"user": "dev"}
metric_data = prom.get_current_metric_value(query, label_config=label_config)
metric_object_list = MetricsList(metric_data)  
# metric_object_list will be initialized as
# a list of Metric objects for all the
# metrics downloaded using get_metric query

# We can see what each of the metric objects look like
for item in metric_object_list:
    print(f"指标名称:{item.metric_name}  标签:{item.label_config}")
```

运行

![image-20221021152443264](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20221021152443264.png)

## 4、使用pandas处理数据

先转换到pandas的DataFrames

使用`MetricSnapshotDataFrame`转换`get_current_metric_value`的数据

使用`MetricRangeDataFrame`转换`get_metric_range_data`的数据

然后就可以pandas处理数据了

```python
query = "up"
label_config = {"user": "dev"}
metric_data = prom.get_current_metric_value(query, label_config=label_config)
frame = MetricSnapshotDataFrame(metric_data)
head = frame.head()
print(head)
print("-" * 20)
query = 'up{instance=~"172.16.90.22:9100"}'
start_time = parse_datetime("2d")
end_time = parse_datetime("now")
chunk_size = timedelta(days=2)
result = prom.get_metric_range_data(query, None, start_time, end_time, chunk_size)
frame2 = MetricRangeDataFrame(result)
print(frame2.head())
```

运行

![image-20221021154811067](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20221021154811067.png)

最后贴一个所有的代码集合，方便查看

```python
import json
from datetime import timedelta

from prometheus_api_client import PrometheusConnect, MetricSnapshotDataFrame, MetricRangeDataFrame, Metric, MetricsList
from prometheus_api_client.utils import parse_datetime

prom = PrometheusConnect(url="http://172.17.140.17:9090", headers=None, disable_ssl=True)
ok = prom.check_prometheus_connection()  # 检查连接状态
print(f"连接Prometheus:{prom.url}, 状态:{'连接成功' if ok else '连接失败'}")
"""
url - (str) url for the prometheus host
headers – (dict) A dictionary of http headers to be used to communicate with the host. Example: {“Authorization”: “bearer my_oauth_token_to_the_host”}
disable_ssl – (bool) If set to True, will disable ssl certificate verification for the http requests made to the prometheus host

url Prometheus的连接url
如果Prometheus需要认证, 则需要在headers中添加认证 {“Authorization”: “bearer my_oauth_token_to_the_host”}
disable_ssl 是否禁止ssl认证 http就禁止 https 则需要验证
"""
all_metrics = prom.all_metrics()
print("------所有指标------")
for metrics in all_metrics:
    print(metrics)
print('------------------')

label_name = "end_time"
params = None
result = prom.get_label_values(label_name, params)
print(f"-------------------- 抓取的标签:{label_name} 参数:{params} --------------------")
print(f"个数:{len(result)}")
for r in result:
    print(r)
"""
调用的接口是 /api/v1/label/end_time/values
"""

query = "up"
label_config = {"user": "dev"}
values = prom.get_current_metric_value(query, label_config=label_config)
"""
调用的接口是 /api/v1/query
"""
for v in values:
    print(v)
print('-' * 20)
query = "up{user='dev'}"
values = prom.custom_query(query)
"""
调用的接口是 /api/v1/query
"""
for v in values:
    print(v)

query = 'up{instance=~"172.16.90.22:9100"}'
start_time = parse_datetime("2d")
end_time = parse_datetime("now")
chunk_size = timedelta(days=2)
result = prom.get_metric_range_data(query, None, start_time, end_time, chunk_size)
"""
调用的接口是  /api/v1/query_range
"""
print(f'result:{result}')
with open('a.json', 'w') as f:
    json.dump(result, f)

query = "up"
label_config = {"user": "dev"}
metric_data = prom.get_current_metric_value(query, label_config=label_config)
metric_object_list: list[Metric] = MetricsList(metric_data)
# metric_object_list will be initialized as
# a list of Metric objects for all the
# metrics downloaded using get_metric query

# We can see what each of the metric objects look like
for item in metric_object_list:
    print(f"指标名称:{item.metric_name}  标签:{item.label_config}")

query = "up"
label_config = {"user": "dev"}
metric_data = prom.get_current_metric_value(query, label_config=label_config)
frame = MetricSnapshotDataFrame(metric_data)
head = frame.head()
print(head)
print("-" * 20)
query = 'up{instance=~"172.16.90.22:9100"}'
start_time = parse_datetime("2d")
end_time = parse_datetime("now")
chunk_size = timedelta(days=2)
result = prom.get_metric_range_data(query, None, start_time, end_time, chunk_size)
frame2 = MetricRangeDataFrame(result)
print(frame2.head())
```