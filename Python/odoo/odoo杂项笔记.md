# 关于One2many和Many2many的增删改查

## One2many

在 Odoo 中，`One2many` 字段用于定义父模型和子模型之间的一对多关系。这种关系允许一个记录（父记录）链接到多个记录（子记录）。我们将讨论如何在 Odoo 中进行增、删、改、查操作。

### 增、删、改、查操作概述

1. **增加（Create）：** 在父记录上创建新的子记录。
2. **删除（Delete）：** 从父记录中删除子记录。
3. **修改（Update）：** 更新子记录的字段值。
4. **查询（Read）：** 查询子记录的信息。

### 增加（Create）

要在父记录中增加子记录，可以通过设置 `One2many` 字段来实现。假设我们有一个父模型 `ParentModel` 和一个子模型 `ChildModel`。

```python
class ChildModel(models.Model):
    _name = 'child.model'
    _description = 'Child Model'
    
    name = fields.Char(string='Name')
    parent_id = fields.Many2one('parent.model', string='Parent')

class ParentModel(models.Model):
    _name = 'parent.model'
    _description = 'Parent Model'
    
    name = fields.Char(string='Name')
    child_ids = fields.One2many('child.model', 'parent_id', string='Children')
```

在创建或更新 `ParentModel` 时，可以通过设置 `child_ids` 字段来增加子记录。例如：

```python
parent = self.env['parent.model'].create({
    'name': 'Parent Record',
    'child_ids': [(0, 0, {'name': 'Child 1'}), 
                  (0, 0, {'name': 'Child 2'})]
})
```

### 删除（Delete）

要删除子记录，可以使用 `(2, id, 0)` 语法。

```python
# 删除一个子记录
parent.write({
    'child_ids': [(2, child_id, 0)]
})
```

### 修改（Update）

要修改子记录，可以使用 `(1, id, values)` 语法，其中 `id` 是子记录的 ID，`values` 是要更新的字段和值。

```python
# 修改一个子记录
parent.write({
    'child_ids': [(1, child_id, {'name': 'Updated Child Name'})]
})
```

### 查询（Read）

要查询父记录的子记录，可以直接访问 `One2many` 字段。

```python
# 获取父记录的子记录
parent = self.env['parent.model'].browse(parent_id)
children = parent.child_ids
for child in children:
    print(child.name)
```

### 示例代码

下面是一个完整的示例代码，展示了如何在 Odoo 中进行 `One2many` 字段的增删改查操作。

```python
class ChildModel(models.Model):
    _name = 'child.model'
    _description = 'Child Model'
    
    name = fields.Char(string='Name')
    parent_id = fields.Many2one('parent.model', string='Parent')

class ParentModel(models.Model):
    _name = 'parent.model'
    _description = 'Parent Model'
    
    name = fields.Char(string='Name')
    child_ids = fields.One2many('child.model', 'parent_id', string='Children')

# 增加操作
parent = self.env['parent.model'].create({
    'name': 'Parent Record',
    'child_ids': [(0, 0, {'name': 'Child 1'}), 
                  (0, 0, {'name': 'Child 2'})]
})

# 查询操作
parent = self.env['parent.model'].browse(parent.id)
children = parent.child_ids
for child in children:
    print(child.name)

# 修改操作
parent.write({
    'child_ids': [(1, children[0].id, {'name': 'Updated Child Name'})]
})

# 删除操作
parent.write({
    'child_ids': [(2, children[1].id, 0)]
})
```

以上代码展示了如何在 Odoo 中进行 `One2many` 字段的增删改查操作。通过这种方式，可以轻松管理父子记录之间的关系。



## Many2many

在 Odoo 中，`Many2many` 字段用于定义两个模型之间的多对多关系。这种关系允许两个模型的多个记录相互关联。我们将讨论如何在 Odoo 中进行 `Many2many` 字段的增、删、改、查操作。

### 增、删、改、查操作概述

1. **增加（Create）：** 在一个记录中增加关联的记录。
2. **删除（Delete）：** 从一个记录中删除关联的记录。
3. **修改（Update）：** 更新关联记录的字段值。
4. **查询（Read）：** 查询关联记录的信息。

### 定义 Many2many 关系

假设我们有两个模型 `ModelA` 和 `ModelB`，它们之间有一个多对多关系。

```python
class ModelA(models.Model):
    _name = 'model.a'
    _description = 'Model A'
    
    name = fields.Char(string='Name')
    model_b_ids = fields.Many2many('model.b', string='Model B Records')

class ModelB(models.Model):
    _name = 'model.b'
    _description = 'Model B'
    
    name = fields.Char(string='Name')
    model_a_ids = fields.Many2many('model.a', string='Model A Records')
```

### 增加（Create）

要在一个记录中增加关联的记录，可以使用 `(4, id)` 或 `(6, 0, ids)` 语法。

```python
# 创建一个新的 ModelA 记录，并关联一些 ModelB 记录
model_a = self.env['model.a'].create({
    'name': 'Record A',
    'model_b_ids': [(6, 0, [b1_id, b2_id])]
})
```

### 删除（Delete）

要删除关联的记录，可以使用 `(3, id)` 或 `(6, 0, ids)` 语法，其中 `id` 是关联记录的 ID。

```python
# 从 ModelA 记录中删除一个 ModelB 关联
model_a.write({
    'model_b_ids': [(3, b1_id)]
})
```

### 修改（Update）

`Many2many` 字段本身没有直接的更新操作，因为它只是表示关系。要修改关联记录的字段值，可以直接访问关联的记录。

```python
# 修改关联的 ModelB 记录
model_b = self.env['model.b'].browse(b1_id)
model_b.write({'name': 'Updated Name'})
```

### 查询（Read）

要查询一个记录的关联记录，可以直接访问 `Many2many` 字段。

```python
# 获取 ModelA 记录的关联记录
model_a = self.env['model.a'].browse(a_id)
associated_records = model_a.model_b_ids
for record in associated_records:
    print(record.name)
```

### 示例代码

下面是一个完整的示例代码，展示了如何在 Odoo 中进行 `Many2many` 字段的增删改查操作。

```python
class ModelA(models.Model):
    _name = 'model.a'
    _description = 'Model A'
    
    name = fields.Char(string='Name')
    model_b_ids = fields.Many2many('model.b', string='Model B Records')

class ModelB(models.Model):
    _name = 'model.b'
    _description = 'Model B'
    
    name = fields.Char(string='Name')
    model_a_ids = fields.Many2many('model.a', string='Model A Records')

# 增加操作
b1 = self.env['model.b'].create({'name': 'Record B1'})
b2 = self.env['model.b'].create({'name': 'Record B2'})

model_a = self.env['model.a'].create({
    'name': 'Record A',
    'model_b_ids': [(6, 0, [b1.id, b2.id])]
})

# 查询操作
model_a = self.env['model.a'].browse(model_a.id)
associated_records = model_a.model_b_ids
for record in associated_records:
    print(record.name)

# 修改操作
model_b = self.env['model.b'].browse(b1.id)
model_b.write({'name': 'Updated Name'})

# 删除操作
model_a.write({
    'model_b_ids': [(3, b2.id)]
})
```

以上代码展示了如何在 Odoo 中进行 `Many2many` 字段的增删改查操作。通过这种方式，可以灵活地管理两个模型之间的多对多关系。

# 判定当前用户是否具有权限

```python
self.env.user.has_group('base.group_user'):
```

# psql执行.sql文件

```shell
psql -U <数据库用户> -p <postgresql端口> -d <数据库> -f <.sql文件路径>
```

# psql配置环境变量：

1. 添加变量PG_HOME 值：postgresql安装目录
2. path中添加%PG_HOME%\bin\

# cmd查找并终止端口服务

```shell
netstat -ano | findstr 8069
```

假设查到进程服务6475，即标有listening的进程服务

```shell
taskkill -PID 6475 -F
```

