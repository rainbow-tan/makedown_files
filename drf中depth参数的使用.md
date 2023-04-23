drf中depth参数的使用

## 1、模型文件

用户User、用户组Group、一个组有多个用户，一个用户只属于一个组，用户组对应用户属于一对多

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

## 2、序列化文件

```python
from rest_framework import serializers

from mymodel.models import User, Group


class UserSerializer(serializers.ModelSerializer):
    class Meta:
        model = User
        fields = '__all__'
        depth = 1


class GroupSerializer(serializers.ModelSerializer):
    class Meta:
        model = Group
        fields = '__all__'
```

### ①`depth`参数指定遍历外键信息

- 当不使用`depth`参数时，外键只显示关联的字段，但是创建和更新可以进行传递该外键

![image-20220623095533900](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220623095533900.png)

- 当使用了`depth`参数时，外键显示所有的字段信息，但是创建和更新不可以进行传递该外键

![image-20220623095755167](C:\Users\dell\AppData\Roaming\Typora\typora-user-images\image-20220623095755167.png)

- `depth`参数应该给一个整数，表示递归遍历到第几层外键

如果想在使用了`depth`参数时，创建和更新又能传递外键，则参考[DRF depth=1的情况下对象的创建与更新问题](https://blog.csdn.net/tmpbook/article/details/53297558)

原理：通过重写get_serializer_class方法，判断请求方式来决定序列化类

序列化文件

```python
from rest_framework import serializers

from mymodel.models import User, Group


class UserSerializer(serializers.ModelSerializer):
    class Meta:
        model = User
        fields = '__all__'
        depth = 1


class UserNodepthSerializer(serializers.ModelSerializer):
    class Meta:
        model = User
        fields = '__all__'


class GroupSerializer(serializers.ModelSerializer):
    class Meta:
        model = Group
        fields = '__all__'
```

视图文件

```python
# Create your views here.
from rest_framework import viewsets

from mymodel.models import User, Group
from mymodel.serializer import UserSerializer, GroupSerializer, UserNodepthSerializer


class UserViewSet(viewsets.ModelViewSet):
    queryset = User.objects.all()
    serializer_class = UserSerializer

    def get_serializer_class(self):
        serializer_class = self.serializer_class
        if self.request.method in ("PUT", "PATCH", "POST"):
            serializer_class = UserNodepthSerializer
        elif self.request.method in ("GET",):
            serializer_class = UserSerializer
        return serializer_class


class GroupViewSet(viewsets.ModelViewSet):
    queryset = Group.objects.all()
    serializer_class = GroupSerializer
```