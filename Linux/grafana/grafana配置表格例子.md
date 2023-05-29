grafana配置表格例子

# 前言

使用了node_exporter采集主机数据和Prometheus数据源来监控主机信息

# 需求

要查询机器的运行时间，以表格的形式显示

# 步骤

1. 先把Prometheus的查询语句写出来，可以直接使用code模式手动写，也可以使用面板的builder模式

   求运行时间就用`node_time_seconds - on(instance) node_boot_time_seconds`

2. 调整右边显示为表格显示
3. 调整下方的format为表格，type为instant
4. 完善一下其他信息

添加看板

![image-20230526151459105](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20230526151459105.png)

添加面板

![image-20230526151520923](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20230526151520923.png)

选择builder模式，先build一下查询语句

![image-20230526153637387](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20230526153637387.png)

选择code模式，点击Run queries就是查询出来数据了

![image-20230526153526935](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20230526153526935.png)

选择表格显示

![image-20230526154049314](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20230526154049314.png)

选择格式化为表格显示

![image-20230526154303436](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20230526154303436.png)

查询点时间，而非范围时间就显示正常咯

但是我们一般都是显示查询范围时间，因此需要适配

![image-20230526154453698](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20230526154453698.png)

设置type为instant就可以保证上面的适配

![image-20230526154901824](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20230526154901824.png)

调整表格显示，完善其他信息

![image-20230526161322883](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20230526161322883.png)

最终效果

![image-20230526161416336](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20230526161416336.png)

# 参考链接

https://zhuanlan.zhihu.com/p/336074453