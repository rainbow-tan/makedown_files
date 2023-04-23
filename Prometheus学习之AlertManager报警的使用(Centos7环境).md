Prometheus学习之AlertManager报警的使用(Centos7环境)

## 1、下载、解压、启动

网址：[Prometheus官网](https://prometheus.io/download/#alertmanager)

![image-20220728185025873](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220728185025873.png)

例如下载0.24.0版本并启动（启动可以自己配置为服务启动）

```sh
wget https://github.com/prometheus/alertmanager/releases/download/v0.24.0/alertmanager-0.24.0.linux-amd64.tar.gz
tar -zxvf alertmanager-0.24.0.linux-amd64.tar.gz
cd alertmanager-0.24.0.linux-amd64
./alertmanager
```

验证：访问9093端口 http://172.16.131.28:9093/ （需要替换自己的IP）（如果访问失败把防火墙禁用后再试一试）

![image-20220728185336870](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220728185336870.png)

## 2、AlertManager配置分解

AlertManager 加载的配置文件就在解压的当前目录下，文件为 alertmanager.yml，有以下几部分组成，这里简单讲解一下（网上抄来的）

#### ①global部分

global：全局配置，包括报警解决后的超时时间、SMTP 相关配置、各种渠道通知的 API 地址等等。

```yaml
global: #全局配置
  resolve_timeout: 5m                           # 在接下来的这个时间内，如果没有此报警信息触发，就发送报警解除消息。
  smtp_smarthost: 'smtp.qq.com:587'             # smtp 地址
  smtp_from: '1310693853@qq.com'                # 发件人邮箱地址
  smtp_auth_username: '1310693853@qq.com'       # 发件人的登陆用户名，默认和发件人邮箱一致
  smtp_auth_password: 'ryholnnjswce'            # 邮箱客户端授权码
  smtp_require_tls: true                        # 是否需要tls协议。默认是true
```

#### ②route部分

route: 用来设置报警的分发策略，它是一个树状结构，按照深度优先从左向右的顺序进行匹配。

```yaml
routes: # 接收器
  - receiver: 'db'                           #接收者是receivers中定义的db
    group_wait: 10s                          #当收到报警时，如果同组内，10秒内出现相同报警，则会合并为一条发送一起发出去
    match:
      class: 'db'                            #匹配到标签class=db就发出消息
  - receiver: 'web'                          #接收者是receivers中定义的web
    group_wait: 10s                          #当收到报警时，如果同组内，10秒内出现相同报警，则会合并为一条发送一起发出去
    match:
      class: 'web'                          #匹配到标签class=web就发出消息
  - receiver: 'db_web'                      #接收者是receivers中定义的db_web
    group_wait: 10s                         #当收到报警时，如果同组内，10秒内出现相同报警，则会合并为一条发送一起发出去
    match:
      class: 'db_web'                       #匹配到标签class=db_web就发出消息
  - receiver: 'db_web'                      #接收者是receivers中定义的db_web
    group_wait: 10s                         #当收到报警时，如果同组内，10秒内出现相同报警，则会合并为一条发送一起发出去
      match_re:                             #匹配正则
        class: db|web                       #匹配到标签class=db或者class=web就发出消息
```

分配策略匹配比较考验逻辑能力，可以去[routing-tree-editor](https://www.prometheus.io/webtools/alerting/routing-tree-editor/)网址来生成路由树，然后输入标签来匹配，验证遇到哪些标签，会发送消息给哪些人，使用方法很简单，就是把 `alertmanager.yml` 的配置信心复制到这个站点，然后点击 `Draw Routing Tree` 按钮生成路由结构树， 然后在 `Match Label Set` 前面输入以 `{<label name> = "<value>"}` 格式的警报标签，然后点击 `Match Label Set` 按钮会显示发送状态图。

#### ③receivers部分

receivers: 配置告警消息接受者信息，例如常用的 email、wechat、slack、webhook 等消息通知方式。 

```yaml
receivers: #定义接收者列表，将告警发送给谁。
  - name: 'web.hook'                          #接收者的唯一名称。
    webhook_configs: #通知方式
      - url: 'http://127.0.0.1:5001/'         #webhook地址

  - name: 'db'                                #接收者的唯一名称。
    email_configs: #通知方式
      - to: '1150646501@qq.com'               # 收件人邮箱地址
        send_resolved: true                   # 当报警解除时是否发送邮件通知，默认为false

  - name: 'web'                               #接收者的唯一名称。
    email_configs: #通知方式
      - to: '1310693853@qq.com'               # 收件人邮箱地址
        send_resolved: true                   # 当报警解除时是否发送邮件通知，默认为false

  - name: 'db_web'                            #接收者的唯一名称。
    email_configs: #通知方式
      - to: '1150646501@qq.com,1310693853@qq.com'   # 收件人邮箱地址
        send_resolved: true                          # 当报警解除时是否发送邮件通知，默认为false
```

#### ④inhibit_rules部分

inhibit_rules: 抑制规则配置，当存在与另一组匹配的警报（源）时，抑制规则将禁用与一组匹配的警报（目标）。

```yaml
inhibit_rules: #报警抑制
  - source_match: #源警报
      severity: 'critical'                                  #自定义标签报警级别为 critical
    target_match: # 目标警报
      severity: 'warning'                                   #报警级别为 warning
    equal: [ 'alertname', 'dev', 'instance' ]                 #源警报和目标警报中必须具有相等值的标签才能使抑制生效。

# 在alertname、dev、instance 三个标签的值相同情况下，severity为critaical 的报警会抑制 warning 级别的报警信息。
# 匹配所有severity为critical的报警和severity为warning的报警，当alertname、dev、instance 三个标签的值相同时，critical抑制warning的，
# 只会发送critical的，而不发送warning的告警
```

整体配置就是这样子

![image-20220812172529110](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220812172529110.png)

配置好 alertmanager.yml记得重启 alertmanager服务

## 3、与Prometheus集成

启动了AlertManager服务后，需要与Prometheus集成，这样才可以处理Prometheus生产的报警信息。原理可自行百度，当Prometheus中产生了报警信息后，会发送到AlertManager服务中，由AlertManager进行后续处理，AlertManager会对Prometheus产生的报警消息进行分组、去重、抑制、静默，然后以邮件的方式或者webhook的方式通知到负责人。

编辑Prometheus配置文件prometheus.yml,并添加以下内容，就可以在Prometheus中集成AlertManager

```yaml
alerting:
  alertmanagers:
    - static_configs:
        - targets: ['localhost:9093']
```

然后重启Prometheus即可

## 4、配置Prometheus的告警规则

Prometheus才是生成告警的源头，产生消息是Prometheus，而AlertManager是处理告警信息的服务，因此，我们需要配置告警的规则

step1：在Prometheus中配置告警的规则文件

vim /home/prometheus-2.37.0.linux-amd64/rules/first.yml

```yaml
groups:
  - name: Nano Vm Warning message        #组的名称
    rules: #规则
      - alert: up status             # 报警名称
        expr: up == 0           # 要计算的PromQL表达式
        for: 1m                # 告警持续时间，超过这个时间才会推送给 Alertmanager
        labels: # 指定要附加到警报中的标签，冲突标签将被覆盖。标签值可以被模板化。
          severity: warning      # 报警等级标签（自定义的）
        annotations: # 添加到每个警报的注释，例如警报描述或运行手册链接。注释值可以被模板化。
          summary: {{$labels.instance}}: 服务宕机      # 概况（自定义的）
          description: {{$labels.instance}}: 服务宕机超过了一分钟     # 描述（自定义的）


#$labels   # 此变量保存警报实例的标签键/值对。用法：{{ $labels.<labelname> }}
#$value    # 变量保存警报实例的评估值。用法：{{ $value }}
```

****配置好告警规则后，需要与Prometheus集成，只需要修改prometheus.yml配置文件即可

```yaml
rule_files:
  - "/home/prometheus-2.37.0.linux-amd64/rules/*.yml"
```

可以直接指定某个文件名，也可以使用正则来表示rule_files，这样Prometheus才可以根据配置文件定时监测，产生告警信息，发送给AlertManager

## 5、Prometheus配置文件示例

贴出一份AlertManager与Prometheus集成的配置文件prometheus.yml

![image-20220815105421391](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220815105421391.png)

## 6、自定义模板

#### 默认的样式

我们如果不自定义模板，默认收到的邮件是这样子的(单个警报)

![image-20220815144655367](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220815144655367.png)

和这样子的（多个警报）

![image-20220815153103450](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220815153103450.png)

我们可以自定义这个模板

#### 参考的网上模板

（1）修改AlertManager的配置文件alertmanager.yml，添加模板的路径，可以直接指向某个具体文件，也可以使用通配符

```yaml
templates:
  - '/home/alertmanager-0.24.0.linux-amd64/template/*.tmpl'
```

(2)在指定的路径下，编写自己的模板

vim /home/alertmanager-0.24.0.linux-amd64/template/custom.tmpl

```yaml
{{ define "custom.email.template" }}
{{- if gt (len .Alerts.Firing) 0 -}}{{ range.Alerts }}
告警名称: {{ .Labels.alertname }} <br>
实例名: {{ .Labels.instance }}  <br>
摘要:  {{ .Annotations.summary }} <br>
详情:  {{ .Annotations.description }} <br>
级别:  {{ .Labels.severity }}  <br>
开始时间:  {{ (.StartsAt.Add 28800e9).Format "2006-01-02 15:04:05" }}<br>
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++<br>
{{ end }}{{ end -}}
{{- if gt (len .Alerts.Resolved) 0 -}}{{ range.Alerts }}
Resolved-告警恢复了。<br>
告警名称: {{ .Labels.alertname }} <br>
实例名: {{ .Labels.instance }}  <br>
摘要:  {{ .Annotations.summary }} <br>
详情:  {{ .Annotations.description }} <br>
级别:  {{ .Labels.severity }}  <br>
开始时间:  {{ (.StartsAt.Add 28800e9).Format "2006-01-02 15:04:05" }}<br>
恢复时间:  {{ (.EndsAt.Add 28800e9).Format "2006-01-02 15:04:05" }}<br>
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++<br>
{{ end }}{{ end -}}
{{- end }}
```

(3)在AlertManager的配置文件alertmanager.yml中指定使用该模板

上面定义了`custom.email.template`，所有模板中就使用`html: '{{template "custom.email.template" . }}'`

```yaml
receivers: #定义接收者列表，将告警发送给谁。
  - name: 'db'                                #接收者的唯一名称。
    email_configs: #通知方式
      - to: '1150646501@qq.com'               # 收件人邮箱地址
        send_resolved: true                   # 当报警解除时是否发送邮件通知，默认为false
        html: '{{template "custom.email.template" . }}'
```

(4)重启AlertManager

邮件就变成了这样子的（单个警报）

![image-20220815180119181](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220815180119181.png)

和这样子的（多个警报）

![image-20220815175527868](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220815175527868.png)

但是恢复的邮件感觉有些问题，这里不打算再研究了，学习的链接是：https://www.jianshu.com/p/654d59325550

#### 自定义模板-自己

后续研究了一下go template的语法，因为这里的模板就是基于go template的语法而来，然后根据官网给出的介绍，进行自定义模板

官网：https://prometheus.io/docs/alerting/latest/notifications/#data

自己学习的go template链接：https://www.cnblogs.com/rainbow-tan/p/16599512.html

从官网可以看得出传入的参数就是一个结构体对象，就是Data，然后根据属性来自定义呗

```go
{{ define "custom.email.template" }}
{{ if gt (len .Alerts.Firing) 0 }}
--------------------触发的告警--------------------<br>
{{ range .Alerts }}
{{  if eq .Status "firing" }}
告警名称:  {{ .Labels.alertname }}<br>
实例对象:  {{ .Labels.instance }}<br>
告警摘要:  {{ .Annotations.summary }} <br>
告警详情:  {{ .Annotations.description }} <br>
开始时间:  {{ (.StartsAt.Add 28800e9).Format "2006-01-02 15:04:05" }}<br>
标签详情：  <br>
{{ range $k, $v := .Labels }}
{{ $k }} = {{ $v }}<br>
{{ end }}
<br>
{{ end }}
{{ end }}
{{ end }}
{{ if gt (len .Alerts.Resolved) 0 }}
--------------------已解决告警--------------------<br>
{{ range  .Alerts  }}
{{  if eq .Status "resolved" }}
告警名称:  {{ .Labels.alertname }}<br>
实例对象:  {{ .Labels.instance }}<br>
告警摘要:  {{ .Annotations.summary }} <br>
告警详情:  {{ .Annotations.description }} <br>
开始时间:  {{ (.StartsAt.Add 28800e9).Format "2006-01-02 15:04:05" }}<br>
恢复时间:  {{ (.EndsAt.Add 28800e9).Format "2006-01-02 15:04:05" }}<br>
标签详情：  <br>
{{ range $k, $v := .Labels }}
{{ $k }} = {{ $v }}<br>
{{ end }}
<br>
{{ end }}
{{ end }}
{{ end }}
{{ end }}
```

一个警报样式

![image-20220819103938825](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220819103938825.png)

多个警报样式

![image-20220819105029966](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220819105029966.png)

解决的警报样式

![image-20220819105203327](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220819105203327.png)

## 7、一些自己使用的告警规则

这些都是网上抄来的，自己测试了一下

#### CPU使用率

```yaml
groups:
- name: Host
  rules:
  - alert: "CPU监测"
    expr: 100 * (1 - avg(irate(node_cpu_seconds_total{mode="idle"}[2m])) by(instance)) > 10
    for: 1m
    labels:
      severity: warning
    annotations:
      summary: "{{$labels.instance}}: CPU高使用率监测"
      description: "{{$labels.instance}}:CPU使用率是:{{$value}}%, 超过了10%, 已经持续了1分钟了"
```

模拟提高CPU使用率可以使用命令`cat /dev/urandom | md5sum`

#### 宕机监测

```yaml
groups:
  - name: Nano Vm Warning message        #组的名称
    rules: #规则
      - alert: "宕机监测"             # 报警名称
        expr: up == 0           # 要计算的PromQL表达式
        for: 10s                # 告警持续时间，超过这个时间才会推送给 Alertmanager
        labels: # 指定要附加到警报中的标签，冲突标签将被覆盖。标签值可以被模板化。
          severity: warning      # 报警等级标签（自定义的）
        annotations: # 添加到每个警报的注释，例如警报描述或运行手册链接。注释值可以被模板化。
          summary: "{{$labels.instance}}: 服务宕机"      # 概况（自定义的）
          description: "{{$labels.instance}}: 服务宕机超过了一分钟"     # 描述（自定义的）


#$labels   # 此变量保存警报实例的标签键/值对。用法：{{ $labels.<labelname> }}
#$value    # 变量保存警报实例的评估值。用法：{{ $value }}
```

#### 内存使用率

```yaml
groups:
- name: Host
  rules:
  - alert: "内存监测"
    expr: (node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes) / node_memory_MemTotal_bytes * 100 > 20
    for: 1m
    labels:
        severity: middle
    annotations:
        summary: "{{$labels.instance}}: 内存高使用率监测"
        description: "{{$labels.instance}}: 内存使用率是:{{ $value }}%, 超过了20%, 已经持续了1分钟了"
```

模拟提高内存使用率可以参考：https://blog.csdn.net/heavenmark/article/details/82805260 恢复内存就重启机器吧

#### 磁盘使用率监测

```yaml
groups:
- name: Host
  rules:
  - alert: "磁盘监测"
    expr: 100 * (node_filesystem_size_bytes{fstype=~"xfs|ext4"} - node_filesystem_avail_bytes) / node_filesystem_size_bytes > 10
    for: 1m
    labels:
      severity: warning
    annotations:
      summary: "{{$labels.instance}}: 磁盘高使用率监测"
      description: "{{$labels.instance}}:磁盘使用率是:{{$value}}%, mountpoint:{{$labels.mountpoint}}, 超过了10%, 已经持续了1分钟了"
```

参考：

[Prometheus Alertmanager 报警（四）](https://www.cpweb.top/1923)

[Prometheus监控神器-Alertmanager篇(1)](https://zhuanlan.zhihu.com/p/179292686)

[prometheus+alertmanager实现CPU、内存、磁盘的监控报警](https://blog.csdn.net/xiaoxiangzi520/article/details/115005765)

[Prometheus-alertmanager告警规则模板](https://www.kococ.cn/20191215/cid=683.html)



