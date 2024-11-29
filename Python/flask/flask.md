# 一 Flask入门

![image-20240226101859897](assets/image-20240226101859897.png)



## 1 debug模式开启

![image-20240226102249491](assets/image-20240226102249491.png)



## 2 修改host

![image-20240226102510598](assets/image-20240226102510598.png)



## 3 修改端口

![image-20240226102857016](assets/image-20240226102857016.png)



# 二 URL与视图

```python
# 带参数的url
# <>表示接受的参数，int表示定义参数类型。
# methods表示该路径能接受的类型请求。
#
@app.route('/smt/<int:smt_id>',methods=['GET','POST'], endpoint='smt')
def smt(smt_id):
    return f'something:{smt_id}'
```

代码中已经包含了URL的一些相关信息，浓缩自行提取。



## 1 自定义转换器

```python
from flask import Flask,request,render_template

# 自定义转换器
class RegexConverter(BaseConverter):

    def __init__(self, url_map, regex):
        # 调用父类方法
        super(RegexConverter,self).__init__(url_map)
        self.regex = regex

    def to_python(self, value):
        # 实现父类方法
        print('to_python方法被调用')
        return value

# 将自定义的转换器类添加到flask应用中
app.url_map.converters['re'] = RegexConverter

# re中可以填入正则表达式 定义规则
@app.route('/hello/<re("1\d{10}"):value>')
def hello(value):
    print(value)
    return "hello Nihao"

```



## 2 访问静态页面

```python
from flask import Flask,request,render_template

@app.route('/index',methods=['GET','POST'])
def index():
    return render_template('index.html')
```



# 三 请求和响应

index.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<form action="" method="post">
    账号：<br/>
    <input type="text" name="name">
    <br/>
    密码：<br/>
    <input type="password" name="pwd">
    <br/>
    <br/>
    <input type="submit" name="submit">
</form>
</body>
</html>
```

## 1 获取请求中的数据

```python
from flask import Flask,request

@app.route('/index',methods=['GET','POST'])
def index():
    if request.method == 'GET':
        return render_template('index.html')
    else:
        name = request.form.get('name')
        pwd = request.form.get('pwd')
        print(name,pwd)
        return "this is post"
```



## 2 重定向

```python
from flask import Flask,request,render_template,redirect

@app.route('/baidu')
def baidu():
    return redirect('https://www.baidu.com/')
```



## 3 响应

```python
from flask import Flask,jsonify

@app.route('/testResponse')
def testResponse():
    return jsonify({
        'name':'zmy'
    })

```



## 4 abort函数

```python
...abort(404) # 即可在网页抛出一个404错误

@app.errorhandler(404) # 处理404错误
def handle_4040_error(err):
    return '出现404错误啦'
```



# 四 模板语法

修改index代码

```python
@app.route('/index',methods=['GET','POST'])
def index():
    if request.method == 'GET':
        list = (1,2,3,4,5,6)
        # 模板传入数据
        data = {
            'name': 'zmy',
            'age': 25,
            'list': list
        }
        return render_template('index.html', data=data)
    else:
        name = request.form.get('name')
        pwd = request.form.get('pwd')
        print(name,pwd)
        return "this is post"
        
```

在html中的页面使用data数据

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<form action="" method="post">
    账号：<br/>
    <input type="text" name="name">
    <br/>
    密码：<br/>
    <input type="password" name="pwd">
    <br/>
    <br/>
    <input type="submit" name="submit">
    <br/>
    {{ data }}
    <br/>
    {{ data.name }}
    <br/>
    {{ data.list[1] }}
</form>
</body>
</html>
```

感觉和vue有点像



# 五 连接数据库

```python
from flask import Flask
from flask_sqlalchemy import SQLAlchemy

app = Flask(__name__)

# 在app.config中设置好链接数据库的信息。
# mysql所在主机名
HOSTNAME = 'localhost'
# 端口
PORT = 3306
# 用户 密码
USERNAME = 'root'
PASSWORD = 'qaz5094515'
# 数据库名称
DATABASE = 'flaskstudy'
app.config['SQLALCHEMY_DATABASE_URI'] = f'mysql+pymysql://{USERNAME}:{PASSWORD}@{HOSTNAME}:{PORT}/{DATABASE}?charset=utf8mb4'

```

测试数据库是否连接成功

```python
db = SQLAlchemy(app)

with app.app_context():
    with db.engine.connect() as conn:
        rs = conn.execute(text("select 1"))
        print(rs.fetchone()) # 如果打印了(1,)证明连接成功

```



# 六 ORM



## 1. 创建实体类

```python
class User(db.Model):
    __tablename__ = 'user'
    id = db.Column(db.Integer, primary_key=True, autoincrement=True)
    username = db.Column(db.String(100), nullable=False)
    password = db.Column(db.String(100), nullable=False)
```

然后更新数据库

```python
with app.app_context():
    db.create_all()
```



## 2. CRUD

```python
@app.route('/user/add')
def add_user():
    # 创建orm对象
    user = User(username='zmy',password='123')
    # 将orm对象添加到db.session中
    db.session.add(user)
    # 将db.session中的改变同步到数据库中
    db.session.commit()

    return "用户创建成功"

@app.route('/user/query')
def query_user():

    # 1.get查找，根据主键进行查找
    user = User.query.get(1)
    print(f'{user.username}:{user.password}')

    # 2.filter_by查找
    # users = User.query.filter_by(username='zmy')
    # for user in users:
    #     print(f'{user.username}:{user.password}')
    return "查询成功"

@app.route('/user/update')
def update_user():

    # 1.get查找，根据主键进行查找
    user = User.query.get(1)
    print(f'{user.username}:{user.password}')

    user.password = '321'

    db.session.commit()

    return "更新成功"

@app.route('/user/delete')
def delete_user():

    # 1.get查找，根据主键进行查找
    user = User.query.get(1)

    db.session.delete(user)

    db.session.commit()

    return "删除成功"

```



## 3. 表关系建立



### 一对多关系建立

```python
class User(db.Model):
    __tablename__ = 'user'
    id = db.Column(db.Integer, primary_key=True, autoincrement=True)
    username = db.Column(db.String(100), nullable=False)
    password = db.Column(db.String(100), nullable=False)

class Article(db.Model):
    __tablename__ = 'article'
    id = db.Column(db.Integer, primary_key=True, autoincrement=True)
    title = db.Column(db.String(100), nullable=False)
    content = db.Column(db.Text, nullable=False)

    # 添加外键
    author_id = db.Column(db.Integer, db.ForeignKey('user.id'))
    # 以下是更方便的关联 可以直接通过article.author直接访问user对象。
    # backref可以自动的让User实体类多出一个articles字段，表示关联了user对象所有的articles，即可以通过author.articles访问所有的articles
    author = db.relationship("User", backref='articles')
```

==注意一下，这边db.Column(db.Integer, db.ForeignKey('user.id'))这个如果你修改了字段名字，提交后字段名不会再数据库中做出改变，会导致后续开发出现问题==



### 一对一关系建立

```python
class Country(db.Model):
	capital = db.relationship('Capital',userlist=False)

class Capital(db.Model):
	country_id = db.Column(db.Integer, db.ForeignKey('country.id'))
	country = db.relationship('Country')

```



### 多对多关系建立

```python
association_table = db.Table(
					'association', 
					db.Column('student_id',db.Integr, db.ForeignKey('student.id')),
					db.Column('teacher_id',db.Integr, db.ForeignKey('teacher.id'))
					)

class Student(db.Model):
	id = db.Column(db.Integer,primary_key=True)
	name = db.Column(db.String(70),unique=True)
	grade = db.Column(db.String(20))
	teachers = db.relationship('Teacher', secondary =association_table, back_populates = 'students' )
	
class Teacher(db.Model):
	id = db.Column(db.Integer,primary_key=True)
	name = db.Column(db.String(70),unique=True)
	office = db.Column(db.String(70))

```



## 4. migrate

废除以下代码

```python
with app.app_context():
    db.create_all()
```

以上代码对已经创建的表不会进行表字段的更新。

```python
app = Flask(__name__)

# 在app.config中设置好链接数据库的信息。
# mysql所在主机名
HOSTNAME = 'localhost'
# 端口
PORT = 3306
# 用户 密码
USERNAME = 'root'
PASSWORD = 'qaz5094515'
# 数据库名称
DATABASE = 'flaskstudy'
app.config['SQLALCHEMY_DATABASE_URI'] = f'mysql+pymysql://{USERNAME}:{PASSWORD}@{HOSTNAME}:{PORT}/{DATABASE}?charset=utf8mb4'

# 然后使用SQLAlchemy（app）创建一个db对象
db = SQLAlchemy(app)

migrate = Migrate(app,db)
```

三步操作

1. flask db init 只需执行一次
2. flask db migrate 识别orm模型的改变，生成迁移脚本
3. flask db upgrade 运行迁移脚本，同步到数据库中



# 七 新规范

在 **SQLAlchemy 2.0+** 中，使用新的 **`Mapped`** 和 **`mapped_column`** 语法来定义外键和各种关系变得更加简洁。以下是如何在这种风格下定义 **外键** 和 **1 对多、多对多** 的关系的详细说明。

### **1. 外键定义**

外键（Foreign Key）是表示一个表中的字段引用另一个表的主键字段。可以使用 `mapped_column` 和 `ForeignKey` 来定义外键关系。

#### **示例：定义外键**

假设有两个表：`User` 和 `Post`，其中 `Post` 表中的 `user_id` 字段是外键，引用了 `User` 表的 `id` 字段。

```python
python复制代码from sqlalchemy import ForeignKey
from sqlalchemy.orm import relationship
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy import Integer, String

Base = declarative_base()

class User(Base):
    __tablename__ = 'user'
    id: Mapped[int] = mapped_column(Integer, primary_key=True, autoincrement=True)
    name: Mapped[str] = mapped_column(String(50))

class Post(Base):
    __tablename__ = 'post'
    id: Mapped[int] = mapped_column(Integer, primary_key=True, autoincrement=True)
    title: Mapped[str] = mapped_column(String(100))
    
    # 外键，引用 User 表的 id 字段
    user_id: Mapped[int] = mapped_column(Integer, ForeignKey('user.id'))

    # 一对多关系（从 Post 到 User）
    user: Mapped['User'] = relationship("User", back_populates="posts")

User.posts: Mapped[list['Post']] = relationship("Post", back_populates="user")
```

#### **解释**

- **`ForeignKey('user.id')`**：定义 `Post` 表中的 `user_id` 字段作为外键，指向 `User` 表的 `id` 字段。
- **`relationship("User", back_populates="posts")`**：定义了 `Post` 与 `User` 的一对多关系，并在 `User` 中通过 `back_populates` 进行反向映射。
- **`User.posts`**：在 `User` 表中反向定义了这个一对多关系，使得 `User` 能够访问相关的 `Post`。

### **2. 一对多（One-to-Many）关系**

一对多关系表示一个父表记录可以有多个子表记录，但每个子表记录只能属于一个父表记录。

#### **示例：一对多关系**

```python
python复制代码class User(Base):
    __tablename__ = 'user'
    id: Mapped[int] = mapped_column(Integer, primary_key=True, autoincrement=True)
    name: Mapped[str] = mapped_column(String(50))

    # 一对多关系，User 可以有多个 Post
    posts: Mapped[list['Post']] = relationship("Post", back_populates="user")

class Post(Base):
    __tablename__ = 'post'
    id: Mapped[int] = mapped_column(Integer, primary_key=True, autoincrement=True)
    title: Mapped[str] = mapped_column(String(100))
    
    # 外键，引用 User 表的 id 字段
    user_id: Mapped[int] = mapped_column(Integer, ForeignKey('user.id'))

    # 反向关系，Post 只属于一个 User
    user: Mapped['User'] = relationship("User", back_populates="posts")
```

#### **解释**

- **`User.posts`**：定义 `User` 表中的一对多关系，表示一个 `User` 可以有多个 `Post`。
- **`Post.user`**：定义反向关系，表示每个 `Post` 只属于一个 `User`。

### **3. 多对一（Many-to-One）关系**

多对一关系是指多个子表记录指向同一个父表记录。实际上，多对一关系就是一对多关系的反向关系，所以在 SQLAlchemy 中可以通过在另一个表中定义外键来表示。

在上面的示例中，`Post` 中的 `user_id` 就是一个多对一关系的例子，因为多个 `Post` 可能指向同一个 `User`。

### **4. 多对多（Many-to-Many）关系**

多对多关系表示两个表中的记录可以互相引用，即一方的多条记录可以与另一方的多条记录相关联。通常，使用中间表来实现多对多关系。

#### **示例：多对多关系**

假设有两个表：`Student` 和 `Course`，一个学生可以选多个课程，一个课程可以有多个学生。中间表 `student_course` 记录了学生和课程的关联。

```python
python复制代码class Student(Base):
    __tablename__ = 'student'
    id: Mapped[int] = mapped_column(Integer, primary_key=True, autoincrement=True)
    name: Mapped[str] = mapped_column(String(50))

    # 多对多关系
    courses: Mapped[list['Course']] = relationship(
        "Course",
        secondary="student_course",  # 使用中间表
        back_populates="students"
    )

class Course(Base):
    __tablename__ = 'course'
    id: Mapped[int] = mapped_column(Integer, primary_key=True, autoincrement=True)
    title: Mapped[str] = mapped_column(String(100))

    # 多对多关系
    students: Mapped[list['Student']] = relationship(
        "Student",
        secondary="student_course",  # 使用中间表
        back_populates="courses"
    )

# 中间表
class StudentCourse(Base):
    __tablename__ = 'student_course'
    student_id: Mapped[int] = mapped_column(Integer, ForeignKey('student.id'), primary_key=True)
    course_id: Mapped[int] = mapped_column(Integer, ForeignKey('course.id'), primary_key=True)
```

#### **解释**

- **`secondary="student_course"`**：在 `Student` 和 `Course` 之间定义了一个多对多关系，并指定了中间表 `student_course` 来存储两者的关联。
- **`StudentCourse`**：这是定义多对多关系的中间表，它连接 `Student` 和 `Course` 两个表。

### **总结**

1. **外键**：使用 `ForeignKey` 映射字段和引用的主键。
2. **一对多**：使用 `relationship` 在父表和子表之间建立关系，定义外键和反向关系。
3. **多对一**：实际上是与一对多关系的反向定义相同，通过外键将多个记录指向一个父记录。
4. **多对多**：通过 `secondary` 指定中间表来实现多对多关系，定义两个表与中间表的关系。

SQLAlchemy 2.0+ 的 `Mapped` 和 `mapped_column` 语法使得这些关系定义更加现代化和简洁，同时保留了 ORM 强大的功能。
