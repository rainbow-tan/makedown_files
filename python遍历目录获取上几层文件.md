## 功能：

python遍历目录，获取上N层的文件，例如获取第一层，第二层，第三层的文件。

## 代码

```python
import os


def traverse_folders_by_layer(folder, layer=999, index=1, data=None):
    """
    提示: 当前目录是第一层 次级目录是第二次 依次类推
    :param folder: 要遍历的路径
    :param layer: 要遍历多少层
    :param index: 当前是第几层,默认第一层,使用时不需要传递该参数
    :param data: 存放遍历的结果
    :return:
    e.g:
    data = traverse_folders_by_layer(".", 1) 遍历至第1层
    data = traverse_folders_by_layer(".", 2) 遍历至第2层
    data = traverse_folders_by_layer(".", 3) 遍历至第3层
    data = traverse_folders_by_layer(".") 遍历至第999层,也可以理解为遍历当前目录所有子目录
    """
    if data is None:
        data = []
    folder = os.path.abspath(folder)
    if index > layer:
        return data
    info = os.listdir(folder)
    for name in info:
        path = os.path.abspath(os.path.join(folder, name))
        if os.path.isfile(path):
            data.append((index, path))
        else:
            data = traverse_folders_by_layer(path, layer, index + 1, data)
    return data


def main():
    for i in range(1, 7):
        data = traverse_folders_by_layer(".", i)
        print(f"第{i}层的文件信息是:")
        for one in data:
            print(one)
        print("\n")


if __name__ == '__main__':
    main()

```

### 验证

目录结构

![image-20230227114324252](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20230227114324252.png)

运行结果

F:\Python3.9.12\python.exe E:/WorkSpace/TMP/PythonProjectTMP/1/traverse_folder_floor.py
第1层的文件信息是:
(1, 'E:\\WorkSpace\\TMP\\PythonProjectTMP\\1\\traverse_folder_floor.py')
(1, 'E:\\WorkSpace\\TMP\\PythonProjectTMP\\1\\新建文本文档 - 副本.txt')
(1, 'E:\\WorkSpace\\TMP\\PythonProjectTMP\\1\\新建文本文档.txt')


第2层的文件信息是:
(2, 'E:\\WorkSpace\\TMP\\PythonProjectTMP\\1\\2\\2.txt')
(2, 'E:\\WorkSpace\\TMP\\PythonProjectTMP\\1\\2\\新建文本文档.txt')
(2, 'E:\\WorkSpace\\TMP\\PythonProjectTMP\\1\\2 - 副本\\新建文本文档.txt')
(2, 'E:\\WorkSpace\\TMP\\PythonProjectTMP\\1\\2 - 副本 (2)\\新建文本文档.txt')
(1, 'E:\\WorkSpace\\TMP\\PythonProjectTMP\\1\\traverse_folder_floor.py')
(1, 'E:\\WorkSpace\\TMP\\PythonProjectTMP\\1\\新建文本文档 - 副本.txt')
(1, 'E:\\WorkSpace\\TMP\\PythonProjectTMP\\1\\新建文本文档.txt')


第3层的文件信息是:
(2, 'E:\\WorkSpace\\TMP\\PythonProjectTMP\\1\\2\\2.txt')
(2, 'E:\\WorkSpace\\TMP\\PythonProjectTMP\\1\\2\\新建文本文档.txt')
(3, 'E:\\WorkSpace\\TMP\\PythonProjectTMP\\1\\2 - 副本\\3\\新建文本文档 - 副本.txt')
(3, 'E:\\WorkSpace\\TMP\\PythonProjectTMP\\1\\2 - 副本\\3\\新建文本文档.txt')
(3, 'E:\\WorkSpace\\TMP\\PythonProjectTMP\\1\\2 - 副本\\3 - 副本\\新建文本文档.txt')
(2, 'E:\\WorkSpace\\TMP\\PythonProjectTMP\\1\\2 - 副本\\新建文本文档.txt')
(2, 'E:\\WorkSpace\\TMP\\PythonProjectTMP\\1\\2 - 副本 (2)\\新建文本文档.txt')
(1, 'E:\\WorkSpace\\TMP\\PythonProjectTMP\\1\\traverse_folder_floor.py')
(1, 'E:\\WorkSpace\\TMP\\PythonProjectTMP\\1\\新建文本文档 - 副本.txt')
(1, 'E:\\WorkSpace\\TMP\\PythonProjectTMP\\1\\新建文本文档.txt')


第4层的文件信息是:
(2, 'E:\\WorkSpace\\TMP\\PythonProjectTMP\\1\\2\\2.txt')
(2, 'E:\\WorkSpace\\TMP\\PythonProjectTMP\\1\\2\\新建文本文档.txt')
(4, 'E:\\WorkSpace\\TMP\\PythonProjectTMP\\1\\2 - 副本\\3\\4\\新建文本文档.txt')
(4, 'E:\\WorkSpace\\TMP\\PythonProjectTMP\\1\\2 - 副本\\3\\4 - 副本\\新建文本文档 - 副本.txt')
(4, 'E:\\WorkSpace\\TMP\\PythonProjectTMP\\1\\2 - 副本\\3\\4 - 副本\\新建文本文档.txt')
(3, 'E:\\WorkSpace\\TMP\\PythonProjectTMP\\1\\2 - 副本\\3\\新建文本文档 - 副本.txt')
(3, 'E:\\WorkSpace\\TMP\\PythonProjectTMP\\1\\2 - 副本\\3\\新建文本文档.txt')
(3, 'E:\\WorkSpace\\TMP\\PythonProjectTMP\\1\\2 - 副本\\3 - 副本\\新建文本文档.txt')
(2, 'E:\\WorkSpace\\TMP\\PythonProjectTMP\\1\\2 - 副本\\新建文本文档.txt')
(2, 'E:\\WorkSpace\\TMP\\PythonProjectTMP\\1\\2 - 副本 (2)\\新建文本文档.txt')
(1, 'E:\\WorkSpace\\TMP\\PythonProjectTMP\\1\\traverse_folder_floor.py')
(1, 'E:\\WorkSpace\\TMP\\PythonProjectTMP\\1\\新建文本文档 - 副本.txt')
(1, 'E:\\WorkSpace\\TMP\\PythonProjectTMP\\1\\新建文本文档.txt')


第5层的文件信息是:
(2, 'E:\\WorkSpace\\TMP\\PythonProjectTMP\\1\\2\\2.txt')
(2, 'E:\\WorkSpace\\TMP\\PythonProjectTMP\\1\\2\\新建文本文档.txt')
(4, 'E:\\WorkSpace\\TMP\\PythonProjectTMP\\1\\2 - 副本\\3\\4\\新建文本文档.txt')
(4, 'E:\\WorkSpace\\TMP\\PythonProjectTMP\\1\\2 - 副本\\3\\4 - 副本\\新建文本文档 - 副本.txt')
(4, 'E:\\WorkSpace\\TMP\\PythonProjectTMP\\1\\2 - 副本\\3\\4 - 副本\\新建文本文档.txt')
(3, 'E:\\WorkSpace\\TMP\\PythonProjectTMP\\1\\2 - 副本\\3\\新建文本文档 - 副本.txt')
(3, 'E:\\WorkSpace\\TMP\\PythonProjectTMP\\1\\2 - 副本\\3\\新建文本文档.txt')
(3, 'E:\\WorkSpace\\TMP\\PythonProjectTMP\\1\\2 - 副本\\3 - 副本\\新建文本文档.txt')
(2, 'E:\\WorkSpace\\TMP\\PythonProjectTMP\\1\\2 - 副本\\新建文本文档.txt')
(2, 'E:\\WorkSpace\\TMP\\PythonProjectTMP\\1\\2 - 副本 (2)\\新建文本文档.txt')
(1, 'E:\\WorkSpace\\TMP\\PythonProjectTMP\\1\\traverse_folder_floor.py')
(1, 'E:\\WorkSpace\\TMP\\PythonProjectTMP\\1\\新建文本文档 - 副本.txt')
(1, 'E:\\WorkSpace\\TMP\\PythonProjectTMP\\1\\新建文本文档.txt')


第6层的文件信息是:
(2, 'E:\\WorkSpace\\TMP\\PythonProjectTMP\\1\\2\\2.txt')
(2, 'E:\\WorkSpace\\TMP\\PythonProjectTMP\\1\\2\\新建文本文档.txt')
(4, 'E:\\WorkSpace\\TMP\\PythonProjectTMP\\1\\2 - 副本\\3\\4\\新建文本文档.txt')
(4, 'E:\\WorkSpace\\TMP\\PythonProjectTMP\\1\\2 - 副本\\3\\4 - 副本\\新建文本文档 - 副本.txt')
(4, 'E:\\WorkSpace\\TMP\\PythonProjectTMP\\1\\2 - 副本\\3\\4 - 副本\\新建文本文档.txt')
(3, 'E:\\WorkSpace\\TMP\\PythonProjectTMP\\1\\2 - 副本\\3\\新建文本文档 - 副本.txt')
(3, 'E:\\WorkSpace\\TMP\\PythonProjectTMP\\1\\2 - 副本\\3\\新建文本文档.txt')
(3, 'E:\\WorkSpace\\TMP\\PythonProjectTMP\\1\\2 - 副本\\3 - 副本\\新建文本文档.txt')
(2, 'E:\\WorkSpace\\TMP\\PythonProjectTMP\\1\\2 - 副本\\新建文本文档.txt')
(2, 'E:\\WorkSpace\\TMP\\PythonProjectTMP\\1\\2 - 副本 (2)\\新建文本文档.txt')
(1, 'E:\\WorkSpace\\TMP\\PythonProjectTMP\\1\\traverse_folder_floor.py')
(1, 'E:\\WorkSpace\\TMP\\PythonProjectTMP\\1\\新建文本文档 - 副本.txt')
(1, 'E:\\WorkSpace\\TMP\\PythonProjectTMP\\1\\新建文本文档.txt')



进程已结束,退出代码0
