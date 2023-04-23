Django显示SQL语句

### 1、使用`connection`显示

导入：from django.db import connection

获取SQL语句：connection.queries

样例

```python
@action(detail=False)
def test_print_sql(self, request):
    from django.db import connection
    groups = Group.objects.all()
    print(f"所有的组:{groups}")
    master = Group.objects.filter(name="master组").first()
    print(f"master组:{master}")
    admin = Group.objects.filter(name="admin组").first()
    print(f"admin组:{admin}")
    in_sql = Group.objects.filter(name__in=["master组", "dev组", "不存在的组"]).first()
    print(f"in语句的sql:{in_sql}")
    Group(gid="test组", name="测试组").save()
    print("插入单条数据")
    Group.objects.bulk_create([Group(gid="组1", name="组1"), Group(gid="组2", name="组2"), Group(gid="组3", name="组3"),
                               Group(gid="组4", name="组4")])
    print("批量创建")
    for o in connection.queries:
        print(f"SQL语句:{o}")
    return Response(status=status.HTTP_200_OK, data="测试显示SQL语句")
```

运行

<img src="C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220712163722779.png" alt="image-20220712163722779" style="zoom:50%;" />

### 2、配置log显示

在Django项目的settings.py文件中，在最后复制粘贴如下代码：

```python
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'handlers': {
        'console':{
            'level':'DEBUG',
            'class':'logging.StreamHandler',
        },
    },
    'loggers': {
        'django.db.backends': {
            'handlers': ['console'],
            'propagate': True,
            'level':'DEBUG',
        },
    }
}
```

### 3、参考链接

[如何查看Django ORM实际执行的原始SQL查询]( https://www.cnblogs.com/zzhaolei/p/13608966.html)

[我该如何在Django运行的过程中看到SQL查询？](https://docs.djangoproject.com/zh-hans/4.0/faq/models/)

[Django终端打印SQL语句](https://www.cnblogs.com/fixdq/p/9210647.html)