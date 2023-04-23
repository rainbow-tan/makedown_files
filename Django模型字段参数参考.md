Django模型的参数笔记

## 1、通用参数

- `max_length` 长度
- `unique` 是否唯一
- `null` 数据库是否可以为null
- `blank` 前端是否必填
- `primary_key` 是否为主键
- `default` 默认值
- `choices` 可选值

## 2、外键参数

### 2.1、`to_field` 指定关联的字段

如果不使用`to_filed`，默认关联的是ID，例如：

group表中有一条ID是1的记录

![image-20220622204241604](https://img2022.cnblogs.com/blog/1768648/202206/1768648-20220622230453465-1389946698.png)

user表中关联的字段就是group中的ID字段

![image-20220622204201325](https://img2022.cnblogs.com/blog/1768648/202206/1768648-20220622230432164-611372789.png)

如果使用`to_filed`，默认关联的是指定字段，如果该字段不存在，则会报错

使用to_filed关联group表的name字段`to_field="name"`，name字段需要是唯一，因为关联是唯一的，例如

group表中有一条ID是1的记录

![image-20220622204241604](https://img2022.cnblogs.com/blog/1768648/202206/1768648-20220622230453465-1389946698.png)

user表中关联的字段就是group中的name字段

![image-20220622204853586](https://img2022.cnblogs.com/blog/1768648/202206/1768648-20220622230513876-815928858.png)

可以看到关联字段是name字段了，但是字段名还是显示以id关联，通过`db_column`指定

### 2.2、`db_column` 指定字段名

使用了`db_column="group_name"`

![image-20220622205238970](https://img2022.cnblogs.com/blog/1768648/202206/1768648-20220622230534780-6047695.png)

### 2.3、`related_name` 反向关系

举例：一个用户组可以拥有多个用户，而一个用户只能属于一个用户组，用户组对于用户属于一对多

正向查询：查询某个用户属于的组

```python
user: User = User.objects.filter(name="admin").first()
user_group = user.group
print(user_group)
```

反向查询：查询某个用户组拥有的用户是哪些

通过模型名称_set来指定反向关系（本例子就是user_set）

```python
admin_group = Group.objects.filter(name="admin组").first()
users = admin_group.user_set.all()
print(users)
```

上面是不使用`related_name`的情况，使用了`related_name`就可以直接使用`related_name="group_users"`指定的名字来反向了

```python
admin_group = Group.objects.filter(name="admin组").first()
users=admin_group.group_users.all()
```

### 2.3、`on_delete`删除关联表中的数据时,当前表字段行为

含义：当删除外键表中的数据时，自己表中的数据字段如何处理，有以下几种值可选择

on_delete = models.CASCADE删除关联数据的时候，该条记录也删除
on_delete = models.DO_NOTHING删除关联数据的时候，什么操作也不做
on_delete = models.PROTRCT删除关联数据的时候，引发报错
on_delete = models.SET_NULL删除关联数据的时候，设置该字段为NULL，前提是null=True
on_delete = models.SET_DEFAULT删除关联数据的时候，设置该字段为默认值，前提是设置了default默认值
on_delete = models.SET删除关联数据时，设置为该值，例如

```python
官方案例**
def get_sentinel_user():
    return get_user_model().objects.get_or_create(username='deleted')[0]

class MyModel(models.Model):
    user = models.ForeignKey(
        settings.AUTH_USER_MODEL,
        on_delete=models.SET(get_sentinel_user),
    )
```

## 3、模型代码

```python
from django.db import models


class User(models.Model):
    id = models.AutoField(primary_key=True)
    uid = models.CharField("用户ID", max_length=30, unique=True, null=False, blank=False)
    name = models.CharField("用户名", max_length=30, unique=True, null=False, blank=False)
    email = models.EmailField("用户邮箱", max_length=30, unique=True, null=False, blank=False)
    group = models.ForeignKey(
        "Group",
        on_delete=models.SET_NULL,
        null=True,
        blank=True,
        default=None,
        to_field="name",  # 指定关联字段             (默认是id)
        db_column="group_name",  # 指定列名          (默认是模型名称_id例如：group_id)
        related_name="group_users"  # 指定反向关系    (默认是模型名称_set例如：user_id)
    )

    created = models.DateTimeField("创建时间", auto_now_add=True)
    updated = models.DateTimeField("修改时间", auto_now=True)


class Group(models.Model):
    id = models.AutoField(primary_key=True)
    gid = models.CharField("用户组ID", max_length=30, unique=True, null=False, blank=False)
    name = models.CharField(
        "用户组名称", max_length=30, unique=True, null=False, blank=False
    )

    created = models.DateTimeField("创建时间", auto_now_add=True)
    updated = models.DateTimeField("修改时间", auto_now=True)
```

参考链接：

[Django 外键ForeignKey中的on_delete](https://blog.csdn.net/lht_521/article/details/80605146#:~:text=%E5%BD%93%E7%94%B1%E4%B8%80%E4%B8%AA%20ForeignKey%20%E5%BC%95%E7%94%A8%20%E7%9A%84%20%E5%AF%B9%E8%B1%A1%E8%A2%AB%E5%88%A0%E9%99%A4%EF%BC%8C%E9%BB%98%E8%AE%A4%E6%83%85%E5%86%B5%E4%B8%8B%EF%BC%8C%20Django%20%E6%A8%A1%E6%8B%9FSQL%20%E7%9A%84,%EF%BC%8C%E4%BD%A0%E6%83%B3%E4%BB%96%E5%BC%95%E7%94%A8%20%E7%9A%84%20%E5%AF%B9%E8%B1%A1%E8%A2%AB%E5%88%A0%E9%99%A4%E6%97%B6%EF%BC%8C%E8%AF%A5%E9%A1%B9%E4%B8%BA%E7%A9%BA%E3%80%82%20user%20%3D%20mod%20el%20s.)

[django数据模型中关于on_delete的使用](https://blog.csdn.net/kuangshp128/article/details/78946316)
