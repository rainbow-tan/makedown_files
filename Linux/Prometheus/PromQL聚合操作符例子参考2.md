# PromQL聚合操作符例子参考2

[参考链接](https://www.prometheus.wang/promql/prometheus-promql-operators-v2.html)

## by

### 含义

根据标签分组，然后使用聚合函数

### 例子

count(up)by(job)

根据job标签分组，然后再求出分组后的各样本数量

![image-20230523113135814](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20230523113135814.png)

## without

### 含义

排除这些标签后，然后分组，与by正好相反

### 例子

![image-20230523135833729](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20230523135833729.png)

### by与without举例

假设标签有a,b,c,d

sum()by(a)则是根据a分组，然后求和，等同于sum()without(b,c,d)

sum()by(a,b)则是根据a,b分组，然后求和，等同于sum()without(c,d)

## on

### 含义

根据给定的标签，匹配查询的样本，组合查询指标

### 例子

node_memory_MemTotal_bytes

![image-20230523143345318](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20230523143345318.png)

 node_filesystem_files{device=~"/dev/sda1|/dev/sda2"}

![image-20230523143354329](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20230523143354329.png)

 如果直接查询node_filesystem_files{device=~"/dev/sda1|/dev/sda2"} and  node_memory_MemTotal_bytes

![image-20230523143631419](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20230523143631419.png)

指定通过user聚合

![image-20230523143750131](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20230523143750131.png)

保留的标签是前一个指标的标签 ![image-20230523143842450](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20230523143842450.png)

 算术运算符也可以

![image-20230523144236577](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20230523144236577.png)

 CPU使用率和磁盘IO均低于2%

![image-20230523145936899](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20230523145936899.png)

###  备注

最终保留下来的标签是左边的指标的标签

与on对应的是ignoreing，用法和by与without一样

## ignoreing

### 含义

排除给定的标签，然后匹配查询的样本，组合查询指标

### 例子

不举例了，参考【用法和by与without一样】

