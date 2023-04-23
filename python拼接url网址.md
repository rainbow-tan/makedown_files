python拼接url网址

1、拼接字典与拆出字典

```python
import urllib.parse


def join_url(url, data):
    """
    拼接基础url和query字典参数
    :param url: 
    :param data: 
    :return: 
    """
    query_string = urllib.parse.urlencode(data)
    new = url + '?' + query_string
    print(f"url:[{url}], data:[{data}] 拼接后:{new}")
    return new


def split_url(url):
    """
    解析出基础url和query字典参数
    :param url: 
    :return: 
    """
    info = urllib.parse.urlparse(url)
    query = info.query
    path = urllib.parse.urlunparse((info.scheme, info.netloc, info.path, "", "", ""))
    data = dict([(k, v[0]) for k, v in urllib.parse.parse_qs(query).items()])
    print(f"url:[{url}] path:[{path}] data:{data}")
    return path, data


if __name__ == '__main__':
    debug_url = "http://127.0.0.1:3000/render/d-solo/me"
    debug_data = {"name": "admin", "age": 18, "like": "墨玉麒麟"}
    debug_new = join_url(debug_url, debug_data)
    debug_split = split_url(debug_new)
```

运行

![image-20221021103036452](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20221021103036452.png)