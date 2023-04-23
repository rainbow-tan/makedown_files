Docker容器的使用例子

### 拉取镜像

```
docker pull centos
```

### 查看镜像

```
docker images
```

### 启动一个交互的容器

```
docker run -it centos:latest /bin/bash
```

参数说明：

- **-i**: 交互式操作。
- **-t**: 终端。
- **centos:latest**: centos镜像。
- **/bin/bash**：放在镜像名后的是命令，这里我们希望有个交互式 Shell，因此用的是 /bin/bash。

要退出终端，直接输入 **exit**:

![1654844841370](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\1654844841370.png)

### 启动一个后台运行的容器

```
docker run -itd centos:latest /bin/bash
```

### 查看运行的容器

```
docker ps
```

### 查看所有容器

```
docker ps -a
```

![1654845131554](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\1654845131554.png)

### 停止运行中的容器

```
docker stop lucid_hugle
```

参数说明：

- `lucid_hugle` 容器的名字（也可以使用容器ID:9314ba033c11）

### 重启停止的容器

```
docker start lucid_hugle
```

参数说明：

- `lucid_hugle` 容器的名字（也可以使用容器ID:9314ba033c11）

### --name 指定容器名称

```
docker run -itd --name centos_name centos:latest /bin/bash
```

![1654845391232](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\1654845391232.png)

### 进入运行的容器

在使用 **-d** 参数时，容器启动后会进入后台。此时想要进入容器，可以通过以下两个指令进入：

### **（1）docker attach**

使用docker attach进入容器，输入exit退出容器时，会导致容器停止

```
 docker attach hungry_brattain
```

![1654858669292](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\1654858669292.png)

### **（2）docker exec**（推荐）

使用docker exec进入容器，输入exit退出容器时，容器不会停止

```
docker exec -it 243c32535da7 /bin/bash
```

![1654858617060](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\1654858617060.png)

### 导出容器快照

```
docker export hungry_brattain > /home/my_hungry_brattain.tar
```

![1654858954387](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\1654858954387.png)

### 导入容器快照为镜像

