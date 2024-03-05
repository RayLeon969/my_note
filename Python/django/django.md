# 一 项目结构

![image-20240222135132338](assets/image-20240222135132338.png)

1. manage.py

   用于项目的管理，启动项目，创建APP，数据管理。不需要动。

2. settings.py

   项目配置，常需要操作。

3. urls.py

   URL和函数的对应关系，常需要操作。

4. asgi.py和wsgi.py

   都是用于接收网络请求，不需要动，却别是asgi.py可以异步。

# 二 APP

```text
项目
	app,用户管理
	app,订单管理
	app,后台管理
	...
```

## app结构

![image-20240222140255678](assets/image-20240222140255678.png)

1. apps.py 

   固定不动。

2. admin.py

   固定不动。

3. migrations

   固定不动。

4. test.py

   固定不动。单元测试

5. **views.py**

   重要，函数。

6. **models.py**

   重要，对数据库操作。

# 三 快速上手



## 1 确保APP已注册

注册位置

![image-20240222141059417](assets/image-20240222141059417.png)

![image-20240222141720344](assets/image-20240222141720344.png)

填入INSTALLED_APPS

![image-20240222141805010](assets/image-20240222141805010.png)



## 2 编写URL和视图函数对应关系

![image-20240222142056570](assets/image-20240222142056570.png)



## 3 编写视图函数

 ![image-20240222142245189](assets/image-20240222142245189.png)



## 4 启动项目并访问

![image-20240222143123458](assets/image-20240222143123458.png)



# 四 模板和静态文件



## 1 模板

![image-20240222145455196](assets/image-20240222145455196.png)

通过url去访问静态资源的时候，会自动去app或者根目录的templates下去搜寻。

注意：

![image-20240222145558481](assets/image-20240222145558481.png)

上图中，如果DIRS有做如上修改，资源的搜索优先级就是先搜寻根目录的templates然后才是各级app中的templates，如果没有则是先搜索app中的templates。



## 2 静态文件

![image-20240223100909195](assets/image-20240223100909195.png)

同样存放在app下，命名必须为static，结构如上图所示。



# 五 请求和响应

```python
def something(request):
    # request是一个对象，封装了用户通过浏览器发过来的请求数据

    # 1.获取请求方式
    print(request.method)

    # 2.在URL上传递值
    print(request.GET)

    # 3.在请求体中提交数据
    print(request.POST)

    return HttpResponse('返回内容')
```



# 六 链接Mysql



## MySQL+pymysql

```python
import pymysql

# 1.连接Mysql
conn = pymysql.connect(host='127.0.0.1', port=3306, user='root', passwd='qaz5094515', charset='utf8', db='test')
cursor = conn.cursor(cursor=pymysql.cursors.DictCursor)

# 2.发送指令
cursor.excute('insert into admin(username,password,mobile)' values('test', 'test', 'test'))
conn.commit()

# 3.关闭
cursor.close()
conn.close()

```



## Django链接数据库

安装第三方模块

```bash
pip install mysqlclient
```

创建数据库后修改成如下所示。

![image-20240225091117731](assets/image-20240225091117731.png)

至此，django会自行链接数据库。



# 七 ORM操作



## 1.通过类创建表

![image-20240225091724870](assets/image-20240225091724870.png)

然后在命令行执行

```bash
python manage.py makemigrations

python manage.py migrate
```



## 2.crud

```python
from app01.models import UserInfo # 请一定注意这里要使用app01.models 不能直接models不然直接报错

def orm(request):

    # 1.新建
    # UserInfo.objects.create('zmy')
     user = UserInfo(
        name= 'zmy',
        password= '123',
        age= 25,
    )
    user.save()

    # 2.删除表 filter是筛选条件
    # UserInfo.objects.filter(id=1).delete()

    # 3.查询表 filter是筛选条件
    # UserInfo.objects.all()  获取所有数据
    # UserInfo.objects.filter(id=1).first()

    # 4.更新表
    UserInfo.objects.all().update(password='111')

    return HttpResponse('成功')
```



## 3. 表关系建立



### 一对一关系建立

1. 在模型任意一边即可，使用models.OneToOneFiled()链接即可。

   ```python
   from django.db import models
   
   # Create your models here.
   class UserInfo(models.Model):
       name = models.CharField(max_length=32)
       password = models.CharField(max_length=64)
       age = models.IntegerField()
   
       # 此处不写明关联 先假设一个作者也只有一本书
   
   class Book(models.Model):# 和用户建立一对一关系
       title = models.CharField(max_length=32)
   
       # 建立一对一关系
       author = models.OneToOneField(UserInfo, on_delete=models.CASCADE, related_name='book')
   
   ```

   

2. 添加没有关系的一边，直接实例化保存就可以。

   ```python
   def add_user(request):
   
       user = UserInfo(
           name= 'zmy',
           password= '123',
           age= 25,
       )
       user.save()
       return HttpResponse('用户添加成功')
   
   ```

   

3. 添加有关系的一边，使用create方法。

```python
def add_book(request):
    author = UserInfo.objects.filter(id=1).first()

    book = Book(title='java', author=author)
    book.save()

    return HttpResponse('书本添加成功')
```

测试：

```python
def show_bookAurthor(request):
    book = Book.objects.filter(title='java').first()

    return HttpResponse(book.author.name)

def show_userBook(request):
    user = UserInfo.objects.filter(id=1).first()

    return HttpResponse(user.book.title)
```

均正常显示



### 一对多关系建立

再一对多中多的一方使用models.ForeignKey

```python
class UserInfo(models.Model):
    name = models.CharField(max_length=32)
    password = models.CharField(max_length=64)
    age = models.IntegerField()
    
class Blog(models.Model):

    title = models.CharField(max_length=32)

    user = models.ForeignKey(UserInfo, on_delete=models.CASCADE, related_name='blogs')
    
```

添加blogs

```python
def add_blogs(request):
    user = UserInfo.objects.filter(id=1).first()

    blog1 = Blog(title='python', user=user)
    blog2 = Blog(title='go', user=user)
    blog1.save()
    blog2.save()

    return HttpResponse('博客添加成功')

```

反向查验blogs

```python
def show_userBlogs(request):

    user = UserInfo.objects.filter(id=1).first()
    res = ''
    for blog in user.blogs.all(): # 此处名为blogs的原因是字段定义中related_name='blogs'
        res = res + " " + blog.title

    return HttpResponse(res)
```



### 多对多关系建立

使用models.ManyToMany定义，只需要定义一边。

```python
class Book(models.Model):
    title = models.CharField(max_length=32)

    # 建立一对一关系
    author = models.OneToOneField(UserInfo, on_delete=models.CASCADE, related_name='book')
    
class Tag(models.Model):
    title = models.CharField(max_length=32)

    books = models.ManyToManyField(Book, related_name='tags')
```

新增tags

```python
def add_tags(request):
    book1 = Book.objects.filter(title='java').first()
    book2 = Book.objects.filter(title='rust').first()

    tag1 = Tag(title='后端')
    tag2 = Tag(title='语言')

    tag1.save()
    tag2.save()

    book1.tags.add(tag1)
    book1.tags.add(tag2)
    book2.tags.add(tag1)
    book2.tags.add(tag2)



    return HttpResponse('标签添加成功')
```

==注意，这里是先保存tags，然后再把book的tags设定为tag==

测试

```python
def show_tags(request):
    book1 = Book.objects.filter(title='java').first()
    book2 = Book.objects.filter(title='rust').first()

    for tag in book1.tags.all():
        print(tag.title)

    for tag in book2.tags.all():
        print(tag.title)

    return HttpResponse('成功')
```





# 八 总结

在django中，大的功能模块通过app去划分，app都需要在settings中进行注册，然后再app中，urls相当于视图层，请求首先要去urls里面查找路径对应的接口，views相当于应用层，可以编写相应的业务逻辑代码，models相当于pojo实体类和持久层，django通过models创建对应实体类的数据库表格。
