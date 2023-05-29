# PromQL聚合操作符例子参考1

[参考链接](https://www.prometheus.wang/promql/prometheus-promql-operators-v2.html)

## sum

### 含义

sum是求和操作，就是把查询到样本的值加起来

### 例子

查询up状态，返回的样本的值是0,0,1,1

![image-20230523100847038](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20230523100847038.png)

使用sum求和，返回的样本的值是0+0+1+1=2

![image-20230523100944988](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20230523100944988.png)

## min & max

### 含义

max和min表示返回查询结果中的最大值和最小值

### 例子

查询node_filesystem_size_bytes

![image-20230523101656208](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20230523101656208.png)

min返回最小值

![image-20230523101855282](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20230523101855282.png)

max返回最大值

![image-20230523101916548](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20230523101916548.png)

## avg

### 含义

avg是求平均数操作，就是把查询到样本的值加起来除以样本个数=sum()/count()

### 例子

使用avg(up)求平均数，就是(0+0+1+1)/4=0.5

![image-20230523102335644](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20230523102335644.png)

## count

### 含义

返回样本的个数（返回Prometheus查询出来的条数）

### 例子

使用count(up)，则返回就是4，即4行数据，4个样本，因为up返回的就是4行数据

![image-20230523102517932](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20230523102517932.png)

## count_values

### 含义

不同于count，count_values返回的是样本的值的个数统计

count返回的是样本的个数，count_values返回的是样本的值的个数

### 用法

count_values(自定义的标签,指标)

第一个参数是自定义的标签，表示这次聚合后的含义

第二个参数就是实际的指标了

### 例子

统计up状态各样本值个数，即启动的和下线的各有多少

![image-20230523104253339](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20230523104253339.png)

### 例子2

![image-20230523104837733](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20230523104837733.png)

## topk和bottomk

### 含义

topk表示前几名

bottomk表示后几名

### 例子

前3名

![image-20230523105517510](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20230523105517510.png)

后两名

![image-20230523105550461](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20230523105550461.png)