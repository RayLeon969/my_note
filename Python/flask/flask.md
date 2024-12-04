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

# 八 关于查询

SQLAlchemy 2.0 引入了一些新的查询语法和概念，主要的变化是 **引入了 2.0 风格的查询 API**，以及对数据库事务和查询执行的更加规范化。这些改动带来了更简洁且一致的查询方式。

在 SQLAlchemy 2.0 中，使用 `select()` 和 `session.execute()` 代替了旧版的 `query()`。下面我将详细讲解 SQLAlchemy 2.0 中的查询方式。

### 1. **引入 `select()` 和 `session.execute()`**

在 SQLAlchemy 2.0 中，查询的方式发生了改变，不再使用传统的 `session.query()` 方法，而是引入了 **`select()`** 查询对象。

- **`select()`**：新的查询方法，代替了旧版本的 `session.query()`，用于构建查询。
- **`session.execute()`**：执行查询对象，返回一个查询结果。

### 2. **基础查询**

假设我们有一个简单的 `User` 模型：

```python
python复制代码from sqlalchemy import Column, Integer, String
from sqlalchemy.ext.declarative import declarative_base

Base = declarative_base()

class User(Base):
    __tablename__ = 'users'
    
    id = Column(Integer, primary_key=True)
    name = Column(String)
    age = Column(Integer)
```

我们可以使用 SQLAlchemy 2.0 来进行查询：

#### 查询所有用户：

```python
python复制代码from sqlalchemy import select

# 创建查询对象
query = select(User)

# 执行查询
result = session.execute(query).scalars().all()

# 输出所有用户
for user in result:
    print(user.id, user.name, user.age)
```

#### 查询单个用户：

```python
python复制代码query = select(User).where(User.id == 1)
result = session.execute(query).scalars().first()

# 输出单个用户
if result:
    print(result.id, result.name, result.age)
```

#### 查询带条件的用户：

```python
python复制代码query = select(User).where(User.age > 25)
result = session.execute(query).scalars().all()

# 输出满足条件的用户
for user in result:
    print(user.id, user.name, user.age)
```

### 3. **`select()` 与 `session.execute()` 的关系**

SQLAlchemy 2.0 推崇一种显式的执行方式，不再使用 `session.query()` 直接查询。相反，使用 `select()` 来构造查询，然后通过 `session.execute()` 来执行查询。这里有两个常用的方法来获取查询结果：

- **`scalars()`**：如果查询的结果是单一列或模型实例（而不是一组元组），你可以使用 `.scalars()` 来获取。
- **`all()`**：获取所有结果。
- **`first()`**：获取第一个结果（如果有）。

#### 示例：查询特定字段

```python
python复制代码query = select(User.name, User.age).where(User.age > 25)
result = session.execute(query).all()

# 输出查询的字段
for name, age in result:
    print(name, age)
```

### 4. **`join()` 连接查询**

SQLAlchemy 2.0 中的连接查询（`join()`）依然与 SQLAlchemy 1.x 类似，唯一的变化是查询的方式不再是通过 `session.query()`，而是通过 `select()` 和 `join()` 构造。

假设我们有两个表，`User` 和 `Address`，它们通过外键 `user_id` 关联。

#### 定义模型：

```python
python复制代码class Address(Base):
    __tablename__ = 'addresses'
    
    id = Column(Integer, primary_key=True)
    user_id = Column(Integer, ForeignKey('users.id'))
    email_address = Column(String)
    
    user = relationship('User', back_populates='addresses')

User.addresses = relationship('Address', order_by=Address.id, back_populates='user')
```

#### 连接查询（`join`）：

```python
python复制代码from sqlalchemy.orm import aliased

# 创建连接查询
query = select(User, Address).join(Address, User.id == Address.user_id)

# 执行查询
result = session.execute(query).all()

# 输出结果
for user, address in result:
    print(f"{user.name} - {address.email_address}")
```

在这个查询中，`select(User, Address)` 选择了两个表，`join(Address, User.id == Address.user_id)` 表示连接 `User` 和 `Address` 表。

### 5. **复杂查询和子查询**

SQLAlchemy 2.0 支持复杂查询和子查询。我们可以通过 `select()` 嵌套 `select()` 来实现子查询。

#### 子查询的例子：

```python
python复制代码subquery = select(User.id).where(User.age > 30).alias()

# 主查询
query = select(Address).where(Address.user_id.in_(subquery))

# 执行查询
result = session.execute(query).all()

# 输出查询结果
for address in result:
    print(address.email_address)
```

在这个例子中，`subquery` 是一个子查询，查询所有年龄大于 30 的用户 ID。主查询选择了这些用户的地址。

### 6. **聚合查询**

SQLAlchemy 2.0 支持 SQL 聚合函数，如 `count()`, `sum()`, `avg()` 等。你可以在 `select()` 中使用这些聚合函数来执行统计查询。

#### 计算用户的总数：

```python
python复制代码from sqlalchemy import func

query = select(func.count(User.id)).label('user_count')

result = session.execute(query).scalar()

print(f"Total users: {result}")
```

### 7. **使用事务（`session.commit()`）**

在 SQLAlchemy 2.0 中，数据库操作还是需要显式地通过事务提交。例如，在修改数据后你需要调用 `session.commit()` 来提交更改。

```python
python复制代码user = session.execute(select(User).where(User.id == 1)).scalars().first()
user.name = "Updated Name"
session.commit()  # 提交更改到数据库
```

### 8. **其他 SQL 操作**

- **排序 (`order_by`)**：

  ```python
  python复制代码query = select(User).order_by(User.age.desc())
  result = session.execute(query).scalars().all()
  ```

- **分页 (`limit`, `offset`)**：

  ```python
  python复制代码query = select(User).limit(10).offset(20)
  result = session.execute(query).scalars().all()
  ```

### 总结：

- **`select()`**：用于构建查询。
- **`session.execute()`**：执行查询并获取结果。
- **`scalars()`**：提取单列或模型实例。
- **`join()`**：连接表进行联合查询。
- **`filter()`**：添加条件过滤。
- **聚合函数**：使用 `func` 进行聚合查询。
- **事务管理**：操作数据后调用 `session.commit()` 提交事务。

SQLAlchemy 2.0 强调更加显式和统一的查询方式，通过 `select()` 和 `session.execute()` 进行查询，不仅清晰明了，而且易于控制和扩展。

在 SQLAlchemy 2.0 中，`scalars()` 是一个非常重要的查询结果处理方法，它用于从查询结果中提取单个列或模型实例的值，特别适用于需要获取单一字段或模型对象的情况。



在 SQLAlchemy 中，可以使用 `select()` 方法来构建复杂的查询条件，尤其是当条件包含 **OR** 和 **AND** 逻辑时。要实现 `(A or B) and C` 这样的查询条件，可以使用 SQLAlchemy 提供的 `or_` 和 `and_` 方法来组合不同的条件。

### 1. **构建查询 `(A or B) and C`**

假设我们有以下模型：

```python
python复制代码from sqlalchemy import create_engine, Column, Integer, String
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
from sqlalchemy import select, or_, and_

Base = declarative_base()

class MyModel(Base):
    __tablename__ = 'my_model'
    id = Column(Integer, primary_key=True)
    field_a = Column(String)
    field_b = Column(String)
    field_c = Column(String)

# 创建数据库引擎和会话
engine = create_engine('sqlite:///:memory:')
Base.metadata.create_all(engine)
Session = sessionmaker(bind=engine)
session = Session()
```

现在我们要查询所有满足以下条件的记录：

- `(field_a = 'value1' OR field_b = 'value2') AND field_c = 'value3'`

### 2. **构建 SQLAlchemy 查询**

使用 `or_()` 来构建 `A or B` 部分，`and_()` 来构建整体的 `AND` 逻辑。

```python
python复制代码# 创建查询条件
query = select(MyModel).where(
    and_(
        or_(
            MyModel.field_a == 'value1',
            MyModel.field_b == 'value2'
        ),
        MyModel.field_c == 'value3'
    )
)

# 执行查询
result = session.execute(query).scalars().all()

# 输出查询结果
for record in result:
    print(record.id, record.field_a, record.field_b, record.field_c)
```

### 3. **解释代码**

- **`or_()`**：用于将多个条件结合成 `OR` 逻辑。在上面的例子中，`field_a = 'value1'` 或 `field_b = 'value2'`。
- **`and_()`**：用于将多个条件结合成 `AND` 逻辑。在上面的例子中，`(field_a = 'value1' OR field_b = 'value2') AND field_c = 'value3'`。

### 4. **查询结果**

执行该查询后，`result` 将包含所有满足条件 `(A or B) and C` 的记录。具体来说，如果某条记录的 `field_a` 为 `'value1'` 或 `field_b` 为 `'value2'` 且 `field_c` 为 `'value3'`，它就会被包含在查询结果中。

### 总结

- **OR 条件**：使用 `or_()` 结合多个条件。
- **AND 条件**：使用 `and_()` 结合多个条件。
- 使用 `select().where()` 将条件传入查询中，构建复杂的逻辑条件。

这种方式非常灵活，可以根据需要组合多个复杂的查询条件。







### `scalars()` 方法的作用：

1. **提取单一列值**： 当你进行查询时，如果你只选择了某个表的单个字段（如 `User.name` 或 `User.age`），那么你可以使用 `scalars()` 来直接提取这个字段的值。
2. **返回模型实例**： 在某些情况下，你可能只需要从查询结果中提取模型实例本身，而不是包含多个字段的元组。

### 基本用法：

#### 1. 查询单一列并使用 `scalars()`

假设我们有一个 `User` 表，你只关心 `name` 字段而不是整个 `User` 对象：

```python
python复制代码from sqlalchemy import select

# 查询用户的名字
query = select(User.name)

# 执行查询并提取单一列的值
result = session.execute(query).scalars().all()

# 输出查询结果
for name in result:
    print(name)
```

- **`scalars()`**：提取查询结果中的单列（`User.name`）。执行完查询后，`result` 是一个 `list`，包含查询到的所有 `name` 字段的值。

  如果查询的结果包含多个字段（比如选择了 `name` 和 `age`），则 `scalars()` 只会返回第一个字段的所有值。

#### 2. 查询模型实例并使用 `scalars()`

如果你查询的是模型本身，而不仅仅是字段值，`scalars()` 会返回一个包含模型实例的列表：

```python
python复制代码# 查询所有用户实例
query = select(User)

# 执行查询并获取用户实例列表
result = session.execute(query).scalars().all()

# 输出每个用户实例的属性
for user in result:
    print(user.id, user.name, user.age)
```

- 这里，`scalars()` 会提取模型实例，而不是一个包含所有字段的元组。如果查询的结果是一个 `User` 实例，那么 `scalars()` 会返回包含这些实例的列表。

#### 3. 查询单个模型实例并使用 `scalars()`

如果你只关心查询的第一个结果，可以使用 `first()`：

```python
python复制代码# 查询第一个用户实例
query = select(User).where(User.id == 1)

# 执行查询并获取单个实例
result = session.execute(query).scalars().first()

# 输出结果
if result:
    print(result.id, result.name, result.age)
```

- **`first()`**：返回查询结果的第一个项，如果没有结果，则返回 `None`。

#### 4. 使用 `scalars()` 处理聚合查询

当你使用聚合函数（如 `COUNT`, `SUM`, `AVG` 等）时，`scalars()` 可以帮助你提取结果。例如，计算用户的数量：

```python
python复制代码from sqlalchemy import func

# 计算用户总数
query = select(func.count(User.id))

# 执行查询并获取计数结果
count = session.execute(query).scalars().first()

print(f"Total users: {count}")
```

- **`first()`**：因为聚合查询通常只返回一个结果，所以我们用 `first()` 来获取这个单一值。

### 什么时候使用 `scalars()`：

- **单列查询**：当查询的结果只有一个字段时，使用 `scalars()` 可以直接获得这个字段的值，而不是一个包含多个字段的元组。
- **模型实例**：如果你只需要模型实例，而不是多个列值，使用 `scalars()` 可以直接提取模型实例。
- **聚合查询结果**：聚合查询通常返回一个单一值，使用 `scalars()` 可以方便地获取这个值。

### 对比 `.all()` 与 `.scalars()`：

- **`all()`**：返回所有查询结果，通常是一个包含多个元组的列表，每个元组代表一行数据（如果选择了多个字段的话）。
- **`scalars()`**：返回查询结果的单一列或模型实例。如果查询了多个字段，`scalars()` 只返回第一个字段的值。如果查询的是模型实例，返回的是模型实例。

### 例子：

1. **`.all()`：**

   ```python
   python复制代码query = select(User.name, User.age)
   result = session.execute(query).all()
   for name, age in result:
       print(f"Name: {name}, Age: {age}")
   ```

   - 这里，`all()` 返回的是一个包含所有元组的列表，每个元组包含了 `name` 和 `age`。

2. **`.scalars()`：**

   ```python
   python复制代码query = select(User.name)
   result = session.execute(query).scalars().all()
   for name in result:
       print(name)
   ```

   - 这里，`scalars()` 返回的是一个包含 `name` 列的单一值列表。

### 总结：

- **`scalars()`** 用于从查询结果中提取单一列或模型实例，简化了从查询结果中提取单一数据的过程。
- 它适用于 **查询单列数据**（如某个字段的所有值）或者 **返回模型实例** 的情况。
- 它是 SQLAlchemy 2.0 中的新特性，使得查询结果的处理更加灵活和简洁

在 SQLAlchemy 中，模糊查询通常使用 `like` 或 `ilike` 来匹配字符串。具体来说，`like` 是区分大小写的模糊查询，而 `ilike` 是不区分大小写的模糊查询。

### 使用 `like` 进行模糊查询

- `like` 用于执行区分大小写的模糊匹配。例如，我们可以查找名字中包含某个子字符串的所有用户。

#### 示例：

假设我们有一个 `User` 模型，它有 `name` 和 `email` 字段，我们想查询所有名字中包含“john”的用户。

```python
python复制代码from sqlalchemy import select

# 查询名字包含 'john' 的所有用户
query = select(User).where(User.name.like('%john%'))

# 执行查询
result = session.execute(query).scalars().all()

# 输出查询结果
for user in result:
    print(user.id, user.name, user.email)
```

这里，`'%john%'` 是一个通配符字符串，表示匹配包含 "john" 的任何位置。

- `%`：表示任意字符（包括空字符）。
- `_`：表示一个任意字符。

例如：

- `'%john%'`：匹配任何地方包含 `john` 的字符串。
- `'john%'`：匹配以 `john` 开头的字符串。
- `'%john'`：匹配以 `john` 结尾的字符串。
- `'j_n%'`：匹配 `j` 和 `n` 之间有一个任意字符的字符串。

### 使用 `ilike` 进行不区分大小写的模糊查询

如果你需要执行不区分大小写的模糊查询，可以使用 `ilike`。`ilike` 与 `like` 相似，但不区分大小写。

#### 示例：

```python
python复制代码# 查询名字包含 'john'，不区分大小写
query = select(User).where(User.name.ilike('%john%'))

# 执行查询
result = session.execute(query).scalars().all()

# 输出查询结果
for user in result:
    print(user.id, user.name, user.email)
```

- `'%john%'` 这里同样表示匹配包含 `john` 的任何地方，但查询将不区分大小写。

### 使用 `like` 和 `ilike` 的查询优化

为了提高性能，通常建议在使用 `like` 或 `ilike` 时结合索引。比如，如果你在 `User.name` 上创建了一个索引，它将有助于加速模糊查询的执行。

#### 创建索引：

```python
python复制代码from sqlalchemy import Index

# 创建一个针对 User.name 字段的索引
Index('ix_user_name', User.name)
```

### 通过 `filter()` 实现模糊查询

你可以使用 `filter()` 来实现模糊查询，通常与 `like` 或 `ilike` 一起使用：

```python
python复制代码query = select(User).filter(User.name.like('%john%'))
result = session.execute(query).scalars().all()
```

或者，不区分大小写的：

```python
python复制代码query = select(User).filter(User.name.ilike('%john%'))
result = session.execute(query).scalars().all()
```

### 组合条件查询

你还可以将模糊查询与其他条件（如 `and_` 或 `or_`）结合使用进行复杂查询。例如，查询名字包含 `john` 且邮箱包含 `gmail` 的用户：

```python
python复制代码from sqlalchemy import and_

query = select(User).filter(and_(User.name.like('%john%'), User.email.like('%gmail%')))
result = session.execute(query).scalars().all()

# 输出查询结果
for user in result:
    print(user.id, user.name, user.email)
```

### 使用 `like` 和 `ilike` 进行多条件查询

如果你想要查询多个字段并使用模糊查询，可以在 `filter()` 中同时使用多个 `like` 或 `ilike` 条件：

```python
python复制代码query = select(User).filter(
    User.name.ilike('%john%'),
    User.email.ilike('%gmail%')
)
result = session.execute(query).scalars().all()

# 输出查询结果
for user in result:
    print(user.id, user.name, user.email)
```

### 总结

- **`like`**：用于区分大小写的模糊查询。
- **`ilike`**：用于不区分大小写的模糊查询。
- 使用 `%` 和 `_` 作为通配符进行模糊匹配。
- `like` 和 `ilike` 可以与 `filter()` 和 `select()` 结合使用进行更复杂的查询。
