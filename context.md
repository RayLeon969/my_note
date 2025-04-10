# Data Sync 模块开发指南

## 1. 模型层设计

### 1.1 基础模型

所有需要同步的模型都应该继承 `data.sync.base`，例如：

```python
class ProductCategory(models.Model):
    _name = 'product.category'
    _inherit = ['product.category', 'data.sync.base']
```

### 1.2 关键方法实现

#### 1.2.1 `_get_sync_fields` 方法

此方法用于获取需要同步的字段值，并处理外键关系：

```python
def _get_sync_fields(self, sys_info_id):
    res = super()._get_sync_fields(sys_info_id)
    res.update({
        'name': self.name,
        'complete_name': self.complete_name,
        'parent_id': self.parent_id.unique_id,
        'hive_off_code_id': {'type': 'many2one', 'model_name': 'hive.off.code', 'unique_ids':[self.hive_off_code_id.unique_id] if self.hive_off_code_id else []},
    })
    return res
```

对于外键字段，需要使用特殊的格式：
- `many2one` 字段：`{'type': 'many2one', 'model_name': '模型名称', 'unique_ids': [唯一ID列表]}`
- `many2many` 字段：`{'type': 'many2many', 'model_name': '模型名称', 'unique_ids': [唯一ID列表]}`

#### 1.2.2 `_data_sync_check_before` 方法

此方法用于同步前的数据验证，确保数据满足同步条件：

```python
def _data_sync_check_before(self, sys_info_id):
    err_msg = ''
    fast_copy = []
    sync_record = self.env['data.sync.record.line']
    for record in self:
        if record.hive_off_code_id:
            hive_off_code_lines = sync_record.search([
                ('model', '=', 'hive.off.code'),
                ('res_id', '=', record.hive_off_code_id.id),
                ('sys_info_id', '=', sys_info_id.id)
            ])
            if not hive_off_code_lines:
                err_msg += f'产品类别：{record.name}的分群码：{record.hive_off_code_id.code}\n'
                fast_copy.append(record.hive_off_code_id.code)
    if err_msg:
        err_msg += f'上述产品类别的分群码并未在{sys_info_id.description}中存在，请先同步该分群码数据！\n'
        err_msg += '分群码快速搜索：' + self.generate_fast_copy_search_string(fast_copy)
        raise exceptions.ValidationError(err_msg)
```

## 2. 向导层设计

### 2.1 特定模型向导

特定模型的向导应该继承 `data.sync.base.wizard`，例如：

```python
class ProductTemplateSyncWizard(models.TransientModel):
    _name = 'product.template.sync.wizard'
    _inherit = ['data.sync.base.wizard']
    _description = '产品同步向导'

    line_ids = fields.One2many('product.template.sync.wizard.line', 'wizard_id', string='同步操作行')
```

### 2.2 向导明细

向导明细应该继承 `data.sync.base.wizard.line`，例如：

```python
class ProductTemplateSyncWizardLine(models.TransientModel):
    _name = 'product.template.sync.wizard.line'
    _inherit = ['data.sync.base.wizard.line']
    _description = '产品同步向导操作行'

    wizard_id = fields.Many2one('product.template.sync.wizard', string='向导')
    sync_data_id = fields.Many2one('product.template', string='同步数据')

    # 添加相关字段，方便在视图中显示
    default_code = fields.Char(string='料号', related='sync_data_id.default_code')
    product_name = fields.Char(string='品名', related='sync_data_id.name')
    x_model_standard = fields.Char(string='规格', related='sync_data_id.x_model_standard')

    # 必须添加这两个字段
    model = fields.Char(string='模型名称', default='product.template')
    res_id = fields.Integer(string='记录ID', related='sync_data_id.id')
```

### 2.3 公共向导

公共向导可以用于通用的同步操作，注意我没有特定需要模型的同步向导时，默认使用公共向导，无需重复添加公共向导视图：

```python
class DataSyncPublicWizard(models.TransientModel):
    _name = 'data.sync.public.wizard'
    _inherit = ['data.sync.base.wizard']
    _description = '数据同步公共向导'

    line_ids = fields.One2many('data.sync.public.wizard.line', 'wizard_id', string='同步操作行')
```

## 3. 视图设计

### 3.1 向导表单视图

```xml
<record id="product_template_sync_wizard_view_form" model="ir.ui.view">
    <field name="name">product.template.sync.wizard.view.form</field>
    <field name="model">product.template.sync.wizard</field>
    <field name="type">form</field>
    <field name="priority">1</field>
    <field name="arch" type="xml">
        <form string="同步向导" version="7.0">
            <group>
                <group string="同步系统">
                    <field name="sys_info_ids_domain" invisible="1"/>
                    <field name="sys_info_id" required="1" options="{'no_create':True,'no_open':True,'no_edit':True}"/>
                </group>
                <group string="同步信息">
                    <field name="is_sync" widget="boolean_toggle" readonly="1"/>
                    <field name="err_msg" readonly="1"/>
                </group>
            </group>

            <separator string="同步明细" style="margin-top: 5px !important;"/>
            <field name="line_ids">
                <tree edit="0" delete="0" create="0">
                    <field name="default_code" />
                    <field name="product_name" />
                    <field name="x_model_standard" />
                </tree>
            </field>

            <footer>
                <button string="确定" name="execute_sync" confirm="确认同步？" type="object" class="btn-primary" attrs="{'invisible': [('is_sync', '=', True)]}"/>
                <button string="取消" special="cancel" class="btn-default" attrs="{'invisible': [('is_sync', '=', True)]}"/>
            </footer>
        </form>
    </field>
</record>
```

### 3.2 公共向导视图

```xml
<record id="view_data_sync_public_wizard_form" model="ir.ui.view">
    <field name="name">data.sync.public.wizard.form</field>
    <field name="model">data.sync.public.wizard</field>
    <field name="arch" type="xml">
        <form string="数据同步公共向导">
            <header>
                <button name="execute_sync" 
                        string="执行同步" 
                        type="object" 
                        class="oe_highlight" 
                        attrs="{'invisible': [('is_sync', '=', True)]}"/>
            </header>
            <sheet>
                <div class="oe_title">
                    <h1>
                        <field name="sys_info_id" required="1" options="{'no_create': True, 'no_open': True}"/>
                    </h1>
                </div>
                <group>
                    <field name="is_sync" invisible="1"/>
                </group>
                <notebook>
                    <page string="同步数据" name="sync_data">
                        <field name="line_ids" 
                               nolabel="1" 
                               attrs="{'readonly': [('is_sync', '=', True)]}">
                            <tree editable="bottom">
                                <field name="sync_data_id" 
                                       options="{'no_create': True}" 
                                       domain="[('id', 'not in', parent.line_ids.mapped('sync_data_id').ids)]"/>
                            </tree>
                        </field>
                    </page>
                    <page string="同步结果" name="sync_result" attrs="{'invisible': [('is_sync', '=', False)]}">
                        <group>
                            <field name="err_msg" nolabel="1" readonly="1"/>
                        </group>
                    </page>
                </notebook>
            </sheet>
            <footer>
                <button string="关闭" 
                        class="btn-secondary" 
                        special="cancel"/>
            </footer>
        </form>
    </field>
</record>
```

## 4. 权限设置

在 `security/ir.model.access.csv` 文件中添加访问权限：

```
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_product_category_sync_wizard_group_erp_manager,product_category_sync_wizard.group_erp_manager,data_sync.model_product_category_sync_wizard,base.group_erp_manager,1,1,1,1
access_product_category_sync_wizard_line_group_erp_manager,product_category_sync_wizard_line.group_erp_manager,data_sync.model_product_category_sync_wizard_line,base.group_erp_manager,1,1,1,1
```

## 5. 清单文件更新

在 `__manifest__.py` 文件中添加新文件：

```python
'data': [
    'security/res_groups.xml',
    'security/ir.model.access.csv',
    'data/data_sync_config.xml',
    'data/data_sync_actions.xml',
    'views/data_sync_config_views.xml',
    'views/data_sync_record_line_views.xml',
    'wizard/product_category_sync_wizard_views.xml',
    'views/menuitem.xml',
],
```

## 6. 同步流程

1. 用户选择一个系统信息（`sys.info`）
2. 系统加载需要同步的数据记录
3. 调用 `remote_create` 方法，将数据发送到远程系统
4. 远程系统创建记录并返回结果
5. 调用 `sync_line_create` 方法，在本地创建同步记录
6. 通过 `remote_update` 和 `remote_unlink` 方法处理记录的更新和删除

## 7. 注意事项

1. 所有需要同步的模型都必须继承 `data.sync.base`
2. 必须实现 `_get_sync_fields` 和 `_data_sync_check_before` 方法
3. 向导明细必须包含 `model` 和 `res_id` 字段
4. 同步记录使用 `data.sync.record.line` 模型
5. 确保在 `ir.model.access.csv` 文件中添加访问权限
6. 在 `__manifest__.py` 文件中添加新文件

这份指南应该能够帮助您按照 `data_sync` 模块的工作模式编写新的同步代码。