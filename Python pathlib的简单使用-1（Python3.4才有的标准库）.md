Python pathlib的简单使用-1（Python3.4才有的标准库）

## 1、继承关系图

pathlib模块提供表示文件系统路径的类，它们的继承关系如图

![image-20220714150254227](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220714150254227.png)

- PurePath 类会将路径看做是一个普通的字符串，它可以实现将多个指定的字符串拼接成适用于当前操作系统的路径格式，同时还可以判断任意两个路径是否相等。注意，使用 PurePath 操作的路径，它并不会关心该路径是否真实有效。
- PurePosixPath 和 PureWindowsPath 是 PurePath 的子类，前者用于操作 UNIX（包括 Mac OS X）风格的路径，后者用于操作 Windows 风格的路径。
- Path 类和以上 3 个类不同，它操作的路径一定是真实有效的。Path 类提供了判断路径是否真实存在的方法。
- PosixPath 和 WindowsPath是 Path 的子类，分别用于操作 Unix（Mac OS X）风格的路径和 Windows 风格的路径。

**个人小提示：**

- **如果在Windows系统导入PosixPath，则会抛出异常，因为它是真实处理Linux的，同理也别在Linux上导入WindowsPath使用**

- **当我们在Windows开发，需要操作Linux的时候，我们可以使用Path或WindowsPath 来操作当前的路径，操作Windows的路径，而使用PurePosixPath 定义Linux的路径，例如，我们想上传Windows本地的文件到Linux机器时，这个技巧比较好用**

例如

```python
from pathlib import Path, PurePosixPath

window_path = Path(r"E:\WorkSpace\TMP\LinuxTool\aa")
print(window_path.parts)
print(window_path.absolute())
linux_path = PurePosixPath("/home/aaa/bbb")
print(linux_path.parts)
# 然后就可以cd到递归进入目录，不存在则创建，存在则进入

```

运行

![image-20220714193508942](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220714193508942.png)

如果定义Linux的目录也用Path，则容易混乱，比如取绝对路径等

```python
from pathlib import Path, PurePosixPath

window_path = Path(r"E:\WorkSpace\TMP\LinuxTool\aa")
print(window_path.parts)
print(window_path.absolute())
linux_path = Path("/home/aaa/bbb")
print(linux_path.parts)
print(linux_path.absolute())
# 然后就可以cd到递归进入目录，不存在则创建，存在则进入
```

![image-20220714193334055](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220714193334055.png)

## 2、PurePath最基 类的用法

创建时根据平台返回对应的对象，Windows是PureWindowsPath Linux是PurePosixPath

```python
from pathlib import PurePath

p = PurePath("main.py")
print(type(p))  # Windows是PureWindowsPath Linux是PurePosixPath
print(p)
```

支持多路径创建

```python
from pathlib import PurePath

p = PurePath("aaa", "bbb", "cccc", "ddd.py")
print(type(p))  # Windows是PureWindowsPath Linux是PurePosixPath
print(p)  # Windows是aaa\bbb\cccc\ddd.py Linux是aaa/bbb/cccc/ddd.py
```

如果想在Windows平台输出UNIX风格的路径，直接使用PurePosixPath类，如果想在Linux平台输出Windows风格的路径，则使用PureWindowsPath类

```python
from pathlib import PurePath, PurePosixPath, PureWindowsPath

p = PurePath("aaa", "bbb", "cccc", "ddd.py")
print(type(p))  # Windows是PureWindowsPath Linux是PurePosixPath
print(p)  # Windows是aaa\bbb\cccc\ddd.py Linux是aaa/bbb/cccc/ddd.py
# 根据操作平台等同于下面
linux_path = PurePosixPath("aaa", "bbb", "cccc", "ddd.py")
print(linux_path)  # aaa/bbb/cccc/ddd.py
windows_path = PureWindowsPath("aaa", "bbb", "cccc", "ddd.py")
print(windows_path)  # aaa\bbb\cccc\ddd.py
```

使用 PurePath 类构造方法时，不传入任何参数，则等同于传入点‘.’（表示当前路径）

```python
from pathlib import PurePath

p = PurePath()
print(p)  # .
p1 = PurePath(".")
print(p1)  # .
p2 = PurePath("./")
print(p2)  # .
print(p1 == p2)  # True
print(p == p1)  # True
print(p == p2)  # True
```

如果传入 PurePath 构造方法中的多个参数中，包含多个根路径，则只会有最后一个根路径及后面的子路径生效

```python
from pathlib import PurePath

path = PurePath('C://', 'D://', 'my_file.txt')
print(path)  # Windows运行：D:\my_file.txt Linux运行：C:/D:/my_file.txt

p = PurePath("/home", "/aaa", "my_file.txt")
print(p)  # Windows运行：\aaa\my_file.txt Linux运行： /aaa/my_file.txt
```

需要注意的是，如果传给 PurePath 构造方法的参数中包含有多余的斜杠或者点（ . ，表示当前路径），会直接被忽略（ .. 不会被忽略）。举个例子：

```python
from pathlib import *

path = PurePath('C://aa/./my_file.txt')
print(path)  # C:\aa\my_file.txt
path = PurePath('C://aa/../my_file.txt')
print(path)  # C:\aa\..\my_file.txt
```

PurePath 类还重载各种比较运算符，多余同种风格的路径字符串来说，可以判断是否相等，也可以比较大小（实际上就是比较字符串的大小）；对于不同种风格的路径字符串之间，只能判断是否相等（显然，不可能相等），但不能比较大小。

```python
from pathlib import *

path1 = PurePath('C:/aa/bb/cc/d.py')
path2 = PurePath('c:/AA/bb/CC/d.py')
print(path1 == path2)  # Windows是True(不区分大小写) Linux是False(区分大小写)

path1 = PurePath('C:/aa/bb/cc/d.py')
path2 = PurePath('C:/aa/bb/cc/ee/d.py')
print(path1 > path2)  # 都是False(字符串比较)

print(PureWindowsPath('C://my_file.txt') == PurePosixPath('C://my_file.txt'))  # 都是False
```

比较特殊的是，PurePath 类对象支持直接使用斜杠（/）作为多个字符串之间的连接符，例如：

```python
from pathlib import *

path1 = PurePath('C:/aa/bb/cc/')
path2 = path1 / 'aa' / 'cc' / 'dd' / 'a.py'
print(type(path2))  # PureWindowsPath
print(path2)  # C:\aa\bb\cc\aa\cc\dd\a.py
```

通过以上方式构建的路径，其本质上就是字符串，因此我们完全可以使用 str() 将 PurePath 对象转换成字符串。

```python
from pathlib import *

path1 = PurePath('C:/aa/bb/cc/d.py')
print(str(path1))
```

PurePath类实例属性和实例方法

| 类实例属性和实例方法名       | 功能描述                                                     |
| ---------------------------- | ------------------------------------------------------------ |
| PurePath.parts               | 返回路径字符串中所包含的各部分。                             |
| PurePath.drive               | 返回路径字符串中的驱动器盘符。                               |
| PurePath.root                | 返回路径字符串中的根路径。                                   |
| PurePath.anchor              | 返回路径字符串中的盘符和根路径。                             |
| PurePath.parents             | 返回当前路径的全部父路径。                                   |
| PurPath.parent               | 返回当前路径的上一级路径，相当于 parents[0] 的返回值。       |
| PurePath.name                | 返回当前路径中的文件名。                                     |
| PurePath.suffixes            | 返回当前路径中的文件所有后缀名。                             |
| PurePath.suffix              | 返回当前路径中的文件后缀名。相当于 suffixes 属性返回的列表的最后一个元素。 |
| PurePath.stem                | 返回当前路径中的主文件名。                                   |
| PurePath.as_posix()          | 返回使用正斜杠（`/`）的路径字符串                            |
| PurePath.as_uri()            | 将当前路径转换成 URL。只有绝对路径才能转换，否则将会引发 ValueError。 |
| PurePath.is_absolute()       | 判断当前路径是否为绝对路径。                                 |
| PurePath.joinpath(*other)    | 将多个路径连接在一起，作用类似于前面介绍的斜杠（/）连接符。  |
| PurePath.match(pattern)      | 判断当前路径是否匹配指定通配符。                             |
| PurePath.relative_to(*other) | 获取当前路径中去除基准路径之后的结果。                       |
| PurePath.with_name(name)     | 将当前路径中的文件名替换成新文件名。如果当前路径中没有文件名，则会引发 ValueError。 |
| PurePath.with_suffix(suffix) | 将当前路径中的文件后缀名替换成新的后缀名。如果当前路径中没有后缀名，则会添加新的后缀名。 |

Windows下使用示例

```python
from pathlib import *
path = PurePath(r"E:\WorkSpace\TMP\LinuxTool\aa\bb\bb.py")
print(path.parts)#('E:\\', 'WorkSpace', 'TMP', 'LinuxTool', 'aa', 'bb', 'bb.py')
print(path.drive)#E:
print(path.root)#\
print(path.anchor)     #E:\
print(list(path.parents))#返回的是迭代器 [PureWindowsPath('E:/WorkSpace/TMP/LinuxTool/aa/bb'), PureWindowsPath('E:/WorkSpace/TMP/LinuxTool/aa'), PureWindowsPath('E:/WorkSpace/TMP/LinuxTool'), PureWindowsPath('E:/WorkSpace/TMP'), PureWindowsPath('E:/WorkSpace'), PureWindowsPath('E:/')]
print(path.parent)#E:\WorkSpace\TMP\LinuxTool\aa\bb
print(path.name)#bb.py
print(path.suffixes)#['.py']
print(path.suffix)#.py
print(path.stem)#bb
print(path.as_posix())#E:/WorkSpace/TMP/LinuxTool/aa/bb/bb.py
print(path.as_uri())#file:///E:/WorkSpace/TMP/LinuxTool/aa/bb/bb.py
print(path.is_absolute())#True
print(path.joinpath("dd","ee","f.py"))#E:\WorkSpace\TMP\LinuxTool\aa\bb\bb.py\dd\ee\f.py
print(path.match("**/*.py"))#True
print(path.relative_to(r"E:\WorkSpace\tmp"))#LinuxTool\aa\bb\bb.py
print(path.with_name("new_name.py"))#E:\WorkSpace\TMP\LinuxTool\aa\bb\new_name
print(path.with_suffix(".txt"))#E:\WorkSpace\TMP\LinuxTool\aa\bb\bb.txt

path = PurePath(r"E:\WorkSpace\TMP\LinuxTool\node_exporter-1.3.1.linux-amd64.tar.gz")
print(path.parts)#('E:\\', 'WorkSpace', 'TMP', 'LinuxTool', 'node_exporter-1.3.1.linux-amd64.tar.gz')
print(path.drive)#E:
print(path.root)#\
print(path.anchor)#E:\
print(list(path.parents))#[PureWindowsPath('E:/WorkSpace/TMP/LinuxTool'), PureWindowsPath('E:/WorkSpace/TMP'), PureWindowsPath('E:/WorkSpace'), PureWindowsPath('E:/')]
print(path.parent)#E:\WorkSpace\TMP\LinuxTool
print(path.name)#node_exporter-1.3.1.linux-amd64.tar.gz
print(path.suffixes)#['.3', '.1', '.linux-amd64', '.tar', '.gz']
print(path.suffix)#.gz
print(path.stem)#node_exporter-1.3.1.linux-amd64.tar
print(path.as_posix())#E:/WorkSpace/TMP/LinuxTool/node_exporter-1.3.1.linux-amd64.tar.gz
print(path.as_uri())#file:///E:/WorkSpace/TMP/LinuxTool/node_exporter-1.3.1.linux-amd64.tar.gz
print(path.is_absolute())#True
print(path.joinpath("dd","ee","f.py"))#E:\WorkSpace\TMP\LinuxTool\node_exporter-1.3.1.linux-amd64.tar.gz\dd\ee\f.py
print(path.match("**/*.py"))#False
print(path.relative_to(r"E:\WorkSpace\tmp"))#LinuxTool\node_exporter-1.3.1.linux-amd64.tar.gz
print(path.with_name("new_name.py"))#E:\WorkSpace\TMP\LinuxTool\new_name.py
print(path.with_suffix(".txt"))#E:\WorkSpace\TMP\LinuxTool\node_exporter-1.3.1.linux-amd64.tar.txt
```

Linux下的示例

```python
from pathlib import *

path = PurePath(r"/home/aaa/bbb/ccc/dddd/eee/ff.py")
print(path.parts,)                         #('/', 'home', 'aaa', 'bbb', 'ccc', 'dddd', 'eee', 'ff.py')
print(path.drive)                          #空串
print(path.root)                           #/
print(path.anchor)                         #/
print(list(path.parents))                  #[PurePosixPath('/home/aaa/bbb/ccc/dddd/eee'), PurePosixPath('/home/aaa/bbb/ccc/dddd'), PurePosixPath('/home/aaa/bbb/ccc'), PurePosixPath('/home/aaa/bbb'), PurePosixPath('/home/aaa'), PurePosixPath('/home'), PurePosixPath('/')]
print(path.parent)                         #/home/aaa/bbb/ccc/dddd/eee
print(path.name)                           #ff.py
print(path.suffixes)                       #['.py']
print(path.suffix)                         #.py
print(path.stem)                           #ff
print(path.as_posix())                     #/home/aaa/bbb/ccc/dddd/eee/ff.py
print(path.as_uri())                       #file:///home/aaa/bbb/ccc/dddd/eee/ff.py
print(path.is_absolute())                  #True
print(path.joinpath("dd", "ee", "f.py"))   #/home/aaa/bbb/ccc/dddd/eee/ff.py/dd/ee/f.py
print(path.match("**/*.py"))               #True
print(path.relative_to(r"/home/aaa"))      #bbb/ccc/dddd/eee/ff.py
print(path.with_name("new_name.py"))       #/home/aaa/bbb/ccc/dddd/eee/new_name.py
print(path.with_suffix(".txt"))            #/home/aaa/bbb/ccc/dddd/eee/ff.txt

print("=======================================")

path = PurePath(r"/home/aaa/bbb/ccc/dddd/eee/node_exporter-1.3.1.linux-amd64.tar.gz")
print(path.parts)                                    #('/', 'home', 'aaa', 'bbb', 'ccc', 'dddd', 'eee', 'node_exporter-1.3.1.linux-amd64.tar.gz')
print(path.drive)                                    #空串
print(path.root)                                     #/ 
print(path.anchor)                                   #/ 
print(list(path.parents))                            #[PurePosixPath('/home/aaa/bbb/ccc/dddd/eee'), PurePosixPath('/home/aaa/bbb/ccc/dddd'), PurePosixPath('/home/aaa/bbb/ccc'), PurePosixPath('/home/aaa/bbb'), PurePosixPath('/home/aaa'), PurePosixPath('/home'), PurePosixPath('/')] 
print(path.parent)                                   #/home/aaa/bbb/ccc/dddd/eee 
print(path.name)                                     #node_exporter-1.3.1.linux-amd64.tar.gz 
print(path.suffixes)                                 #['.3', '.1', '.linux-amd64', '.tar', '.gz'] 
print(path.suffix)                                   #.gz 
print(path.stem)                                     #node_exporter-1.3.1.linux-amd64.tar  
print(path.as_posix())                               #/home/aaa/bbb/ccc/dddd/eee/node_exporter-1.3.1.linux-amd64.tar.gz 
print(path.as_uri())                                 #file:///home/aaa/bbb/ccc/dddd/eee/node_exporter-1.3.1.linux-amd64.tar.gz 
print(path.is_absolute())                            #True  
print(path.joinpath("dd", "ee", "f.py"))             #/home/aaa/bbb/ccc/dddd/eee/node_exporter-1.3.1.linux-amd64.tar.gz/dd/ee/f.py 
print(path.match("**/*.py"))                         #False 
print(path.relative_to(r"/home/aaa"))                #bbb/ccc/dddd/eee/node_exporter-1.3.1.linux-amd64.tar.gz 
print(path.with_name("new_name.py"))                 #/home/aaa/bbb/ccc/dddd/eee/new_name.py 
print(path.with_suffix(".txt"))                      #/home/aaa/bbb/ccc/dddd/eee/node_exporter-1.3.1.linux-amd64.tar.txt 
```

### Path类的功能和用法

和 PurPath 类相比，Path 类的最大不同，就是支持对路径的真实性进行判断。

从继承图 可以轻易看出，Path 是 PurePath 的子类，因此 Path 类除了支持 PurePath 提供的各种构造函数、实例属性以及实例方法之外，还提供甄别路径字符串有效性的方法，甚至还可以判断该路径对应的是文件还是文件夹，如果是文件，还支持对文件进行读写等操作。

和 PurePath 一样，Path 同样有 2 个子类，分别为 PosixPath（表示 UNIX 风格的路径）和 WindowsPath（表示 Windows 风格的路径）。

后面会写一个Path类的简单使用，有兴趣可以看看。

### 参考：

[Python pathlib模块用法详解](http://c.biancheng.net/view/2541.html) 

[官网](https://docs.python.org/zh-tw/3.9/library/pathlib.html)















