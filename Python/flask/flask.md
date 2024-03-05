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
