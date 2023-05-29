go template模板的语法

### 当前对象

#### {{ . }} 表示当前对象，当前对象针对于作用域而言

- 例子1：当前传入字符串对象，{{ . }}直接表示字符串

![image-20220818143448916](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220818143448916.png)

这里的{{ . }}就是传入的hello world

- 例子2：当前传入结构体对象，{{ . }} 表示结构体  {{ .Name }}和{{ .Age }}获取结构体属性

![image-20220818143845275](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220818143845275.png)

这里的{{ . }} 就是匿名结构体 {{ .Name }} 获取姓名 {{ .Age }}获取年龄

如果解析错误，则终止解析，例如结构体没有.Sex属性，而使用{{ .Sex }}获取Sex属性

- 错误示例：尝试解析没有存在的结构体属性会导致解析异常，终止解析

![image-20220818144245257](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220818144245257.png)

### 移除空白

#### {{- . }}移除前面的空白，数据本身的空白不移除

#### {{- . }}移除后面的空白，数据本身的空白不移除

#### {{- . -}}移除前面和后面的空白，数据本身的空白不移除

演示的时候用apipost模拟，因为浏览器会自动忽略空格

- 例子1：正常有空格的情况

![image-20220818145359307](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220818145359307.png)

- 例子2：去除左边的空格

![image-20220818145553732](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220818145553732.png)

- 例子3：去除右边的空格

![image-20220818145642124](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220818145642124.png)

- 例子4：去除左右两边的空格

![image-20220818145747666](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220818145747666.png)

- 例子5：数据本身的空格不会去掉

![image-20220818150040795](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220818150040795.png)

### 注释

#### {{/* 这是注释 */}} 表示注释

- 例子1：注释也会占一行

![image-20220818150521226](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220818150521226.png)

- 例子2：移除注释的左空白

![image-20220818150942253](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220818150942253.png)

- 例子3：移除注释的右空白

![image-20220818151056369](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220818151056369.png)

- 例子4：不能同时移除注释的左右空白

![image-20220818151218666](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220818151218666.png)

### 变量和赋值

#### {{ $var := 123 }} 定义变量var 值是123

#### {{ $var = 456 }} 把变量var改为456

一般写成{{- $var := 123 }}和{{- $var := 123 }}去除空白

- 例子1：正常定义与赋值

![image-20220818153511955](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220818153511955.png)

### 判断

{{ if pipeline }} T1 {{ end }}

{{ if pipeline }} T1 {{ else }} T0 {{ end }}

{{ if pipeline }} T1 {{ else if pipeline }} T0 {{ end }}

{{ if pipeline }} T2 {{ else if pipeline }} T1 {{ else }}T0 {{ end }}

例子1：

![image-20220818155236374](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220818155236374.png)

### 遍历

#### {{ range $k, $v := . }} {{ $k }} => {{ $v }} {{ end }} 遍历map时是key和value，遍历数组和切片时，是index和value

- 例子1：遍历map

![image-20220818160251346](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220818160251346.png)

- 例子2：遍历切片

![image-20220818160359038](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220818160359038.png)

#### {{ range $k, $v := . }} {{ $k }} => {{ $v }} {{ else }}T0{{ end }} 在上面的基础上添加了一个else块

#### 例子3：传入的map或者切片或数组长度为0的情况

![image-20220818160821364](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220818160821364.png)

### 内置函数

#### eq arg1 arg2 判断相等

#### ne arg1 arg2 判断不等

#### lt arg1 arg2 小于

#### le arg1 arg2 小于等于

#### gt arg1 arg2 大于

#### ge arg1 arg2 大于等于

#### len arg1 arg2 返回长度

#### print go语言中的fmt.Sprint

#### printf go语言中的Sprintf

#### println go语言中的Sprintln

### 定义模板

可以把代码块定义为一个模板，方便后面的嵌套，重用

{{ define "template1" }}{{ end }} 定义模板名称为template1

{{template “template1” . }} 使用模板template1，传入当前的对象进去，对象就是 {{ . }}

![image-20220818163804825](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220818163804825.png)

参考：https://www.cnblogs.com/f-ck-need-u/p/10053124.html