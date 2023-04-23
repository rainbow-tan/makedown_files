Python单例模式的创建

### 1、重写`__new__`方法

因为实例化对象时，会先调用`__new__`方法，然后再调用`__init__`方法，因此重写`__new__`方法完成单例模式

```python
import threading


class A:
    __instance = None

    def __init__(self):
        pass

    def __new__(cls, *args, **kwargs):
        if cls.__instance is None:
            instance = super(A, cls).__new__(cls, *args, **kwargs)
            cls.__instance = instance
        return cls.__instance


def create_instance_normal(n, class_name):
    # 普通创建方式,看创建的是否是单例模式
    for i in range(n):
        o = class_name()
        print(f"创建出来的实例是:{o}, ID是:{id(o)}")


def create_instance_by_thread(n, class_name):
    # 开启线程创建,看创建的是否是单例模式
    def create(name):
        o = name()
        print(f"创建出来的实例是:{o}, ID是:{id(o)}")

    for i in range(n):
        threading.Thread(target=create, args=(class_name,)).start()


if __name__ == '__main__':
    create_instance_normal(3, A)
    create_instance_by_thread(3, A)
```

运行

![image-20220614103226673](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220614103226673.png)

说明

可以看到无论是单线程创建，还是多线程创建，都是一个对象

### 2、模块实现

Python 的模块就是天然的单例模式，因为模块在第一次导入时，会生成 .pyc 文件，当第二次导入时，就会直接加载 .pyc 文件，而不会再次执行模块代码。因此，我们只需把相关的函数和数据定义在一个模块中，就可以获得一个单例对象了。

创建a1.py，实例化A类，实例化出的o就是一个单例，其他文件`from a1 import o`即可

```python
class A:
    __instance = None

    def __init__(self):
        pass

    @staticmethod
    def test(info):
        print(f"test函数:{info}")


o = A()
o.test("a1.py")
print(f"a1.py创建出来的实例是:{o}, ID是:{id(o)}")
```

其他文件使用

![image-20220614104853248](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220614104853248.png)

运行a4.py

![image-20220614104914402](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220614104914402.png)

### 3、使用装饰器

```python
import threading
import time


def singleton(cls, *args, **kw):
    instances = {}

    def getinstance():
        if cls not in instances:
            instances[cls] = cls(*args, **kw)
        return instances[cls]

    return getinstance


@singleton
class A:
    __instance = None

    def __init__(self):
        pass


@singleton
class B:
    __instance = None

    def __init__(self):
        pass


def create_instance_normal(n, class_name):
    # 普通创建方式,看创建的是否是单例模式
    for i in range(n):
        o = class_name()
        print(f"创建出来的实例是:{o}, ID是:{id(o)}")


def create_instance_by_thread(n, class_name):
    # 开启线程创建,看创建的是否是单例模式
    def create(name):
        o = name()
        print(f"创建出来的实例是:{o}, ID是:{id(o)}")

    for i in range(n):
        threading.Thread(target=create, args=(class_name,)).start()


if __name__ == '__main__':
    create_instance_normal(5, A)
    create_instance_by_thread(5, A)
    time.sleep(1)
    create_instance_normal(5, B)
    create_instance_by_thread(5, B)
```

运行

![image-20220614105551548](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220614105551548.png)